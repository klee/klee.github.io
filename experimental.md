---
layout: default
title: Getting Started (Experimental)
subtitle: Building and Running KLEE with LLVM 3.4
slug: experimental
---

The current procedure for building KLEE with LLVM 3.4 (experimental) is outlined below.
If you want to build KLEE with LLVM 2.9 (stable), [click here]({{site.baseurl}}/getting-started).

1. **Install dependencies:** KLEE requires all the dependencies of LLVM, which are discussed [here](http://llvm.org/docs/GettingStarted.html#requirements). In particular, you should install the following programs and libraries, listed below as Ubuntu packages:  

   ```bash
   $ sudo apt-get install build-essential curl git bison flex bc libcap-dev git cmake libboost-all-dev libncurses5-dev python-minimal python-pip unzip
   ```

   You will need gcc/g++ 4.8 or later installed on your system. For Ubuntu 12.04 and 13.04, you can follow the instructions [here](http://ubuntuhandbook.org/index.php/2013/08/install-gcc-4-8-via-ppa-in-ubuntu-12-04-13-04/).   

   On some architectures, you might also need to set the following environment variables (best to put them in a config file like **.bashrc**):  

   ```bash
   $ export C_INCLUDE_PATH=/usr/include/x86_64-linux-gnu  
   $ export CPLUS_INCLUDE_PATH=/usr/include/x86_64-linux-gnu
   ```

2. **Install LLVM 3.4:** KLEE is built on top of [LLVM](http://llvm.org); the first steps are to get a working LLVM installation. See [Getting Started with the LLVM System](http://llvm.org/docs/GettingStarted.html) for more information.

   _**NOTE:** Currently, KLEE has only experimental support for **LLVM 3.4**. The only stable LLVM version for KLEE is **LLVM 2.9**. KLEE is currently tested on **Linux x86-64**, and might break on x86-32. KLEE will **not** compile with LLVM versions prior to 2.9._

   If you are using a recent Ubuntu (≥ 12.04, e.g. 14.04 LTS) or Debian, we recommend you to use the LLVM packages provided by LLVM itself. Use LLVM Package Repository to add the appropriate line to your `/etc/apt/sources.list`. As an example, for Ubuntu 14.04, the following lines should be added:  

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

3. **Build STP:** KLEE is based on the STP constraint solver. STP does not make frequent releases, and its GitHub repository is under constant development and may be unstable. The instructions below are for the upstream version. If you would like to use a version, which we have tested and used successfully, please refer to the [Getting Started guide]({{site.baseurl}}/getting-started). _Please let us know if you have successfully and extensively used KLEE with a more recent version of STP._  

   ```bash
   $ git clone https://github.com/stp/stp.git  
   $ mkdir stp/build  
   $ cd stp/build
   ```

   Shared STP libraries cause problems for KLEE, so we have to disable them ([see this mailing list thread](https://www.mail-archive.com/klee-dev@imperial.ac.uk/msg01704.html)). The python interface requires shared libraries, so we have to disable that, too.  

   ```bash
   $ cmake -DBUILD_SHARED_LIBS:BOOL=OFF -DENABLE_PYTHON_INTERFACE:BOOL=OFF ..  
   $ make  
   $ sudo make install  
   $ cd ..
   ```

   **NOTE:** If you are using an older Linux release (e.g. Ubuntu ≤12.04), then you will have to manually install cmake 2.8.8 or newer (you can follow the instructions [here](http://cameo54321.blogspot.com/2014/02/installing-cmake-288-or-higher-on.html)).    

   As documented on the STP website, it is essential to run the following command before using STP (and thus KLEE):  

   ```bash
   $ ulimit -s unlimited
   ```

   You can make this persistent by editing the `/etc/security/limits.conf` file.<br/><br/>  

4. (Optional) **Build uclibc and the POSIX environment model:** By default, KLEE works on closed programs (programs that don't use any external code such as C library functions). However, if you want to use KLEE to run real programs you will want to enable the KLEE POSIX runtime, which is built on top of the [uClibc](http://uclibc.org) C library.  

   ```bash
   $ git clone https://github.com/klee/klee-uclibc.git  
   $ cd klee-uclibc  
   $ ./configure --make-llvm-lib  
   $ make -j2  
   $ cd .. 
   ```

   **NOTE:** If you are on a different target (i.e., not i386 or x64), you will need to run make config and select the correct target. The defaults for the other uClibc configuration variables should be fine.<br/><br/>  

5. (Optional) **Build libgtest:**

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
   $ mkdir build  
   $ cd build  
   $ ../configure --with-stp=/full/path/to/stp/build --with-uclibc=/full/path/to/klee-uclibc --enable-posix-runtime
   ```

   **NOTE:** If LLVM is not found or you have multiple LLVM versions installed, you can add `--with-llvmsrc=/usr/lib/llvm-3.4/build --with-llvmobj=/usr/lib/llvm-3.4/build --with-llvmcc=/usr/bin/clang-3.4 --with-llvmcxx=/usr/bin/clang++-3.4`.  
If you skipped step 4, simply remove the `--with-uclibc` and `--enable-posix-runtime` options.<br/><br/>  

8. **Build KLEE:**  

   ```bash
   $ make DISABLE_ASSERTIONS=0 ENABLE_OPTIMIZED=1 ENABLE_SHARED=0 -j2
   ```

   **NOTE:** You can add `/full/path/to/klee/build/Release+Asserts/bin` to your path.

9. **Run the tests to verify your build:**

   1. Install lit (it is not installed with the newer LLVM versions):  

      ```bash
      $ sudo pip install lit
      ```

   2. Run KLEE regression tests:  
   
      ```bash
      $ cd /full/path/to/klee/build/test  
      $ make lit.site.cfg DISABLE_ASSERTIONS=0 ENABLE_OPTIMIZED=1 ENABLE_SHARED=0  
      $ lit -v .
      ```

   3. Get the LLVM unit tests makefile (not included in default installation):  
   
      ```bash
      $ sudo mkdir -p /usr/lib/llvm-3.4/build/unittests/  
      $ sudo curl -L http://llvm.org/svn/llvm-project/llvm/branches/release_34/unittests/Makefile.unittest -o /usr/lib/llvm-3.4/build/unittests/Makefile.unittest  
      ```

   4. Run KLEE unit tests:  

      ```bash
      $ make CPPFLAGS=-I/full/path/to/gtest-1.7.0/include LDFLAGS=-L/full/path/to/gtest-1.7.0 DISABLE_ASSERTIONS=0 ENABLE_OPTIMIZED=1 ENABLE_SHARED=0 unittests
      ```
    **NOTE:** The flags (DISABLE_ASSERTIONS, ENABLE_OPTIMIZED, ENABLE_SHARED) have to be the same as the ones used for building KLEE.  
    **NOTE:** For testing real applications (e.g. Coreutils), you may need to increase your system's open file limit (ulimit -n). Something between 10000 and 999999 should work. In most cases, the hard limit will have to be increased first, so it is best to directly edit the `/etc/security/limits.conf` file.<br/><br/>

10. You're ready to go! Check the [Tutorials]({{site.baseurl}}/tutorials) page to try KLEE.
