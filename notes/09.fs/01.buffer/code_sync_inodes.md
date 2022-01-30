#1.sync_inodes

```cpp
//// 同步所有i 节点。
// 同步内存与设备上的所有i 节点信息。
sync_inodes
--for(i=0 ; i<NR_INODE ; i++,inode++)
----wait_on_inode(inode);
----if (inode->i_dirt && !inode->i_pipe)
------write_inode(inode);
--------sb=get_super(inode->i_dev)

```

#2. write_inode

```
write_inode
--sb=get_super(inode->i_dev)
// 该i 节点所在的逻辑块号= (启动块+超级块) + i 节点位图占用的块数+ 逻辑块位图占用的块数+
// (i 节点号-1)/每块含有的i 节点数。
--block = 2 + sb->s_imap_blocks + sb->s_zmap_blocks +(inode->i_num-1)/INODES_PER_BLOCK;
// 从设备上读取该i 节点所在的逻辑块。
--bh=bread(inode->i_dev,block)
// 将该i 节点信息复制到逻辑块对应该i 节点的项中。
--((struct d_inode *)bh->b_data)[(inode->i_num-1)%INODES_PER_BLOCK] = *(struct d_inode *)inode;
// 置缓冲区已修改标志，而i 节点修改标志置零。然后释放该含有i 节点的缓冲区，并解锁该i 节点。
    bh->b_dirt=1;
    inode->i_dirt=0;
    brelse(bh);
```