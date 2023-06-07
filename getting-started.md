---
layout: default
title: Getting Started
subtitle: Getting Started with KLEE
slug: getting-started
---

## Web Interface

Run tiny code examples in your browser with [KLEE web](http://klee.doc.ic.ac.uk/).

## Docker

[Our Docker images]({{site.baseurl}}/docker) are one of the fastest ways to get started.

## Installation via Package Manager

* [Use your package manager](https://repology.org/project/klee/versions) for Arch Linux, openSUSE Tumbleweed, and several other distributions
* [Running with Nix]({{site.baseurl}}/nix): this is even faster than Docker if you have Nix.
* [FreeBSD package](https://www.freshports.org/security/klee): FreeBSD users can install latest release with `pkg install klee` or by building `security/klee` port themselves.
* [Homebrew package]({{site.baseurl}}/install-brew): install the latest release using [Homebrew](https://brew.sh)

## Manual Installation

* [Build from source against LLVM 13]({{site.baseurl}}/build-llvm13): this is the currently recommended version
* [Building arbitrary KLEE configurations]({{site.baseurl}}/build-script): to build different configurations of KLEE and its dependencies
