#1.sys_umount

```cpp
//// 卸载文件系统的系统调用函数。
// 参数dev_name 是设备文件名。
sys_umount
// 首先根据设备文件名找到对应的i 节点，并取其中的设备号。
--inode = namei (dev_name)
--dev = inode->i_zone[0];//特殊情况，见sys_mknod()
// 释放设备文件名的i 节点。
--iput (inode);
// 如果设备是根文件系统，则不能被卸载，返回出错号。
--if (dev == ROOT_DEV)
----return -EBUSY;
// 查找i 节点表，看是否有进程在使用该设备上的文件，如果有则返回忙出错码。
--for (inode = inode_table + 0; inode < inode_table + NR_INODE; inode++)
----if (inode->i_dev == dev && inode->i_count)
------return -EBUSY;
// 复位被安装到的i 节点的安装标志，释放该i 节点。
//!!!!!!这里被安装节点和namei返回的特殊节点不是一回事。。。。namei中需要仔细理解。。。
--sb->s_imount->i_mount = 0;
----iput (sb->s_imount);
// 置超级块中被安装i 节点字段为空，并释放设备文件系统的根i 节点，置超级块中被安装系统
// 根i 节点指针为空。
--sb->s_imount = NULL;
--iput (sb->s_isup);
--sb->s_isup = NULL;
// 释放该设备的超级块以及位图占用的缓冲块，并对该设备执行高速缓冲与设备上数据的同步操作。
--put_super (dev);
--sync_dev (dev);
```

#2.note

```
sb->s_isup
sb->s_imount
inode = namei (dev_name)
三者什么关系？？？
```
