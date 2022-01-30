#1.sys_kill

```cpp
/*
* 为了向进程组等发送信号，XXX 需要检查许可。kill()的许可机制非常巧妙!
*/
//// kill()系统调用可用于向任何进程或进程组发送任何信号。
// 如果pid 值>0，则信号被发送给pid。
// 如果pid=0，那么信号就会被发送给当前进程的进程组中的所有进程。
// 如果pid=-1，则信号sig 就会发送给除第一个进程外的所有进程。
// 如果pid < -1，则信号sig 将发送给进程组-pid 的所有进程。
// 如果信号sig 为0，则不发送信号，但仍会进行错误检查。如果成功则返回0。
int sys_kill (int pid, int sig)
{
  struct task_struct **p = NR_TASKS + task;
  int err, retval = 0;

  if (!pid)
    while (--p > &FIRST_TASK)
    {
        if (*p && (*p)->pgrp == current->pid)
          if (err = send_sig (sig, *p, 1))
            retval = err;
    }
  else if (pid > 0)
    while (--p > &FIRST_TASK)
    {
    if (*p && (*p)->pid == pid)
      if (err = send_sig (sig, *p, 0))
        retval = err;
    }
  else if (pid == -1)
    while (--p > &FIRST_TASK)
      if (err = send_sig (sig, *p, 0))
        retval = err;
      else
        while (--p > &FIRST_TASK)
          if (*p && (*p)->pgrp == -pid)
            if (err = send_sig (sig, *p, 0))
              retval = err;
  return retval;
}
```

#2.caller

```

```