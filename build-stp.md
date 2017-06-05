---
layout: default
title: Building KLEE
subtitle: Building STP
slug: build-stp
---

KLEE is based on the STP constraint solver. STP does not make frequent releases, and its GitHub repository is under constant development and may be unstable.
The instructions below are for the release 2.1.2. If you would like to use the upstream version, do not perform the checkout command `git checkout tags/2.1.2`.

_Please let us know if you have successfully and extensively used KLEE with a more recent version of STP._

STP has a few external dependencies they are listed below as an install command for Ubuntu 14.04LTS.

```bash
sudo apt-get install cmake bison flex libboost-all-dev python perl zlib1g-dev
```

**NOTE:** If you are using an older Linux release (e.g. Ubuntu ≤12.04), then you will have to manually install cmake 2.8.8 or newer (you can follow the instructions [here](http://cameo54321.blogspot.com/2014/02/installing-cmake-288-or-higher-on.html)).    


```bash
$ git clone https://github.com/stp/minisat.git
$ cd minisat
$ mkdir build
$ cd build
$ cmake -DSTATIC_BINARIES=ON -DCMAKE_INSTALL_PREFIX=/usr/local/ ../
$ sudo make install
$ cd ../../
$ git clone https://github.com/stp/stp.git
$ cd stp
$ git checkout tags/2.1.2
$ mkdir build
$ cd build
```

<!-- TODO: Once we switch to CMake drop building the static library. Using the shared library works fine when KLEE is built with CMake -->
Shared STP libraries cause problems for KLEE, so we have to disable them ([see this mailing list thread](https://www.mail-archive.com/klee-dev@imperial.ac.uk/msg01704.html)). The python interface requires shared libraries, so we have to disable that, too.

```bash
$ cmake -DBUILD_SHARED_LIBS:BOOL=OFF -DENABLE_PYTHON_INTERFACE:BOOL=OFF ..
$ make
$ sudo make install
$ cd ..
```
As documented on the STP website, it is essential to run the following command before using STP (and thus KLEE):  

```bash
$ ulimit -s unlimited
```

You can make this persistent by editing the `/etc/security/limits.conf` file.<br/><br/>  
