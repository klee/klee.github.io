---
layout: default
title: Getting Started
subtitle: Installing KLEE
slug: getting-started
---

There are multiple ways to get started with KLEE.

* [KLEE web](http://klee.doc.ic.ac.uk/): run tiny code examples in your browser
* [Our Docker images]({{site.baseurl}}/docker): this is one of the fastest ways to get started.
* [Running with Nix]({{site.baseurl}}/nix): this is even faster than Docker if you have Nix.
* [Build from source against LLVM 9]({{site.baseurl}}/build-llvm9): this is the currently recommended version
* [Build from source against LLVM 3.8]({{site.baseurl}}/build-llvm38): this version of LLVM is the earliest non-deprecated version supported by KLEE
* [Fedora package](https://src.fedoraproject.org/rpms/klee): install the latest release using `dnf install klee`
* [FreeBSD package](https://www.freshports.org/security/klee): FreeBSD users can install latest release with `pkg install klee` or by building `security/klee` port themselves.
* [Homebrew package]({{site.baseurl}}/install-brew): install the latest release using [Homebrew](https://brew.sh)
* [Building arbitrary KLEE configurations]({{site.baseurl}}/build-script): to build different configurations of KLEE and its dependencies
