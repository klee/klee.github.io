---
layout: default
title: Solver Chain
subtitle: Overview of solver chain and related command-line options
slug: documentation
---

{:.toc}
Contents
{:.toc__title .no_toc}
* Table of contents placeholder
{:.toc__list .list-anchor}
{:toc}

# The solver chain

KLEE has a number of command line options that control the solver chain that
is shared both by the `klee` and `kleaver` tools.

The solver chain is implemented internally using the decorator design pattern.
Thus when a solver query is issued it travels through a nested stack of solvers
and may eventually call the underlying SMT solver (core solver).

The reasons for this architecture are:

1. Allows transforming the solver query into another form that is hopefully
   faster for the core solver to solve (e.g. the independence solver).
2. Allows querying the core solver to be avoided in some cases (e.g. the
   counterexample caching solver).
3. Allows logging of queries at different stages in the solver chain.

The `constructSolverChain()` function is used to create the solver chain.

## Core solver

This is the underlying SMT solver. Several solvers are currently supported.
The default is dependent on what solvers were available when KLEE was built.
The solver can be selected using the `-solver-backend=` option.

### MetaSMT

**metaSMT:** An Embedded Domain Specific Language for SMT [link](http://www.informatik.uni-bremen.de/agra/eng/metasmt.php). Use `-solver-backend=metasmt`.

These are options that only affect the MetaSMT solver.

`-metasmt-backend=<backend>`

This specifies the solver that metaSMT should use. Valid values for `<backend>`
are `stp` (STP), `z3` (Z3), and `btor` (Boolector). Note that Z3 can also be
used directly through its API via the Z3 core solver.

`-use-construct-hash-metasmt`

When set to true (**default: true**) constructed MetaSMT expressions will be
cached (and then cleared) for each constraint to facilitate expression re-use.

### STP

**STP:** Simple Theorem Prover SMT solver [link](http://stp.github.io). Use `-solver-backend=stp`.

These are options that only affect the STP solver.

`-debug-dump-stp-queries`

When set to true (**default: false**) every query that the STP solver receives
will be written to standard error in its native format.

`-ignore-solver-failures`

When set to true (**default: false**) unexpected solver failures in the STP
solver will be ignored.

`-use-construct-hash`

When set to true (**default: true**) constructed STP expressions will be cached
(and then cleared) for each constraint to facilitate expression re-use.

`-stp-sat-solver=<minisat|simpleminisat|cryptominisat|riss>`

Set the underlying SAT solver for STP (**default: cryptominisat**) when it is available.


### Z3

**Z3:** The Z3 Theorem Prover [link](https://github.com/Z3Prover/z3). Use `-solver-backend=z3`.

These are options that only affect the Z3 solver.

`-debug-z3-dump-queries=<path>`

When set every query that the Z3 solver receives will be logged to the file
at `<path>` in the SMT-LIBv2 format.

`-debug-z3-log-api-interaction=<path>`

When set Z3's C API interaction will be logged to the file at `<path>`.
This log can be replayed by the Z3 binary using its `-log` option.
This option is useful for precisely reproducing KLEE's interaction with Z3
outside of KLEE.

`-debug-z3-validate-models`

When set to true (**default: false**), Z3 models will be substituted back into
the Z3 expressions for every query that is satisfiable. This is used to check
that the models that Z3 provides are satisfiable in Z3's own constraint
language. If a model does not satisfy the constraints information about the
failure is printed to standard error and then an `abort()` is called.

Note Z3 models being satisfiable in Z3's constraint language does not necessarily
imply that the model is satisfiable when substituted into KLEE's own expression
language. This could happen if there was an unintentional semantic mismatch
between Z3's and KLEE's expression language. To check that a model satisfies
KLEE's constraint language use `-debug-assignment-validating-solver`.

`-debug-z3-verbosity=<N>`

When set, Z3's verbosity level will be set to level `<N>` (**default: 0**).
This is equivalent to the Z3 binary's `-v:<N>` option. This is useful for
observing Z3's internal behaviour (e.g. what tactic is being applied).

`-use-construct-hash-z3`

When set to true (**default: true**) constructed Z3 expressions will be cached
(and then cleared) for each query to facilitate expression re-use.

### Common core solver options

These are options shared by two or more core solvers.

`-solver-optimize-divides`

This only affects the MetaSMT and STP solvers. When set to true expressions involving
division will be optimized (if possible) before being given to the solver.

`-use-forked-solver`

This only affects the MetaSMT and STP solvers. When set to true (**default:
true**) the solver is created in a separate process that is forked from the
`klee`/`kleaver` process. When set to false the solver is created in the
`klee`/`kleaver` process.

## Caching Solvers

These solvers cache previous queries to avoid calling the underlying solver when possible.

### Caching solver

This solver caches previous queries and their result to avoid calling the underlying solver
in certain situations. The queries are canonicalized to increase the cache hit rate.

This solver can be enabled with the `-use-cache` option.

### Counterexample caching solver

This solver caches satisfying assignments to queries (i.e. "counterexamples"
to validity) which can be used to avoid calling the underlying solver in
certain situations.

This solver can be enabled with the `-use-cex-cache` option.

<!-- TODO: Explain this solver in greater depth -->

<!-- TODO: Document fastcex solver -->

## Independence solver

This solver splits a query into disjoint sets of independent constraints and
then calls the underlying solver on each set to solve them independently.

This solver can be enabled with the `-use-independent-solver` option.

## Debugging solvers

The remaining solvers are used for debugging.

### Assignment validating solver

This solver checks that assignments to satisfiable queries are satisfiable
when substituted into the expressions in the query.

This is useful for checking the consistency of KLEE's constraint language and
that of the core solver.

This option can be enabled using the `-debug-assignment-validating-solver`
option.

### Debug validating solver

This solver is meant for debugging and currently is only useful if building with
assertions is enabled. The solver can be enabled with the `-debug-validate-solver`
option.

This solver invokes the underlying solver and a separate solver for every query and
then checks that the solvers agree. The separate solver can be set with the
`-debug-crosscheck-core-solver=` option which takes the same arguments as
`-solver-backend=`.

This is useful for comparing two completely independent solvers (e.g. STP and Z3) and
for checking that invoking the solver chain and a solver directly obtains the
same results.

### Query logging solver

The query logging solver logs queries at different positions in the chain.

The option to enable this is the `--use-query-log=` option. The format of the option
is `--use-query-log=TYPE:FORMAT`, where:
  - **TYPE** is either **all** to log all the queries given to the solver chain. This is before any optimisation (e.g. caching, constraint independence) is performed, or **solver** to log only the queries passed to the core solver. Note that it is possible that some of the unoptimized queries are never executed or are modified before being executed by KLEE's underlying solver.

  - **FORMAT** is the format in which queries are logged and can be either **kquery** for the [KQuery]({{site.baseurl}}/docs/kquery) format, or **smt2** for the [SMT-LIBv2](http://www.smtlib.org) format.

In addition there are several options that change how these queries are logged.

- `--min-query-time-to-log=TIME` is used to log only queries that exceed a certain time limit (default=0s)
- `--log-timed-out-queries` is used to log queries that exceed `--max-solver-time` (default=true)
- `--log-partial-queries-early=true` is used to dump the query to the log file before the next part of the solver chain is called.  Normally, KLEE prints the query and its solution after it has been solved. But if KLEE crashes inside the solver chain, the suspicious query will not be logged. Enable this option to debug such cases. This option comes with a performance penalty as the log buffer gets always flushed.
- `--compress-query-log` is used to compress query log files (default=off)

