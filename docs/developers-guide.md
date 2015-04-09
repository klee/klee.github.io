---
layout: default
title: Developer's Guide
subtitle: Working with KLEE source code
lead: This guide covers several areas of KLEE that may not be imediately obvious to new developers.
slug: developers-guide
---

{: .toc }
Contents
{:.toc__title .no_toc}
* Replaced with ToC
{:.toc__list .list-anchor}
{:toc}

## GitHub Environment

KLEE's codebase is currently hosted on [GitHub](https://github.com/ccadar/klee). For those unfamiliar with GitHub, a good starting point is [here](https://help.github.com/categories/54/articles).

We are using a fork&amppull model in KLEE, based on pull requests. For those of you unfamiliar with the process, you can find more information [here](https://help.github.com/articles/using-pull-requests).

When submitting patches to KLEE, please open a separate pull request for each independent feature or bug fix. This makes it easier to review and approve patches.

## Build System

KLEE uses LLVM's ability to build third-party projects, which is described [here](http://llvm.org/docs/Projects.html). The build system uses [GNU Autoconf](http://www.gnu.org/software/autoconf/) and [AutoHeader](http://www.gnu.org/savannah-checkouts/gnu/autoconf/manual/autoconf-2.69/html_node/autoheader-Invocation.html) to configure the build, but does not use the rest of GNU Autotools (e.g. automake).

LLVM's build system supports out-of-source builds and therefore so does KLEE. It is highly recommended you take advantage of this. For example, you could create three builds (Release, Release with debug symbols, Debug) that all use the same source tree. This allows you keep your source tree clean and allows multiple configurations to be tested from a single source tree.

### Setting up a debug build of KLEE

Setting up a debug build of KLEE (we'll assume it is an out-of-source build) is very similar to the build process described in [Getting Started]({{site.baseurl}}/getting-started), with the exception of steps 6 and 7.

1.  Now we will configure KLEE. Notice that we are forcing the compiler to produce unoptimised code, this isn't the default behaviour.

    ```bash
    $ mkdir path/to/build-dir  
    $ cd path/to/build-dir  
    $ CXXFLAGS="-g -O0" CFLAGS="-g -O0" path/to/source-dir/configure --with-llvm=path/to/llvm --with-stp=path/to/stp/install --with-uclibc=path/to/klee-uclibc --enable-posix-runtime --with-runtime=Debug+Asserts
    ```

    Note if you're using an out-of-source build of LLVM you will need to use `--with-llvmsrc=` and `--with-llvmobj=` configure options instead of `--with-llvm=`
    
2.  Now we can build KLEE.

    ```bash
    $ make -j
    ```

    Note that we are using the `-j` option of make to speed up the compilation process.

Note that KLEE depends on LLVM and STP. If you need to debug KLEE's calls to that code, then you will need to build LLVM/STP with debug support too.

### Adding a class

Because KLEE uses LLVM's build system, adding a class to an existing library in KLEE is very simple. For example, to add a class to libkleaverExpr, the following steps would be followed:

1. Create the header file (.h) for the class and place it somewhere inside include/ (the location isn't really important except that #include is relative to the include/ directory).
2. Create the source file (.cpp) for the class place it in lib/Expr/. You can confirm that the library in which your new class will be included is kleaverExpr by looking at the Makefile in lib/Expr.

That's it! Now LLVM's build system will detect the new .cpp file and add it to the library that is generated when you run make.

### Building code documentation

KLEE uses [Doxygen](http://www.doxygen.org) to generate code documentation. To generate it yourself you can run the following from KLEE's build directory root.

{% highlight bash %}
$ make doxygen
{% endhighlight %}

This will generate documentation in `path/to/build-dir/docs/doxygen/` folder. You can also find KLEE's latest official code documentation [here](http://test.minormatter.com/~ddunbar/klee-doxygen/index.html)

## Regression Testing Framework

KLEE uses LLVM's testing infrastructure for its regression tests. In KLEE these are...

* External tests executed by [llvm-lit](http://llvm.org/docs/CommandGuide/lit.html). These are the tests that are executed by the ``make check`` command. These test the KLEE tools externally.
* Internal tests that are driven using [GoogleTest](https://code.google.com/p/googletest/). These are the tests that are executed by the ``make unittests`` command.  These test KLEE's internal APIs.

### External tests

``llvm-lit`` is used to test KLEE's tools by invoking them as shell commands.

KLEE's tests are currently divided into categories, each of which is a subdirectory in ``test/`` in the source tree (e.g. ``test/Feature``).

``llvm-lit`` is passed one or more paths to test files or directories which it will search recursively for tests. The ``llvm-lit`` tool needs to know what test-suite the tests belong to and how to run them. This information is in the ``lit.site.cfg`` (generated by the ``Makefile``). This file itself refers to ``lit.cfg`` which tells ``llvm-lit`` how to run the test suite. At the time of writing the ``lit.cfg`` instructs ``llvm-lit`` to treat files with the following file extensions as tests:`` .ll .c .cpp .pc``.

It is desirable to disable some tests (e.g. disable klee-uclibc tests if support was not compiled in) or change which file extensions to look for. This is done by adding a ``lit.local.cfg`` file to a directory which can be used to customise how tests in that directory are executed. For example to change the file extensions searched for to ``.cxx`` and ``.txt`` the following could be used in a ``lit.local.cfg`` file:

{% highlight python %}
config.suffixes = ['.cxx', '.txt' ]
{% endhighlight %}

To disable execution of tests in a directory based on a condition you can
mark them as unsupported by placing the following inside a
``lit.local.cfg`` file in that directory (where ``some_condition`` is any Python expression):

{% highlight python %}
if some_condition:  config.unsupported = True
{% endhighlight %}

All ``llvm-lit`` configuration files are Python scripts loaded by ``llvm-lit`` so you have the full power of Python at your disposal.

The actions performed in each test are specified by special comments in the file. For example, in ``test/Feature/ByteSwap.c`` the first two lines are:

{% highlight c %}
// RUN: %llvmgcc %s -emit-llvm -O0 -c -o %t1.bc
// RUN: %klee --libc=klee --exit-on-error %t1.bc
{% endhighlight %}

This first runs ``llvm-gcc`` (or ``clang``) on the source file (``%s``) and generates a temporary file (``%t1.bc``). Then KLEE is executed on this generated temporary file. If either program returns a non-zero exit code (or crashes) then test is considered to have failed. More information on the available substitution variables (such as ``%s</tt>``) can be found (here)[http://llvm.org/docs/TestingGuide.html#variables-and-substitutions].

To run the entire test suite run:

{% highlight bash %}
$ cd path/to/klee/build/test
$ make check
{% endhighlight %}

or you can use ``llvm-lit`` directy

{% highlight bash %}
$ cd path/to/klee/build/test
$ llvm-lit .
{% endhighlight %}

If you want to run a subset of the test suite simply pass the filenames of the tests or directories to search for tests to ``llvm-lit``. For example the commands below will execute the ``Feature/DoubleFree.c`` and ``CXX/ArrayNew.cpp`` test and all tests nested in the ``regression/`` folder.

{% highlight bash %}
$ cd path/to/klee/build/test
$ llvm-lit Feature/DoubleFree.c CXX/ArrayNew.cpp  regression/
{% endhighlight %}

Sometimes it can be useful to pass extra command line options to ``klee`` or ``kleaver`` when running tests. This could be used for example to quickly see if changing the default value for a command line option changes the passing/failing of a test. This is **not** a substitute for writing new tests for ``klee`` or ``kleaver``! If you add a new feature that is exposed by a new command line option a new test should be added that tests this behaviour.

Here is an example:

{% highlight bash %}
$ cd test/
$ llvm-lit --param klee_opts=-use-forked-solver=0 --param kleaver_opts="-use-forked-solver=0 -use-query-log=all:smt2" .
{% endhighlight %}

In this example when running ``klee`` in tests an extra option ``-use-forked-solver=0`` is passed to ``klee`` and when running ``kleaver`` the ``-use-forked-solver=0`` and ``-use-query-log=all:smt2`` options will be passed to ``kleaver``. It is important to realise if the test already invokes ``klee`` or ``kleaver`` with a particular option you will not be able to override that option because the ``klee_opts`` and ``kleaver_opts`` are substituted just after the tool name so subsequent options used in the test will override these.

For more information on ``llvm-lit`` tests see [LLVM's testing infrastructure documentation](http://llvm.org/docs/TestingGuide.html#id1) and the [``llvm-lit`` tool documentation](http://llvm.org/docs/CommandGuide/lit.html).

### Internal tests

These tests are located in ``unittests/`` and can be executed by running:
{% highlight bash %}
$ cd path/to/klee/build
$ make unittests
{% endhighlight %}

These test use [Google's C++ testing framework](https://code.google.com/p/googletest/) and is well [documented](https://code.google.com/p/googletest/wiki/Documentation).

## Miscellaneous

### Writing messages to standard error

The kleeCore library (lib/Core) provides several functions that can be used similarly to `printf()` in C. See `lib/Core/Common.h` for more information.

### Adding a command line option to a tool

KLEE uses LLVM's CommandLine library for adding options to tools in KLEE, which is well documented [here](http://llvm.org/docs/CommandLine.html). See lib/core/Executor.cpp for examples.

### Run-time libraries

KLEE searches for run-time libraries in install and build paths. These are hard-coded to the binary, so if the filesystem tree changes, KLEE will not find them until recompiled. This behaviour can be overridden by setting _KLEE_RUNTIME_LIBRARY_PATH_ environment variable to the path to the libraries.
