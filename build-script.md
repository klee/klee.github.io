---
layout: default
title: Building KLEE and its Dependencies
subtitle: The build infrastructure for KLEE
slug: getting-started
---

# Building KLEE and its dependencies

KLEE is a symbolic execution framework that can be built with a multitude of different components (compiler infrastructure: multiple LLVM versions, solvers: STP, Z3, MetaSMT) and run on a variety of systems (e.g., Linux (Ubuntu), macOS, FreeBSD).

Managing and building the different combinations of dependencies can be tedious. Moreover, not all the different requested versions of each dependency are available for every system. To simplify this process, we provide a build script `scripts/build/build.sh` that allows you to manage those tasks automatically.

## TL; DR

To locally build our standard configuration, use the following option:
* LLVM: 9.0 - optimized with debug information
* Solvers: STP and Z3
* uclibc: to test applications with systems interaction
* libc++: to test C++ applications

```
COVERAGE=0 USE_TCMALLOC=1 BASE=$HOME/klee_deps LLVM_VERSION=9.0 ENABLE_OPTIMIZED=1 ENABLE_DEBUG=1 DISABLE_ASSERTIONS=0 REQUIRES_RTTI=0 SOLVERS=STP:Z3 GTEST_VERSION=1.7.0 UCLIBC_VERSION=klee_uclibc_v1.2 TCMALLOC_VERSION=2.7 SANITIZER_BUILD= STP_VERSION=2.3.3 MINISAT_VERSION=master Z3_VERSION=4.8.4 USE_LIBCXX=1 KLEE_RUNTIME_BUILD="Debug+Asserts" ./scripts/build/build.sh klee --install-system-deps
```

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

### Invoking the script - installing a solver

The script builds components (e.g., `solver`) and their dependencies. KLEE is handled as one component as well (i.e., `klee`).

Every component might require configuration options. If the script is invoked, but not all the necessary options are provided, the user is informed.

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

If the build was successful, all the sources and build artifacts can be found in the `$BASE` directory.
```
$ ls $BASE
minisat minisat-build minisat-install stp-2.3.3 stp-2.3.3-build stp-2.3.3-install
```

If builds fail, build dependencies might be missing.
To fix this - assuming your system is supported - the script can be suffixed with: ` --install-system-deps`.
In this case, the required packages will be installed.
Similarly, if specific packages for the system are available, the script will try to install them instead of compiling them from scratch.

#### Getting inspired by the Travis Builds
To test KLEE, we build different software combinations. The `.travis.yml` reflects those combinations.

Everything under `global` specifies the default configuration we use.
Every line under `matrix` overrides a specific option.
Use those options to mix and match your required setup.

To locally build our standard configuration, use the following option:

```
COVERAGE=0 USE_TCMALLOC=1 BASE=$HOME/klee_deps LLVM_VERSION=9.0 ENABLE_OPTIMIZED=1 ENABLE_DEBUG=1 DISABLE_ASSERTIONS=0 REQUIRES_RTTI=0 SOLVERS=STP:Z3 GTEST_VERSION=1.7.0 UCLIBC_VERSION=klee_uclibc_v1.2 TCMALLOC_VERSION=2.7 SANITIZER_BUILD= STP_VERSION=2.3.3 MINISAT_VERSION=master Z3_VERSION=4.8.4 USE_LIBCXX=1 KLEE_RUNTIME_BUILD="Debug+Asserts" ./scripts/build/build.sh klee
```

For example, to have address sanitized builds with LLVM 7.0 use:
```
SANITIZER_BUILD=address DISABLE_ASSERTIONS=0 ENABLE_OPTIMIZED=0 USE_TCMALLOC=0 SOLVERS=STP USE_LIBCXX=0 LLVM_VERSION=7.0 STP_VERSION="2.3.3" ENABLE_DEBUG=1 UCLIBC_VERSION=klee_uclibc_v1.2 REQUIRES_RTTI=0 ./scripts/build/build.sh klee
```

#### Mix and Match

Different versions and build setups can be built and used in parallel.
Every component gets a version and a build-type suffix. This allows you to re-use and combine different setups and use KLEE with varying versions of any component.

### Docker

Besides, the script allows building docker images of KLEE.
For that, append `--docker` to the script invocation. The only variable required is `BASE_IMAGE`, which refers to the docker image used as a base system, e.g., `ubuntu:xenial-20181005`.

Internally, the different independent components will be built as separate docker images and combined for the final image.

For your docker repository, provide the `REPOSITORY` as an argument.
Use the default `REPOSITORY=klee` to utilize KLEE's upstream images.

## Developers

The build scripts provide a framework that can be extended easily.

The basic idea is to describe different components that are required to build a specific artifact.

All the required elements are in the `scripts/build/` directory or beneath.

The main build script `build.sh` automatically uses components that are available in the same directory. Every component is described with a set of `*.inc` files.

### How to add a new component?

A component consists of two parts: a required virtual description (`v-COMPONENT.inc`) and optional build description that are platform-specific (`p-COMPONENT-PLATFORM.inc`).

The build script automatically detects under which platform it runs (e.g., Linux, macOS, ...), which platform-specific versions (e.g., Debian, Ubuntu, ...), and which version number (e.g., Ubuntu 16.04, Ubuntu 18.04, ...).

This platform information is used to resort to required build instructions from the most platform-specific version to the least.

As example, to build the __STP__ solver, `build.sh` finds the general component information in `v-stp.inc`. To build STP under a Linux Ubuntu 20.04, the build script will try to acquire information from `p-stp-linux-ubuntu-20.04.inc`, which it won't find. In subsequent steps, it will try to resolve functionality from `p-stp-linux-ubuntu.inc`, then `p-stp-linux.inc` and last `p-stp.inc`.

Due to the nature of bash, all component-specific functions are suffixed with `_COMPONENT` to avoid name clashes and re-definitions.

#### The virtual description of `COMPONENT`

We will use __STP__ as an example of how to add a new package.

The virtual description file contains specific instructions that are necessary to describe the package.

##### Required Variables
For example, the version number of a component is important.
To require such information, a function or array `required_variables_COMPONENT=()` is added to the `v-COMPONENT.inc` file. The `build.sh` script will check if such an argument is provided before starting the build process.

For STP, the `STP_VERSION` and `MINISAT_VERSION` is required, e.g.

```
required_variables_stp=(
  "STP_VERSION"
  "MINISAT_VERSION"
  )
```

If no additional information is required, an empty array can be used.
Bash-specifics require it to have one entry:
```
required_variables_stp=("")
```

To validate if required variables have a specific schema, e.g., a boolean value is provided, the `required_variables_check_COMPONENT() {}` function can be provided.

For example, for the meta-package __SOLVERS__, which references potential solvers, the package checks if the user-provided variable contains a valid solver, i.e., _z3_, _stp_, or _metasmt_.


##### Artifact dependencies

To specify the dependencies of components, `artifact_dependency_COMPONENT` should be used. In the case of __SOLVERS__, the selected solvers become dependents.

#### Providing build instructions

Build instructions define how a component is provided.

It uses the following steps that can be customized as described later:

1. Setup general variables to support each of the following steps (`setup_artifact_variables_COMPONENT()`)
2. Check if the artifact is already available in the build environment, i.e., it was built already or is pre-installed (`is_installed_COMPONENT()`).
3. Install a package that contains the artifact, e.g., a pre-compiled binary version (`install_binary_artifact_COMPONENT()`)
4. Setup build variables (`setup_build_variables_COMPONENT()`)
5. Install build dependencies (`install_build_dependencies_COMPONENT()`)
6. Download source code for component (`download_COMPONENT`)
7. Build component (`build_COMPONENT`)
8. Install component (`install_COMPONENT`)

In a nutshell, if the component is not available, try to get a pre-built version or built it.

##### Building from source

Let's start with the case that the package has to be built from scratch. For this, the last four steps are needed, and the associated functions should be added to the `p-COMPONENT.inc` file if needed.

For __STP__, `setup_build_variables_stp()` in `p-stp.inc` will provide different build variables required for the next steps.

To download (`download_stp`), build (`build_stp`), and (`install_stp`) provide each function. To speed up the build process, many functions should be guarded, e.g.

```
download_COMPONENT() {
  // Check if downloaded already
  [[ - f "{BASE}/COMPONENT/.downloaded"]] && return 0

  // Do an expensive download

  // Mark as downloaded
  touch "{BASE}/COMPONENT/.downloaded"
}
```

This helps to save bandwidth and compile time.

Building software will often require build dependencies installed.
But this depends on the specific system where the software is built.

To install these dependencies, write an `install_build_dependencies_COMPONENT()` function in the system-specific file.

For example, if the compiler is `clang` and the build system is `cmake`, the following files could be provided like in the following examples:

* Specific Ubuntu version: `p-stp-linux-ubuntu-18.04.inc`:
```
install_build_dependencies_stp() {
  apt -y --no-install-recommends install clang-8 cmake
}
```


* Any Ubuntu: `p-stp-linux-ubuntu.inc`:
```
install_build_dependencies_stp() {
  apt -y --no-install-recommends install clang cmake
}
```

* Arch: `p-stp-linux-arch.inc`
```
install_build_dependencies_stp() {
  pacman -S clang cmake
}
```

* macOS: `p-stp-osx.inc`
```
install_build_dependencies_stp() {
  brew install clang@8 cmake
}
```

The install script tries to find a satisfying function depending on the system and the provided instructions by evaluating the most specific to the least specific version. With the previously defined functions, on Ubuntu 20.04, the script will use the more general `p-stp-linux-ubuntu.inc` as a `p-stp-linux-ubuntu-20.04.inc` is not available.

To foster reusability, source-built components should not be installed into systems directories, instead, they should be kept in separate directories.
For example, for __STP__, it is build in something like `STP_BUILD_PATH="${BASE}/stp-${STP_VERSION}-build"` and installed into `STP_INSTALL_PATH="${BASE}/stp-${STP_VERSION}-install`. This allows us to have multiple software versions and differently optimized versions simultaneously on a system and link them appropriately.

##### Using pre-built packages

If a component can be provided as a pre-built package, the function `install_binary_artifact_COMPONENT()` can be specified similarly to the previous section. It should be defined in the most-specific systems file required.

##### Checking if a component is installed

To handle pre-built packages and source-based installations, the `is_installed_COMPONENT` should be provided. Again, they can be specified in multiple files from more-specific systems files to least specific build files.

For example, for Ubuntu:

* Specific Ubuntu version: `p-stp-linux-ubuntu-16.04.inc`:
```
is_installed_stp() {
  # Check if library exists
  [[ -f "/usr/lib/libstp.so" ]] && return 0

  # Not found
  return 1
}
```

* General : `p-stp.inc`:
```
is_installed_stp() {
  # Check if library exists
  [[ -f "{STP_INSTALL_PATH}/lib/libstp.so" ]] && return 0

  # Not found
  return 1
}
```

### Dockerizing components

To save time re-compiling code, components can be dockerized:
If `build.sh` is executed with `--docker`, the script will use dockerized components if possible; otherwise, it will install or compile components inside the docker environment.

To enable this functionality, two functions have to be provided: `get_docker_config_id_COMPONENT()` and `get_build_artifacts_COMPONENT()`.

The `get_docker_config_id_COMPONENT` provides the name of the artifact that the build script should look for, e.g., a version string or build configuration.
The build script tries to access the docker image: `${REPOSITORY}/${COMPONENT}:${ID}`, if found it will be added as a dependency of the current docker environment.

For example, if KLEE should be built and depends on STP version 2.3.3, `klee/stp:2.3.3_ubuntu` is added as a dependent image if available. The built artifact is transferred from the dependent image into the target image.

The `get_build_artifacts_COMPONENT()` function is specified to describe which build results should be kept. As an example, in the case of __STP__, neither the source nor the build directory is of interest; only the files installed are important. The following code provides the installation directories of _MiniSAT_ and _STP_.

```
get_build_artifacts_stp() {
  (
    echo "${MINISAT_INSTALL_PATH}"
    echo "${STP_INSTALL_PATH}"
  )
}
```

The provided artifact directories are copied into a new `BASE_IMAGE` docker image to keep the overall size small.


## Building Docker images for Travis CI

`scripts/build/build-travis-container.py` uses the `.travis.yml` file in the root directory to build all the images required for a Travis CI run. This takes a substantial amount of time and space (memory and disk space).
