
# Overview

This program, called process-run.py, allows you to see how the state of a
process state changes as it runs on a CPU. As described in the chapter, 
processes can be in a few different states:

这个名为 process-run.py 的程序，可以使你看到进程状态是如何在 CPU 上发生变化的。
如本章所述，进程可以分为不同的状态：

```sh
RUNNING - the process is using the CPU right now
READY   - the process could be using the CPU right now
          but (alas) some other process is
WAITING - the process is waiting on I/O
          (e.g., it issued a request to a disk)
DONE    - the process is finished executing
```

```sh
运行态 - 进程正在使用 CPU
就绪态 - 进程可以被 CPU 立即使用，但有其他进程在
阻塞态 - 进程等待 I/O（例如，它向磁盘发起请求）
完成态 - 进程已经执行完毕
```

In this homework, we'll see how these process states change as a program
runs, and thus learn a little bit better how these things work.

在本次作业中，我们将观察当程序运行时进程状态如何变更，从而更好地理解它们的工作原理。

To run the program and get its options, do this:

若要运行该程序并且获取可选项，请这样做：

```sh
prompt> ./process-run.py -h
```

If this doesn't work, type `python` before the command, like this:

如果这不生效，在执行命令前输入 `pyhton` ，像这样

```sh
prompt> python process-run.py -h
```


What you should see is this:

你将会看到的是:

```sh
Usage: process-run.py [options]

Options:
  -h, --help            show this help message and exit
  -s SEED, --seed=SEED  the random seed
  -l PROCESS_LIST, --processlist=PROCESS_LIST
                        a comma-separated list of processes to 
                        run, in the
                        form X1:Y1,X2:Y2,... where X is the number of
                        instructions that process should run, and Y the
                        chances (from 0 to 100) that an instruction will use
                        the CPU or issue an IO
                        一个类似 X1:Y1,X2:Y2,.. 这样以逗号为分隔的要运行的进程列表，X 表示指令运行的数量，Y 表明指令的几率（从 0 到 100）使用 CPU 或调用 IO。

  -L IO_LENGTH, --iolength=IO_LENGTH
                        how long an IO takes
                        IO 执行了多少时间
  -S PROCESS_SWITCH_BEHAVIOR, --switch=PROCESS_SWITCH_BEHAVIOR
                        when to switch between processes: SWITCH_ON_IO,
                        SWITCH_ON_END
                        何时切换两个进程:
                        SWITCH_ON_IO，
                        SWITCH_ON_END
  -I IO_DONE_BEHAVIOR, --iodone=IO_DONE_BEHAVIOR
                        type of behavior when IO ends: IO_RUN_LATER,
                        IO_RUN_IMMEDIATE
                        当 IO 结束后输出行为：
                        IO_RUN_LATER,
                        IO_RUN_IMMEDIATE
  -c                    compute answers for me
                        告知答案
  -p, --printstats      print statistics at end; only useful with -c flag
                        (otherwise stats are not printed)
                        在结尾处打印统计数据；只有在使用 -c 标志下才有用（否则不打印统计状态）                     
```

```sh
Usage: process-run.py [options]

Options:
  -h, --help            show this help message and exit
  -s SEED, --seed=SEED  the random seed
  -l PROCESS_LIST, --processlist=PROCESS_LIST
                        a comma-separated list of processes to run, in the
                        form X1:Y1,X2:Y2,... where X is the number of
                        instructions that process should run, and Y the
                        chances (from 0 to 100) that an instruction will use
                        the CPU or issue an IO
  -L IO_LENGTH, --iolength=IO_LENGTH
                        how long an IO takes
  -S PROCESS_SWITCH_BEHAVIOR, --switch=PROCESS_SWITCH_BEHAVIOR
                        when to switch between processes: SWITCH_ON_IO,
                        SWITCH_ON_END
  -I IO_DONE_BEHAVIOR, --iodone=IO_DONE_BEHAVIOR
                        type of behavior when IO ends: IO_RUN_LATER,
                        IO_RUN_IMMEDIATE
  -c                    compute answers for me
  -p, --printstats      print statistics at end; only useful with -c flag
                        (otherwise stats are not printed)
```

The most important option to understand is the PROCESS_LIST (as specified by
the -l or --processlist flags) which specifies exactly what each running
program (or 'process') will do. A process consists of instructions, and each
instruction can just do one of two things: 
- use the CPU 
- issue an IO (and wait for it to complete)

最重要的一项是理解 PROCESS_LIST（由
或 --l 或 --processlist 来标志），它指定了每个运行的程序（或进程）会做的事情，一个进程由指令组成，而每个指令只能做两件事中的一个：
- 使用 CPU
- 调用 IO(等待它完成)

When a process uses the CPU (and does no IO at all), it should simply
alternate between RUNNING on the CPU or being READY to run. For example, here
is a simple run that just has one program being run, and that program only
uses the CPU (it does no IO).

当一个进程使用 CPU（完全没有 IO 操作)，它应该简单地在 CPU 的运行态和就绪态之间交替运行。
比如，这里一个样例，只有一个程序在运行，而在这个程序只使用 CPU (没有 IO 操作)

```sh
prompt> ./process-run.py -l 5:100 
Produce a trace of what would happen when you run these processes:
Process 0
  cpu
  cpu
  cpu
  cpu
  cpu

Important behaviors:
  System will switch when the current process is FINISHED or ISSUES AN IO
  After IOs, the process issuing the IO will run LATER (when it is its turn)

prompt> 
```

Here, the process we specified is "5:100" which means it should consist of 5
instructions, and the chances that each instruction is a CPU instruction are
100%.

在这里，我们指定 "5:100" 的进程表示它包含 5 条指令，并且每条指令是 CPU 指令的几率是 100%。

You can see what happens to the process by using the -c flag, which computes the
answers for you:

你可以通过使用 -c 标记来查看进程发生了什么，它会为你计算出答案。

```sh
prompt> ./process-run.py -l 5:100 -c
Time     PID: 0        CPU        IOs
  1     RUN:cpu          1
  2     RUN:cpu          1
  3     RUN:cpu          1
  4     RUN:cpu          1
  5     RUN:cpu          1
```

This result is not too interesting: the process is simple in the RUN state and
then finishes, using the CPU the whole time and thus keeping the CPU busy the
entire run, and not doing any I/Os.

结论不算太有趣：进程非常简单地开始然后结束，整段时间都在使用 CPU 以使其处于忙碌地运行，
并且不做任何 I/O 操作。

Let's make it slightly more complex by running two processes:

让我们运行两个进程来让例子变得稍微复杂一些：

```sh
prompt> ./process-run.py -l 5:100,5:100
Produce a trace of what would happen when you run these processes:
Process 0
  cpu
  cpu
  cpu
  cpu
  cpu

Process 1
  cpu
  cpu
  cpu
  cpu
  cpu

Important behaviors:
  Scheduler will switch when the current process is FINISHED or ISSUES AN IO
  After IOs, the process issuing the IO will run LATER (when it is its turn)
```

In this case, two different processes run, each again just using the CPU. What
happens when the operating system runs them? Let's find out:

在本例中，两个不同的进程运行，每个进程仅使用 CPU。 当操作系统运行它们时，会发生什么？
我们一起来看：

```sh
prompt> ./process-run.py -l 5:100,5:100 -c
Time     PID: 0     PID: 1        CPU        IOs
  1     RUN:cpu      READY          1
  2     RUN:cpu      READY          1
  3     RUN:cpu      READY          1
  4     RUN:cpu      READY          1
  5     RUN:cpu      READY          1
  6        DONE    RUN:cpu          1
  7        DONE    RUN:cpu          1
  8        DONE    RUN:cpu          1
  9        DONE    RUN:cpu          1
 10        DONE    RUN:cpu          1
```

As you can see above, first the process with "process ID" (or "PID") 0 runs,
while process 1 is READY to run but just waits until 0 is done. When 0 is
finished, it moves to the DONE state, while 1 runs. When 1 finishes, the trace
is done.

如你所见，第一个进程为"process ID"(或者是 "PID") 0 运行时，进程1 准备就绪。但需要等待直到 0 完成。当进程0 完成，它进入 DONE 状态，此时 进程1 开始运行。当 进程1 结束后，调用栈结束。

Let's look at one more example before getting to some questions. In this
example, the process just issues I/O requests. We specify here that I/Os take 5
time units to complete with the flag -L.

在谈一些问题之前，我们再看一个例子。在本例中，进程只发起 I/O 请求。我们指定 I/O 为 5 单位时间，通过 L 标记来完成。

```sh
prompt> ./process-run.py -l 3:0 -L 5
Produce a trace of what would happen when you run these processes:
Process 0
  io-start
  io-start
  io-start

Important behaviors:
  System will switch when the current process is FINISHED or ISSUES AN IO
  After IOs, the process issuing the IO will run LATER (when it is its turn)
```

What do you think the execution trace will look like? Let's find out:

你认为执行栈应该是怎样的？让我们一探究竟：

```sh
prompt> ./process-run.py -l 3:0 -L 5 -c
Time     PID: 0        CPU        IOs
  1  RUN:io-start          1
  2     WAITING                     1
  3     WAITING                     1
  4     WAITING                     1
  5     WAITING                     1
  6* RUN:io-start          1
  7     WAITING                     1
  8     WAITING                     1
  9     WAITING                     1
 10     WAITING                     1
 11* RUN:io-start          1
 12     WAITING                     1
 13     WAITING                     1
 14     WAITING                     1
 15     WAITING                     1
 16*       DONE
```

As you can see, the program just issues three I/Os. When each I/O is issued,
the process moves to a WAITING state, and while the device is busy servicing
the I/O, the CPU is idle.

如你所见，这个程序只发起了 3 个 I/O。 当每个 I/O 发出时，进程会转移到等待状态，同时设备也忙于服务 I/O，CPU 是空闲的。

Let's print some stats (run the same command as above, but with the -p flag)
to see some overall behaviors: 

我们打印一些状态（运行和上面一样的命令，但需要加上 -p 标记）来看观察一些整体行为:

```sh
Stats: Total Time 16
Stats: CPU Busy 3 (25.00%)
Stats: IO Busy  12 (75.00%)
```

As you can see, the trace took 16 clock ticks to run, but the CPU was only
busy less than 20% of the time. The IO device, on the other hand, was quite
busy. In general, we'd like to keep all the devices busy, as that is a better
use of resources.

如你所见，调用栈执行需要 16 个时钟计数，但是 CPU 仅忙碌不到 20% 的时间。另外一方面，IO 设备确实相当忙碌。通常，我们期望保持所有设备忙碌，这样能更好地利用资源。

There are a few other important flags:
```sh
  -s SEED, --seed=SEED  the random seed  
    this gives you way to create a bunch of different jobs randomly

  -L IO_LENGTH, --iolength=IO_LENGTH
    this determines how long IOs take to complete (default is 5 ticks)
    
  -S PROCESS_SWITCH_BEHAVIOR, --switch=PROCESS_SWITCH_BEHAVIOR
                        when to switch between processes: SWITCH_ON_IO, SWITCH_ON_END
    this determines when we switch to another process:
    - SWITCH_ON_IO, the system will switch when a process issues an IO
    - SWITCH_ON_END, the system will only switch when the current process is done 

  -I IO_DONE_BEHAVIOR, --iodone=IO_DONE_BEHAVIOR
                        type of behavior when IO ends: IO_RUN_LATER, IO_RUN_IMMEDIATE
    this determines when a process runs after it issues an IO:
    - IO_RUN_IMMEDIATE: switch to this process right now
    - IO_RUN_LATER: switch to this process when it is natural to 
      (e.g., depending on process-switching behavior)
```

Now go answer the questions at the back of the chapter to learn more.

现在，尝试去回答章节后的问题巩固练习。
