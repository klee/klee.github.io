---
layout: default
title: UBSan
subtitle: Taking Advantage of The Undefined Behavior Sanitizer
slug: documentation
---
KLEE introduced handlers for the UBSan runtime calls inserted by the LLVM Undefined Behavior Sanitizer in 
[release 3.0](https://github.com/klee/klee/releases#release-v3.0). This functionality is intended to supplement the checks
that KLEE already includes internally to detect division by zero and overshift overflow (KLEE can reliably detect these 
specific behaviors, since they are visible without viewing the source-language metadata) as well as the undefined behavior 
that will be located by KLEE during the symbolic execution phase itself. It is profitable for KLEE to utilize the UBSan 
instrumentation machinery implemented by the LLVM frontend itself as certain checks, such as signed integer overflow and
locating misaligned pointers, require information that is absent or ambiguous in the LLVM IR itself, such as whether an 
integer operation originated from a signed type to provide an accurate determination. This tutorial provides an example 
of a basic C program that exhibits signed integer overflow and demonstrates how KLEE is capable of catching this behavior
should the program be compiled with the appropriate UBSan flags.  

Let us consider the following program 'badDivision.c':
{% highlight c %}
#include <stdint.h>
#include <math.h>
#include <stdio.h>

#include <klee/klee.h>

double noUB(void) {
  double a = 85.3;
  double b = 19.8;
  double c = 0.11;
  return (-b + sqrt(b*b - 4*a*c)) / (2*a);
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
      printf("clean path: %lf\n", noUB());
    break;
    case 1:
      signed_division_overflow();
      printf("A valid division operation occurred in this path!\n");
    break;
  }
  
  return 0;
}
{% endhighlight %}

# Compiling The Program

If one wishes to take advantage of the UBSan handlers in KLEE, it is first compulsory to enable UBSan instrumentation 
within the appropriate LLVM frontend during the compilation process itself. Considering C, one can enable this with the 
`-fsanitize` flag in `clang`. In this case, we only enable instrumentation for signed division overflow with the 
`-fsanitize=signed-integer-overflow` flag, though one can enable most checks by utilizing the `-fsanitize=undefined` 
flag (it enables Clang’s standard subset of undefined-behavior checks, though omits checks such as unsigned integer 
overflow and implicit conversions). Additional details regarding UBSan and its specific checks can be found [here](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html).  


Command to compile our example program:  
{% highlight bash %}
$ clang -c -g -O0 -Xclang -disable-O0-optnone -emit-llvm -fsanitize=signed-integer-overflow ./badDivision.c -o ./badDivision.bc
{% endhighlight %}

# Demonstrating The UBSan Handlers Found In KLEE

When we execute this program with KLEE, we can see that the path that contains signed division overflow causes KLEE to 
throw an error and terminate the offending execution state. Please note that the flag `--ubsan-runtime` is required to 
enable KLEE's UBSan handlers.
{% highlight bash %}
$ klee --libc=uclibc --ubsan-runtime --posix-runtime ./badDivision.bc
KLEE: NOTE: Using POSIX model: /tmp/klee_build130stp_z3/runtime/lib/libkleeRuntimePOSIX64_Debug+Asserts.bca
KLEE: NOTE: Using klee-uclibc : /tmp/klee_build130stp_z3/runtime/lib/klee-uclibc.bca
KLEE: output directory is "/home/klee/klee_ubsan_test/./klee-out-0"
KLEE: Using STP solver backend
KLEE: SAT solver: MiniSat
KLEE: Deterministic allocator: Using quarantine queue size 8
KLEE: Deterministic allocator: globals (start-address=0x7f40af200000 size=10 GiB)
KLEE: Deterministic allocator: constants (start-address=0x7f3e2f200000 size=10 GiB)
KLEE: Deterministic allocator: heap (start-address=0x7e3e2f200000 size=1024 GiB)
KLEE: Deterministic allocator: stack (start-address=0x7e1e2f200000 size=128 GiB)
KLEE: WARNING: undefined reference to function: sqrt
KLEE: WARNING ONCE: calling external: syscall(16, 0, 21505, 132543126962176) at klee_src/runtime/POSIX/fd.c:997 10                                                                                                                  
KLEE: WARNING ONCE: Alignment of memory from call "malloc" is not modelled. Using alignment of 8.                                                                                                                                   
KLEE: WARNING ONCE: calling __klee_posix_wrapped_main with extra arguments.                                                                                                                                                         
KLEE: ERROR: ./badDivision.c:21: divide by zero                                                                                                                                                                                     
KLEE: NOTE: now ignoring this error at this location                                                                                                                                                                                
KLEE: WARNING ONCE: calling external: sqrt(4644944186881844708) at ./badDivision.c:11 14                                                                                                                                            
KLEE: WARNING ONCE: calling external: printf(133741053739008) at ./badDivision.c:37 5                                                                                                                                               
A valid division operation occurred in this path!
clean path: -0.005695                                                                                                                                                                            
KLEE: ERROR: klee_src/runtime/Sanitizer/ubsan/ubsan_handlers.cpp:37: integer division overflow
KLEE: NOTE: now ignoring this error at this location

KLEE: done: total instructions = 13532                                                                                                                                                                                              
KLEE: done: completed paths = 2                                                                                                                                                                                                     
KLEE: done: partially completed paths = 2                                                                                                                                                                                           
KLEE: done: generated tests = 4   
{% endhighlight %}

Notice that the two invalid cases are detected differently. Division by zero is detected directly by KLEE while 
interpreting the LLVM division instruction, whereas signed division overflow is reported through the UBSan handler
inserted by Clang.
