#1.read_super

```cpp
//// 从设备上读取超级块到缓冲区中。
// 如果该设备的超级块已经在高速缓冲中并且有效，则直接返回该超级块的指针。
read_super
// 首先检查该设备是否可更换过盘片（也即是否是软盘设备），如果更换过盘，则高速缓冲区有关该
// 设备的所有缓冲块均失效，需要进行失效处理（释放原来加载的文件系统）。
--check_disk_change (dev);
// 如果该设备的超级块已经在高速缓冲中，则直接返回该超级块的指针。
--s = get_super (dev)
// 否则，首先在超级块数组中找出一个空项(也即其s_dev=0 的项)。如果数组已经占满则返回空指针。
--for (s = 0 + super_block;; s++)
----if (s >= NR_SUPER + super_block)
------return NULL;
----if (!s->s_dev)
------break;
// 找到超级块空项后，就将该超级块用于指定设备，对该超级块的内存项进行部分初始化。
--s->s_dev = dev;
  s->s_isup = NULL;
  s->s_imount = NULL;
  s->s_time = 0;
  s->s_rd_only = 0;
  s->s_dirt = 0;
// 然后锁定该超级块，并从设备上读取超级块信息到bh 指向的缓冲区中。如果读超级块操作失败，
// 则释放上面选定的超级块数组中的项，并解锁该项，返回空指针退出。
--lock_super (s);
--bh = bread (dev, 1)
// 将设备上读取的超级块信息复制到超级块数组相应项结构中。并释放存放读取信息的高速缓冲块。
--*((struct d_super_block *) s) = *((struct d_super_block *) bh->b_data);
--brelse (bh);
// 下面开始读取设备上i 节点位图和逻辑块位图数据。首先初始化内存超级块结构中位图空间。
    for (i = 0; i < I_MAP_SLOTS; i++)
        s->s_imap[i] = NULL;
    for (i = 0; i < Z_MAP_SLOTS; i++)
        s->s_zmap[i] = NULL;
// 然后从设备上读取i 节点位图和逻辑块位图信息，并存放在超级块对应字段中。
    block = 2;
    for (i = 0; i < s->s_imap_blocks; i++)
        if (s->s_imap[i] = bread (dev, block))
            block++;
        else
            break;
    for (i = 0; i < s->s_zmap_blocks; i++)
        if (s->s_zmap[i] = bread (dev, block))
            block++;
        else
            break;
// 否则一切成功。对于申请空闲i 节点的函数来讲，如果设备上所有的i 节点已经全被使用，则查找
// 函数会返回0 值。因此0 号i 节点是不能用的，所以这里将位图中的最低位设置为1，以防止文件
// 系统分配0 号i 节点。同样的道理，也将逻辑块位图的最低位设置为1。
--s->s_imap[0]->b_data[0] |= 1;
--s->s_zmap[0]->b_data[0] |= 1;
// 解锁该超级块，并返回超级块指针。
--free_super (s);        
--return s;    
```

#2.caller

```
- sys_mount
- mount_root
```