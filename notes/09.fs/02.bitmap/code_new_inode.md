#1.new_inode

```cpp
new_inode
--get_empty_inode
--get_super
// 扫描i 节点位图，寻找首个0 比特位，寻找空闲节点，获取放置该i 节点的节点号。
--j = 8192;
--for (i=0 ; i<8 ; i++)
----if (bh=sb->s_imap[i])
------if ((j=find_first_zero(bh->b_data))<8192)
--------break
// 如果全部扫描完还没找到，或者位图所在的缓冲块无效(bh=NULL)则返回0，退出（没有空闲i 节点）。
--if (!bh || j >= 8192 || j+i*8192 > sb->s_ninodes)
----iput(inode)
----return NULL;
// 置位对应新i 节点的i 节点位图相应比特位，如果已经置位，则出错。
--set_bit(j,bh->b_data)

// 置i 节点位图所在缓冲区已修改标志。
    bh->b_dirt = 1;
// 初始化该i 节点结构。
    inode->i_count=1;       // 引用计数。
    inode->i_nlinks=1;      // 文件目录项链接数。
    inode->i_dev=dev;       // i 节点所在的设备号。
    inode->i_uid=current->euid;     // i 节点所属用户id。
    inode->i_gid=current->egid;     // 组id。
    inode->i_dirt=1;            // 已修改标志置位。
    inode->i_num = j + i*8192;  // 对应设备中的i 节点号。
    inode->i_mtime = inode->i_atime = inode->i_ctime = CURRENT_TIME;    // 设置时间。
    return inode;   // 返回该i 节点指针。
```

#2.get_empty_inode

```cpp
get_empty_inode
--while (inode->i_count)
----for (i = NR_INODE; i ; i--) 
------if (!last_inode->i_count)
--------inode = last_inode;
--------if (!inode->i_dirt && !inode->i_lock)
----------break;//for (i = NR_INODE; i ; i--) 
----if (!inode)
------panic("No free inodes in mem");
----wait_on_inode(inode);
----while (inode->i_dirt)
------write_inode(inode);
------wait_on_inode(inode);
--memset(inode,0,sizeof(*inode));
--inode->i_count = 1;
--return inode;
```