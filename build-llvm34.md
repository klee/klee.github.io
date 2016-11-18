---
layout: default
title: Building KLEE (recommended)
subtitle: with LLVM 3.4
slug: build-llvm34
---

{% include version_warning.md %}

The current procedure for building KLEE with LLVM 3.4 (recommended) is outlined below.
These instructions use KLEE's new CMake based build system. If you wish to build with
KLEE's older build system then see [here]({{site.baseurl}}/build-llvm34-autoconf).

If you want to build KLEE with LLVM 2.9, [click here]({{site.baseurl}}/build-llvm29).


1. **Install dependencies:** KLEE requires all the dependencies of LLVM, which are discussed [here](http://llvm.org/docs/GettingStarted.html#requirements). In particular, you should install the following programs and libraries, listed below as Ubuntu packages:  

   ```bash
   $ sudo apt-get install build-essential curl libcap-dev git cmake libncurses5-dev python-minimal python-pip unzip
   ```

   You will need gcc/g++ 4.8 or later installed on your system. For Ubuntu 12.04 and 13.04, you can follow the instructions [here](http://ubuntuhandbook.org/index.php/2013/08/install-gcc-4-8-via-ppa-in-ubuntu-12-04-13-04/).   

2. **Install LLVM 3.4:** KLEE is built on top of [LLVM](http://llvm.org); the first steps are to get a working LLVM installation. See [Getting Started with the LLVM System](http://llvm.org/docs/GettingStarted.html) for more information.

   _**NOTE:** Currently, KLEE has only experimental support for **LLVM 3.4**. The only stable LLVM version for KLEE is **LLVM 2.9**. KLEE is currently tested on **Linux x86-64**, and might break on x86-32. KLEE will **not** compile with LLVM versions prior to 2.9._

   If you are using a recent Ubuntu (≥ 12.04 and ≤ 15.10, e.g. 14.04 LTS) or Debian, we recommend you to use the LLVM packages provided by LLVM itself. Use [LLVM Package Repository](http://llvm.org/apt/) to add the appropriate line to your `/etc/apt/sources.list`. As an example, for Ubuntu 14.04, the following lines should be added:  

   ```bash
   deb http://llvm.org/apt/trusty/ llvm-toolchain-trusty-3.4 main  
   deb-src http://llvm.org/apt/trusty/ llvm-toolchain-trusty-3.4 main
   ```

   Then add the repository key and install the 3.4 packages:  

   ```bash
   $ wget -O - http://llvm.org/apt/llvm-snapshot.gpg.key|sudo apt-key add -  
   $ sudo apt-get update  
   $ sudo apt-get install clang-3.4 llvm-3.4 llvm-3.4-dev llvm-3.4-tools  
   ```

   Finally, make sure llvm-config is in your path:   

   That's it for LLVM. If you want to install it manually, please refer to the official [LLVM Getting Started documentation](http://www.llvm.org/docs/GettingStarted.html).<br/><br/>  

3. **Install constraint solver(s)**

   KLEE supports multiple different constraint solvers. You must install at least one to build KLEE.

   * [STP](https://github.com/stp/stp) Historically KLEE was built around only this solver so support for this solver is the most stable. For build instructions see [here]({{site.baseurl}}/build-stp).
   * [Z3](https://github.com/z3prover/z3) Z3 support is much more recent addition to KLEE but is reasonably stable. You should use Z3 version >= 4.4. For build instructions see [here](https://github.com/Z3Prover/z3/blob/master/README.md).
   * [metaSMT](https://github.com/agra-uni-bremen/metaSMT) **experimental** For build instructions see [here](https://github.com/agra-uni-bremen/metaSMT).

4. **(Optional) Build uclibc and the POSIX environment model:** By default, KLEE works on closed programs (programs that don't use any external code such as C library functions). However, if you want to use KLEE to run real programs you will want to enable the KLEE POSIX runtime, which is built on top of the [uClibc](http://uclibc.org) C library.  

   ```bash
   $ git clone https://github.com/klee/klee-uclibc.git  
   $ cd klee-uclibc  
   $ ./configure --make-llvm-lib  
   $ make -j2  
   $ cd .. 
   ```

   **NOTE:** If you are on a different target (i.e., not i386 or x64), you will need to run make config and select the correct target. The defaults for the other uClibc configuration variables should be fine.<br/><br/>  

5. **(Optional) Get Google test sources:**

   For unit tests we use the Google test libraries. If you don't want to run the unit tests you can skip this step but you will
   need to pass `-DENABLE_UNIT_TESTS=OFF` to CMake when configuring KLEE in step 9.

   We depend on a version `1.7.0` right now so grab the sources for it.

   ```bash
   $ curl -OL https://github.com/google/googletest/archive/release-1.7.0.zip
   $ unzip release-1.7.0.zip
   ```

   This will create a directory called `googletest-release-1.7.0`.

6. **(Optional) Install lit:**

   For testing the `lit` tool is used. If you LLVM from a build tree you can
   skip this step as the build system will try to use `llvm-lit` in the
   directory containing the LLVM binaries.

   If you don't want to run the tests you can skip this step but you will need
   to pass `-DENABLE_TESTS=OFF` to CMake when configuring KLEE in step 9.

   ```bash
   $ pip install lit
   ```

7. **(Optional) Install tcmalloc:**

   By default, KLEE uses `malloc_info()` to observe and to restrict its memory usage.
   Due to limitations of `malloc_info()`, the maximum limit is set to 2 GB. To support bigger limits, KLEE can use TCMalloc as an alternative allocator. It is thus necessary to install TCMalloc:

   ```bash
   $ sudo apt-get install libtcmalloc-minimal4 libgoogle-perftools-dev
   ```

   When configuring KLEE in step 9 pass `-DENABLE_TCMALLOC=ON` to CMake when configuring KLEE.

8. **Get KLEE source:**  

   ```bash
   $ git clone https://github.com/klee/klee.git
   ```

9. **Configure KLEE:**

   KLEE must be built "out of source" so first make a binary build directory. You can create this where ever you like.

   ```bash
   $ mkdir klee_build_dir
   ```

   Now `cd` into the build directory and run CMake to configure KLEE where `<KLEE_SRC_DIRECTORY>` is the path
   to the KLEE git repository you cloned in step 7.

   ```bash
   $ cd klee_build_dir
   $ cmake <CMAKE_OPTIONS> <KLEE_SRC_DIRECTORY>
   ```

   `<CMAKE_OPTIONS>` are the configuration options. These are documented in [README-CMake.md](https://github.com/klee/klee/blob/master/README-CMake.md).

   For example if KLEE was being built with STP, the POSIX runtime, klee-uclibc and testing then the
   command line would look something like this

   ```bash
   cmake \
     -DENABLE_SOLVER_STP=ON \
     -DENABLE_POSIX_RUNTIME=ON \
     -DENABLE_KLEE_UCLIBC \
     -DKLEE_UCLIBC_PATH=<KLEE_UCLIBC_SOURCE_DIR> \
     -DGTEST_SRC_DIR=<GTEST_SOURCE_DIR> \
     -DENABLE_TESTS=ON \
     -DENABLE_SYSTEM_TESTS=ON \
     -DENABLE_UNIT_TESTS=ON \
     <KLEE_SRC_DIRECTORY>
   ```

   Where `<KLEE_UCLIBC_SOURCE_DIR>` is the absolute path the klee-uclibc source tree,
   `<GTEST_SOURCE_DIR>` is the absolute path to the Google Test source tree.


   **NOTE:** If LLVM is not found or you need a particular version to be used you can pass `-DLLVM_CONFIG_BINARY=<LLVM_CONFIG_BINARY>` to CMake where `<LLVM_CONFIG_BINARY>` is the absolute path to the
   relevant `llvm-config` binary. Similary KLEE needs a C and C++ compiler that can create LLVM bitcode that is compatible with the version of LLVM KLEE is using. If these are not detected automatically `-DLLVMCC=<PATH_TO_CLANG>` and `-DLLVMCXX=<PATH_TO_CLANG++>` can be passed to explicitly set these compilers where `<PATH_TO_CLANG>` is the absolute path to `clang` and `<PATH_TO_CLANG++>` is the absolute path to `clang++`.


9. **Build KLEE:**

   From the ``klee_build_dir`` directory created in the previous step run.

   ```bash
   $ make
   ```

9. **(Optional) Run the main regression test suite**

   If KLEE was configured with system tests enabled
   then you can run them like this.
   
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

   If KLEE was configured with unit tests enabled then you can build and run the
   unit tests like this.

   ```bash
   $ make unittests
   ```

9. **You're ready to go! Check the [Tutorials]({{site.baseurl}}/tutorials) page to try KLEE.**

**NOTE:** For testing real applications (e.g. Coreutils), you may need to increase your system's open file limit (ulimit -n). Something between 10000 and 999999 should work. In most cases, the hard limit will have to be increased first, so it is best to directly edit the `/etc/security/limits.conf` file.<br/><br/>
