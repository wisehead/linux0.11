#1.sys_waitpid

```cpp
//// 系统调用waitpid()。挂起当前进程，直到pid 指定的子进程退出（终止）或者收到要求终止
// 该进程的信号，或者是需要调用一个信号句柄（信号处理程序）。如果pid 所指的子进程早已
// 退出（已成所谓的僵死进程），则本调用将立刻返回。子进程使用的所有资源将释放。
// 如果pid > 0, 表示等待进程号等于pid 的子进程。
// 如果pid = 0, 表示等待进程组号等于当前进程的任何子进程。
// 如果pid < -1, 表示等待进程组号等于pid 绝对值的任何子进程。
// [ 如果pid = -1, 表示等待任何子进程。]
// 若options = WUNTRACED，表示如果子进程是停止的，也马上返回。
// 若options = WNOHANG，表示如果没有子进程退出或终止就马上返回。
// 如果stat_addr 不为空，则就将状态信息保存到那里。

sys_waitpid
--verify_area (stat_addr, 4);
--repeat:
--for (p = &LAST_TASK; p > &FIRST_TASK; --p)
----switch ((*p)->state)
------case TASK_STOPPED:
--------if (!(options & WUNTRACED))
----------continue
--------put_fs_long (0x7f, stat_addr);
------case TASK_ZOMBIE:
--------current->cutime += (*p)->utime;   // 更新当前进程的子进程用户
               current->cstime += (*p)->stime;   // 态和核心态运行时间。
               flag = (*p)->pid;
               code = (*p)->exit_code;   // 取子进程的退出码。
               release (*p);     // 释放该子进程。
               put_fs_long (code, stat_addr);    // 置状态信息为退出码值。
               return flag;      // 退出，返回子进程的pid.
------default:
--------flag = 1;     // 如果子进程不在停止或僵死状态，则flag=1。
--------continue
--if (flag)
---- // 如果子进程没有处于退出或僵死状态
----if (options & WNOHANG)    // 并且options = WNOHANG，则立刻返回。
------return 0;
----current->state = TASK_INTERRUPTIBLE;  // 置当前进程为可中断等待状态。
----schedule ();      // 重新调度。
----if (!(current->signal &= ~(1 << (SIGCHLD - 1))))  // 又开始执行本进程时，
------goto repeat;        // 如果进程没有收到除SIGCHLD 的信号，则还是重复处理。
----else
------return -EINTR;      // 退出，返回出错码。
               
```