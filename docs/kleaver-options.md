---
layout: default
title: Kleaver's Options
subtitle: Overview of Kleaver’s main command-line options
slug: kleaver-options
---

{:.toc}
Contents
{:.toc__title .no_toc}
* Table of contents placeholder
{:.toc__list .list-anchor}
{:toc}

## Kleaver usage

{% highlight bash %}
$ kleaver [options] <input query log>
{% endhighlight %}

In particular, `<input query log>` must be a file containing a query in [KQuery]({{site.baseurl}}/docs/kquery) format.
To get a complete list of Kleaver's command-line options run: `kleaver --help`

The remainder of this page illustrate Kleaver's distinctive command-line options.

## Parsing

Kleaver parses the input query log assuming the most general definition of the [KQuery]({{site.baseurl}}/docs/kquery) format. A KQuery source file consists of a sequence of declarations:

{% highlight ebnf %}
kquery = { array-declaration | query-command }
{% endhighlight %}

In particular, the set of array declarations can be used in several distinct queries.

### Optimize parsing in case of independent queries

An input query log may contain independent queries, that is query declarations that immediately follow their own array declarations. For instance, KLEE logs the queries issued to the solver as a list of independent queries. In this scenario, it is possible to optimize Kleaver runtime to substantially reduce its memory footprint by running Kleaver with the option `-clear-array-decls-after-query`. For example:

{% highlight bash %}
$ kleaver --clear-array-decls-after-query=true klee-queries.kquery
{% endhighlight %}

## Solver  

Kleaver can leverage several backend SMT solvers to compute the query:

1.  **STP (Default):** Simple Theorem Prover SMT solver [link](http://stp.github.io)
2.  **Z3:** The Z3 Theorem Prover [link](https://github.com/Z3Prover/z3)
3.  **metaSMT:** An Embedded Domain Specific Language for SMT [link](http://www.informatik.uni-bremen.de/agra/eng/metasmt.php)

To select a specific SMT solver, use the `-solver-backend` option provided by Kleaver. For example:

{% highlight bash %}
$ kleaver --solver-backend=stp query.kquery
{% endhighlight %}

## Query Logging

To log the queries issued by Kleaver to the underlying solver, you can use the same options described for KLEE [here]({{site.baseurl}}/docs/options/#query-logging).

To select where to store the log, you can use the `-query-log-dir` option. The default value is the current working directory. For example:

{% highlight bash %}
$ kleaver --query-log-dir=kleaver-log query.kquery
{% endhighlight %}
