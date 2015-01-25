title: OSLAB notes 2
date: 2014-12-12 21:02:04
tags:
- linux
- fork
- signal
- pipe
categories:
- LerningNotes
---

This time, we are asked to play with the `fork()`, `signal()`, `kill()` and some other POSIX APIs related with process control and communication.

## Fork and pass signal
The first task is to use `fork` to create 2 child-processes and then use soft interrupt (signal) to make interprocess communication.

###Fork
Well, first of all you need to create 2 child-processes. And how to achieve that, this?
```C
pid1 = fork();
pid2 = fork();
```

Well, young man, you're simply naiveeee.

Before making further explanation, you'd better run this code yourself.

```C
pid_t pid1, pid2;
pid1 = fork();
pid2 = fork();
printf("The father %d created, pid1: %d \t pid2: %d\n", getpid(), pid1, pid2);
```

The output goes like this,

    The father 6281 created, pid1: 6282     pid2: 6283
    The father 6282 created, pid1: 0        pid2: 6284
    The father 6284 created, pid1: 0        pid2: 0
    The father 6283 created, pid1: 6282     pid2: 0

Strange, huh? There is 4 processes created, rather than 3 that we expected; and the child process 6282 has also created one process 6284.

Well, let's explain this. The `fork` will create a duplicate of the running process, and the process created will be the child of the original process. Since the child is a replica, it will have the same local variables ,same PC(program counter) and a bunch of other stuff you may see its [man page](http://man7.org/linux/man-pages/man2/fork.2.html) for details.

See the child and the parent processes are almost identical, the child will have the same PC as the parent's, which means, it will continue to execute the code after where `fork` is called. And that is the reason of the "wrong" behavior we have above. The execution flow goes like this.

---

1. Parent 6281 created child 6282, then 6281 & 6282 will continue to execute `pid2 = fork()`
2. 6281 created another child 6283, while child 6282 also created one child of its own, 6284.

---

Then how shall we distinguish the child and the parent? Notice that some pids are 0. After `fork` returned the child and the parent will receive 2 different value. The parent will get the pid of its child, meanwhile the child will have a 0. Also -1 for failed duplicate. And we can work on that.

###Signal and Kill
In this task, we used software interrupt to achieve interprocess communication.
That is, one process listen for a specific signal (interrupt) and then the other one send that signal. These are done by `signal` and `kill` respectively.

The `signal(signum, handler)` will bind the specified signal with id `signum` with the handler function `handler`. And `kill(pid, sig)` will send signal `sig` to process `pid`.

###Design
Then everything is simple, as the task asked us to let the parent listen for keyboard interrupt and then send signal to kill the children. We shall simply let the parent listen to the `SIGINT`, and then bind a handler function that will send signals to the 2 children, the children then wait for the signals to commit suicide.

However, there are few things to note.

###Notes
The guide provided is... well, it made few mistakes. Only on some specific OS the `break` and `delete` will generate the `SIGINT`, mostly it's for `Ctrl+c`. And since we are using `SIGINT`, it's vital to overwrite the orignal handler for it. Or it will cause the processes to terminate, though the outcome is same, it's not what we want.

All in all, it's my code below.
```C
/**
 * Author: Ply_py
 * OSLAB: fork and signal passing
 */
#include <stdio.h>
#include <signal.h>
#include <unistd.h>
#include <sys/types.h>
#include <stdlib.h>

void killer(int);
void suicide(int);

pid_t pid1, pid2;

int
main(int argc, int *argv[])
{
    // make all process of this group ignore the normal soft interrupt
    signal(SIGINT, SIG_IGN);
    // create child process 1
    while (-1 == (pid1 = fork())) continue;

    if (pid1 > 0) { // parent
        // create child process 2
        while (-1 == (pid2 = fork())) continue;

        if (pid2 > 0) { // parent
            printf("Parent: %ld\t Child1: %ld\t Child2: %ld\n",
                (long) getpid(), (long) pid1, (long) pid2);

            signal(SIGINT, killer);
            // wait for death of 2 children
            wait(0);
            wait(0);
            puts("The parent has killed his children");
            exit(EXIT_SUCCESS);
        } else { // child 2
            printf("child 2: %ld created\n", (long) getpid());
            signal(16, suicide);
            while (1) sleep(5);
        }

    } else { // child 1
        printf("child 1: %ld created\n", (long) getpid());
        signal(17, suicide);
        while (1) sleep(5);
    }
    printf("I, %ld can get here\n", (long) getpid());
}

void
killer(int signum)
{
    static counter = 0;
    printf("process %ld catching signal %d\n", (long) getpid(), signum);
    if (0 == counter) {
        kill(pid1, 17); // kill child 1
    } else {
        kill(pid2, 16); // kill child 2
    }
    ++counter;
}

void
suicide(int signum)
{
    printf("Process %ld is committing suicide, by signal %d\n",
        (long) getpid(), signum);
    exit(EXIT_SUCCESS);
}
```

## Pipe
The task 2 demand us to implement another interprocess communication via `pipe`.

Well it's rather easy, as we already got here. Use [pipe](http://man7.org/linux/man-pages/man2/pipe.2.html) to create a communication tunnel, and use that to pass messages. But notice there are 2 process writing to that pipe concurently, so proper lock is needed. Check [lockf](http://man7.org/linux/man-pages/man3/lockf.3.html).

Code here,

```C
/**
 * Author: Ply_py
 * OSLAB: pipe
 */
#include <unistd.h>
#include <signal.h>
#include <stdio.h>
#include <stdlib.h>

#define BUF_SIZE 105
pid_t pid1,pid2;

int
main(int argc, char const *argv[])
{
    int fd[2];
    // fd[0] dest, fd[1] source
    char in_buf[BUF_SIZE];
    char out_buf[BUF_SIZE];

    if (-1 == pipe(fd)) {
        perror("pipe");
        exit(EXIT_FAILURE);
    }

    while (-1 == (pid1 = fork())) continue;

    if (0 == pid1) {//child1
        lockf(fd[1], F_LOCK, 0);
        sprintf(out_buf, "\nChild process 1 is sending a message\n");
        write(fd[1], out_buf, BUF_SIZE);
        lockf(fd[1], F_ULOCK, 0);
        exit(EXIT_SUCCESS);
    } else {
        while (-1 == (pid2 = fork())) continue;

        if (0 == pid2) {// child2
            lockf(fd[1], F_LOCK, 0);
            sprintf(out_buf, "\nChild process 2 is sending a message\n");
            write(fd[1], out_buf, BUF_SIZE);
            lockf(fd[1], F_ULOCK, 0);
            exit(EXIT_SUCCESS);
        } else { // parent
            read(fd[0], in_buf, BUF_SIZE);
            puts(in_buf);
            wait(pid1);
            read(fd[0], in_buf, BUF_SIZE);
            puts(in_buf);
            wait(pid2);
            exit(EXIT_SUCCESS);
        }
    }
    return 0;
}
```
