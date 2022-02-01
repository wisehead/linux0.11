#1.sys_close

```cpp
// 关闭文件系统调用函数。
// 参数fd 是文件句柄。
// 成功则返回0，否则返回出错码。
int sys_close(unsigned int fd)
{
    struct file * filp;

// 若文件句柄值大于程序同时能打开的文件数，则返回出错码。
    if (fd >= NR_OPEN)
        return -EINVAL;
// 复位进程的执行时关闭文件句柄位图对应位。
    current->close_on_exec &= ~(1<<fd);
// 若该文件句柄对应的文件结构指针是NULL，则返回出错码。
    if (!(filp = current->filp[fd]))
        return -EINVAL;
// 置该文件句柄的文件结构指针为NULL。
    current->filp[fd] = NULL;
// 若在关闭文件之前，对应文件结构中的句柄引用计数已经为0，则说明内核出错，死机。
    if (filp->f_count == 0)
        panic("Close: file count is 0");
// 否则将对应文件结构的句柄引用计数减1，如果还不为0，则返回0（成功）。若已等于0，说明该
// 文件已经没有句柄引用，则释放该文件i 节点，返回0。
    if (--filp->f_count)
        return (0);
    iput(filp->f_inode);
    return (0);
}

```