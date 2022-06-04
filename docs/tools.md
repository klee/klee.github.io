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

_klee-stats_ is a Python script used to extract and present in a tabular form runtime statistics for a KLEE execution. The runtime statistics include:

* The number of executed instructions
* Instruction coverage in the LLVM bitcode (%)
* Branch coverage in the LLVM bitcode (%)
* Total static instructions in the LLVM bitcode
* The number of currently active states
* Megabytes of memory currently used
* The number of queries issued to the SMT solver
* The average number of query constructs per query
* Various time statistics:
  * Total user time
  * Total wall time
  * Time spent in the constraint solver
  * Time spent in the counterexample caching code
  * Time spent forking
  * Time spent in object resolution

_klee-stats_ extracts statistics information from the `run.stats` file present in the `klee-out-*` directory created during a KLEE execution. The exact usage of _klee-stats_ is as follows:

```
klee-stats [options] directories
```

The _directories_ parameter is a list of `klee-out-*` directories created by KLEE. A common scenario is to simply run _klee-stats_ on _klee-last_.

In order to limit printed information only to the values of measured times, the following options can be used:

* `--print-rel-times` display time values relative to measured execution time
* `--print-abs-times` display absolute time values

The `--precision` option can be used to configure the number of fractional digits displayed in floating point values. By default, 2 fractional digits are displayed, but in some cases that might be not sufficient—if the value is very small, e.g. 0.0001, with 2-digits precision it will be printed as 0.00.

Several table styles are supported, e.g. `latex_booktabs` or `html`, and can be enabled with `--table-format=<format>`.

Various other options can be used to specify what values are displayed and how they are displayed. Options for comparison of statistics are also provided. More information about available options can be obtained using the command:

{% highlight bash %}
$ klee-stats --help
{% endhighlight %}


### Conversion to comma-separated values (csv)

Starting with version 2.0, KLEE switched from csv to [SQLite3](https://www.sqlite.org) to store its statistics.
Of course, these files can be opened and queried with any SQLite client, e.g.:

```
$ sqlite3 <klee-out-dir>/run.stats
> SELECT * FROM stats
```

The easiest way to convert all statistics to comma-separated values (csv) is to use `klee-stats` with `--to-csv` flag.
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
klee-stats --grafana <klee-out-dir>
```

Which starts on port 5000 by default. Then you can start the preconfigured
Grafana Docker image with:

```
docker run -d --net=host --name=grafana klee/grafana
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
docker stop grafana
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
