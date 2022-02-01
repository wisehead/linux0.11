#1.namei

```cpp
/*
 *  namei()
 *
 * 该函数被许多简单的命令用于取得指定路径名称的i 节点。open、link 等则使用它们
 * 自己的相应函数，但对于象修改模式'chmod'等这样的命令，该函数已足够用了。
 */
//// 取指定路径名的i 节点。
// 参数：pathname - 路径名。
// 返回：对应的i 节点。

namei
// 首先查找指定路径的最顶层目录的目录名及其i 节点，若不存在，则返回NULL，退出。
--dir = dir_nam// 在返回的顶层目录中寻找指定文件名的目录项的i 节点。因为如果最后也是一个目录名，但其后没
// 有加'/'，则不会返回该最后目录的i 节点！比如：/var/log/httpd，将只返回log/目录的i 节点。
// 因此dir_namei()将不以'/'结束的最后一个名字当作一个文件名来看待。因此这里需要单独对这种
// 情况使用寻找目录项i 节点函数find_entry()进行处理。
--bh = find_entry(&dir,basename,namelen,&de);
// 取该目录项的i 节点号和目录的设备号，并释放包含该目录项的高速缓冲区以及目录i 节点。
    inr = de->inode;
    dev = dir->i_dev;
    brelse(bh);
    iput(dir);
// 取对应节号的i 节点，修改其被访问时间为当前时间，并置已修改标志。最后返回该i 节点指针。
    dir=iget(dev,inr);
    if (dir) {
        dir->i_atime=CURRENT_TIME;
        dir->i_dirt=1;
    }
    return dir;
```