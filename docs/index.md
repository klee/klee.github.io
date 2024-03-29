---
layout: default
title: Documentation
subtitle: Learn How to Use KLEE
slug: documentation
---

{% include version_warning.md %}

---

### If you have a question on using KLEE:
1. Please first check the documentation below
1. Then check the [searchable mailing list archive](http://www.mail-archive.com/klee-dev@imperial.ac.uk/)
1. If this still doesn't answer your questions then:
   * If you think it is a bug, please open an [issue on GitHub](https://github.com/klee/klee/issues)
   * If it is a general question, then please send an email to the [klee-dev mailing list]({{site.baseurl}}/klee-dev/)

---

## Usage

1. [Intrinsics]({{site.baseurl}}/docs/intrinsics/): Overview of the main KLEE intrinsic functions.
1. [KLEE Options]({{site.baseurl}}/docs/options/): Overview of KLEE's main command-line options.
1. [Generated Files]({{site.baseurl}}/docs/files/): Overview of the main files generated by KLEE.
1. [Tools]({{site.baseurl}}/docs/tools/): Overview of the main auxiliary tools provided by KLEE.
1. [Solver Chain]({{site.baseurl}}/docs/solver-chain/): Overview of the solver chain and related command-line arguments.
1. [Kleaver Options]({{site.baseurl}}/docs/kleaver-options/): Overview of Kleaver's main command-line options.
1. [KQuery]({{site.baseurl}}/docs/kquery): The reference manual for the KQuery language, used for interacting with Kleaver.
1. [Coreutils Experiments]({{site.baseurl}}/docs/coreutils-experiments): Some information about the Coreutils experiments presented in our [KLEE OSDI'08 paper](http://www.doc.ic.ac.uk/~cristic/papers/klee-osdi-08.pdf).

## Tutorials

1. [First tutorial]({{site.baseurl}}/tutorials/testing-function/): Testing a small function.
2. [Second tutorial]({{site.baseurl}}/tutorials/testing-regex/): Testing a simple regular expression library.
3. [Using a symbolic environment]({{site.baseurl}}/tutorials/using-symbolic/): Guide with examples on how to use the symbolic environment such as symbolic files and command-line arguments for the program under test.
4. [Testing Coreutils]({{site.baseurl}}/tutorials/testing-coreutils/): In-depth description of how to use KLEE to test GNU Coreutils.

## External Resources

We also recommend the following external resources.  When using them, please take into account that they might be using old versions of KLEE. 

1. [Symbolic execution with KLEE](https://adalogics.com/blog/symbolic-execution-with-klee): A set of four instructional videos introducing KLEE, starting with how to get started with KLEE and ending with a demo that finds memory corruption bugs in real code.
1. [Solving a maze with KLEE](http://feliam.wordpress.com/2010/10/07/the-symbolic-maze/): A nice explanation of how symbolic execution can be used to generate interesting program inputs. The example shows how to use KLEE to find all the solutions to a maze game.
1. [Keygenning with KLEE and Hex-Rays](https://gitlab.com/Manouchehri/Matryoshka-Stage-2/blob/master/stage2.md): A beginners explanation of using symbolic execution to solve a small binary's pseudocode.
1. [Keygenning with KLEE](https://doar-e.github.io/blog/2015/08/18/keygenning-with-klee/): A more in-depth guide to using KLEE for key generation.
1. [How to generate tests automatically?](https://verificationglasses.wordpress.com/2020/10/02/symbolic-execution-klee/): A basic introduction to symbolic execution with KLEE.
1. [Using KLEE to generate tests for binary search](https://verificationglasses.wordpress.com/2020/10/17/klee-binary-search/): A tutorial on using KLEE to test a binary search implementation).
1. [A Guide to KLEE LLVM Execution Engine Internals](https://www.alchemistowl.org/pocorgtfo/pocorgtfo18.pdf): A quick overview of KLEE's internals and core classes in [PoC\|\|GTFO](https://github.com/angea/pocorgtfo) 0x18 (pp. 51).
1. [Testing Rust programs with KLEE](https://github.com/project-oak/rust-verification-tools/): A collection of tools/libraries to support testing of Rust programs with KLEE.
1. [SAT/SMT by example](https://yurichev.com/writings/SAT_SMT_by_example.pdf): A comprehensive guide focusing on using SAT and SMT solvers, which includes lots of interesting examples involving KLEE.
1. [Measuring the coverage achieved by symbolic execution](http://ccadar.blogspot.com/2020/07/measuring-coverage-achieved-by-symbolic.html): A blog post that discusses the difference between the internal coverage reported by KLEE and the external coverage reported by a tool such as GCov.
1. [Binary symbolic execution with KLEE-Native](https://blog.trailofbits.com/2019/08/30/binary-symbolic-execution-with-klee-native/): An extension of KLEE that analyses binaries by lifting them to LLVM IR
1. [SHA-3 Buffer Overflow](https://mouha.be/sha-3-buffer-overflow-part-2/): Finding a bug in a sha-3 implementation with KLEE
