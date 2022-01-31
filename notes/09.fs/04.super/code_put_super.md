#1.put_super

```cpp
//// 释放指定设备的超级块。
// 释放设备所使用的超级块数组项（置s_dev=0），并释放该设备i 节点位图和逻辑块位图所占用
// 的高速缓冲块。如果超级块对应的文件系统是根文件系统，或者其i 节点上已经安装有其它的文件
// 系统，则不能释放该超级块。
void
put_super (int dev)
{
    struct super_block *sb;
//  struct m_inode *inode;
    int i;

// 如果指定设备是根文件系统设备，则显示警告信息“根系统盘改变了，准备生死决战吧”，并返回。
    if (dev == ROOT_DEV)
    {
        printk ("root diskette changed: prepare for armageddon\n\r");
        return;
    }
// 如果找不到指定设备的超级块，则返回。
    if (!(sb = get_super (dev)))
        return;
// 如果该超级块指明本文件系统i 节点上安装有其它的文件系统，则显示警告信息，返回。
    if (sb->s_imount)
    {
        printk ("Mounted disk changed - tssk, tssk\n\r");
        return;
    }
// 找到指定设备的超级块后，首先锁定该超级块，然后置该超级块对应的设备号字段为0，也即即将
// 放弃该超级块。
    lock_super (sb);
    sb->s_dev = 0;
// 然后释放该设备i 节点位图和逻辑块位图在缓冲区中所占用的缓冲块。
    for (i = 0; i < I_MAP_SLOTS; i++)
        brelse (sb->s_imap[i]);
    for (i = 0; i < Z_MAP_SLOTS; i++)
        brelse (sb->s_zmap[i]);
// 最后对该超级块解锁，并返回。
    free_super (sb);
    return;
}
```