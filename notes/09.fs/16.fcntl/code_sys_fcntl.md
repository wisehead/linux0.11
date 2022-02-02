#1.sys_fcntl

```cpp
//// 文件控制系统调用函数。
// 参数fd 是文件句柄，cmd 是操作命令(参见include/fcntl.h，23-30 行)。
int sys_fcntl(unsigned int fd, unsigned int cmd, unsigned long arg)
{
    struct file * filp;

// 如果文件句柄值大于一个进程最多打开文件数NR_OPEN，或者该句柄的文件结构指针为空，则出错，
// 返回出错码并退出。
    if (fd >= NR_OPEN || !(filp = current->filp[fd]))
        return -EBADF;
// 根据不同命令cmd 进行分别处理。
    switch (cmd) {
        case F_DUPFD:   // 复制文件句柄。
            return dupfd(fd,arg);
        case F_GETFD:   // 取文件句柄的执行时关闭标志。
            return (current->close_on_exec>>fd)&1;
        case F_SETFD:   // 设置句柄执行时关闭标志。arg 位0 置位是设置，否则关闭。
            if (arg&1)
                current->close_on_exec |= (1<<fd);
            else
                current->close_on_exec &= ~(1<<fd);
            return 0;
        case F_GETFL:   // 取文件状态标志和访问模式。
            return filp->f_flags;
        case F_SETFL:   // 设置文件状态和访问模式(根据arg 设置添加、非阻塞标志)。
            filp->f_flags &= ~(O_APPEND | O_NONBLOCK);
            filp->f_flags |= arg & (O_APPEND | O_NONBLOCK);
            return 0;
        case F_GETLK:   case F_SETLK:   case F_SETLKW:  // 未实现。
            return -1;
        default:
            return -1;
    }
}

```