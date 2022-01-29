#1.sys_signal

```cpp
// signal()系统调用。类似于sigaction()。为指定的信号安装新的信号句柄(信号处理程序)。
// 信号句柄可以是用户指定的函数，也可以是SIG_DFL（默认句柄）或SIG_IGN（忽略）。
// 参数signum --指定的信号；handler -- 指定的句柄；restorer –原程序当前执行的地址位置。
// 函数返回原信号句柄。

sys_signal
--tmp.sa_handler = (void (*)(int)) handler;   // 指定的信号处理句柄
--tmp.sa_mask = 0;        // 执行时的信号屏蔽码。
-- tmp.sa_flags = SA_ONESHOT | SA_NOMASK;  // 该句柄只使用1 次后就恢复到默认值，
    // 并允许信号在自己的处理句柄中收到。
--tmp.sa_restorer = (void (*)(void)) restorer;    // 保存返回地址。
--handler = (long) current->sigaction[signum - 1].sa_handler;
--current->sigaction[signum - 1] = tmp;
```

#2.caller

```
main_rename
--sched_init
----set_system_gate (0x80, &system_call);
------_system_call//system_call.s
```