#1.sys_mount

```cpp
//// 安装文件系统调用函数。
// 参数dev_name 是设备文件名，dir_name 是安装到的目录名，rw_flag 被安装文件的读写标志。
// 将被加载的地方必须是一个目录名，并且对应的i 节点没有被其它程序占用。

sys_mount
// 首先根据设备文件名找到对应的i 节点，并取其中的设备号。
// 对于块特殊设备文件，设备号在i 节点的i_zone[0]中。
--dev_i = namei (dev_name)
--dev = dev_i->i_zone[0];
// 释放该设备文件的i 节点dev_i。
--iput (dev_i);
// 根据给定的目录文件名找到对应的i 节点dir_i。
--if (!(dir_i = namei (dir_name)))
----return -ENOENT;
// 如果该i 节点的引用计数不为1（仅在这里引用），或者该i 节点的节点号是根文件系统的节点
// 号1，则释放该i 节点，返回出错码。
    if (dir_i->i_count != 1 || dir_i->i_num == ROOT_INO)
    {
        iput (dir_i);
        return -EBUSY;
    }
--sb = read_super (dev)
// 如果将要被安装的文件系统已经安装在其它地方，则释放该i 节点，返回出错码。
--if (sb->s_imount)
    {
        iput (dir_i);
        return -EBUSY;
    }
// 如果将要安装到的i 节点已经安装了文件系统(安装标志已经置位)，则释放该i 节点，返回出错码。
--if (dir_i->i_mount)
    {
        iput (dir_i);
        return -EPERM;
    }  
// 被安装文件系统超级块的“被安装到i 节点”字段指向安装到的目录名的i 节点。
--sb->s_imount = dir_i;
// 设置安装位置i 节点的安装标志和节点已修改标志。/* 注意！这里没有iput(dir_i) */
--dir_i->i_mount = 1;     /* 这将在umount 内操作 */
--dir_i->i_dirt = 1;      /* NOTE! we don't iput(dir_i) */    
```