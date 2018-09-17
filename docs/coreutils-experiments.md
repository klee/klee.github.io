---
layout: default
title: Coreutils Experiments
subtitle: OSDI'08 Coreutils Experiments
slug: coreutils-experiment
---

This document is meant to give additional information regarding the Coreutils experiments discussed in our [KLEE OSDI'08 paper](http://llvm.org/pubs/2008-12-OSDI-KLEE.html). However, please note that in the last several years, KLEE and its dependencies (particularly LLVM and STP), have undergone major changes, which have resulted in considerable different behavior on several benchmarks, including Coreutils.

This document is structured as a series of FAQs:

1.  How did you build Coreutils?  
   
    Please follow the instructions in the [Coreutils Tutorial]({{site.baseurl}}/tutorials/testing-coreutils).

2.  What version of LLVM was used in the OSDI paper?  

    We generally kept in sync with the LLVM top-of-tree, which at the time was somewhere around LLVM 2.2 and 2.3.

3.  What version of STP was used in the OSDI paper?  

    An old version of STP, which is still available as part of KLEE's repository, in revisions up to r161056.

4.  On what OS did you run your experiments?  

    We ran most experiments on a 32-bit Fedora machine with SELinux support. The most important aspect is that this was a 32-bit system: the constraints generated on a 64-bit system are typically more complex and memory consumption might also increase.

5.  What are the 89 Coreutils applications that you tested?   

    ```
    [ base64 basename cat chcon chgrp chmod chown chroot cksum comm cp csplit cut
    date dd df dircolors dirname du echo env expand expr factor false fmt fold
    head hostid hostname id ginstall join kill link ln logname ls md5sum mkdir
    mkfifo mknod mktemp mv nice nl nohup od paste pathchk pinky pr printenv printf
    ptx pwd readlink rm rmdir runcon seq setuidgid shred shuf sleep sort split
    stat stty sum sync tac tail tee touch tr tsort tty uname unexpand uniq unlink
    uptime users wc whoami who yes
    ```

6.  What modification did you make in `sort`?  

    We changed `#define INPUT_FILE_SIZE_GUESS (1024 * 1024)`

    to `#define INPUT_FILE_SIZE_GUESS 1024` in file `sort.c`
    

7.  What options did you run KLEE with?   

    We used the following options (the command below is for paste):

    ```bash
    $ klee --simplify-sym-indices --write-cvcs --write-cov --output-module \  
    \--max-memory=1000 --disable-inlining --optimize --use-forked-solver \  
    \--use-cex-cache --with-libc --with-file-model=release \  
    \--allow-external-sym-calls --only-output-states-covering-new \  
    \--exclude-libc-cov --exclude-cov-file=./../lib/functions.txt \  
    \--environ=test.env --run-in=/tmp/sandbox --output-dir=paste-data-1h \  
    \--max-sym-array-size=4096 --max-instruction-time=10. --max-time=3600. \  
    \--watchdog --max-memory-inhibit=false --max-static-fork-pct=1 \  
    \--max-static-solve-pct=1 --max-static-cpfork-pct=1 --switch-type=internal \  
    \--randomize-fork --use-random-path --use-interleaved-covnew-NURS \  
    \--use-batching-search --batch-instructions 10000 --init-env \  
    ./paste.bc --sym-args 0 1 10 --sym-args 0 2 2 --sym-files 1 8 --sym-stdout
    ```

    Some of these options have been renamed or removed in the current version of KLEE. Most notably, the options `--exclude-libc-cov` and `--exclude-cov-file` were implemented in a fragile way and we decided to remove them from KLEE. The idea was to treat the functions in libc or specified in a text file as "covered". (For the Coreutils experiments, we were interested in covering the code in the tools themselves, as opposed to library code, see the paper for more details). If you plan to reimplement these options in a clean way, please consider contributing your code to the mainline.

    Option `--randomize-fork` was used in the experiments but recently removed (commit [`5133b98`](https://github.com/klee/klee/commit/5133b98f1d989af94902366c6d02eb6447458aa1)).


8.  What are the options closest to the ones above that work with the current version of KLEE?

    Try the following: 

    ```bash
    $ klee --simplify-sym-indices --write-cvcs --write-cov --output-module \  
    \--max-memory=1000 --disable-inlining --optimize --use-forked-solver \  
    \--use-cex-cache --libc=uclibc --posix-runtime \  
    \--allow-external-sym-calls --only-output-states-covering-new \  
    \--environ=test.env --run-in=/tmp/sandbox \  
    \--max-sym-array-size=4096 --max-instruction-time=30. --max-time=3600. \  
    \--watchdog --max-memory-inhibit=false --max-static-fork-pct=1 \  
    \--max-static-solve-pct=1 --max-static-cpfork-pct=1 --switch-type=internal \  
    \--search=random-path --search=nurs:covnew \  
    \--use-batching-search --batch-instructions=10000 \  
    ./paste.bc --sym-args 0 1 10 --sym-args 0 2 2 --sym-files 1 8 --sym-stdin 8 --sym-stdout
    ```

9.  How do I generate test.env and /tmp/sandbox?   

    We used a simple environment and a "sandbox" directory to make our experiments more deterministic. To recreate them, follow these steps:

    1.  Download `testing-env.sh` by clicking [here](http://www.doc.ic.ac.uk/~cristic/klee/klee-cu-testing-env.html), and place it in the current directory.
    
    2.  Create `test.env` by running:
    
        ```bash
        $ env -i /bin/bash -c '(source testing-env.sh; env >test.env)'
        ```

    3.  Download `sandbox.tgz` by clicking [here](http://www.doc.ic.ac.uk/~cristic/klee/klee-cu-sandbox.html), place it in `/tmp`, and run: 

        ```bash
        $ cd /tmp $ tar xzfv sandbox.tgz
        ```

0.  What symbolic arguments did you use in your experiments?

    We ran most utilities using the arguments below. Our choice was based on a high-level understanding of the Coreutils applications: most behavior can be triggered with no more than two short options, one long option, and two small input streams (stdin and one file).

    ```bash
    --sym-args 0 1 10 --sym-args 0 2 2 --sym-files 1 8 --sym-stdin 8 --sym-stdout
    ```

    For eight tools where the coverage results were unsatisfactory, we consulted the man page and increased the number and size of arguments and files as follows:

    * **dd:** `--sym-args 0 3 10 --sym-files 1 8 --sym-stdin 8 --sym-stdout`
    * **dircolors:** `--sym-args 0 3 10 --sym-files 2 12 --sym-stdin 12 --sym-stdout`
    * **echo:** `--sym-args 0 4 300 --sym-files 2 30 --sym-stdin 30 --sym-stdout`   
    * **expr:** `--sym-args 0 1 10 --sym-args 0 3 2 --sym-stdout`
    * **mknod:** `--sym-args 0 1 10 --sym-args 0 3 2 --sym-files 1 8 --sym-stdin 8 --sym-stdout`
    * **od:** `--sym-args 0 3 10 --sym-files 2 12 --sym-stdin 12 --sym-stdout`
    * **pathchk:** `--sym-args 0 1 2 --sym-args 0 1 300 --sym-files 1 8 --sym-stdin 8 --sym-stdout`
    * **printf:** `--sym-args 0 3 10 --sym-files 2 12 --sym-stdin 12 --sym-stdout`

1. How did you measure coverage?

   As mentioned in the paper, we replayed the test inputs generated by KLEE and used `gcov` to measure coverage.  One thing to take into account is that `gcov` does not record coverage on `_exit`.  This is because `_exit` terminates execution without calling any handlers, including the handler installed by `gcov` to dump coverage information.  The simplest solution is to replace `_exit` with `exit`.
