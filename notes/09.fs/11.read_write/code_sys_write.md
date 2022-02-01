#1.sys_write

```cpp

sys_write
// 取文件对应的i 节点。若是管道文件，并且是写管道文件模式，则进行写管道操作，若成功则返回
// 写入的字节数，否则返回出错码，退出。
--inode = file->f_inode;
--if (inode->i_pipe)
----return (file->f_mode & 2) ? write_pipe (inode, buf, count) : -EIO;
// 如果是字符型文件，则进行写字符设备操作，返回写入的字符数，退出。
--if (S_ISCHR (inode->i_mode))
----return rw_char (WRITE, inode->i_zone[0], buf, count, &file->f_pos);
// 如果是块设备文件，则进行块设备写操作，并返回写入的字节数，退出。
--if (S_ISBLK (inode->i_mode))
----return block_write (inode->i_zone[0], &file->f_pos, buf, count);
// 若是常规文件，则执行文件写操作，并返回写入的字节数，退出。
--if (S_ISREG (inode->i_mode))
----return file_write (inode, file, buf, count);
```