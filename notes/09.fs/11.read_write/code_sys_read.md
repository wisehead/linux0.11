#1.sys_read

```cpp
//// 读文件系统调用函数。
// 参数fd 是文件句柄，buf 是缓冲区，count 是欲读字节数。

sys_read
// 验证存放数据的缓冲区内存限制。
--verify_area (buf, count);
// 取文件对应的i 节点。若是管道文件，并且是读管道文件模式，则进行读管道操作，若成功则返回
// 读取的字节数，否则返回出错码，退出。
--inode = file->f_inode;
--if (inode->i_pipe)
----return (file->f_mode & 1) ? read_pipe (inode, buf, count) : -EIO;
// 如果是字符型文件，则进行读字符设备操作，返回读取的字符数。
--if (S_ISCHR (inode->i_mode))
----return rw_char (READ, inode->i_zone[0], buf, count, &file->f_pos);
// 如果是块设备文件，则执行块设备读操作，并返回读取的字节数。
--if (S_ISBLK (inode->i_mode))
--return block_read (inode->i_zone[0], &file->f_pos, buf, count);

// 如果是目录文件或者是常规文件，则首先验证读取数count 的有效性并进行调整（若读取字节数加上
// 文件当前读写指针值大于文件大小，则重新设置读取字节数为文件长度-当前读写指针值，若读取数
// 等于0，则返回0 退出），然后执行文件读操作，返回读取的字节数并退出。
--if (S_ISDIR (inode->i_mode) || S_ISREG (inode->i_mode))
--{
        if (count + file->f_pos > inode->i_size)
            count = inode->i_size - file->f_pos;
        if (count <= 0)
            return 0;
        return file_read (inode, file, buf, count);
--}
```