#1.read_inode

```cpp
//// 从设备上读取指定i 节点的信息到内存中（缓冲区中）。

read_inode
--lock_inode(inode);
--sb=get_super(inode->i_dev)	
// 该i 节点所在的逻辑块号= (启动块+超级块) + i 节点位图占用的块数+ 逻辑块位图占用的块数+
// (i 节点号-1)/每块含有的i 节点数。
--block = 2 + sb->s_imap_blocks + sb->s_zmap_blocks +(inode->i_num-1)/INODES_PER_BLOCK;
// 从设备上读取该i 节点所在的逻辑块，并将该inode 指针指向对应i 节点信息。
--bh=bread(inode->i_dev,block)
--*(struct d_inode *)inode = ((struct d_inode *)bh->b_data)[(inode->i_num-1)%INODES_PER_BLOCK];
// 最后释放读入的缓冲区，并解锁该i 节点。
--brelse(bh);
--unlock_inode(inode);
```