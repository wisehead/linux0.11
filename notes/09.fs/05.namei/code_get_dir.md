#1.get_dir

```cpp
/*
 *  get_dir()
 *
 * 该函数根据给出的路径名进行搜索，直到达到最顶端的目录。
 * 如果失败则返回NULL。
 */
//// 搜寻指定路径名的目录。
// 参数：pathname - 路径名。
// 返回：目录的i 节点指针。失败时返回NULL。

get_dir
// 如果用户指定的路径名的第1 个字符是'/'，则说明路径名是绝对路径名。则从根i 节点开始操作。
    if ((c=get_fs_byte(pathname))=='/') {
        inode = current->root;
        pathname++;
// 否则若第一个字符是其它字符，则表示给定的是相对路径名。应从进程的当前工作目录开始操作。
// 则取进程当前工作目录的i 节点。
    } else if (c)
        inode = current->pwd;
// 否则表示路径名为空，出错。返回NULL，退出。
    else
        return NULL;    /* 空的路径名是错误的 */
// 将取得的i 节点引用计数增1。
    inode->i_count++;
    while (1) {
// 若该i 节点不是目录节点，或者没有可进入的访问许可，则释放该i 节点，返回NULL，退出。
        thisname = pathname;
        if (!S_ISDIR(inode->i_mode) || !permission(inode,MAY_EXEC)) {
            iput(inode);
            return NULL;
        }
// 从路径名开始起搜索检测字符，直到字符已是结尾符(NULL)或者是'/'，此时namelen 正好是当前处理
// 目录名的长度。如果最后也是一个目录名，但其后没有加'/'，则不会返回该最后目录的i 节点！
// 比如：/var/log/httpd，将只返回log/目录的i 节点。
        for(namelen=0;(c=get_fs_byte(pathname++))&&(c!='/');namelen++)
            /* nothing */ ;
// 若字符是结尾符NULL，则表明已经到达指定目录，则返回该i 节点指针，退出。
        if (!c)
            return inode;
// 调用查找指定目录和文件名的目录项函数，在当前处理目录中寻找子目录项。如果没有找到，
// 则释放该i 节点，并返回NULL，退出。
        if (!(bh = find_entry(&inode,thisname,namelen,&de))) {
            iput(inode);
            return NULL;
        }
// 取该子目录项的i 节点号inr 和设备号idev，释放包含该目录项的高速缓冲块和该i 节点。
        inr = de->inode;
        idev = inode->i_dev;
        brelse(bh);
        iput(inode);
// 取节点号inr 的i 节点信息，若失败，则返回NULL，退出。否则继续以该子目录的i 节点进行操作。
        if (!(inode = iget(idev,inr)))
            return NULL;
    }
            
```