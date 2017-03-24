---
layout: default
title: Solver Chain
subtitle: Overview of solver chain and related command-line options
slug: solver-chain
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

1.  **STP:** Simple Theorem Prover SMT solver [link](http://stp.github.io). Use `-solver-backend=stp`.
2.  **Z3:** The Z3 Theorem Prover [link](https://github.com/Z3Prover/z3). Use `-solver-backend=z3`.
3.  **metaSMT:** An Embedded Domain Specific Language for SMT [link](http://www.informatik.uni-bremen.de/agra/eng/metasmt.php). Use `-solver-backend=metasmt`.

When using metaSMT as the core solver, its backend can be specified using the
`-metasmt-backend=` option.

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

This is useful for comparing two completly independent solvers (e.g. STP and Z3) and
for checking that invoking the solver chain and a solver directly obtains the
same results.

### Query logging solver

The query logging solver logs queries at different positions in the chain.

The option to enable this is the `--use-query-log=` option. The format of the option
is `--use-query-log=TYPE:FORMAT`, where:
  - **TYPE** is either **all** to log all the queries given to the solver chain. This is before any optimisation (e.g. caching, constraint independence) is performed, or **solver** to log only the queries passed to the core solver. Note that it is possible that some of the unoptimized queries are never executed or are modified before being executed by KLEE's underlying solver.

  - **FORMAT** is the format in which queries are logged and can be either **kquery** for the [KQuery]({{site.baseurl}}/docs/kquery) format, or **smt2** for the [SMT-LIBv2](http://www.smtlib.org) format.

In addition there are several options that change how these queries are logged.

- `--min-query-time-to-log=TIME` (in ms) is used to log only queries that exceed a certain time limit. **TIME** can be:
    - **0** (default): to log all queries
    - **&lt;0**: a negative value specifies that only queries that timed out should be logged. The timeout value is specified via the `--max-solver-time` option.
    - **&gt;0**: only queries that took more that **TIME** milliseconds should be logged.
-  `--log-partial-queries-early=true` is used to dump the query to the log file before the next part of the solver chain is called.  Normally, KLEE prints the query and its solution after it has been solved. But if KLEE crashes inside the solver chain, the suspicious query will not be logged. Enable this option to debug such cases. This option comes with a performance penalty as the log buffer gets always flushed.
-  `--compress-query-log` is used to compress query log files (**default=off**)
