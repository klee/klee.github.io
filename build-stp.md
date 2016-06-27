---
layout: default
title: Building KLEE
subtitle: Building STP
slug: build-stp
---

KLEE is based on the STP constraint solver. STP does not make frequent releases, and its GitHub repository is under constant development and may be unstable.
The instructions below are for the release 2.1.2. If you would like to use the upstream version, do not perform the checkout command `git checkout tags/2.1.2`.

_Please let us know if you have successfully and extensively used KLEE with a more recent version of STP._  

```bash
$ git clone https://github.com/stp/minisat.git
$ cd minisat
$ mkdir build
$ cd build
$ cmake -DSTATICCOMPILE=ON -DCMAKE_INSTALL_PREFIX=/usr/ ../
$ sudo make install
$ cd ../../
$ git clone https://github.com/stp/stp.git
$ cd stp
$ git checkout tags/2.1.2
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
