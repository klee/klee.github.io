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

KLEE uses LLVM's testing infrastructure for its regression tests, which itself uses [DejaGnu](http://www.gnu.org/software/dejagnu/). These are the tests that are executed by the make check command. Some documentation on LLVM's testing infrastructure can be found [here](http://llvm.org/docs/TestingGuide.html#rtcustom).

KLEE's tests are currently divided into categories, each of which is a subdirectory in test/ in the source tree (e.g. test/Feature) . The dg.exp file in each subdirectory instructs the LLVM testing infrastructure which files in the subdirectory are to be used as tests. For example, test/Expr/dg.exp contains:

{% highlight tcl %}
load_lib llvm.exp
    
RunLLVMTests [lsort [glob -nocomplain $srcdir/$subdir/*.{pc}]]
{% endhighlight %}

This instructs the testing infrastructure that every .pc file in test/Expr should be used as a test.

The actions performed in each test are specified by special comments in the file. For example, in test/Feature/ByteSwap.c the first two lines are

{% highlight c %}
// RUN: %llvmgcc %s -emit-llvm -O0 -c -o %t1.bc  
// RUN: %klee --libc=klee --exit-on-error %t1.bc  
{% endhighlight %}

This first runs llvm-gcc on the source file (`%s`) and generates a temporary file (`%t1.bc`). Then KLEE is executed on this generated temporary file. If either program returns a non-zero exit code (or crashes) then test is considered to have failed. More information on the available substitution variables (such as `%s`) can be found [here](http://llvm.org/releases/3.1/docs/TestingGuide.html#rtvars).

It is useful to be able to execute just a single test rather than all of them. KLEE provides a makefile target for doing so which can used as shown below. Note that _category/test-name_ should be the test that one would like to execute, e.g. Feature/ByteSwap.c.

{% highlight bash %}
$ cd path/to/build-dir/test  
$ make check-one TESTONE=path/to/source-dir/test/category/test-name
{% endhighlight %}

# Internal tests
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
