---
layout: default
title: Getting Started
subtitle: Getting Started with KLEE
slug: getting-started
---

## Web Interface

<s>Run tiny code examples in your browser with
[KLEE web](http://klee.doc.ic.ac.uk/).</s>

KLEE web is not maintained anymore.
If you would like to take over the maintainer role, please get in touch with [Cristian Cadar](https://www.doc.ic.ac.uk/~cristic/).

## Docker

[Our Docker images]({{site.baseurl}}/docker) are one of the fastest ways to get started.

## Installation via Package Manager

* [Get it from the Snap Store](https://snapcraft.io/klee). It works on Ubuntu, Fedora, Debian, and other major Linux distributions.
* [Use your package manager](https://repology.org/project/klee/versions) for Arch Linux, openSUSE Tumbleweed, and several other distributions
* [Running with Nix]({{site.baseurl}}/nix): this is even faster than Docker if you have Nix.
* [FreeBSD package](https://www.freshports.org/security/klee): FreeBSD users can install latest release with `pkg install klee` or by building `security/klee` port themselves.
* [Homebrew package]({{site.baseurl}}/install-brew): install the latest release using [Homebrew](https://brew.sh)

## Manual Installation

* [Build from source against LLVM 13]({{site.baseurl}}/build/build-llvm13): this is the currently recommended version
* [Building arbitrary KLEE configurations]({{site.baseurl}}/build/build-script): to build different configurations of KLEE and its dependencies
