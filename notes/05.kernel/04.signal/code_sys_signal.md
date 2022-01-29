#1.sys_signal

```cpp
// signal()系统调用。类似于sigaction()。为指定的信号安装新的信号句柄(信号处理程序)。
// 信号句柄可以是用户指定的函数，也可以是SIG_DFL（默认句柄）或SIG_IGN（忽略）。
// 参数signum --指定的信号；handler -- 指定的句柄；restorer –原程序当前执行的地址位置。
// 函数返回原信号句柄。

sys_signal
--
```