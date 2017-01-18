---
layout: default
title: Building KLEE
subtitle: with LLVM 2.9
slug: build-llvm29
---

{% include version_warning.md %}

The current procedure for building KLEE with LLVM 2.9 is outlined below.
If you want to build KLEE with LLVM 3.4 (recommended), [click here]({{site.baseurl}}/build-llvm34).

1. **Install dependencies:** KLEE requires all the dependencies of LLVM, which are discussed [here](http://llvm.org/docs/GettingStarted.html#requirements). In particular, you should have the following packages (the list is likely not complete): g++, curl, dejagnu, subversion, bison, flex, bc, libcap-dev(el):

   ```bash
   $ sudo apt-get install g++ python curl cmake git bison flex bc libcap-dev # Ubuntu
   $ sudo dnf install g++ python curl cmake git bison flex bc libcap-devel # Fedora 21+
   ```

   On some architectures, you might also need to set the following environment variables (best to put them in a config file like **.bashrc**):

   ```bash
   $ export C_INCLUDE_PATH=/usr/include/x86_64-linux-gnu  
   $ export CPLUS_INCLUDE_PATH=/usr/include/x86_64-linux-gnu
   ```

   **(Optional) Build KLEE with TCMalloc support:** By default, KLEE uses malloc_info() to observe and to restrict its memory usage. Due to limitations of malloc_info(), the maximum limit is set to 2 GB. To support bigger limits, KLEE can use TCMalloc as an alternative allocator. It is thus necessary to install TCMalloc:

   ```bash
   $ sudo apt-get install libtcmalloc-minimal4 libgoogle-perftools-dev
   ```

2. **Build LLVM 2.9:** KLEE is built on top of [LLVM](http://llvm.org); the first steps are to get a working LLVM installation. See [Getting Started with the LLVM System](http://llvm.org/docs/GettingStarted.html) for more information.

   **NOTE:** KLEE is currently tested on **Linux x86-64**, and might break on x86-32. KLEE will **not** compile with LLVM versions prior to 2.9.

   1. Install llvm-gcc:
      * Download and install the LLVM 2.9 release of llvm-gcc from  [here](http://llvm.org/releases/download.html#2.9). On an x86-64 Linux platform you are going to need the archive  [LLVM-GCC 4.2 Front End Binaries for Linux x86-64](http://llvm.org/releases/2.9/llvm-gcc4.2-2.9-x86_64-linux.tar.bz2). 
      * Add llvm-gcc to your PATH. It is important to do this first so that llvm-gcc is found in subsequent configure steps. llvm-gcc will be used later to compile programs that KLEE can execute. _Forgetting to add llvm-gcc to your PATH at this point is by far the most common source of build errors reported by new users._

   2. Download and build [LLVM 2.9](http://llvm.org/releases/2.9/llvm-2.9.tgz):

      ```bash
      $ tar zxvf llvm-2.9.tgz  
      $ cd llvm-2.9  
      $ ./configure --enable-optimized --enable-assertions  
      $ make
      ```
      The `--enable-optimized` configure argument is not necessary, but KLEE runs very slowly in Debug mode.
      _You may run into compilation issues if you use new kernels/glibc versions. Please see [this mailing list post](http://www.mail-archive.com/klee-dev@imperial.ac.uk/msg01302.html) for details on how to fix this issue._

3. **Build STP:** KLEE is based on the STP constraint solver, you can find the instructions [here]({{site.baseurl}}/build-stp).

4. (Optional) **Build uclibc and the POSIX environment model:** By default, KLEE works on closed programs (programs that don't use any external code such as C library functions). However, if you want to use KLEE to run real programs you will want to enable the KLEE POSIX runtime, which is built on top of the [uClibc](http://uclibc.org) C library.

   ```bash
   $ git clone https://github.com/klee/klee-uclibc.git
   $ cd klee-uclibc
   $ ./configure --make-llvm-lib
   $ make -j2
   $ cd ..
   ```

   **NOTE:** If you are on a different target (i.e., not i386 or x64), you will need to run make config and select the correct target. The defaults for the other uClibc configuration variables should be fine.

5. **Download KLEE:**

   ```bash
   $ git clone https://github.com/klee/klee.git
   ```

6. **Configure KLEE:** From the KLEE source directory, run:

   ```bash
   $ ./configure --with-llvm=full-path-to-llvm --with-stp=full-path-to-stp/install --with-uclibc=full-path-to-klee-uclibc --enable-posix-runtime
   ```

   **NOTE:** If you skipped step 4, simply remove the `--with-uclibc` and `--enable-posix-runtime` options.

7. **Build KLEE:**

   ```bash
   $ make ENABLE_OPTIMIZED=1
   ```

8. **Run the regression suite to verify your build:**

   ```bash
   $ make systemtests
   $ make unittests
   ```

   **NOTE:** For testing real applications (e.g. Coreutils), you may need to increase your system's open file limit (ulimit -n). Something between 10000 and 999999 should work. In most cases, the hard limit will have to be increased first, so it is best to directly edit the `/etc/security/limits.conf` file.<br/><br/>

9. **You're ready to go! Check the [Tutorials]({{site.baseurl}}/tutorials) page to try KLEE.**
