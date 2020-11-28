---
layout: default
title: LLVM 9 (recommended)
subtitle: Building KLEE with LLVM 9
slug: getting-started
---

{% include version_warning.md %}

The current procedure for building KLEE with LLVM 9 (recommended) is outlined below.


**NOTE:** KLEE is currently tested on Linux x86-64 (particularly
Ubuntu), FreeBSD and macOS.  There is no support for uClibc and the
POSIX environment under macOS. KLEE might not work under x86-32.

1. **Install dependencies:** KLEE requires all the dependencies of LLVM (see [here](http://llvm.org/docs/GettingStarted.html#requirements), and some more. In particular, you should install the programs and libraries listed below.

   Under Ubuntu, use:
   ```bash
   $ sudo apt-get install build-essential curl libcap-dev git cmake libncurses5-dev python-minimal python-pip unzip libtcmalloc-minimal4 libgoogle-perftools-dev libsqlite3-dev doxygen
   ```
   For Ubuntu >= 18.04, some additional packages may be required along with the ones mentioned above:
   ```bash
   $ sudo apt-get install python3 python3-pip gcc-multilib g++-multilib 
   ```
   Under macOS, run:
   ```bash
   $ brew install curl git cmake python unzip gperftools sqlite3 doxygen bash
   ```

	You should also install `lit` to enable testing, `tabulate` to
    support additional features in `klee-stats` and `wllvm` to
    make it easier to compile programs to LLVM bitcode:

	```bash
	$ pip3 install lit tabulate wllvm
	```
	(use `--user` to install it for the current user only)

2. **Install LLVM 9:** KLEE is built on top of [LLVM](http://llvm.org); the first steps are to get a working LLVM installation. See [Getting Started with the LLVM System](http://llvm.org/docs/GettingStarted.html) for more information.

   If you are using a recent Ubuntu (e.g. 18.04 LTS) or Debian, we recommend you to use the LLVM packages provided by LLVM itself. 

   ```bash
   $ sudo apt-get install clang-9 llvm-9 llvm-9-dev llvm-9-tools  
   ```

   If you are using macOS, you can install older LLVM packages using brew:
   ```bash
   $ brew install llvm@9  
   ```   

   That's it for LLVM. If you want to install it manually, please refer to the official [LLVM Getting Started documentation](http://releases.llvm.org/3.8.0/docs/GettingStarted.html).

3. **Install constraint solver(s)**

   KLEE supports multiple different constraint solvers. You must install at least one to build KLEE.

   * [STP](https://github.com/stp/stp) Historically KLEE was built around STP so support for this solver is the most stable. For build instructions, see [here]({{site.baseurl}}/build-stp).
   * [Z3](https://github.com/z3prover/z3) is a more recent addition to KLEE but is reasonably stable. You should use Z3 version ≥ 4.4. For build instructions, see [here](https://github.com/Z3Prover/z3/blob/master/README.md).
   * [metaSMT](https://github.com/agra-uni-bremen/metaSMT) supports
     various solvers, including Boolector, CVC4, STP, Z3 and Yices.  We recommend branch v4.rc1 (`git clone -b v4.rc1 ...`). For build instructions, see [here](https://github.com/agra-uni-bremen/metaSMT).

4. **(Optional) Get Google test sources:**

   For unit tests we use the Google test libraries. If you want to run the unit tests you need to perform this step and also pass `-DENABLE_UNIT_TESTS=ON` to CMake when configuring KLEE in step 8.

   We depend on a version `1.7.0` right now so grab the sources for it.

   ```bash
   $ curl -OL https://github.com/google/googletest/archive/release-1.7.0.zip
   $ unzip release-1.7.0.zip
   ```

   This will create a directory called `googletest-release-1.7.0`.

5. **(Optional) Build uClibc and the POSIX environment model (not supported on macOS):** By default, KLEE works on closed programs (programs that don't use any external code such as C library functions). However, if you want to use KLEE to run real programs you will want to enable the KLEE POSIX runtime, which is built on top of the [uClibc](http://uclibc.org) C library.

   ```bash
   $ git clone https://github.com/klee/klee-uclibc.git  
   $ cd klee-uclibc  
   $ ./configure --make-llvm-lib  
   $ make -j2  
   $ cd .. 
   ```
   When `clang` or `llvm-config` are not in your `PATH` or have a custom prefix/suffix, `configure` may fail to detect their location. You can use the `--with-cc` and `--with-llvm-config` flags to set the paths manually.

   **NOTE:** If you are on a different target (i.e., not i386 or x64), you will need to run `make config` and select the correct target. The defaults for the other uClibc configuration variables should be fine.  <br/><br/>  

   To tell KLEE to use both klee-uclibc and the POSIX runtime, pass
   `-DENABLE_POSIX_RUNTIME=ON`, `-DENABLE_KLEE_UCLIBC=ON` and `-DKLEE_UCLIBC_PATH=<KLEE_UCLIBC_SOURCE_DIR>`
   to CMake when configuring KLEE in step 8 where `<KLEE_UCLIBC_SOURCE_DIR>` is
   the absolute path to the cloned `klee-uclibc` git repository.<br/><br/>

6. **Get KLEE source:**

   ```bash
   $ git clone https://github.com/klee/klee.git
   ```

7. **(Optional) Build libc++:** To be able to run C++ code, you also need to enable support for the C++ standard library.

   Make sure that `clang++-9` is in your path. Then, run from the main KLEE source directory:

   ```bash
   $ LLVM_VERSION=9 SANITIZER_BUILD= BASE=<LIBCXX_DIR> REQUIRES_RTTI=1 DISABLE_ASSERTIONS=1 ENABLE_DEBUG=0 ENABLE_OPTIMIZED=1 ./scripts/build/build.sh libcxx
   ```
   where `<LIBCXX_DIR>` is the absolute path where libc++ should be cloned and built.

   To tell KLEE to use libc++, pass the following flags to CMake when you configure KLEE in step 8:

   ```bash
   -DENABLE_KLEE_LIBCXX=ON -DKLEE_LIBCXX_DIR=<LIBCXX_DIR>/libc++-install-9/ -DKLEE_LIBCXX_INCLUDE_DIR=<LIBCXX_DIR>/libc++-install-9/include/c++/v1/
   ```

   To additionally enable KLEE's exception handling support for C++, pass the following flags to CMake when you configure KLEE in step 8:

   ```bash
   -DENABLE_KLEE_EH_CXX=ON -DKLEE_LIBCXXABI_SRC_DIR=<LIBCXX_DIR>/llvm-9/libcxxabi/
   ```

   Note that `<LIBCXX_DIR>` must currently be an absolute path. Note that if you want to
   build libc++ in your user home path, that in some environments (such as Ubuntu 18.04), `~`
   may not be an absolute path, but you can use `$HOME` instead.

8. **Configure KLEE:**

   KLEE must be built "out of source", so first create a build directory. You can create this wherever you like.  Below, we assume you create this directory inside KLEE's repository.

   ```bash
   $ mkdir build
   ```

   Now `cd` into the build directory and run CMake to configure KLEE where `<KLEE_SRC_DIRECTORY>` is the path to the KLEE git repository you cloned in the previous step.
   
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
     -DLLVM_CONFIG_BINARY=<PATH_TO_llvm-config-9> \
     -DLLVMCC=<PATH_TO_clang-9> \
     -DLLVMCXX=<PATH_TO_clang++-9> \
     <KLEE_SRC_DIRECTORY>
   ```

   Where `<KLEE_UCLIBC_SOURCE_DIR>` is the absolute path to the klee-uclibc source tree,
   `<GTEST_SOURCE_DIR>` is the absolute path to the Google Test source tree.

   **NOTE 1:** You can simply type `cmake ..` to use the default options for KLEE (but note that these will not include support for uClibC and the POSIX runtime).

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
