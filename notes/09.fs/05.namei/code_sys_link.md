#1.sys_link

```cpp
//// 系统调用函数- 为文件建立一个文件名。
// 为一个已经存在的文件创建一个新连接(也称为硬连接- hard link)。
// 参数：oldname - 原路径名；newname - 新的路径名。
// 返回：若成功则返回0，否则返回出错号。

sys_link
// 取原文件路径名对应的i 节点oldinode。如果为0，则表示出错，返回出错号。
--oldinode=namei(oldname);
// 查找新路径名的最顶层目录的i 节点，并返回最后的文件名及其长度。如果目录的i 节点没有找到，
// 则释放原路径名的i 节点，返回出错号。
--dir = dir_namei(newname,&namelen,&basename);
// 查询该新路径名是否已经存在，如果存在，则也不能建立连接，于是释放包含该已存在目录项的
// 高速缓冲区，释放新路径名目录的i 节点和原路径名的i 节点，返回出错号。
--bh = find_entry(&dir,basename,namelen,&de);
// 在新目录中添加一个目录项。若失败则释放该目录的i 节点和原路径名的i 节点，返回出错号。
--bh = add_entry(dir,basename,namelen,&de);
// 否则初始设置该目录项的i 节点号等于原路径名的i 节点号，并置包含该新添目录项的高速缓冲区                                
// 已修改标志，释放该缓冲区，释放目录的i 节点。                                                      
--de->inode = oldinode->i_num;                                                  
--bh->b_dirt = 1;                                                               
--brelse(bh);                                                                   
--iput(dir);                                                                    
// 将原节点的应用计数加1，修改其改变时间为当前时间，并设置i 节点已修改标志，最后释放原                                  
// 路径名的i 节点，并返回0(成功)。                                                           
--oldinode->i_nlinks++;                                                         
--oldinode->i_ctime = CURRENT_TIME;                                             
--oldinode->i_dirt = 1;                                                         
--iput(oldinode);                                                               
--return 0;                                                                     

```