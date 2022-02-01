#1.sys_mknod

```cpp
//// 系统调用函数- 创建一个特殊文件或普通文件节点(node)。
// 创建名称为filename，由mode 和dev 指定的文件系统节点(普通文件、设备特殊文件或命名管道)。
// 参数：filename - 路径名；mode - 指定使用许可以及所创建节点的类型；dev - 设备号。
// 返回：成功则返回0，否则返回出错码。

sys_mknod
--dir = dir_namei(filename,&namelen,&basename)
// 如果对应路径名上最后的文件名的目录项已经存在，则释放包含该目录项的高速缓冲区，释放目录
// 的i 节点，返回文件已经存在出错码，退出。
--bh = find_entry(&dir,basename,namelen,&de);
// 申请一个新的i 节点，如果不成功，则释放目录的i 节点，返回无空间出错码，退出。
--inode = new_inode(dir->i_dev);
// 设置该i 节点的属性模式。如果要创建的是块设备文件或者是字符设备文件，则令i 节点的直接块
// 指针0 等于设备号。
--inode->i_mode = mode;
--if (S_ISBLK(mode) || S_ISCHR(mode))
----inode->i_zone[0] = dev;
// 设置该i 节点的修改时间、访问时间为当前时间。
--inode->i_mtime = inode->i_atime = CURRENT_TIME;
--inode->i_dirt = 1;
// 在目录中新添加一个目录项，如果失败(包含该目录项的高速缓冲区指针为NULL)，则释放目录的
// i 节点；所申请的i 节点引用连接计数复位，并释放该i 节点。返回出错码，退出。
--bh = add_entry(dir,basename,namelen,&de);
// 令该目录项的i 节点字段等于新i 节点号，置高速缓冲区已修改标志，释放目录和新的i 节点，
// 释放高速缓冲区，最后返回0(成功)。
--de->inode = inode->i_num;        
--bh->b_dirt = 1;                  
--iput(dir);                       
--iput(inode);                     
--brelse(bh);                      
--return 0;                        

```