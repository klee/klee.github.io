---
layout: default
title: Getting Involved
subtitle: Getting Involved
slug: getting-involved
lead: If you are interested in contributing to KLEE, here is some relevant information.
---

## License and Copyright

KLEE is made available under the liberal [UIUC open source license](https://opensource.org/licenses/UoI-NCSA.php). All contributors to the KLEE codebase agree to release their contributions under this license.  

The copyright for the code is held by the individual contributors of the code under the terms of KLEE's license.


## Mailing Lists

For both users and developers, the [klee-dev](https://mailman.ic.ac.uk/mailman/listinfo/klee-dev) mailing list is the central place for discussions, questions, and announcements.

If you have queries about KLEE that are not answered on this website, please send a message to the list rather than opening a GitHub issue.

However, before sending your message, please check [klee-dev's searchable archive](http://www.mail-archive.com/klee-dev@imperial.ac.uk/) to see if your question has already been answered.

Please note that only subscribers can post messages to the list, so you would need to subscribe to the list first.  The first post of each member is moderated to avoid spam, so please allow up to a few days for your first message to be approved.

Commit messages to the KLEE repository go to [klee-commits](https://mailman.ic.ac.uk/mailman/listinfo/klee-commits).

## Bug Reports

If you find a bug in KLEE, please fill a bug report on [GitHub](https://github.com/klee/klee/issues/new). You might also want to report it on [klee-dev](#mailing-lists).

## Working with the Code

You should first check [KLEE's developer's guide]({{site.baseurl}}/docs/developers-guide).

Developer documentation is written in [Doxygen]({{site.baseurl}}/doxygen/html/).

Many parts of KLEE rely on the LLVM infrastructure, so you might also want to look at [LLVM's General Programming
Documentation](http://llvm.org/docs/#llvmprog).

## Open Projects

This section lists a variety of open projects that are natural (and tractable) extensions of KLEE and things that we would love to see people work on. If you are interested in tackling any of the projects, please mail [klee-dev](#mailing-lists) with your ideas -- or even better, submit your PRs directly on GitHub!

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


* _Support for Floating-point Arithmetic:_

	KLEE lacks support for symbolic floating-point computation.  An extension of KLEE for floating point exists (see [KLEE-Float](https://srg.doc.ic.ac.uk/projects/klee-float/)), but it makes intrusive changes to the expression representation and relies on constraint solvers for floating point, which are slow.  In this project, you would add support for floating-point computations using fixed-point arithmetic.  While this approach has obvious limitations, it should be relatively easy to add to KLEE and lead to significantly higher performance in many cases.

For other project suggestions, see [Suggestions for improvement and possible KLEE enhancements](https://github.com/klee/klee/projects/2), [KLEE Extensions](https://github.com/klee/klee/projects/4) and [KLEE's Issues](https://github.com/klee/klee/issues).
