---
layout: default
title: Building KLEE
subtitle: Building STP
slug: build-stp
---

STP is the recommended solver in KLEE.  The instructions below are for release 2.3.3. If you would like to use the upstream version, do not perform the checkout command `git checkout tags/2.3.3` in the instructions below.

STP has a few external dependencies that are listed here as an install command for Ubuntu 18.04:  

```bash
sudo apt-get install cmake bison flex libboost-all-dev python perl zlib1g-dev minisat
```

The line above installs MiniSAT. If you need to install MiniSAT manually, run:  

```bash
$ git clone https://github.com/stp/minisat.git
$ cd minisat
$ mkdir build
$ cd build
$ cmake -DCMAKE_INSTALL_PREFIX=/usr/local/ ../
$ sudo make install
```

To install STP, run:  

```bash
$ git clone https://github.com/stp/stp.git
$ cd stp
$ git checkout tags/2.3.3
$ mkdir build
$ cd build
$ cmake ..
$ make
$ sudo make install
```
Before running KLEE with STP on larger benchmarks, it is essential to run the following command:  

```bash
$ ulimit -s unlimited
```

You can make this persistent by editing the `/etc/security/limits.conf` file.<br/><br/>  
