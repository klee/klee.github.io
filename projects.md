---
layout: default
title: Projects
subtitle: KLEE Open Projects
slug: projects
---

This page lists a variety of open projects that are natural (and tractable) extensions of KLEE and things that we would love to see people work on. If you are interested in tackling any of the projects, please mail [klee-dev](/klee-dev/) with your ideas -- or even better, your patches!

* _Implement SMT support for Kleaver:_

  We would like to make Kleaver (KLEE's constraint solver) a more viable standalone product. One of the steps in this direction is to implement SMT support. This would also allow us to enter Kleaver into the [SMT-COMP](http://www.smtcomp.org/2011/) theorem prover competition, which would be a good proving ground for our optimizations and benchmarking our underlying constraint solver (STP, currently) against other implementations.

* _Distributed Constraint Solving:_

  Much of the execution time of KLEE is spent inside the constraint solver. A natural extension would be to implement support for a distributed constraint solver, which would run KLEE on a single machine, but would farm out the constraints to be solved on a network of machines.

* _Improve User Interface:_

  This is not a very glamorous project, but it is still important.Currently, KLEE has a myriad of command line options and flags, most of which are left over from its research project roots. In order to promote KLEE's use as a user tool, we would like to clean up most of the UI so that the default behavior matches best practice, and so that more arcane or research-only options are hidden by default.

* _Experiment With Other Constraint Solvers:_

  For the most part, KLEE has been written so that it is possible toswap in other constraint solvers, but we have never tried anything other than STP. We would love to see the results of using other contraint solvers (like Yices or Z3) with KLEE.

* _Implement Expression Level Constraint Optimization:_

  KLEE does not currently do much optimization of constraint expressions before sending them to the constraint solver. We would like to have an internal framework for doing optimization of constraint expressions(e.g., (A & ~A) => 0) so that these optimizations are only done once instead of on every solver query.

  In general, because KLEE is dealing with expressions which result from dynamic execution traces, many expressions end up having constant components. This means we can frequently apply the same optimizations a compiler would do, but to much greater effect because we are more likely to see constant values. For reference, see the kinds of optimizations done by LLVM's `InstCombine` pass [here](http://llvm.org/viewvc/llvm-project/llvm/trunk/lib/Transforms/InstCombine/).

  The bulk of this project involves defining a good framework for us to apply optimizations to expressions, and for deciding when to attempt tooptimization expressions.
