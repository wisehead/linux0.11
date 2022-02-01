#1.read_pipe

```cpp
//// 管道读操作函数。
// 参数inode 是管道对应的i 节点，buf 是数据缓冲区指针，count 是读取的字节数。

read_pipe
// 若欲读取的字节计数值count 大于0，则循环执行以下操作。
--while (count>0) {
// 若当前管道中没有数据(size=0)，则唤醒等待该节点的进程，如果已没有写管道者，则返回已读
// 字节数，退出。否则在该i 节点上睡眠，等待信息。
        while (!(size=PIPE_SIZE(*inode))) {
            wake_up(&inode->i_wait);
            if (inode->i_count != 2) /* are there any writers? */
                return read;
            sleep_on(&inode->i_wait);
        }
// 取管道尾到缓冲区末端的字节数chars。如果其大于还需要读取的字节数count，则令其等于count。
// 如果chars 大于当前管道中含有数据的长度size，则令其等于size。
        chars = PAGE_SIZE-PIPE_TAIL(*inode);
        if (chars > count)
            chars = count;
        if (chars > size)
            chars = size;
// 读字节计数减去此次可读的字节数chars，并累加已读字节数。
        count -= chars;
        read += chars;
// 令size 指向管道尾部，调整当前管道尾指针（前移chars 字节）。
        size = PIPE_TAIL(*inode);
        PIPE_TAIL(*inode) += chars;
        PIPE_TAIL(*inode) &= (PAGE_SIZE-1);
// 将管道中的数据复制到用户缓冲区中。对于管道i 节点，其i_size 字段中是管道缓冲块指针。
        while (chars-->0)
            put_fs_byte(((char *)inode->i_size)[size++],buf++);
--}
// 唤醒等待该管道i 节点的进程，并返回读取的字节数。
--wake_up(&inode->i_wait);
--return read;
```