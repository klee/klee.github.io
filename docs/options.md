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

## KLEE usage

{% highlight bash %}
$ klee [klee-options] <program.bc> [program-options]
{% endhighlight %}

The general form of a KLEE command-line is first the arguments for KLEE itself (`[klee-options]`), then the LLVM bitcode file to execute (`program.bc`), and then any arguments to pass to the application (`[program-options]`).
In particular, the KLEE option `-posix-runtime` enables the use of the symbolic environment options as part of the program's options.

To get a complete list of KLEE's command-line options run: `klee --help`. The remainder of this page illustrate KLEE's main command-line options.

## Symbolic Environment

KLEE provides several options as part of its symbolic environment:

1.  `-sym-arg <N>` - Replace by a symbolic argument with length N.
2.  `-sym-args <MIN> <MAX> <N>` - Replace by at least MIN arguments and at most MAX arguments, each with maximum length N.
3.  `-sym-files <NUM> <N>` - Make NUM symbolic files ('A', 'B', 'C', etc.), each with size N (excluding stdin)
4.  `-sym-stdin <N>` - Make stdin symbolic with size N.
5.  `-sym-stdout` - Make stdout symbolic.
6.  `-save-all-writes` - Allow write operations to execute as expected even if they exceed the file size (*default=on*). When *off*, all writes exceeding the initial file size are discarded. **Note:** file offset is *always* incremented.
7.  `-max-fail <N>` - Allow up to N injected failures.
8.  `-fd-fail` - Shortcut for '-max-fail 1'.

Some usage examples:

{% highlight bash %}
$ klee -posix-runtime <program.bc> -sym-arg 16
{% endhighlight %}
{% highlight bash %}
$ klee -posix-runtime <program.bc> -sym-files 2 30 -sym-stdin
{% endhighlight %}

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
    - **FORMAT** is the format in which queries are logged and can be either **kquery** for the [KQuery]({{site.baseurl}}/docs/kquery) format, or **smt2** for the [SMT-LIBv2](http://www.smtlib.org) format.
2.  `--min-query-time-to-log=TIME` (in ms) is used to log only queries that exceed a certain time limit. **TIME** can be:
    - **0** (default): to log all queries
    - **&lt;0**: a negative value specifies that only queries that timed out should be logged. The timeout value is specified via the `--max-solver-time` option.
    - **&gt;0**: only queries that took more that **TIME** milliseconds should be logged.
3. `--log-partial-queries-early=true` is used to dump the query to the log file before the next part of the solver chain is called.  Normally, KLEE prints the query and its solution after it has been solved. But if KLEE crashes inside the solver chain, the suspicious query will not be logged. Enable this option to debug such cases. This option comes with a performance penalty as the log buffer gets always flushed.
4. `--compress-query-log` is used to compress query log files (**default=off**)

## Entry Point

To change the entry point you can use the option `-entry-point=FUNCTION_NAME`, where **FUNCTION_NAME** is the name of the function to use as the entry point for execution.

## Calls to `klee-assume`

By default, KLEE will report an error if the assumed condition is infeasible. The option `-silent-klee-assume` can be used to sliently terminate the current path exploration in such cases.

## Statistics

By default, KLEE generates two files containing statistics concerning the code exploration:

* **run.stats**: This is a text file containing various statistics emitted by KLEE. While this file can be inspected manually, you should use the [klee-stats]({{site.baseurl}}/docs/tools) tool for that.
* **run.istats**: This is a text file in Callgrind format containing global statistics emitted by KLEE for each line of code in the program. This file can be inspected with frontends which are able to read it (e.g. [kcachegrind](https://kcachegrind.github.io/))

There are several options to modify how KLEE outputs statistics:

* `-stats`                             - Enable statistics output from program (*default=on*)
* `-output-stats`                      - Write running stats trace file (*default=on*)
* `-output-istats`                     - Write instruction level statistics in callgrind format (*default=on*)
* `-stats-write-interval=TIME`         - Approximate number of seconds between stats writes (*default=1.0s*)
* `-istats-write-interval=TIME`        - Approximate number of seconds between istats writes (*default=10.0s*)
* `-stats-write-after-instructions=N`  - Write statistics after each `N` instructions, 0 to disable (*default=0*)
* `-istats-write-after-instructions=N` - Write istats after each `N` instructions, 0 to disable (*default=0*)

## KLEE debug

KLEE provides several debugging options:

* `-debug-print-instructions=FORMAT`     - Log the LLVM instructions executed by KLEE (**default=off**).

  The output may include: the source code file and line (`src`), the instruction identifier as assigned by KLEE (`inst_id`), and the LLVM instruction with debugging informations (`llvm_inst`). The output format can be controlled with the following options:
  
  - `=all:stderr`     - Log all instructions to stderr in format `[src, inst_id, llvm_inst]`
  - `=src:stderr`     - Log all instructions to stderr in format `[src, inst_id]`
  - `=compact:stderr` - Log all instructions to stderr in format `[inst_id]`
  - `=all:file`       - Log all instructions to file instructions.txt in format `[src, inst_id, llvm_inst]`
  - `=src:file`       - Log all instructions to file instructions.txt in format `[src, inst_id]`
  - `=compact:file`   - Log all instructions to file instructions.txt in format `[inst_id]`

* `-debug-compress-instructions`  - Compress the `instructions.txt` file (**default=off**)

## Memory Management

KLEE explicitly intercepts calls for memory management (like `malloc()` and `free()`) and forwards to an existing memory allocators.
To change this behaviour, following options are provided:

* `--allocate-determ`                  - Enable support to allocate memory deterministically for the executed application (*default=off*)
* `--allocate-determ-size`             - For deterministic allocation, the buffer of the specified size is pre-allocated in MB (*default=100*)
* `--allocate-determ-start-addres`     - Controls the required start address of the pre-allocated memory. This address has to be page aligned. If null is provided, the memory is mapped to an arbitrary address.
* `--return-null-on-zero-malloc`       - Controls if a NULL pointer should be returned in case the size argument is zero (*default=off*)
* `--red-zone-space`                   - Controls the space kept unused between two adjacent allocations in byte (*default=10*)

## Making KLEE Exit on Events

KLEE does not exit if a bug is found in the analyzed application by default. On
the other hand, KLEE implicitly exits on some failures. This behaviour can be
changed by the following options:

* `-exit-on-error`                     - Exit on the first arbitrary error.
* `-exit-on-error-type=TYPE`           - Exit on the first error of type `TYPE`. This parameter can be repeated to exit after more types. `TYPE` can one of the following:
  - `=Abort`       - The program crashed
  - `=Assert`      - An assertion was hit
  - `=Exec`        - Trying to execute an unexpected instruction
  - `=External`    - External objects referenced
  - `=Free`        - Freeing invalid memory
  - `=Model`       - Memory model limit hit
  - `=Overflow`    - An overflow occurred
  - `=Ptr`         - Pointer error
  - `=ReadOnly`    - Write to read-only memory
  - `=ReportError` - `klee_report_error` called
  - `=User`        - Wrong `klee_*` functions invocation
  - `=Unhandled`   - Unhandled instruction hit

* `-ignore-solver-failures`            - Do not exit upon solver failure.

Examples:
{% highlight bash %}
klee -exit-on-error input.bc
{% endhighlight %}
{% highlight bash %}
klee -exit-on-error-type=Assert -exit-on-error-type=Ptr input.bc
{% endhighlight %}

## Linking External Libraries

Definitions of undefined functions are taken from files given using the option
`-link-llvm-lib`.

If some functions in the input file are defined in an external LLVM IR file, an
archive (.a) of LLVM IR files, or a shared object with LLVM IR code, these
external files can be *linked in* using the option `-link-llvm-lib=LIB_FILENAME`.

For example, the following command runs KLEE on the program `test.bc`, linking
a helper library:

{% highlight bash %}
$ klee -link-llvm-lib=libhelper.so.bc test.bc
{% endhighlight %}

The option can be provided multiple times. For instance, linking two libraries,
helper and helper2, can be done with the following command:

{% highlight bash %}
$ klee -link-llvm-lib=libhelper.so.bc -link-llvm-lib=libhelper2.so.bc test.bc
{% endhighlight %}
