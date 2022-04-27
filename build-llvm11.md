---
layout: default
title: LLVM 11 (recommended)
subtitle: Building KLEE with LLVM 11
slug: getting-started
---

{% include version_warning.md %}

The current procedure for building KLEE manually with LLVM 11 (recommended) on Debian/Ubuntu-based distributions or macOS is outlined below.
However, in case you trust our installation scripts for continuous integration testing, you can re-use them on your Ubuntu/macOS-based host system.
You can find detailed instructions in: [Building arbitrary KLEE configurations]({{site.baseurl}}/build-script).

## Manual Installation

**NOTE:** KLEE is currently tested on Linux x86-64 (particularly Ubuntu), FreeBSD and macOS.
There is no support for uClibc and the POSIX environment under macOS.
KLEE does not work under x86-32.

1. **Install dependencies:** KLEE requires all the dependencies of LLVM (see [here](http://llvm.org/docs/GettingStarted.html#requirements)), and some more. In particular, you should install the programs and libraries listed below. `graphviz/doxygen` are optional and only needed to generate the source code documentation.

   Under Ubuntu, use:
   ```bash
   $ sudo apt-get install build-essential cmake curl file g++-multilib gcc-multilib git libcap-dev libgoogle-perftools-dev libncurses5-dev libsqlite3-dev libtcmalloc-minimal4 python3-pip unzip graphviz doxygen
   ```

   Under macOS, run:
   ```bash
   $ brew install curl git cmake python unzip gperftools sqlite3 graphviz doxygen bash
   ```

   You should also install `lit` to enable testing, `tabulate` to support additional features in `klee-stats` and `wllvm` to make it easier to compile programs to LLVM bitcode:

   ```bash
   $ sudo pip3 install lit wllvm
   $ sudo apt-get install python3-tabulate
   ```

   Use `--user` for `pip3` to install packages for the current user only:

   ```bash
   $ pip3 install --user lit wllvm
   ```

   and make sure that e.g. `~/.local/bin` (check with `python3 -m site --user-base` on your system) is in your `PATH`.

2. **Install LLVM 11:** KLEE is built on top of [LLVM](http://llvm.org); the first steps are to get a working LLVM installation. See [Getting Started with the LLVM System](http://llvm.org/docs/GettingStarted.html) for more information.

   If you are using a recent Ubuntu (e.g. 21.10) or Debian, we recommend to use the LLVM packages provided by LLVM itself. 

   ```bash
   $ sudo apt-get install clang-11 llvm-11 llvm-11-dev llvm-11-tools
   ```

   If you are using macOS, you can install older LLVM packages using brew:
   ```bash
   $ brew install llvm@11 
   ```   

   That's it for LLVM.
   If you want to install it manually, please refer to the official [LLVM Getting Started documentation](https://releases.llvm.org/11.0.1/docs/GettingStarted.html).

3. **Install constraint solver(s)**

   KLEE supports multiple different constraint solvers. You must install at least one to build KLEE.

   * [STP](https://github.com/stp/stp) Historically KLEE was built around STP so support for this solver is the most stable. For build instructions, see [here]({{site.baseurl}}/build-stp).
   * [Z3](https://github.com/z3prover/z3) is another solver supported by KLEE that is reasonably stable. You should use Z3 version ≥ 4.4. Z3 is packaged by [many distributions](https://repology.org/project/z3/versions). For build instructions, see [here](https://github.com/Z3Prover/z3/blob/master/README.md).
   * [metaSMT](https://github.com/agra-uni-bremen/metaSMT) supports
     various solvers, including Boolector, CVC4, STP, Z3 and Yices.  We recommend branch v4.rc1 (`git clone -b v4.rc1 ...`). For build instructions, see [here](https://github.com/agra-uni-bremen/metaSMT).

4. **(Optional) Get Google test sources:**

   For unit tests we use the Google test libraries.
   If you want to run the unit tests you need to perform this step and also pass `-DENABLE_UNIT_TESTS=ON` to CMake when configuring KLEE in step 8.

   We currently recommend version `1.11.0`, so grab the sources for it.

   ```bash
   $ curl -OL https://github.com/google/googletest/archive/release-1.11.0.zip
   $ unzip release-1.11.0.zip
   ```

   This will create a directory called `googletest-release-1.11.0`.

5. **(Optional) Build uClibc and the POSIX environment model (not supported on macOS):** By default, KLEE works on closed programs (programs that don't use any external code such as C library functions). However, if you want to use KLEE to run real programs you will want to enable the KLEE POSIX runtime, which is built on top of the [uClibc](http://uclibc.org) C library.

   ```bash
   $ git clone https://github.com/klee/klee-uclibc.git
   $ cd klee-uclibc
   $ ./configure --make-llvm-lib # --with-cc clang-11 --with-llvm-config llvm-config-11
   $ make -j2
   $ cd ..
   ```
   When `clang` or `llvm-config` are not in your `PATH` or have a custom prefix/suffix, `configure` may fail to detect their location.
   You can use the `--with-cc` and `--with-llvm-config` flags to set the paths manually.

   **NOTE:** If you are on a different target (i.e., not i386 or x64), you will need to run `make config` and select the correct target.
   The defaults for the other uClibc configuration variables should be fine.

   To tell KLEE to use both klee-uclibc and the POSIX runtime, pass `-DENABLE_POSIX_RUNTIME=ON` and `-DKLEE_UCLIBC_PATH=<KLEE_UCLIBC_SOURCE_DIR>` to CMake when configuring KLEE in step 8 where `<KLEE_UCLIBC_SOURCE_DIR>` is the absolute path to the cloned `klee-uclibc` git repository.

6. **Get KLEE source**

   ```bash
   $ git clone https://github.com/klee/klee.git
   ```

7. **(Optional) Build libc++:** To be able to run C++ code, you also need to enable support for the C++ standard library.

   Make sure that `clang++-11` is in your path. Then, run from the main KLEE source directory:

   ```bash
   $ LLVM_VERSION=11 BASE=<LIBCXX_DIR> ./scripts/build/build.sh libcxx
   ```
   where `<LIBCXX_DIR>` is the absolute path where libc++ should be cloned and built.

   To tell KLEE to use libc++, pass the following flags to CMake when you configure KLEE in step 8:

   `-DENABLE_KLEE_LIBCXX=ON -DKLEE_LIBCXX_DIR=<LIBCXX_DIR>/libc++-install-110/ -DKLEE_LIBCXX_INCLUDE_DIR=<LIBCXX_DIR>/libc++-install-110/include/c++/v1/`

   To additionally enable KLEE's exception handling support for C++, pass the following flags to CMake when you configure KLEE in step 8:

   `-DENABLE_KLEE_EH_CXX=ON -DKLEE_LIBCXXABI_SRC_DIR=<LIBCXX_DIR>/llvm-110/libcxxabi/`

   `<LIBCXX_DIR>` must currently be an absolute path.
   If you want to build libc++ in your home path, note that in some environments (such as Ubuntu 18.04) `~` may not be an absolute path.
   You can use `$HOME` instead.

8. **Configure KLEE:**

   KLEE must be built "out of source", so first create a build directory.
   You can create this wherever you like.
   Below, we assume you create this directory inside KLEE's repository.

   ```bash
   $ mkdir build
   ```

   Now `cd` into the build directory and run CMake to configure KLEE where `<KLEE_SRC_DIRECTORY>` is the path to the KLEE git repository you cloned in the previous step.

   ```bash
   $ cd build
   $ cmake <CMAKE_OPTIONS> <KLEE_SRC_DIRECTORY>
   ```

   `<CMAKE_OPTIONS>` are the configuration options. These are documented in [README-CMake.md](https://github.com/klee/klee/blob/master/README-CMake.md).

   For example, if you want to build KLEE with STP, the POSIX runtime, klee-uclibc and unit testing then the command line would look something like this:

   ```bash
   $ cmake -DENABLE_SOLVER_STP=ON -DENABLE_POSIX_RUNTIME=ON -DKLEE_UCLIBC_PATH=<KLEE_UCLIBC_SOURCE_DIR> -DENABLE_UNIT_TESTS=ON -DGTEST_SRC_DIR=<GTEST_SOURCE_DIR> <KLEE_SRC_DIRECTORY>
   ```

   Where `<KLEE_UCLIBC_SOURCE_DIR>` is the absolute path to the klee-uclibc source tree and `<GTEST_SOURCE_DIR>` is the absolute path to the Google Test source tree.

   Or more concretely, with `/src` as working directory, `/src/klee/build` as build directory, and libcxx support enabled:
   ```bash
   $ cmake -DENABLE_SOLVER_STP=ON -DENABLE_POSIX_RUNTIME=ON -DKLEE_UCLIBC_PATH=/src/klee-uclibc -DENABLE_UNIT_TESTS=ON -DLLVM_CONFIG_BINARY=/usr/bin/llvm-config-11 -DGTEST_SRC_DIR=/src/googletest-release-1.11.0/ -DENABLE_KLEE_LIBCXX=ON -DKLEE_LIBCXX_DIR=/src/libcxx/libc++-install-110/ -DKLEE_LIBCXX_INCLUDE_DIR=/src/libcxx/libc++-install-110/include/c++/v1/ -DENABLE_KLEE_EH_CXX=ON -DKLEE_LIBCXXABI_SRC_DIR=/src/libcxx/llvm-110/libcxxabi/ ..
   ```

   **NOTE 1:** You can simply type `cmake ..` to use the default options for KLEE but these will not include support for uClibC and the POSIX runtime.

   **NOTE 2:** If LLVM is not found or you need a particular version to be used, you can pass `-DLLVM_CONFIG_BINARY=<LLVM_CONFIG_BINARY>` to CMake where `<LLVM_CONFIG_BINARY>` is the absolute path to the relevant `llvm-config` binary.
   Similarly, KLEE needs a C and C++ compiler that can create LLVM bitcode that is compatible with the LLVM version KLEE is using.
   If these are not detected automatically, `-DLLVMCC=<PATH_TO_CLANG>` and `-DLLVMCXX=<PATH_TO_CLANG++>` can be passed to explicitly set these compilers, where `<PATH_TO_CLANG>` is the absolute path to `clang` and `<PATH_TO_CLANG++>` is the absolute path to `clang++`.

   **NOTE 3:** By default, KLEE uses tcmalloc as its allocator, to support reporting of memory usage above 2GB.
   If you don't want to install tcmalloc (`libtcmalloc-minimal4 libgoogle-perftools-dev` Ubuntu packages) on your system or prefer to use the glibc allocator, pass `-DENABLE_TCMALLOC=OFF` to CMake when configuring KLEE.

9. **Build KLEE:**

   From the ``build`` directory created in the previous step run:

   ```bash
   $ make
   ```

9. **(Optional) Run the main regression test suite**

   If KLEE was configured with system tests enabled then you can run them like this:

   ```bash
   $ make systemtests
   ```

   If you want to invoke `lit` manually use:

   ```bash
   $ lit test/
   ```

   This way you can run individual tests or subsets of the suite:

   ```bash
   $ lit test/regression
   ```

9. **(Optional) Build and run the unit tests:**

   If KLEE was configured with unit tests enabled then you can build and run the unit tests:

   ```bash
   $ make unittests
   ```

   **NOTE:** You can run both, the system and unit tests, with `make check`.

9. **You're ready to go! Check the [Tutorials]({{site.baseurl}}/tutorials) page to try KLEE.**


**NOTE:** For testing real applications (e.g. Coreutils), you may need to increase your system's open file limit (ulimit -n).
Something between 10000 and 999999 should work.
In most cases, the hard limit will have to be increased first, so it is best to directly edit the corresponding configuration file (e.g., `/etc/security/limits.conf`).
