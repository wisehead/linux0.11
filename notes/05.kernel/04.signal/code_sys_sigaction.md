#1.sys_sigaction

```cpp
// sigaction()系统调用。改变进程在收到一个信号时的操作。signum 是除了SIGKILL 以外的任何
// 信号。[如果新操作(action)不为空]则新操作被安装。如果oldaction 指针不为空，则原操作
// 被保留到oldaction。成功则返回0，否则为-1。

sys_sigaction
--tmp = current->sigaction[signum - 1];
  // 在信号的sigaction 结构中设置新的操作（动作）。
--get_new ((char *) action, (char *) (signum - 1 + current->sigaction));
--// 如果oldaction 指针不为空的话，则将原操作指针保存到oldaction 所指的位置。
--save_old ((char *) &tmp, (char *) oldaction);
--// 如果允许信号在自己的信号句柄中收到，则令屏蔽码为0，否则设置屏蔽本信号。
--if (current->sigaction[signum - 1].sa_flags & SA_NOMASK)
----current->sigaction[signum - 1].sa_mask = 0;
--else
----current->sigaction[signum - 1].sa_mask |= (1 << (signum - 1));
```