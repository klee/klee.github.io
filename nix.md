---
layout: default
title: Nix
subtitle: Running KLEE with Nix
slug: nix
---

# Quick Start

KLEE has a package on [nixpkgs](https://search.nixos.org/packages?query=klee&show=klee) that is usable on x86_64 Linux systems.
Getting started with it is as fast as or faster than using the [Docker version]({{site.baseurl}}/docker), and carries all the [reproducible build guarantees](https://r13y.com/) of the
Nix build system.

To make the package available in an ephemeral shell, ensure [you have Nix or are running NixOS](https://nixos.org/download.html), and run:

```bash
$ nix-shell -p klee
```

If the package was built on [Hydra](https://hydra.nixos.org/) and is available on the [NixOS binary cache](https://cache.nixos.org/), it will simply download and be instantly available to run.
Otherwise, the package will build.

# What is Nix?

- Nix is a build system focused on reproducibility. Build scripts are written in a functional language, also called Nix.
- [nixpkgs](https://github.com/NixOS/nixpkgs) contains knowledge on how to build many packages for Linux and MacOS systems.
- NixOS is an entire Linux-based operating system built with nixpkgs. NixOS is just Nix to its logical conclusion - the entire system configuration is immutable
  and built functionally from several packages.

The instructions for building KLEE in nixpkgs are [located here](https://github.com/NixOS/nixpkgs/blob/master/pkgs/applications/science/logic/klee/default.nix). The Nix
package runs the KLEE unit and system test suite as a sanity check before the build completes.

# Installing KLEE systemwide in NixOS

If you are running NixOS and wish to install KLEE systemwide, simply include it in `environment.systemPackages` in your NixOS configuration.

```nix
environment.systemPackages = with pkgs; [
  klee
];
```

# Advanced package use

Currently, the Nix package for KLEE takes a "debug" argument that defaults to false. If you want a debug build of KLEE,
you would use the [nixpkgs override pattern](https://nixos.org/guides/nix-pills/nixpkgs-overriding-packages.html) to force it to true.

NOTE: If any of the build inputs change from the defaults, Nix will no longer be able to fetch it from the NixOS binary cache, and a local build will occur.

```bash
$ nix-shell -p 'klee.override { debug = true; }'
```
