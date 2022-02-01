#1.sys_mkdir

```cpp
//// 系统调用函数- 创建目录。
// 参数：pathname - 路径名；mode - 目录使用的权限属性。
// 返回：成功则返回0，否则返回出错码。

sys_mkdir
// 如果找不到对应路径名目录的i 节点，则返回出错码。
--dir = dir_namei(pathname,&namelen,&basename)
// 如果最顶端的文件名长度为0，则说明给出的路径名最后没有指定文件名，释放该目录i 节点，返回
// 出错码，退出。
--if (!namelen) {
        iput(dir);
        return -ENOENT;
--}
// 如果对应路径名上最后的文件名的目录项已经存在，则释放包含该目录项的高速缓冲区，释放目录
// 的i 节点，返回文件已经存在出错码，退出。
--bh = find_entry(&dir,basename,namelen,&de);
// 申请一个新的i 节点，如果不成功，则释放目录的i 节点，返回无空间出错码，退出。
--inode = new_inode(dir->i_dev);
// 置该新i 节点对应的文件长度为32(一个目录项的大小)，置节点已修改标志，以及节点的修改时间
// 和访问时间。
--inode->i_size = 32;
--inode->i_dirt = 1;
--inode->i_mtime = inode->i_atime = CURRENT_TIME;
// 为该i 节点申请一磁盘块，并令节点第一个直接块指针等于该块号。如果申请失败，则释放对应目录
// 的i 节点；复位新申请的i 节点连接计数；释放该新的i 节点，返回没有空间出错码，退出。
--inode->i_zone[0]=new_block(inode->i_dev)
// 置该新的i 节点已修改标志。
    inode->i_dirt = 1;
// 读新申请的磁盘块。若出错，则释放对应目录的i 节点；释放申请的磁盘块；复位新申请的i 节点
// 连接计数；释放该新的i 节点，返回没有空间出错码，退出。
--dir_block=bread(inode->i_dev,inode->i_zone[0])
// 令de 指向目录项数据块，置该目录项的i 节点号字段等于新申请的i 节点号，名字字段等于"."。
--de = (struct dir_entry *) dir_block->b_data;
--de->inode=inode->i_num;
--strcpy(de->name,".");
// 然后de 指向下一个目录项结构，该结构用于存放上级目录的节点号和名字".."。
--de++;                         
--de->inode = dir->i_num;       
--strcpy(de->name,"..");        
--inode->i_nlinks = 2;          
                                
// 然后设置该高速缓冲区已修改标志，并释放该缓冲区。    
--dir_block->b_dirt = 1;       
--brelse(dir_block);           
// 初始化设置新i 节点的模式字段，并置该i 节点已修改标志
--inode->i_mode = I_DIRECTORY |
--inode->i_dirt = 1;           

// 在目录中新添加一个目录项，如果失败(包含该目录项的高速缓冲区指针为NULL)，则释放目录的
// i 节点；所申请的i 节点引用连接计数复位，并释放该i 节点。返回出错码，退出。
--bh = add_entry(dir,basename,namelen,&de);

// 令该目录项的i 节点字段等于新i 节点号，置高速缓冲区已修改标志，释放目录和新的i 节点，释放                                          
// 高速缓冲区，最后返回0(成功)。                                                                         
--de->inode = inode->i_num;                                                                 
--bh->b_dirt = 1;                               。                                           
--dir->i_nlinks++;                               (mode & 0777 & ~current->umask);           
--dir->i_dirt = 1;                                                                          
--iput(dir);                                                                                
--iput(inode);                                                                              
--brelse(bh);                                                                               
--return 0;                                                                                 
```