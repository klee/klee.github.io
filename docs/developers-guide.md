---
layout: default
title: Developer's Guide
subtitle: Working with KLEE source code
slug: developers-guide
---



{: .toc }
Contents
{:.toc__title .no_toc}
* Replaced with ToC
{:.toc__list .list-anchor}
{:toc}

## GitHub Environment

KLEE's codebase is currently hosted on [GitHub](https://github.com/klee/klee). For those unfamiliar with GitHub, a good starting point is [here](https://help.github.com/categories/54/articles).

We are using a fork & pull model in KLEE, based on pull requests. For those of you unfamiliar with the process, you can find more information [here](https://help.github.com/articles/using-pull-requests).

When submitting patches to KLEE, please open a separate pull request for each independent feature or bug fix. This makes it easier to review and approve patches.

## Build System

KLEE uses [cmake](https://cmake.org/) as build system. The very basic build
setup similar to what KLEE uses is presented in LLVM's [Writing an LLVM pass
tutorial](http://llvm.org/docs/WritingAnLLVMPass.html#setting-up-the-build-environment).


LLVM's build system supports out-of-source builds and so does KLEE.  It is
highly recommended you take advantage of this. For example, you could create
three builds (Release, Release with debug symbols, Debug) that all use the same
source tree. This allows you to keep your source tree clean and to build with
multiple configurations from a single source tree.

### Setting up a debug build of KLEE

Setting up a debug build of KLEE (we'll assume it is an out-of-source build) is
very similar to the build process described in [Getting
Started]({{site.baseurl}}/getting-started). KLEE uses standard [LLVM build type
idioms](http://llvm.org/docs/CMake.html#frequently-used-cmake-variables), so
building a Debug build means just setting the `CMAKE_BUILD_TYPE` variable to
`Debug` for example:

```bash
   cmake \
     -DCMAKE_BUILD_TYPE=Debug \
     -DENABLE_SOLVER_STP=ON \
     -DENABLE_POSIX_RUNTIME=ON \
     -DENABLE_KLEE_UCLIBC=ON \
     -DKLEE_UCLIBC_PATH=<KLEE_UCLIBC_SOURCE_DIR> \
     -DGTEST_SRC_DIR=<GTEST_SOURCE_DIR> \
     -DENABLE_SYSTEM_TESTS=ON \
     -DENABLE_UNIT_TESTS=ON \
     -DLLVM_CONFIG_BINARY=<PATH_TO_LLVM_llvm-config> \
     -DLLVMCC=<PATH_TO_clang> \
     -DLLVMCXX=<PATH_TO_clang++>
     <KLEE_SRC_DIRECTORY>
```

The rest of the build process is exactly the same as in our build guides. Note
that we only provide build guides for some popular LLVM versions, however KLEE
builds with many more (at the time of writing LLVM 3.4 - LLVM 10).  The build
process is exactly the same, cmake only needs `LLVM_CONFIG_BINARY`,
`LLVMCC` and `LLVMCXX` to point to versions of LLVM you want to build with. 


Note that KLEE depends on LLVM and STP (and optionally Z3). If you need to
debug KLEE's calls to that code, then you will need to build LLVM/STP/Z3 with
debug support too.

## Source overview

This section gives a brief overview of how KLEE's source code is structured:

* `include/` contains the publicly exported header files. That is header files
  that are accessible throughout the source code.

* `tools/` has the `main` functions for all KLEE binaries in the `bin/`
  directory. Note that some are Python scripts.

* `lib/` contains most of the code.
    * `lib/Core` contains code that interprets and executes the LLVM bitcode and
      KLEE's memory model. The `Executor.cpp` class is usually a good starting
      point for any KLEE extension.
    * `lib/Expr` has KLEE's expression library.
    * `lib/Solver` contains all the solvers (STP, Z3, MetaSMT) as well as all
      the solvers in the solver chain (Independent Solver and Counterexample
      Cache).
    * `lib/Module` deals with manipulating the LLVM bitcode before it is
      executed. It links in the (POSIX) runtime functions, runs
      optimisations and other passes. Some of these put the LLVM bitcode in the state
      KLEE expects and insert some runtime checks (such as instrumenting the
      division operations to check for div by zero errors).

* `runtime/` contains the various runtime KLEE supports. That is code that is
  linked in with the program KLEE analyses before execution.

* `tests/` contains small C programs and LLVM bitcode that is used as a
  regression test suite for KLEE.

### Adding a class

Because KLEE uses LLVM's build system, adding a class to an existing library in
KLEE is very simple. For example, to add a class to libkleaverExpr
(``lib/Expr``), the
following steps would be followed:

1. Create the header file (``.h``) for the class and place it somewhere inside
``include/`` (the location isn't really important except that ``#include`` is
relative to the ``include/`` directory). Note that if you only require the
header in the same directory you can also put it next to the source file (ie.
``lib/Expr/``).

2. Create the source file (``.cpp``) for the class place it in ``lib/Expr/``.

3. Add your ``.cpp`` file to the ``lib/Expr/CMakeLists.txt`` as an argument to
the ``klee_add_component`` function.

That's it! 

### Building code documentation

KLEE uses [Doxygen](http://www.doxygen.org) to generate code documentation. To generate it yourself you can run the following from KLEE's build directory root.

{% highlight bash %}
$ make docs
{% endhighlight %}

This will generate the documentation in `path/to/build-dir/docs/doxygen/`.

## Regression Testing Framework

KLEE uses LLVM's testing infrastructure for its regression tests. In KLEE these are...

* External tests executed by [llvm-lit](http://llvm.org/docs/CommandGuide/lit.html). These are the tests that are executed by the ``make check`` command. These test the KLEE tools externally.
* Internal tests that are driven using [GoogleTest](https://code.google.com/p/googletest/). These are the tests that are executed by the ``make unittests`` command.  These test KLEE's internal APIs.

### External tests

``llvm-lit`` is used to test KLEE's tools by invoking them as shell commands.

KLEE's tests are currently divided into categories, each of which is a subdirectory in ``test/`` in the source tree (e.g. ``test/Feature``).

``llvm-lit`` is passed one or more paths to test files or directories which it will search recursively for tests. The ``llvm-lit`` tool needs to know what test-suite the tests belong to and how to run them. This information is in the ``lit.site.cfg`` (generated by the ``Makefile``). This file itself refers to ``lit.cfg`` which tells ``llvm-lit`` how to run the test suite. At the time of writing the ``lit.cfg`` instructs ``llvm-lit`` to treat files with the following file extensions as tests:`` .ll .c .cpp .kquery``.

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
// RUN: %clang %s -emit-llvm %O0opt -c -o %t1.bc
// RUN: %klee --libc=klee --exit-on-error %t1.bc
{% endhighlight %}

This first runs ``clang`` on the source file (``%s``) and generates a temporary file (``%t1.bc``). Then, KLEE symbolically executes this bitcode file with one of its runtimes (here ``--libc=klee``). If either program returns a non-zero exit code (or crashes), the test is considered to have failed. More information on the available substitution variables (such as ``%s``) can be found [here](http://llvm.org/docs/TestingGuide.html#variables-and-substitutions).

For LLVM versions greater than 5.0 programs that are to be analysed with KLEE
should not be compiled with `-O0`, since it disables KLEE's ability to run
optimisation passes. Therefore, we have the `%O0opt` variable, which
substitutes to appropriate flags. At the time of writing these are: `-O0
-Xclang -disable-O0-optnone`. For more details see this
[issue](https://github.com/klee/klee/issues/902). 

To run the entire test suite run:

{% highlight bash %}
$ cd path/to/klee/build/test
$ make check
{% endhighlight %}

or you can use ``llvm-lit`` directly

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

The kleeCore library (``lib/Core``) provides several functions that can be used similarly to `printf()` in C. See `include/klee/Internal/Support/ErrorHandling.h` for more information.

### Adding a command line option to a tool

KLEE uses LLVM's CommandLine library for adding options to tools in KLEE, which is well documented [here](http://llvm.org/docs/CommandLine.html). See ``lib/Core/Executor.cpp`` for examples.

### Run-time libraries

KLEE searches for run-time libraries in install and build paths. These are hard-coded to the binary, so if the filesystem tree changes, KLEE will not find them until recompiled. This behaviour can be overridden by setting KLEE_RUNTIME_LIBRARY_PATH environment variable to the path to the libraries.
