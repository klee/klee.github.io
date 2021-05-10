---
layout: default
title: Symbolic Environment
subtitle: Using Symbolic Environment
slug: tutorials
---

As mentioned in the [overview of KLEE's basic command-line options]({{site.baseurl}}/docs/options/), KLEE provides several options as part of its symbolic environment. Their usage, however, is often not easily understood by new users. This tutorial provides basic usage examples for `-sym-arg` and `-sym-files`, which are perhaps the most essential among the options.

# `-sym-arg` Usage

We note that `-sym-arg <N>` provides a command-line argument of length N to the program under test. Its variant `-sym-args <MIN> <MAX> <N>` provides at least MIN arguments and at most MAX symbolic arguments, each with maximum length N. To demonstrate `-sym-arg`, let us first consider the following program `password.c` that checks its command-line argument for a match with a hard-coded password.
{% highlight c %}
#include <stdio.h>

int check_password(char *buf) {
  if (buf[0] == 'h' && buf[1] == 'e' &&
      buf[2] == 'l' && buf[3] == 'l' &&
      buf[4] == 'o')
    return 1;
  return 0;
}

int main(int argc, char **argv) {
  if (argc < 2)
     return 1;
  
  if (check_password(argv[1])) {
    printf("Password found!\n");
    return 0;
  }

  return 1;
}
{% endhighlight %}
To enable symbolic environment, KLEE has to be given the `-posix-runtime` option. We run KLEE given the bitcode of `password.c` as input and using `-sym-arg` option as follows.
{% highlight bash %}
$ clang -emit-llvm -c -g -O0 -Xclang -disable-O0-optnone password.c
$ klee -posix-runtime password.bc -sym-arg 5
KLEE: NOTE: Using model: /home/klee/klee/build/Release+Debug+Asserts/lib/libkleeRuntimePOSIX64_Release+Debug+Asserts.bca
KLEE: output directory is "/home/klee/klee-out-0"
KLEE: Using STP solver backend
KLEE: WARNING: undefined reference to function: printf
KLEE: WARNING: undefined reference to function: strlen
KLEE: WARNING: undefined reference to function: strncmp
KLEE: WARNING ONCE: Alignment of memory from call "malloc" is not modelled. Using alignment of 8.
KLEE: WARNING ONCE: calling external: syscall(4, 94328563719264, 94328559327712) at /home/klee/klee/runtime/POSIX/fd.c:544 12
KLEE: WARNING ONCE: calling __klee_posix_wrapped_main with extra arguments.
KLEE: WARNING ONCE: calling external: printf(94328563829504) at password.c:17 5
Password found!

KLEE: done: total instructions = 1190
KLEE: done: completed paths = 6
KLEE: done: partially completed paths = 0
KLEE: done: generated tests = 6
{% endhighlight %}
As can be seen, due to the command-line argument being symbolic, KLEE executed six paths, with one of the path having the command-line argument match the password.

# `-sym-files` Usage

The option `-sym-files <NUM> <N>` creates NUM symbolic files, where the first file is named 'A', the second 'B', and so on, each with size N. Its sibling options `-sym-stdin` and `-sym-stdout` make the standard input and output symbolic, respectively.

Let us now consider a password checker, still called `password.c` that reads a string from a file specified by the user and checks if it matches a hard-coded password. If the file name is not specified, or if there is an error when opening the file, it reads the string from the standard input.
{% highlight c %}
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdio.h>
#include <unistd.h>

int check_password(int fd) {
  char buf[5];
  if (read(fd, buf, 5) != -1) {
    if (buf[0] == 'h' && buf[1] == 'e' &&
	buf[2] == 'l' && buf[3] == 'l' &&
	buf[4] == 'o')
      return 1;
  }
  return 0;
}

int main(int argc, char **argv) {
  int fd;

  if (argc >= 2) {
    if ((fd = open(argv[1], O_RDONLY)) != -1) {
      if (check_password(fd)) {
        printf("Password found in %s\n", argv[1]);
        close(fd);
        return 0;
      }
      close(fd);
      return 1;
    }
  }

  if (check_password(0)) {
    printf("Password found in standard input\n");
    return 0;
  }

  return 1;
}
{% endhighlight %}
We now run the program using KLEE. For the program not to get stuck trying to read data, we need to provide some input. In our first run, we provide a symbolic standard input using `-sym-stdin` option. The symbolic input will make KLEE explore the path with successful password check.
{% highlight bash %}
$ clang -emit-llvm -c -g -O0 -Xclang -disable-O0-optnone password.c
$ klee -posix-runtime password.bc -sym-stdin 10
KLEE: NOTE: Using model: /home/klee/klee/build/Release+Debug+Asserts/lib/libkleeRuntimePOSIX64_Release+Debug+Asserts.bca
KLEE: output directory is "/home/klee/klee-out-1"
KLEE: Using STP solver backend
KLEE: WARNING: undefined reference to function: printf
KLEE: WARNING: undefined reference to function: strlen
KLEE: WARNING: undefined reference to function: strncmp
KLEE: WARNING ONCE: Alignment of memory from call "malloc" is not modelled. Using alignment of 8.
KLEE: WARNING ONCE: calling external: syscall(4, 94571123861232, 94571119847248) at /home/klee/klee/runtime/POSIX/fd.c:544 12
KLEE: WARNING ONCE: calling __klee_posix_wrapped_main with extra arguments.
KLEE: WARNING ONCE: calling external: printf(94571123946704) at password.c:35 5
Password found in standard input

KLEE: done: total instructions = 1856
KLEE: done: completed paths = 6
KLEE: done: partially completed paths = 0
KLEE: done: generated tests = 6
{% endhighlight %}
We have now discovered the password using KLEE.

Our program can also read the password from a disk file, but we want to read a file with symbolic content so that KLEE executes the path where the password check is successful. The `-sym-files` option provides several such files named 'A', 'B', 'C', and so on. By specifying the option `-sym-files 1 10` below, we ask KLEE to provide one symbolic file of size 10 bytes, and that file is named 'A' by KLEE. We therefore provide this file name as an argument to our program.
{% highlight bash %}
$ klee -posix-runtime password.bc A -sym-files 1 10
KLEE: NOTE: Using model: /home/klee/klee/build/Release+Debug+Asserts/lib/libkleeRuntimePOSIX64_Release+Debug+Asserts.bca
KLEE: output directory is "/home/klee/klee-out-2"
KLEE: Using STP solver backend
KLEE: WARNING: undefined reference to function: printf
KLEE: WARNING: undefined reference to function: strlen
KLEE: WARNING: undefined reference to function: strncmp
KLEE: WARNING ONCE: Alignment of memory from call "malloc" is not modelled. Using alignment of 8.
KLEE: WARNING ONCE: calling external: syscall(4, 94110989166360, 94110985152336) at /home/klee/klee/runtime/POSIX/fd.c:544 12
KLEE: WARNING ONCE: calling __klee_posix_wrapped_main with extra arguments.
KLEE: WARNING ONCE: calling external: printf(94110989239968, 94110989166176) at password.c:25 15
Password found in A

KLEE: done: total instructions = 4395
KLEE: done: completed paths = 6
KLEE: done: partially completed paths = 0
KLEE: done: generated tests = 6
{% endhighlight %}
The password was successfully read from the symbolic file A in one of the execution paths.
