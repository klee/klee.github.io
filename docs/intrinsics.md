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

`klee_assume` is used to constrain the values symbolic variables can take. The remainder of the program's execution will only consider variable values which satisfy _condition_. Conceptually, `klee_assume(condition)` is equivalent to wrapping the rest of the program in an `if(condition) { }` statement, except that the former prints an error if the condition is unsatisfiable. Technically speaking, `klee_assume(condition)` adds _condition_ to the current path constraints.

### Interaction with short-circuit operators

When _condition_ contains [short-circuit operators](https://en.wikipedia.org/wiki/Short-circuit_evaluation), the results of the `klee_assume` instrinsics may come unexpected. For example, consider the following code and the corresponding KLEE output: Note: the output was obtained after compilation with llvm-gcc 2.9. Other compilers/versions may yield slightly different results.

{% highlight c %}
int main()
{
  int c,d;
  klee_make_symbolic(&c, sizeof(c), "c");
  klee_make_symbolic(&d, sizeof(d), "d");

  klee_assume((c==2)||(d==3));

  return 0;
}
{% endhighlight %}

{% highlight bash %}
$ llvm-gcc -c -emit-llvm p.c -o p.bc
$ klee p.bc
KLEE: output directory = "klee-out-0"
KLEE: ERROR: invalid klee_assume call (provably false)
KLEE: NOTE: now ignoring this error at this location

KLEE: done: total instructions = 53
KLEE: done: completed paths = 3
KLEE: done: generated tests = 3
{% endhighlight %}

One might reasonably expect a single path through the progam and no error reports, while KLEE finds 3 paths, one of which triggers an error. The reason lies in the way compilers handle short-circuit operators. Upon compilation, the above code is transformed into 

{% highlight c %}
int main()
{
  int c,d;
  klee_make_symbolic(&c, sizeof(c), "c");
  klee_make_symbolic(&d, sizeof(d), "d");
  
  int tmp;
  if (c == 2) {
    tmp = 1;
  } else if (d == 3) {
    tmp = 1;
  } else {
    tmp = 0;
  }
  klee_assume(tmp);
  
  return 0;
}
{% endhighlight %}

Since the program contains 3 paths and all are feasible, `klee_assume` will be called 3 times with trivial arguments:

{% highlight c %}
klee_assume(1);
klee_assume(1);
klee_assume(0);
{% endhighlight %}

The first two paths go further while the 3rd is terminated with an error, which is the intended behavior, albeit in a rather non-straightforward way. Ideally the paths which satisfy the assume would be merged but klee currently has only some experimental support for path merging. 

As a side note, it is possible to obtain the 'one path' behavior by replacing the logical `&&` and `||` operators with their bitwise counterparts. To correctly do this, ensure that all operands have boolean values and no side effects.

## `klee_prefer_cex(object, condition)`

This function tells KLEE to prefer certain values when generating test cases as output. A KLEE state can correspond to many different possible test cases. For example, in this code:

{% highlight c %}
char input[4];
klee_make_symbolic(input, sizeof input);
assert(input[0] == 'Q');
{% endhighlight %}

KLEE will have a single failing state that corresponds to `input = "aaaa"`, `input = "1234"`, and every other input that fails the assertion. Normally, when KLEE generates a test case for this failure, it chooses an arbitrary one of these inputs. The result could be `input = "\0\0\0\0"` or `input = "\xff\xff\xff\xff"` or some other ugly value. We can make it more readable by using `klee_prefer_cex` after `klee_make_symbolic`:

{% highlight c %}
for (int i = 0; i < 4; i++)
  klee_prefer_cex(input, 32 <= input[i] && input[i] <= 126); // assume ASCII
{% endhighlight %}

Now, when KLEE has a choice between many equivalent test cases, it will prefer to use printable characters when possible. When KLEE finds paths that conflict with the `klee_prefer_cex` condition, it will ignore the preference and generate (ugly) test cases anyway. Thus, the generated test cases will follow exactly the same paths and trigger exactly the same crashes as before, but some of them will look nicer.

The POSIX runtime uses `klee_prefer_cex` in a few places, for instance to prefer printable characters in symbolic command-line arguments. You can cause KLEE to ignore all preferences by giving it the `--no-prefer-cex` option.
