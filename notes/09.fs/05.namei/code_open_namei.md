#1.open_namei

```cpp
/*
 *  open_namei()
 *
 * open()所使用的namei 函数- 这其实几乎是完整的打开文件程序。
 */
//// 文件打开namei 函数。
// 参数：pathname - 文件路径名；flag - 文件打开标志；mode - 文件访问许可属性；
// 返回：成功返回0，否则返回出错码；res_inode - 返回的对应文件路径名的的i 节点指针。

open_namei
--// 根据路径名寻找到对应的i 节点，以及最顶端文件名及其长度。
--dir = dir_namei(pathname,&namelen,&basename)
// 在dir 节点对应的目录中取文件名对应的目录项结构de 和该目录项所在的高速缓冲区。
--bh = find_entry(&dir,basename,namelen,&de);
--if (!bh)
// 如果不是创建文件，则释放该目录的i 节点，返回出错号退出。
----if (!(flag & O_CREAT)) {
            iput(dir);
            return -ENOENT;
----}
// 在目录节点对应的设备上申请一个新i 节点，若失败，则释放目录的i 节点，并返回没有空间出错码。
----inode = new_inode(dir->i_dev);
// 否则使用该新i 节点，对其进行初始设置：置节点的用户id；对应节点访问模式；置已修改标志。
----inode->i_uid = current->euid;
----inode->i_mode = mode;
----inode->i_dirt = 1;
// 然后在指定目录dir 中添加一新目录项。
----bh = add_entry(dir,basename,namelen,&de);        
// 初始设置该新目录项：置i 节点号为新申请到的i 节点的号码；并置高速缓冲区已修改标志。然后
// 释放该高速缓冲区，释放目录的i 节点。返回新目录项的i 节点指针，退出。
----de->inode = inode->i_num;
----bh->b_dirt = 1;
----brelse(bh);
----iput(dir);
----*res_inode = inode;
----return 0;
--//end of if (!bh)

// 若上面在目录中取文件名对应的目录项结构操作成功(也即bh 不为NULL)，取出该目录项的i 节点号
// 和其所在的设备号，并释放该高速缓冲区以及目录的i 节点。
    inr = de->inode;
    dev = dir->i_dev;
    brelse(bh);
    iput(dir);
// 如果取该目录项对应i 节点的操作失败，则返回访问出错码，退出。    
--inode=iget(dev,inr)
// 更新该i 节点的访问时间字段为当前时间。
--inode->i_atime = CURRENT_TIME;
// 如果设立了截0 标志，则将该i 节点的文件长度截为0。
--if (flag & O_TRUNC)
----truncate(inode);
// 最后返回该目录项i 节点的指针，并返回0（成功）。
--*res_inode = inode;
```