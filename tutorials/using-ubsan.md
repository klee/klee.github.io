---
layout: default
title: UBSan
subtitle: Taking Advantage of the Undefined Behavior Sanitizer (UBSan)
slug: documentation
---
KLEE already includes checks that detect division by zero, overshift overflow, and out-of-bounds memory accesses (KLEE can reliably detect these specific behaviours since they are visible without viewing the source-language metadata).

To enable KLEE to find additional types of undefined behaviour, KLEE introduced handlers for the UBSan runtime calls inserted by the LLVM Undefined Behavior Sanitizer (UBSan) in [release 3.0](https://github.com/klee/klee/releases#release-v3.0). 
It is profitable for KLEE to utilize the UBSan instrumentation machinery implemented by the LLVM frontend itself as certain checks, such as signed integer overflow and locating misaligned pointers, require information that is absent or ambiguous in the LLVM IR itself, such as whether an integer operation originated from a signed type.

This tutorial provides an example of a basic C program that exhibits signed integer overflow and demonstrates how KLEE is capable of catching this behavior should the program be compiled with the appropriate UBSan flags.  

Let us consider the following program 'badDivision.c':
{% highlight c %}
#include <limits.h>
#include <stdio.h>

#include <klee/klee.h>

static int noUB(void) {
  int x;
  int y;

  klee_make_symbolic(&x, sizeof(x), "x");
  klee_assume(x != INT_MIN);
  klee_make_symbolic(&y, sizeof(y), "y");
  klee_assume(y != 0);

  return x / y;
}

static int signed_division_overflow(void) {
  int x;
  int y;

  klee_make_symbolic(&x, sizeof(x), "x");
  klee_make_symbolic(&y, sizeof(y), "y");

  return x / y;
}

int main(void) {
  unsigned path;

  klee_make_symbolic(&path, sizeof(path), "path");
  klee_assume(path <= 1);

  switch (path) {
  case 0:
    noUB();
    printf("Clean path. This result was expected.\n");
    break;
  case 1:
    signed_division_overflow();
    printf("A valid division operation occurred in this path!\n");
    break;
  }

  return 0;
}
{% endhighlight %}


# Compiling the Program

To take advantage of the UBSan handlers in KLEE, you must first enable UBSan instrumentation during the compilation process itself.  
Considering C, one can enable this with the `-fsanitize` flag in `clang`. 
In this case, we only enable instrumentation for signed division overflow with the `-fsanitize=signed-integer-overflow` flag, though one can enable most checks by utilizing the `-fsanitize=undefined` flag (it enables Clang’s standard subset of undefined-behavior checks, though omits checks such as unsigned integer overflow and implicit conversions). Additional details regarding UBSan and its specific checks can be found [here](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html).  

Command to compile our example program:  
{% highlight bash %}
$ clang -c -g -O0 -Xclang -disable-O0-optnone -emit-llvm -fsanitize=signed-integer-overflow ./badDivision.c -o ./badDivision.bc
{% endhighlight %}


# Running KLEE with the UBSan Handlers

When we execute this program with KLEE, we can see that the path that contains signed division overflow causes KLEE to throw an error and terminate the offending execution state. 
Please note that the flag `--ubsan-runtime` is required to enable KLEE's UBSan handlers:
{% highlight bash %}
$ klee --ubsan-runtime ./badDivision.bc
KLEE: WARNING: undefined reference to function: printf
KLEE: ERROR: ./badDivision.c:25: divide by zero                                                                                                                   
KLEE: NOTE: now ignoring this error at this location                                                                                                              
KLEE: WARNING ONCE: calling external: printf(133192071774208) at ./badDivision.c:42 5                                                                             
A valid division operation occurred in this path!                                                                                                                 
Clean path. This result was expected.
KLEE: ERROR: klee_src/runtime/Sanitizer/ubsan/ubsan_handlers.cpp:37: integer division overflow
KLEE: NOTE: now ignoring this error at this location

KLEE: done: total instructions = 138                                                                                                                              
KLEE: done: completed paths = 2                                                                                                                                   
KLEE: done: partially completed paths = 2                                                                                                                         
KLEE: done: generated tests = 4                      
{% endhighlight %}

Notice that the two invalid cases are detected differently. 
Division by zero is detected directly by KLEE while interpreting the LLVM division instruction, whereas signed division overflow is reported through the UBSan handler inserted by Clang.
