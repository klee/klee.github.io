---
layout: default
title: Getting Started
subtitle: Installing KLEE
slug: getting-started
---

There are multiple ways to get started with KLEE.

* [KLEE web](http://klee.doc.ic.ac.uk/): run tiny code examples in your browser
* [Our Docker images]({{site.baseurl}}/docker): this is one of the fastest ways to get started.
* [Use your package manager](https://repology.org/project/klee/versions) for Arch Linux, openSUSE Tumbleweed, and several other distributions
* [Running with Nix]({{site.baseurl}}/nix): this is even faster than Docker if you have Nix.
* [Build from source against LLVM 11]({{site.baseurl}}/build-llvm11): this is the currently recommended version
* [Fedora package](https://src.fedoraproject.org/rpms/klee): install the latest release using `dnf install klee`
* [FreeBSD package](https://www.freshports.org/security/klee): FreeBSD users can install latest release with `pkg install klee` or by building `security/klee` port themselves.
* [Homebrew package]({{site.baseurl}}/install-brew): install the latest release using [Homebrew](https://brew.sh)
* [Building arbitrary KLEE configurations]({{site.baseurl}}/build-script): to build different configurations of KLEE and its dependencies
