---
layout: default
title: Projects
subtitle: KLEE Open Projects
slug: projects
---

This page lists a variety of open projects that are natural (and tractable) extensions of KLEE and things that we would love to see people work on. If you are interested in tackling any of the projects, please mail [klee-dev](/klee-dev/) with your ideas -- or even better, submit your PRs directly on GitHub!

* _Package KLEE for Ubuntu and Other Distributions:_

	KLEE is already packaged for [FreeBSD](https://www.freshports.org/security/klee).  We would like to find maintainers for other popular distributions, particularly Ubuntu, but also macOS, Arch Linux, Gentoo Linux, OpenSUSE etc. 

* _Implement SMT Support for Kleaver:_

	We would like to make Kleaver (KLEE's constraint solver) a more viable standalone product. One of the steps in this direction is to implement [SMT-LIB](http://smtlib.cs.uiowa.edu/) support, in particular a parser for .smt2 files. This would also allow us to enter Kleaver into the [SMT-COMP](https://smt-comp.github.io/) constraint solving competition, which would be a good proving ground for our optimizations and benchmarking our underlying constraint solvers.


* _Improve KLEE Web:_

	[KLEE Web](http://klee.doc.ic.ac.uk/) makes it possible for users to quickly try KLEE in the browser.  Currently, the project lacks an active maintainers, and there are many features which would be useful to have, such as a better UI for retrieving results, better usage tracking, more examples and support for C++.  KLEE Web is hosted [here](https://github.com/klee/klee-web).


* _Implement Expression-Level Constraint Optimization:_

	KLEE performs limited optimizations of constraints before sending them to the constraint solver. We would like to have an internal framework for doing optimization of constraint expressions (e.g., (A & ~A) => 0) so that these optimizations are only done once instead of on every solver query.  In general, because KLEE is dealing with expressions which result from dynamic execution traces, many expressions end up having constant components. This means we can frequently apply the same optimizations a compiler would do, but to much greater effect because we are more likely to see constant values. For reference, see the kinds of optimizations done by LLVM's `InstCombine` pass [here](http://llvm.org/viewvc/llvm-project/llvm/trunk/lib/Transforms/InstCombine/).  The bulk of this project involves defining a good framework for us to apply optimizations to expressions, and for deciding when to attempt to optimize expressions.


* _Add Support for Missing LLVM Intrinsics:_

	There are various LLVM intrinsics which KLEE still doesn't support.  We track them [here](https://github.com/klee/klee/issues/678).


* _Add Support for Multithreaded Applications:_

	KLEE lacks support for multi-threaded applications, although other extensions of KLEE provide such support, particularly [Cloud9](http://cloud9.epfl.ch/).  An attempt to add such support can be found [here](https://github.com/klee/klee/pull/240).


* _More Sophisticated Logging Support:_

	Currently, most of the log data dumped to files (info, run.stats, .kquery files) is not necessarily easy to parse. Allowing more sophisticated logging mechanisms (e.g. based on YAML) would allow us to combine readability and parseability.  


* _Support for Another LibC:_

	KLEE currently relies on ucLibC, which is not maintained anymore.  The project would add support for another libc, e.g., see this [attempt to add musl](https://github.com/klee/klee/pull/483).


For other project suggestions, see [Suggestions for improvement and possible KLEE enhancements](https://github.com/klee/klee/projects/2), [KLEE Extensions](https://github.com/klee/klee/projects/4) and [KLEE's Issues](https://github.com/klee/klee/issues).
