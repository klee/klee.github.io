---
layout: default
title: Intrinsics
subtitle: Overview of the main KLEE intrinsic functions
slug: documentation
---

{:.toc}
Contents
{:.toc__title .no_toc}
* Table of contents placeholder
{:.toc__list .list-anchor}
{:toc}
 
KLEE provides a set of special functions which are useful in the context of symbolic execution. Whenever a program calls one of these functions, KLEE handles internally the call, hence their intrinsic nature. The functions are declared in `include/klee/klee.h`. The most often used intrinsic is [klee_make_symbolic]({{site.baseurl}}/tutorials/testing-function), which creates an unconstrained symbolic object.

## `klee_assume(condition)`

### Usage

`klee_assume(condition)` is used to constrain the values symbolic variables can take. The remainder of the program's execution will only consider variable values which satisfy _condition_. Conceptually, `klee_assume(condition)` is equivalent to wrapping the rest of the program in an `if(condition){ }` statement, except that the former prints an error if the condition is unsatisfiable. Technically speaking, `klee_assume(condition)` adds _condition_ to the current path constraints.

### Interaction with short-circuit operators

When _condition_ contains [short-circuit operators](https://en.wikipedia.org/wiki/Short-circuit_evaluation), the results of the `klee_assume` intrinsics may come unexpected. For example, consider the following code and the corresponding KLEE output:

{% highlight c %}
#include "klee/klee.h"

int main() {
  int c,d;
  klee_make_symbolic(&c, sizeof(c), "c");
  klee_make_symbolic(&d, sizeof(d), "d");

  klee_assume((c==2) && (d==3));

  return 0;
}
{% endhighlight %}

{% highlight bash %}
$ clang -O0 -I klee_path/include/ -g -c -emit-llvm p.c -o p.bc
$ klee p.bc
KLEE: output directory is "/path/klee-out-0"
KLEE: Using STP solver backend
KLEE: ERROR: /path/p.c:8: invalid klee_assume call (provably false)
KLEE: NOTE: now ignoring this error at this location

KLEE: done: total instructions = 23
KLEE: done: completed paths = 2
KLEE: done: generated tests = 2
{% endhighlight %}

One might reasonably expect a single path through the program, while KLEE finds 2 paths.
The reason lies in the way compilers handle short-circuit operators.
Upon compilation, the above code is transformed into LLVM bitcode similar to the following C code:

{% highlight c %}
#include "klee/klee.h"

int main() {
  int c,d;
  klee_make_symbolic(&c, sizeof(c), "c");
  klee_make_symbolic(&d, sizeof(d), "d");
  
  int tmp;
  if (c == 2) 
    tmp = d == 3;
  else
    tmp = 0;

  klee_assume(tmp);

  return 0;
}
{% endhighlight %}

Since the program contains two paths and both are feasible, `klee_assume` will be called two times: once with the comparison expression "d == 3" and once with the trivial constant argument "0".
As "0" is equivalent to _false_ and no path can satisfy this condition, KLEE prints out an error message and terminates the corresponding path.

More aggressive optimisations (e.g. `-Os`) reduce the program to a single path and `klee_assume` is called once with the expression "c == 2 && d == 3" as expected.
Remember, symbolic execution engines use expressions internally to represent computations over symbolic variables.
The C API of `klee_assume` still requires a boolean condition.

As a side note, it is also possible to obtain the 'one path'-behaviour by replacing the logical `&&` and `||` operators with their bitwise counterparts.
To correctly do this, ensure that all operands have boolean values and no side effects.

Note: the output was obtained after compilation with Clang 6.
Other compilers/versions may yield slightly different results.

## `klee_prefer_cex(object, condition)`

This function tells KLEE to prefer certain values when generating test cases as output. A KLEE state can correspond to many different possible test cases. For example, in this code:

{% highlight c %}
char input[4];
klee_make_symbolic(input, sizeof(input), "input");
assert(input[0] == 'Q');
{% endhighlight %}

KLEE will have a single failing state that corresponds to `input = "aaaa"`, `input = "1234"`, and every other input that fails the assertion. Normally, when KLEE generates a test case for this failure, it can choose any of these valid inputs. The result could be `input = "\0\0\0\0"` or `input = "\xff\xff\xff\xff"` or some other unreadable value. We can make it more readable by using `klee_prefer_cex` after `klee_make_symbolic`:

{% highlight c %}
for (int i = 0; i < 4; i++)
  klee_prefer_cex(input, 32 <= input[i] && input[i] <= 126); // assume ASCII
{% endhighlight %}

**NOTE:** Only use `klee_prefer_cex` immediately after a `klee_make_symbolic` call.  It currently cannot be used after a `klee_range` call.

Now, when KLEE has a choice between many possible test cases, it will prefer to use printable characters when possible. When KLEE finds paths that conflict with the `klee_prefer_cex` condition, it will ignore the preference and generate (potentially unreadable) test cases anyway.

The POSIX runtime uses `klee_prefer_cex` internally, in particular to prefer printable characters in symbolic command-line arguments.  To enable this option, use `-readable-posix-inputs`.  It is disabled by default, as `klee_prefer_cex` can be expensive when used extensively.
