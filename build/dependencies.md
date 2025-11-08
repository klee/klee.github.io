---
layout: default
title: Building KLEE
subtitle: Dependencies
slug: getting-started
---

KLEE requires all the dependencies of LLVM (see [here](http://llvm.org/docs/GettingStarted.html#requirements)), and some more.

In particular, you should install the programs and libraries listed below. `graphviz/doxygen` are optional and only needed to generate the source code documentation.

   Under Ubuntu, use:
   ```bash
   $ sudo apt-get install build-essential cmake curl file g++-multilib gcc-multilib git libcap-dev libgoogle-perftools-dev libncurses-dev libsqlite3-dev libtcmalloc-minimal4 python3-pip unzip graphviz doxygen
   ```

   Under macOS, run:
   ```bash
   $ brew install curl git cmake python unzip gperftools sqlite3 graphviz doxygen bash
   ```

   You should also install `lit` to enable testing, `tabulate` to support additional features in `klee-stats` and `wllvm` to make it easier to compile programs to LLVM bitcode:

   ```bash
   $ sudo apt-get install pipx
   $ pipx install lit wllvm
   ```

   In older Ubuntu distributions, you might need instead to use:

   ```bash
   $ sudo pip3 install lit wllvm
   $ sudo apt-get install python3-tabulate
   ```

   Make sure that e.g. `~/.local/bin` (check with `python3 -m site --user-base` on your system) is in your `PATH`.
