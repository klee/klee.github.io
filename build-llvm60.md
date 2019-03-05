---
layout: default
title: Building KLEE (recommended)
subtitle: with LLVM 6.0
slug: build-llvm60
---

{% include version_warning.md %}

The current procedure for building KLEE with LLVM 6.0 (recommended) is outlined below. 

**NOTE:** KLEE is currently tested on **Linux x86-64**, and might break on x86-32.

1. **Install dependencies:** KLEE requires all the dependencies of LLVM, which are discussed [here](http://llvm.org/docs/GettingStarted.html#requirements). In particular, you should install the following programs and libraries, listed below as Ubuntu packages:  

   ```bash
   $ sudo apt-get install build-essential curl libcap-dev git cmake libncurses5-dev python-minimal python-pip unzip libtcmalloc-minimal4 libgoogle-perftools-dev
   ```

   You will need gcc/g++ 4.8 or later installed on your system. For Ubuntu 12.04 and 13.04, you can follow the instructions [here](http://ubuntuhandbook.org/index.php/2013/08/install-gcc-4-8-via-ppa-in-ubuntu-12-04-13-04/).   

2. **Install LLVM 6.0:** KLEE is built on top of [LLVM](http://llvm.org); the first steps are to get a working LLVM installation. See [Getting Started with the LLVM System](http://llvm.org/docs/GettingStarted.html) for more information.

   If you are using a recent Ubuntu (e.g. 18.04 LTS) or Debian, we recommend you to use the LLVM packages provided by LLVM itself. 


   ```bash
   $ sudo apt-get install clang-6.0 llvm-6.0 llvm-6.0-dev llvm-6.0-tools  
   ```

   That's it for LLVM. If you want to install it manually, please refer to the official [LLVM Getting Started documentation](http://releases.llvm.org/3.8.0/docs/GettingStarted.html).

3. **Install constraint solver(s)**

   KLEE supports multiple different constraint solvers. You must install at least one to build KLEE.

   * [STP](https://github.com/stp/stp) Historically KLEE was built around STP so support for this solver is the most stable. For build instructions, see [here]({{site.baseurl}}/build-stp).
   * [Z3](https://github.com/z3prover/z3) is a more recent addition to KLEE but is reasonably stable. You should use Z3 version ≥ 4.4. For build instructions, see [here](https://github.com/Z3Prover/z3/blob/master/README.md).
   * [metaSMT](https://github.com/agra-uni-bremen/metaSMT) supports
     various solvers, including Boolector, CVC4, STP, Z3 and Yices.  We recommend branch v4.rc1 (`git clone -b v4.rc1 ...`). For build instructions, see [here](https://github.com/agra-uni-bremen/metaSMT).

4. **(Optional) Build uclibc and the POSIX environment model:** By default, KLEE works on closed programs (programs that don't use any external code such as C library functions). However, if you want to use KLEE to run real programs you will want to enable the KLEE POSIX runtime, which is built on top of the [uClibc](http://uclibc.org) C library.

   **To build klee-uclibc run:**

   ```bash
   $ git clone https://github.com/klee/klee-uclibc.git  
   $ cd klee-uclibc  
   $ ./configure --make-llvm-lib  
   $ make -j2  
   $ cd .. 
   ```
   **NOTE:** If you are on a different target (i.e., not i386 or x64), you will need to run make config and select the correct target. The defaults for the other uClibc configuration variables should be fine.  <br/><br/>  

   To tell KLEE to use both klee-uclibc and the POSIX runtime, pass
   `-DENABLE_POSIX_RUNTIME=ON` and `-DKLEE_UCLIBC_PATH=<KLEE_UCLIBC_SOURCE_DIR>`
   to CMake when configuring KLEE in step 8 where `<KLEE_UCLIBC_SOURCE_DIR>` is
   the absolute path to the cloned `klee-uclibc` git repository.<br/><br/>  

5. **(Optional) Get Google test sources:**

   For unit tests we use the Google test libraries. If you want to run the unit tests you need to perform this step and also pass `-DENABLE_UNIT_TESTS=ON` to CMake when configuring KLEE in step 8.

   We depend on a version `1.7.0` right now so grab the sources for it.

   ```bash
   $ curl -OL https://github.com/google/googletest/archive/release-1.7.0.zip
   $ unzip release-1.7.0.zip
   ```

   This will create a directory called `googletest-release-1.7.0`.

6. **(Optional) Install lit:**

   For testing, the `lit` tool is used. If you installed LLVM from a build tree, you can
   skip this step as the build system will try to use `llvm-lit` in the
   directory containing the LLVM binaries.

   If you don't want to run the tests you can skip this step but you will need
   to pass `-DENABLE_UNIT_TESTS=OFF` and `-DENABLE_SYSTEM_TESTS=OFF` to CMake
   when configuring KLEE in step 9.

   ```bash
   $ pip install lit
   ```
   (use `--user` to install it for the current user only)

7. **Get KLEE source:**  

   ```bash
   $ git clone https://github.com/klee/klee.git
   ```
   

8. **Configure KLEE:**  

   KLEE must be built "out of source", so first create a build directory. You can create this wherever you like.  Below, we assume you create this directory inside KLEE's repository.

   ```bash
   $ mkdir build
   ```

   Now `cd` into the build directory and run CMake to configure KLEE where `<KLEE_SRC_DIRECTORY>` is the path
   to the KLEE git repository you cloned in step 7.
   
   ```bash
   $ cd build
   $ cmake <CMAKE_OPTIONS> <KLEE_SRC_DIRECTORY>
   ```

   `<CMAKE_OPTIONS>` are the configuration options. These are documented in [README-CMake.md](https://github.com/klee/klee/blob/master/README-CMake.md).

   For example, if you want to build KLEE with STP, the POSIX runtime, klee-uclibc and unit testing then the
   command line would look something like this

   ```bash
   cmake \
     -DENABLE_SOLVER_STP=ON \
     -DENABLE_POSIX_RUNTIME=ON \
     -DENABLE_KLEE_UCLIBC=ON \
     -DKLEE_UCLIBC_PATH=<KLEE_UCLIBC_SOURCE_DIR> \
     -DENABLE_UNIT_TESTS=ON \
     -DGTEST_SRC_DIR=<GTEST_SOURCE_DIR> \
     -DLLVM_CONFIG_BINARY=<PATH_TO_llvm-config-6.0> \
     -DLLVMCC=<PATH_TO_clang-6.0> \
     -DLLVMCXX=<PATH_TO_clang++-6.0>
     <KLEE_SRC_DIRECTORY>
   ```

   Where `<KLEE_UCLIBC_SOURCE_DIR>` is the absolute path to the klee-uclibc source tree,
   `<GTEST_SOURCE_DIR>` is the absolute path to the Google Test source tree.

   **NOTE 1:** You can simply type `cmake ..` to use the default options for KLEE (but note that these will not include support for uClibC and the POSIX runtime.

   **NOTE 2:** If LLVM is not found or you need a particular version to be used, you can pass `-DLLVM_CONFIG_BINARY=<LLVM_CONFIG_BINARY>` to CMake where `<LLVM_CONFIG_BINARY>` is the absolute path to the
   relevant `llvm-config` binary. Similarly, KLEE needs a C and C++ compiler that can create LLVM bitcode that is compatible with the LLVM version KLEE is using. If these are not detected automatically, `-DLLVMCC=<PATH_TO_CLANG>` and `-DLLVMCXX=<PATH_TO_CLANG++>` can be passed to explicitly set these compilers, where `<PATH_TO_CLANG>` is the absolute path to `clang` and `<PATH_TO_CLANG++>` is the absolute path to `clang++`.

   **NOTE 3:** By default, KLEE uses tcmalloc as its allocator, to support reporting of memory usage above 2GB. If you don't want to install tcmalloc (`libtcmalloc-minimal4 libgoogle-perftools-dev` Ubuntu packages) on your system or prefer to use glibc allocator, pass `-DENABLE_TCMALLOC=OFF` to CMake when configuring KLEE.


9. **Build KLEE:**

   From the ``build`` directory created in the previous step run.

   ```bash
   $ make
   ```

   **NOTE:** If you see linker errors involving `cxx11`, you may be running into the [dual ABI issue](https://github.com/klee/klee/issues/336#issuecomment-181827009).
   Here's an example:

   ```
/usr/lib/llvm-3.4/include/llvm/Support/CommandLine.h:905: undefined reference to `vtable for llvm::cl::parser<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > >'
CMakeFiles/kleaver.dir/main.cpp.o: In function `main':
/home/user/programs/klee/klee/tools/kleaver/main.cpp:413: undefined reference to `llvm::error_code::message[abi:cxx11]() const'
CMakeFiles/kleaver.dir/main.cpp.o: In function `llvm::cl::opt<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >, false, llvm::cl::parser<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > > >::~opt()':
   ```

   This is caused by a mismatch between the ABI used to build LLVM and the ABI used to build KLEE. To fix this delete your KLEE build directory and rerun `cmake` in this way:

   ```bash
   $ CXXFLAGS="-D_GLIBCXX_USE_CXX11_ABI=0" cmake <CMAKE_OPTIONS> <KLEE_SRC_DIRECTORY>
   ```

   <a name="rtti_link_error">**NOTE:**</a> If you see linker errors involving undefined references to `typeinfo` this is likely an [RTTI issue](https://github.com/klee/klee/issues/508).
   Here's an example:

   ```
[ 81%] Linking CXX executable ../../bin/kleaver
CMakeFiles/kleaver.dir/main.cpp.o:(.rodata+0x1238): undefined reference to `typeinfo for llvm::cl::Option'
CMakeFiles/kleaver.dir/main.cpp.o:(.rodata+0x1270): undefined reference to `typeinfo for llvm::cl::generic_parser_base'
CMakeFiles/kleaver.dir/main.cpp.o:(.rodata+0x12d0): undefined reference to `typeinfo for llvm::cl::GenericOptionValue'
CMakeFiles/kleaver.dir/main.cpp.o:(.rodata+0x12f8): undefined reference to `typeinfo for llvm::cl::Option'
CMakeFiles/kleaver.dir/main.cpp.o:(.rodata+0x1330): undefined reference to `typeinfo for llvm::cl::generic_parser_base'
CMakeFiles/kleaver.dir/main.cpp.o:(.rodata+0x1390): undefined reference to `typeinfo for llvm::cl::GenericOptionValue'
   ```

   The issue here is that LLVM was built without RTTI, but KLEE is trying to build with RTTI. This is caused by the `llvm-config` binary not correctly reporting
   that `-fno-rtti` needs to be passed to the compiler. To fix this delete your KLEE build directory and rerun `cmake` like so

   ```
   $ CXXFLAGS="-fno-rtti" cmake <CMAKE_OPTIONS> <KLEE_SRC_DIRECTORY>
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
   
   **NOTE:** You can run both the system and unit tests with `make check`

9. **You're ready to go! Check the [Tutorials]({{site.baseurl}}/tutorials) page to try KLEE.**

**NOTE:** For testing real applications (e.g. Coreutils), you may need to increase your system's open file limit (ulimit -n). Something between 10000 and 999999 should work. In most cases, the hard limit will have to be increased first, so it is best to directly edit the corresponding configuration file (e.g., `/etc/security/limits.conf`).<br/><br/>
