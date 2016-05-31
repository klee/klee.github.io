---
layout: default
title: Options
subtitle: Overview of KLEE’s main command-line options
slug: options
---

{:.toc}
Contents
{:.toc__title .no_toc}
* Table of contents placeholder
{:.toc__list .list-anchor}
{:toc}

## Search Heuristics

### Main search heuristics

KLEE provides four main search heuristics:

1.  **Depth-First Search (DFS):** Traverses states in depth-first order.
2.  **Random State Search:**Randomly selects a state to explore.
3.  **Random Path Selection:** Described in our [KLEE OSDI'08](http://www.doc.ic.ac.uk/~cristic/papers/klee-osdi-08.pdf) paper.
4.  **Non Uniform Random Search (NURS):** Selects a state randomly according to a given distribution. The distribution can be based on the minimum distance to an uncovered instruction (MD2U), the query cost, etc.

To select a search heuristic, use the `--search` option provided by KLEE. For example:

{% highlight bash %}
$ klee --search=dfs demo.o
{% endhighlight %}

runs `demo.o` using DFS, while

{% highlight bash %}
$ klee --search=random-path demo.o
{% endhighlight %}

runs it using the random path selection strategy. The full list of options is shown in KLEE's help message:

{% highlight bash %}
$ klee --help
-search - Specify the search heuristic (default=random-path interleaved with nurs:covnew)
  =dfs - use Depth First Search (DFS)
  =random-state - randomly select a state to explore
  =random-path - use Random Path Selection (see OSDI'08 paper)
  =nurs:covnew - use Non Uniform Random Search (NURS) with Coverage-New heuristic
  =nurs:md2u - use NURS with Min-Dist-to-Uncovered heuristic
  =nurs:depth - use NURS with 2^depth heuristic
  =nurs:icnt - use NURS with Instr-Count heuristic
  =nurs:cpicnt - use NURS with CallPath-Instr-Count heuristic
  =nurs:qc - use NURS with Query-Cost heuristic
{% endhighlight %}

### Interleaving search heuristics

Search heuristics in KLEE can be interleaved in a round-robin fashion. To interleave several search heuristics to be interleaved, use the `--search` multiple times. For example:

{% highlight bash %}
$ klee --search=random-state --search=nurs:md2u demo.o
{% endhighlight %}

interleaves the Random State and the NURS:MD2U heuristics in a round robin fashion.  

### Default search heuristics

The default heuristics used by KLEE are `random-path` interleaved with `nurs:covnew`.

## Query Logging

To log the queries issued by KLEE during symbolic execution, you can use the following options:

1.  `--use-query-log=TYPE:FORMAT`, where:
    - **TYPE** is either **all** to log all the queries KLEE made during execution before any optimisation (e.g. caching, constraint independence) is performed, or **solver** to log only the queries passed to KLEE's underlying solver. Note that it is possible that some of the unoptimized queries are never executed or are modified before being executed by KLEE's underlying solver.
    - **FORMAT** is the format in which queries are logged and can be either **pc** for the [KQuery]({{site.baseurl}}/docs/kquery) format, or **smt2** for the [SMT-LIBv2](http://www.smtlib.org) format.
2.  `--min-query-time-to-log=TIME` (in ms) is used to log only queries that exceed a certain time limit. **TIME** can be:
    - **0** (default): to log all queries
    - **&lt;0**: a negative value specifies that only queries that timed out should be logged. The timeout value is specified via the `--max-solver-time` option.
    - **&gt;0**: only queries that took more that **TIME** milliseconds should be logged.
3. `--log-partial-queries-early=true` is used to dump the query to the log file before the next part of the solver chain is called.  Normally, KLEE prints the query and its solution after it has been solved. But if KLEE crashes inside the solver chain, the suspicious query will not be logged. Enable this option to debug such cases. This option comes with a performance penalty as the log buffer gets always flushed.

## Entry Point

To change the entry point you can use the option `-entry-point=FUNCTION_NAME`, where **FUNCTION_NAME** is the name of the function to use as the entry point for execution.

## Calls to `klee-assume`

By default, KLEE will report an error if the assumed condition is infeasible. The option `-silent-klee-assume` can be used to sliently terminate the current path exploration in such cases.

## Linking External Libraries

Definitions of undefined functions are taken from files given using the option
`-link-llvm-lib`.

If some functions in the input file are defined in an external LLVM IR file, an
archive (.a) of LLVM IR files, or a shared object with LLVM IR code, these
external files can be *linked in* using the option `-link-llvm-lib=LIB_FILENAME`.
