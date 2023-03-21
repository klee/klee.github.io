---
layout: default
title: Tools
subtitle: Overview of the auxiliary tools provided by KLEE
slug: documentation
---

{:.toc}
Contents
{:.toc__title .no_toc}
* Table of contents placeholder
{:.toc__list .list-anchor}
{:toc}
 
## ktest-tool

KLEE can be configured to output `.ktest` [files]({{site.baseurl}}/docs/files) whenever it finds an error, covers new code or terminates a path.
The content of a `.ktest` file describes the program input that is needed to guide a concrete execution exactly along the corresponding execution path.
Typically, it comprises concrete values for symbolic input files, symbolic arguments, and symbolic variables introduced with `klee_make_symbolic`.
The `ktest-tool` is a Python script that converts the contents of a `.ktest` file into human-readable form.
For instance for the [get_sign.c](https://github.com/klee/klee/blob/f9aa2a3534ac47e07cd1f8b21bafb784b7a0c6c6/examples/get_sign/get_sign.c#L19) example from the KLEE directory it would print a concrete value for the symbolic 32bit integer `a` in different representations (Python byte string, hexadecimal, little-endian uint/int, ...):
```
$ ktest-tool klee-last/test000003.ktest
ktest file : 'klee-last/test000003.ktest'
args       : ['get_sign.bc']
num objects: 1
object 0: name: 'a'
object 0: size: 4
object 0: data: b'\\x00\\x00\\x00\\x80'
object 0: hex : 0x00000080
object 0: int : -2147483648
object 0: uint: 2147483648
object 0: text: ....
```

## klee-stats

`klee-stats` is a Python script used to extract and present statistics from `run.stats` files present in KLEE's `klee-out-*` directories.
`klee-stats` can be invoked on a single directory or list of directories:

```
$ klee-stats klee-out-2
-------------------------------------------------------------------------
|   Path   |  Instrs|  Time(s)|  ICov(%)|  BCov(%)|  ICount|  TSolver(%)|
-------------------------------------------------------------------------
|klee-out-2|     499|     0.09|    45.69|    39.13|     394|        0.03|
-------------------------------------------------------------------------
$ klee-stats .
-------------------------------------------------------------------------
|   Path   |  Instrs|  Time(s)|  ICov(%)|  BCov(%)|  ICount|  TSolver(%)|
-------------------------------------------------------------------------
|klee-last |     499|     0.13|    45.69|    39.13|     394|        0.04|
|klee-out-3|     499|     0.03|    45.69|    39.13|     394|        0.12|
|klee-out-2|     499|     0.09|    45.69|    39.13|     394|        0.03|
|klee-out-8|     499|     0.13|    45.69|    39.13|     394|        0.04|
|klee-out-4|     499|     0.02|    45.69|    39.13|     394|        0.14|
-------------------------------------------------------------------------
|Total (5) |    2495|     0.40|    45.69|    39.13|    1970|        0.07|
-------------------------------------------------------------------------
```

This is only a small subset of statistics that KLEE keeps track of during execution.
By using `--print-all`, a much larger set can be displayed:

|Statistic|Description|
|:-|:-|
|Instrs|number of executed instructions|
|Time(s)|total wall time|
|ICov(%)|instruction coverage in the LLVM bitcode|
|BCov(%)|conditional branch (`br`) coverage in the LLVM bitcode|
|ICount|total static instructions in the LLVM bitcode|
|TSolver(%)|relative time spent in the solver chain wrt wall time (incl. caches and constraint solver)|
|ICovered|total covered instructions in the LLVM bitcode|
|IUncovered|total uncovered instructions in the LLVM bitcode|
|Branches|number of conditional branch (`br`) instructions in the LLVM bitcode|
|FullBranches|number of fully-explored conditional branch (`br`) instructions in the LLVM bitcode|
|PartialBranches|number of partially-explored conditional branch (`br`) instructions in the LLVM bitcode|
|ExternalCalls|number of external calls|
|TUser(s)|total user time|
|TResolve(s)|time spent in object resolution|
|TResolve(%)|relative time spent in object resolution wrt wall time|
|TCex(s)|time spent in the counterexample caching code (incl. constraint solver)|
|TCex(%)|relative time spent in the counterexample caching code wrt wall time (incl. constraint solver)|
|TQuery(s)|time spent in the constraint solver|
|TSolver(s)|time spent in the solver chain (incl. caches and constraint solver)|
|States|number of created states|
|ActiveStates|number of currently active states (`0` after successful termination)|
|MaxActiveStates|maximum number of active states|
|AvgActiveStates|average number of active states|
|InhibitedForks|number of inhibited state forks due to e.g. memory pressure|
|Queries|number of queries issued to the solver chain|
|SolverQueries|number of queries issued to the constraint solver|
|SolverQueryConstructs|number of query constructs for all queries send to the constraint solver|
|AvgSolverQuerySize|average number of query constructs per query issued to the constraint solver|
|QCacheMisses|Query cache misses|
|QCacheHits|Query cache hits|
|QCexCacheMisses|Counterexample cache misses|
|QCexCacheHits|Counterexample cache hits|
|Allocations|number of allocated heap objects of the program under test|
|Mem(MiB)|mebibytes of memory currently used|
|MaxMem(MiB)|maximum memory usage|
|AvgMem(MiB)|average memory usage|
|BrConditional|number of forks caused by symbolic branch conditions (`br`)|
|BrIndirect|number of forks caused by indirect branches (`indirectbr`) with symbolic address|
|BrSwitch|number of forks caused by switch with symbolic value|
|BrCall|number of forks caused by symbolic function pointers|
|BrMemOp|number of forks caused by memory operation with symbolic address|
|BrResolvePointer|number of forks caused by symbolic pointers|
|BrAlloc|number of forks caused by symbolic allocation size|
|BrRealloc|number of forks caused by symbolic reallocation size|
|BrFree|number of forks caused by freeing a symbolic pointer|
|BrGetVal|number of forks caused by user-invoked concretization while seeding|
|TermExit|number of states that reached end of execution path|
|TermEarly|number of early terminated states (e.g. due to memory pressure, state limt)|
|TermSolverErr|number of states terminated due to solver errors|
|TermProgrErr|number of states terminated due to program errors (e.g. division by zero)|
|TermUserErr|number of states terminated due to user errors (e.g. misuse of KLEE API)|
|TermExecErr|number of states terminated due to execution errors (e.g. unsupported intrinsics)|
|TermEarlyAlgo|number of state terminations required by algorithm (e.g. state merging or replaying)|
|TermEarlyUser|number of states terminated via klee_silent_exit()|
|TArrayHash(s)|time spent hashing arrays (if `KLEE_ARRAY_DEBUG` enabled, otherwise `-1`)|
|TFork(s)|time spent forking states|
|TFork(%)|relative time spent forking states wrt wall time|
|TUser(%)|relative user time wrt wall time|

In order to limit printed information only to the values of measured times, the following options can be used:

* `--print-rel-times` display time values relative to measured execution time
* `--print-abs-times` display absolute time values

Several table styles are supported (e.g. `csv`, `latex_booktabs` or `html`) that can be enabled with `--table-format=<format>`, e.g.:
```
$ klee-stats --table-format=readable-csv klee-out-2 klee-out-3
Path      ,  Instrs,  Time(s),  ICov(%),  BCov(%),  ICount,  TSolver(%)
klee-out-2,     499,     0.09,    45.69,    39.13,     394,        0.03
klee-out-3,     499,     0.03,    45.69,    39.13,     394,        0.12
```

Various other options can be used to specify what values are displayed and how they are displayed.
Options for comparison of statistics are also provided.
More information about available options can be obtained using the command:

```
$ klee-stats --help
```


### Conversion to comma-separated values (csv)

Starting with version 2.0, KLEE switched from csv to [SQLite3](https://www.sqlite.org) to store its statistics.
Of course, these files can be opened and queried with any SQLite client, e.g.:

```
$ sqlite3 <klee-out-dir>/run.stats
> SELECT * FROM stats
```

The easiest way to convert the entries for all statistics from a single `run.stats` file to comma-separated values (csv) is to use `klee-stats` with the `--to-csv` flag.
If the output needs to be modified or limited to specific columns and rows an SQLite client such as `sqlite3` comes handy:

```
$ sqlite3 -csv -header run.stats "select Instructions,printf(\"%.2f\",100.0*CoveredInstructions/(CoveredInstructions+UncoveredInstructions)) AS 'Icov(%)',printf(\"%.2f\",1.0*SolverTime/60000000) AS 'SolverTime(min)',NumQueries from stats ORDER BY WallTime DESC LIMIT 1" 
Instructions,Icov(%),SolverTime(min),NumQueries
2376923,3.96,51.77,514
```


### Live-monitoring with Grafana

`klee-stats` can also be used as a [Grafana](https://grafana.com/) data-source. This enables you to
create Grafana dashboards for live monitoring of your KLEE process. First,
`klee-stats` needs to be started with the `-grafana` flag to start serving the
data:

```
$ klee-stats --grafana <klee-out-dir>
```

Which starts on port 5000 by default. Then you can start the preconfigured
Grafana Docker image with:

```
$ docker run -d --net=host --name=grafana klee/grafana
```

This will create a daemon container running Grafana on port 3000. The image may
take half a minute or so to start up. Go to <http://localhost:3000>, then click
on 'Home' in the top left hand corner and select the dashboard named 'KLEE' from
the dropdown.

If you would like to see the progress as Grafana starts, you can instead run
Grafana in the foreground by omitting the ```-d``` flag. Grafana is ready when
the output stops and you see a line like this:
```
t=... lvl=info msg="HTTP Server Listen" logger=http.server address=0.0.0.0:3000 protocol=http subUrl= socket=
```

If you are using Grafana to view the statistics of a KLEE run that has already
finished, make sure to select a time range that includes the time when KLEE was
running. The time range can be changed by with the dropdown in the top right
corner.

You can then of course customize your dashboard, add more panels change time
ranges and enjoy the live monitoring of KLEE.

To stop Grafana:

```
$ docker stop grafana
```
Or if Grafana is running in the foreground then use Ctrl-C.

### Logging granularity

The intervals at which KLEE writes its statistics are [configurable]({{site.baseurl}}/docs/options/#statistics).
All times are lower bounds and a long running solver query might prevent KLEE from writing new entries.

## ktest-gen

A tool for generating a `.ktest` [file]({{site.baseurl}}/docs/files) from a concrete input.
The contents and format of the generated `.ktest` is the same as that described above (similarly, it can be converted into a human-readable form using `ktest-tool`).
The `.ktest` file can be replayed in KLEE (e.g., to generate the path conditions for a concrete input) and used as an interesting seed.

For example, suppose that you had previous fuzzed a target application with the [American Fuzzy Lop](https://github.com/google/AFL) (AFL) fuzzer.
After fuzzing, the input `queue/` contains the set of testcases that produced new state transitions.
The testcases in the queue can be converted to `.ktest` files so that they can be further-explored in KLEE:

```bash
# Assumes that you are in the AFL output directory (specified via the `-o` option when fuzzing.
# Ignores hidden directories.
# AFL-generated testcases always begin with 'id:'

find ./queue -not -path '*/\.*' -type f -name 'id:*'    \
    -exec ktest-gen --bout-file {}.ktest --sym-file {} \;
```

KLEE can subsequently be run with the `-seed-dir` option to seed further exploration.

## ktest-randgen

Similar to `ktest-gen`, except that it generates random data for the `.ktest` file.
