#1.write_pipe

```cpp
//// 管道写操作函数。
// 参数inode 是管道对应的i 节点，buf 是数据缓冲区指针，count 是将写入管道的字节数。

write_pipe
// 若将写入的字节计数值count 还大于0，则循环执行以下操作。
--while (count>0) {
// 若当前管道中没有已经满了(size=0)，则唤醒等待该节点的进程，如果已没有读管道者，则向进程
// 发送SIGPIPE 信号，并返回已写入的字节数并退出。若写入0 字节，则返回-1。否则在该i 节点上
// 睡眠，等待管道腾出空间。
        while (!(size=(PAGE_SIZE-1)-PIPE_SIZE(*inode))) {
            wake_up(&inode->i_wait);
            if (inode->i_count != 2) { /* no readers */
                current->signal |= (1<<(SIGPIPE-1));
                return written?written:-1;
            }
            sleep_on(&inode->i_wait);
        }
// 取管道头部到缓冲区末端空间字节数chars。如果其大于还需要写入的字节数count，则令其等于
// count。如果chars 大于当前管道中空闲空间长度size，则令其等于size。
        chars = PAGE_SIZE-PIPE_HEAD(*inode);
        if (chars > count)
            chars = count;
        if (chars > size)
            chars = size;
// 写入字节计数减去此次可写入的字节数chars，并累加已写字节数到written。
        count -= chars;
        written += chars;
// 令size 指向管道数据头部，调整当前管道数据头部指针（前移chars 字节）。
        size = PIPE_HEAD(*inode);
        PIPE_HEAD(*inode) += chars;
        PIPE_HEAD(*inode) &= (PAGE_SIZE-1);
// 从用户缓冲区复制chars 个字节到管道中。对于管道i 节点，其i_size 字段中是管道缓冲块指针。
        while (chars-->0)
            ((char *)inode->i_size)[size++]=get_fs_byte(buf++);
--}
// 唤醒等待该i 节点的进程，返回已写入的字节数，退出。
--wake_up(&inode->i_wait);
--return written;
```