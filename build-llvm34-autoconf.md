---
layout: default
title: Building KLEE
subtitle: with LLVM 3.4 and Autoconf build system
slug: build-llvm34-autoconf
---

{% include version_warning.md %}

The current procedure for building KLEE with LLVM 3.4 (recommended) using
KLEE's older Autoconf/Makefile build system is outlined below.
If you want to build KLEE with LLVM 2.9, [click here]({{site.baseurl}}/build-llvm29).

1. **Install dependencies:** KLEE requires all the dependencies of LLVM, which are discussed [here](http://llvm.org/docs/GettingStarted.html#requirements). In particular, you should install the following programs and libraries, listed below as Ubuntu packages:  

   ```bash
   $ sudo apt-get install build-essential curl libcap-dev git cmake libncurses5-dev python-minimal python-pip unzip
   ```

   You will need gcc/g++ 4.8 or later installed on your system. For Ubuntu 12.04 and 13.04, you can follow the instructions [here](http://ubuntuhandbook.org/index.php/2013/08/install-gcc-4-8-via-ppa-in-ubuntu-12-04-13-04/).   

   **(Optional) Build KLEE with TCMalloc support:** By default, KLEE uses malloc_info() to observe and to restrict its memory usage. Due to limitations of malloc_info(), the maximum limit is set to 2 GB. To support bigger limits, KLEE can use TCMalloc as an alternative allocator. It is thus necessary to install TCMalloc:

   ```bash
   $ sudo apt-get install libtcmalloc-minimal4 libgoogle-perftools-dev
   ```

2. **Install LLVM 3.4:** KLEE is built on top of [LLVM](http://llvm.org); the first steps are to get a working LLVM installation. See [Getting Started with the LLVM System](http://llvm.org/docs/GettingStarted.html) for more information.

   **NOTE:** KLEE is currently tested on **Linux x86-64**, and might break on x86-32.

   If you are using a recent Ubuntu (≥ 12.04, e.g. 14.04 LTS) or Debian, we recommend you to use the LLVM packages provided by LLVM itself. Use [LLVM Package Repository](http://llvm.org/apt/) to add the appropriate line to your `/etc/apt/sources.list`. As an example, for Ubuntu 14.04, the following lines should be added:  

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

   ```bash
   $ sudo ln -sf /usr/bin/llvm-config-3.4 /usr/bin/llvm-config
   ```

   That's it for LLVM. If you want to install it manually, please refer to the official [LLVM Getting Started documentation](http://www.llvm.org/docs/GettingStarted.html).<br/><br/>  

3. **Build STP:** KLEE is based on the STP constraint solver, you can find the instructions [here]({{site.baseurl}}/build-stp).

4. **(Optional) Build uclibc and the POSIX environment model:** By default, KLEE works on closed programs (programs that don't use any external code such as C library functions). However, if you want to use KLEE to run real programs you will want to enable the KLEE POSIX runtime, which is built on top of the [uClibc](http://uclibc.org) C library.  

   ```bash
   $ git clone https://github.com/klee/klee-uclibc.git  
   $ cd klee-uclibc  
   $ ./configure --make-llvm-lib  
   $ make -j2  
   $ cd .. 
   ```

   **NOTE:** If you are on a different target (i.e., not i386 or x64), you will need to run make config and select the correct target. The defaults for the other uClibc configuration variables should be fine.<br/><br/>  

5. **(Optional) Build libgtest:**

   Build Google test libraries for unit tests. We do a manual build, because the libgtest-dev package (version 1.6) installed through apt does not work for us.  

   ```bash
   $ curl -OL https://googletest.googlecode.com/files/gtest-1.7.0.zip  
   $ unzip gtest-1.7.0.zip  
   $ cd gtest-1.7.0  
   $ cmake .  
   $ make  
   $ cd ..
   ```

6. **Get KLEE source:**  

   ```bash
   $ git clone https://github.com/klee/klee.git
   ```

7. **Configure KLEE:** From the KLEE source directory, run:  

   ```bash
   $ ./configure --with-stp=/full/path/to/stp/build --with-uclibc=/full/path/to/klee-uclibc --enable-posix-runtime
   ```

   **NOTE:** If LLVM is not found or you have multiple LLVM versions installed, you can add `--with-llvmsrc=/usr/lib/llvm-3.4/build --with-llvmobj=/usr/lib/llvm-3.4/build --with-llvmcc=/usr/bin/clang-3.4 --with-llvmcxx=/usr/bin/clang++-3.4`.  
If you skipped step 4, simply remove the `--with-uclibc` and `--enable-posix-runtime` options.<br/><br/>  

8. **Build KLEE:**  

   ```bash
   $ make  
   ```
   <!-- make DISABLE_ASSERTIONS=0 ENABLE_OPTIMIZED=1 ENABLE_SHARED=0 -j2-->

   **NOTE:** You can add `/full/path/to/klee/build/Release/bin` to your path.<br/><br/>


9. **Run the main regression test suite to verify your build:**
   
   ```bash
   $ make test
   ```
   
   If you want to invoke `lit` manually use:
   ```bash
   $ /usr/lib/llvm-3.4/build/utils/lit/lit.py test/
   ```
   
   This way you can run individual tests or subsets of the suite:
   ```bash
   $ /usr/lib/llvm-3.4/build/utils/lit/lit.py test/regression
   ```
   
10. **(Optional) Run the unit tests:**

    If you did not install the LLVM upstream or Debian packages,
    install the LLVM unit tests makefile:
   
    ```bash
    $ sudo mkdir -p /usr/lib/llvm-3.4/build/unittests/  
    $ sudo curl -L http://llvm.org/svn/llvm-project/llvm/branches/release_34/unittests/Makefile.unittest -o /usr/lib/llvm-3.4/build/unittests/Makefile.unittest  
    ```

    Run KLEE unit tests:

    ```bash
    $ make CPPFLAGS=-I/full/path/to/gtest-1.7.0/include LDFLAGS=-L/full/path/to/gtest-1.7.0 unittests
    ```
11. **You're ready to go! Check the [Tutorials]({{site.baseurl}}/tutorials) page to try KLEE.**

<!--    **NOTE:** The flags (DISABLE_ASSERTIONS, ENABLE_OPTIMIZED, ENABLE_SHARED) have to be the same as the ones used for building KLEE. -->

**NOTE:** For testing real applications (e.g. Coreutils), you may need to increase your system's open file limit (ulimit -n). Something between 10000 and 999999 should work. In most cases, the hard limit will have to be increased first, so it is best to directly edit the `/etc/security/limits.conf` file.<br/><br/>
