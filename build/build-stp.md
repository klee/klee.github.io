---
layout: default
title: Building KLEE
subtitle: Building STP
slug: getting-started
---

STP is the recommended solver in KLEE. The build instructions below are for release 2.3.3. If you would like to use the upstream version, do not perform the checkout command `git checkout tags/2.3.3` in the instructions below.

Before you attempt to build STP, you should check first if it is not already [supported by your package manager](https://repology.org/project/stp/versions).

STP has a few external dependencies that are first listed here as an install command for Ubuntu 18.04:  

```bash
sudo apt-get install cmake bison flex libboost-all-dev python perl zlib1g-dev minisat
```

Under macOS, you can run:

```bash
brew install cmake bison flex python perl minisat
```

If your distribution does not offer MiniSAT, you need to install it manually:

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
Before running KLEE with STP on larger benchmarks, it is essential to
set the size of the stack to a very large value:

```bash
$ ulimit -s unlimited
```

On macOS, you'll likely have to run:
```bash
$ ulimit -s 65532
```


You can make this persistent by editing the
`/etc/security/limits.conf` file or adding the `ulimit` command into
e.g, `.bashrc`.

<br/><br/>  
