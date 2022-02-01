#1.sys_rmdir

```cpp
//// 系统调用函数- 删除指定名称的目录。
// 参数： name - 目录名(路径名)。
// 返回：返回0 表示成功，否则返回出错号。

sys_rmdir
--dir = dir_namei(name,&namelen,&basename)
// 如果对应路径名上最后的文件名的目录项不存在，则释放包含该目录项的高速缓冲区，释放目录
// 的i 节点，返回文件已经存在出错码，退出。否则dir 是包含要被删除目录名的目录i 节点，de
// 是要被删除目录的目录项结构。
--bh = find_entry(&dir,basename,namelen,&de);
// 取该目录项指明的i 节点。若出错则释放目录的i 节点，并释放含有目录项的高速缓冲区，返回
// 出错号。
--inode = iget(dir->i_dev, de->inode)
// 置该需被删除目录的目录项的i 节点号字段为0，表示该目录项不再使用，并置含有该目录项的高速
// 缓冲区已修改标志，并释放该缓冲区。
    de->inode = 0;
    bh->b_dirt = 1;
    brelse(bh);
// 置被删除目录的i 节点的连接数为0，并置i 节点已修改标志。
    inode->i_nlinks=0;
    inode->i_dirt=1;
// 将包含被删除目录名的目录的i 节点引用计数减1，修改其改变时间和修改时间为当前时间，并置
// 该节点已修改标志。
    dir->i_nlinks--;
    dir->i_ctime = dir->i_mtime = CURRENT_TIME;
    dir->i_dirt=1;
// 最后释放包含要删除目录名的目录i 节点和该要删除目录的i 节点，返回0(成功)。
    iput(dir);
    iput(inode);
```