---
layout: default
title: Using KLEE with Docker
slug: documentation
---
# What is Docker?

[Docker](https://www.docker.com/) provides tools for deploying applications within containers. Containers are (mostly) isolated from each other and the underlying system. This allows you to make a KLEE container , tinker with it and then throw it away when you're done without affecting the underlying sytem or other containers.

A Docker container is built from a Docker image. A Docker image encapsulates an application which in this case is KLEE. This application level encapsulation is useful because it provides a "portable" and reproducible environment to run KLEE in.

**Note that to run Docker natively on your machine you need to be using a Linux distribution with Docker installed. Users of Windows and OSX can use [Boot2Docker](http://boot2docker.io/) to run Docker inside a lightweight Linux based virtual machine.**

# Getting the KLEE Docker image

There are two ways of obtaining the [KLEE Docker image](https://hub.docker.com/r/klee/klee/).

## Pulling from the Docker Hub

Our GitHub repository is linked to the DockerHub so that changes to particular branches trigger an automatic rebuild of the corresponding Docker image.

To pull down the latest build of a particular Docker image run:

```bash
$ docker pull klee/klee
```

If you want to use a tagged revision of KLEE you should instead run:

```bash
$ docker pull klee/klee:<TAG>
```

Where ``<TAG>`` is [one of the tags listed on the DockerHub](https://hub.docker.com/r/klee/klee/tags/). Typically this is either ``latest`` (corresponds to the ``master`` branch) or a version number (e.g. ``1.0.0``).

**Note this process pulls images containing code compiled by a third-party service. We do not accept resonsibility for the contents of the image**

## Building the Docker image locally

If you want to build the Docker image manually instead of pulling the image from DockerHub you can do so by running

```bash
$ git clone https://github.com/klee/klee.git
$ cd klee
$ docker build -t klee/klee
```

This will use the contents of the ``Dockerfile`` in the root of the repository to build the Docker image.

# Creating a KLEE Docker container

Now that you have a KLEE Docker image you can try creating a container from the image.

The simplest thing to try is creating a temporary container and gain shell access. To do this run

```bash
$ docker run --rm -ti klee/klee
```

**Note if you wanted to use a tagged KLEE image replace** ``klee/klee`` **with** ``klee/klee:<TAG>`` **where** ``<TAG>`` **is the tag you wish to use**

If this worked correctly your shell prompt will have changed and you will be the ``klee`` user.

```bash
klee@3c098b05ca85:~$ whoami
klee
klee@3c098b05ca85:~$
```

You can now try running KLEE inside the container

```bash
klee@3c098b05ca85:~$ klee --version
KLEE 1.0.0 (https://klee.github.io)
  Built Sep 21 2015 (17:03:14)
  Build mode: Release+Asserts
  Build revision: unknown

LLVM (http://llvm.org/):
  LLVM version 3.4
  
  Optimized build.
  Built Mar  5 2014 (17:05:10).
  Default target: x86_64-pc-linux-gnu
  Host CPU: core-avx2
```

and Clang

```bash
$ clang --version
Ubuntu clang version 3.4-1ubuntu3 (tags/RELEASE_34/final) (based on LLVM 3.4)
Target: x86_64-pc-linux-gnu
Thread model: posix
```

Now exit the container

```bash
klee@3c098b05ca85:~$ exit
```

Because the ``--rm`` flag was used with the ``docker run`` command the container was destroyed (not visible in ``docker ps -a``) when the application running in the container (``/bin/bash``) terminated.


# Persistent Containers

The earlier example didn't do anything interesting with KLEE and threw the created container away. Here is a simple
example that does something slightly more interesting with KLEE and also shows how a container can be persisted.

To create and enter the container run:

```bash
$ docker run --ti --name=my_first_klee_container klee/klee
```

Notice we didn't use ``--rm`` so the container will not be destroyed when we exit it from it and we also gave the container a name using the ``-t`` flag.

Now inside the container you can try running the following:

```bash
klee@3c098b05ca85:~$ pwd
/home/klee
klee@3c098b05ca85:~$ echo "int main(int argn, char** argv) { return 0; }" > test.c
klee@3c098b05ca85:~$ clang -emit-llvm -g -c test.c -o test.bc
klee@3c098b05ca85:~$ klee --libc=uclibc --posix-runtime test.bc
KLEE: NOTE: Using klee-uclibc : /home/klee/klee_build/klee/Release+Asserts/lib/klee-uclibc.bca
KLEE: NOTE: Using model: /home/klee/klee_build/klee/Release+Asserts/lib/libkleeRuntimePOSIX.bca
KLEE: output directory is "/home/klee/klee-out-0"
KLEE: WARNING: undefined reference to function: klee_posix_prefer_cex
KLEE: WARNING ONCE: calling external: syscall(16, 0, 21505, 44070352)
KLEE: WARNING ONCE: calling __user_main with extra arguments.

KLEE: done: total instructions = 5047
KLEE: done: completed paths = 1
KLEE: done: generated tests = 1
klee@3c098b05ca85:~$ ls
klee-last  klee-out-0  klee_build  klee_src  test.bc  test.c
```

Now exit the container

```bash
klee@3c098b05ca85:~$ exit
```

We can check that the container still exists by running

```bash
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS                     PORTS               NAMES
1c408467bdf7        klee/klee           "/bin/bash"         About a minute ago   Exited (0) 2 seconds ago                       my_first_klee_container
```

You can restart the container by running

```bash
$ docker start -ai my_first_klee_container
klee@1c408467bdf7:~$ ls
klee-last  klee-out-0  klee_build  klee_src  test.bc  test.c
klee@1c408467bdf7:~$ exit
```

As you can see the files we created earlier are still present.

When you are finished with the container you can run

```bash
$ docker rm my_first_klee_container
```

to remove it.

# Working with KLEE containers built from the KLEE Docker image

There are a few useful things to know about KLEE Docker containers created using the KLEE Docker image.

* The Docker image is based on an Ubuntu 12.04 LTS image.
* Inside the Docker image the ``klee`` user has sudo access (password is ``klee``) so that you can install other applications inside the container (e.g. a text editor). Given that the default user has sudo access this image **should never be used in a production environment**.
* You may want files on your native filesystem available in the container. By default the host filesystem is not visible inside the container.  You can use the ``--volumes-from`` option to ``docker run`` to mount directories on the host filesystem into the container.
* These Docker images use LLVM 3.4 so you need to use ``clang`` (not ``llvm-gcc``) to create LLVM bitcode. So when trying out the tutorials
  use ``clang`` and not ``llvm-gcc``.
* ``gcc`` is not installed in the Docker image. If you need to build native code inside the container just use ``clang``.
* ``/home/klee/klee_src`` contains the source code used to build KLEE.
* ``/home/klee/klee_build`` contains the build of KLEE built from ``/home/klee/klee_src``
* All the previous examples implicitly run ``/bin/bash`` inside the container. This is the default but it is also possible to run KLEE directly (useful for scripting) by specifying the command line to use to ``docker run``.

This should give you everything you need to start playing with KLEE using Docker. If you are unfamilar with Docker and wish to learn more a good place to start is [Docker's documentation](https://docs.docker.com/).
