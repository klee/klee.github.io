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

For advanced use cases, such as data conversion to comma-separated values (csv) or live monitoring with Grafana, see the [Developer documentation]({{site.baseurl}}/docs/developers-guide/#klee-statistics).
