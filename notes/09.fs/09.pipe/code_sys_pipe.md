#1.sys_pipe

```cpp
//// 创建管道系统调用函数。
// 在fildes 所指的数组中创建一对文件句柄(描述符)。这对文件句柄指向一管道i 节点。fildes[0]
// 用于读管道中数据，fildes[1]用于向管道中写入数据。
// 成功时返回0，出错时返回-1。

sys_pipe
// 从系统文件表中取两个空闲项（引用计数字段为0 的项），并分别设置引用计数为1。
--j=0;
--for(i=0;j<2 && i<NR_FILE;i++)
        if (!file_table[i].f_count)
            (f[j++]=i+file_table)->f_count++;
            
// 针对上面取得的两个文件结构项，分别分配一文件句柄，并使进程的文件结构指针分别指向这两个
// 文件结构。
--j=0;
--for(i=0;j<2 && i<NR_OPEN;i++)
        if (!current->filp[i]) {
            current->filp[ fd[j]=i ] = f[j];
            j++;
        }
// 申请管道i 节点，并为管道分配缓冲区（1 页内存）。如果不成功，则相应释放两个文件句柄和文
// 件结构项，并返回-1。        
--inode=get_pipe_inode())
// 初始化两个文件结构，都指向同一个i 节点，读写指针都置零。第1 个文件结构的文件模式置为读，                                
// 第2 个文件结构的文件模式置为写。                                                             
--f[0]->f_inode = f[1]->f_inode = inode;                                         
--f[0]->f_pos = f[1]->f_pos = 0;                                                 
--f[0]->f_mode = 1;       /* read */                                             
--f[1]->f_mode = 2;       /* write */                                            
// 将文件句柄数组复制到对应的用户数组中，并返回0，退出。                                                   
--put_fs_long(fd[0],0+fildes);                                                   
--put_fs_long(fd[1],1+fildes);                                                   
                                                                                 
```
