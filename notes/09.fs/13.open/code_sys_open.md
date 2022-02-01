#1.sys_open

```cpp
//// 打开（或创建）文件系统调用函数。
// 参数filename 是文件名，flag 是打开文件标志：只读O_RDONLY、只写O_WRONLY 或读写O_RDWR，
// 以及O_CREAT、O_EXCL、O_APPEND 等其它一些标志的组合，若本函数创建了一个新文件，则mode
// 用于指定使用文件的许可属性，这些属性有S_IRWXU(文件宿主具有读、写和执行权限)、S_IRUSR
// (用户具有读文件权限)、S_IRWXG(组成员具有读、写和执行权限)等等。对于新创建的文件，这些
// 属性只应用于将来对文件的访问，创建了只读文件的打开调用也将返回一个可读写的文件句柄。
// 若操作成功则返回文件句柄(文件描述符)，否则返回出错码。(参见sys/stat.h, fcntl.h)

sys_open
// 将用户设置的模式与进程的模式屏蔽码相与，产生许可的文件模式。
--mode &= 0777 & ~current->umask;
// 搜索进程结构中文件结构指针数组，查找一个空闲项，若已经没有空闲项，则返回出错码。
--for(fd=0 ; fd<NR_OPEN ; fd++)
        if (!current->filp[fd])
            break;

// 设置执行时关闭文件句柄位图，复位对应比特位。
--current->close_on_exec &= ~(1<<fd);
// 令f 指向文件表数组开始处。搜索空闲文件结构项(句柄引用计数为0 的项)，若已经没有空闲
// 文件表结构项，则返回出错码。
--f=0+file_table;
--for (i=0 ; i<NR_FILE ; i++,f++)
        if (!f->f_count) break;

// 让进程的对应文件句柄的文件结构指针指向搜索到的文件结构，并令句柄引用计数递增1。
--(current->filp[fd]=f)->f_count++;
--i=open_namei(filename,flag,mode,&inode)

/* ttys 有些特殊（ttyxx 主号==4，tty 主号==5）*/
// 如果是字符设备文件，那么如果设备号是4 的话，则设置当前进程的tty 号为该i 节点的子设备号。
// 并设置当前进程tty 对应的tty 表项的父进程组号等于进程的父进程组号。
--if (S_ISCHR(inode->i_mode))
----if (MAJOR(inode->i_zone[0])==4) {
            if (current->leader && current->tty<0) {
                current->tty = MINOR(inode->i_zone[0]);
                tty_table[current->tty].pgrp = current->pgrp;
            }
// 否则如果该字符文件设备号是5 的话，若当前进程没有tty，则说明出错，释放i 节点和申请到的
// 文件结构，返回出错码。
----} else if (MAJOR(inode->i_zone[0])==5)
            if (current->tty<0) {
                iput(inode);
                current->filp[fd]=NULL;
                f->f_count=0;
                return -EPERM;
            }       

/* 同样对于块设备文件：需要检查盘片是否被更换 */
// 如果打开的是块设备文件，则检查盘片是否更换，若更换则需要是高速缓冲中对应该设备的所有
// 缓冲块失效。
--if (S_ISBLK(inode->i_mode))
        check_disk_change(inode->i_zone[0]);
// 初始化文件结构。置文件结构属性和标志，置句柄引用计数为1，设置i 节点字段，文件读写指针                                  
// 初始化为0。返回文件句柄。                                                                 
--f->f_mode = inode->i_mode;                                                     
--f->f_flags = flag;                                                             
--f->f_count = 1;                                                                
--f->f_inode = inode;                                                            
--f->f_pos = 0;                                                                  
--return (fd);                                                                                                                                               
                                                                                      
```