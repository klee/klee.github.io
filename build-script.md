---
layout: default
title: Building KLEE and its Dependencies
subtitle: The build infrastructure for KLEE
slug: build-klee-script
---

# Building KLEE and its Dependencies

KLEE is a symbolic execution framework that can be built with a multitude of different components (compiler infrastructure: LLVM 3.8 - ..., solvers: STP, Z3, MetaSMT) and run on a variety of systems (e.g. Linux (Ubuntu), Mac OS X, FreeBSD).

Managing and building the different combinations of dependencies can be tedious. Moreover, not all the different requested versions of each dependency are available for every system. To simplify this process, we provide a build script `scripts/build/build.sh` that allows you to manage those tasks automatically.


## Users

To get started, check out the KLEE source code:
```
git clone https://github.com/klee/klee.git
cd klee
```

Start the build script with `./scripts/build/build.sh` to get an overview of the available options.

The output looks like:
```
build.sh [--docker] [--keep-dockerfile] [--install-system-deps] component*
Available components:
clang
gtest
klee
libcxx
llvm
metasmt
sanitizer
sanitizer_compiler
solvers
sqlite
stp
tcmalloc
uclibc
z3
```

### Invoking the script - Installing a solver

The script builds components (e.g. `solver`), and their dependencies. KLEE is handled as one component as well (i.e. `klee`).

Every component might require configuration options. If the script is invoked but not all the necessary options are provided, the user is informed.

`./scripts/build/build.sh solvers` generates:

```
SOLVERS Required but unset
```

Prepending the script call with the option fixes that: `SOLVERS="stp" ./scripts/build/build.sh solvers` will ask for additional variables.

The most crucial variable is `BASE`. This defines where components are installed if they are not part of system software.

A full invocation to install STP (version 2.3.3) with minisat as SAT solver (version git master) could be :
```
BASE="$HOME/klee_deps" MINISAT_VERSION="master" STP_VERSION="2.3.3" SOLVERS="stp" ./scripts/build/build.sh solvers
```

If the build was successful, all the sources and build artefacts can be found in the `$BASE` directory.
```
$ ls $BASE
minisat minisat-build minisat-install stp-2.3.3 stp-2.3.3-build stp-2.3.3-install
```

If builds fail, build dependencies might be missing.
To fix this - assuming your system is supported - the script can be suffixed with: ` --install-system-deps`.
In this case, required packages will be installed.
Similarly, if specific packages for the system are available, the script will try to install them instead of compiling them from scratch.

#### Getting inspired by the Travis builds
To test KLEE, we build different software combinations. The `.travis.yml` reflects those combinations.

Everything under `global` specifies the default configuration we use.
Every line under `matrix` overrides a specific option.
Use those options to mix and match the setup towards your requirements.

To locally build our standard configuration, use the following option:

```
COVERAGE=0 USE_TCMALLOC=1 BASE=$HOME/klee_deps LLVM_VERSION=6.0 ENABLE_OPTIMIZED=1 ENABLE_DEBUG=1 DISABLE_ASSERTIONS=0 REQUIRES_RTTI=0 SOLVERS=STP:Z3 GTEST_VERSION=1.7.0 UCLIBC_VERSION=klee_uclibc_v1.2 LLVM_VERSION=6.0 TCMALLOC_VERSION=2.7 SANITIZER_BUILD= LLVM_VERSION=6.0 STP_VERSION=2.3.3 MINISAT_VERSION=master Z3_VERSION=4.8.4 USE_LIBCXX=1 KLEE_RUNTIME_BUILD="Debug+Asserts" ./scripts/build/build.sh klee
```

For example, to have address sanitized builds with LLVM 7.0 use:
```
SANITIZER_BUILD=address DISABLE_ASSERTIONS=0 ENABLE_OPTIMIZED=0 USE_TCMALLOC=0 SOLVERS=STP USE_LIBCXX=0 LLVM_VERSION=7.0 STP_VERSION="2.3.3" ENABLE_DEBUG=1 UCLIBC_VERSION=klee_uclibc_v1.2 REQUIRES_RTTI=0 ./scripts/build/build.sh klee
```

#### Mix and Match

Different versions and build setups can be built and used in parallel.
Every component gets a version and build-type suffix. This allows you to re-use and combine different setups and use KLEE with varying versions of any component.

### Docker

Besides, the script allows building docker images of KLEE.
For that, append `--docker` to the script invocation. The only variable required is `BASE_IMAGE`, which refers to the docker image used as a base system, e.g. `ubuntu:xenial-20181005`.

Internally, the different independent components will be built as separate docker images and combined for the final image.

For your docker repository, provide the `REPOSITORY` as an argument.
Use the default `REPOSITORY=klee` to utilise KLEE's upstream images.

## Developers


- KLEE
  - build
    - LLVM
    - optional depends on uclibc
    - optional depends on stp
    - optional tcmalloc
  - runtime
    - clang

- experiment
  - build
    - clang

- exp_klee
  - build
    - exp
    - klee
  - runtime
    - exp
    - klee

- uclibc
  - build
    - clang

- clang
  - runtime
    llvm

- llvm
  - build
    - sanitizer
config -> potential dependencies
