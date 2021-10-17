6. 另一个重要的行为是I/O完成时要做什么。利用-I IO_RUN_LATER，当I/O完成时，发出它的进程不一定马上运行。相反，当时运行的进程一直运行。当你运行这个进程组合时会发生什么？（./process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_LATER -c -p）系统资源是否被有效利用？

第一个 IO 执行之后，剩下两个 IO 均需要等待占用 100% CPU 的指令执行完。
实际上，剩下的 IO 完全可以先调用，然后利用进程切换后执行 CPU 指令。

正确的解法，使用 `IO_RUN_IMMEDIATE`。
```sh
./process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_IMMEDIATE -c -p
```
