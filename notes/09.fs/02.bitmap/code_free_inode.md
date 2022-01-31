#1.free_inode

```cpp
//// 释放指定的i 节点。
// 复位对应i 节点位图比特位。
void free_inode(struct m_inode * inode)
{
    struct super_block * sb;
    struct buffer_head * bh;

    // 如果i 节点指针=NULL，则退出。
    if (!inode)
        return;
// 如果i 节点上的设备号字段为0，说明该节点无用，则用0 清空对应i 节点所占内存区，并返回。
    if (!inode->i_dev) {
        memset(inode,0,sizeof(*inode));
        return;
    }
// 如果此i 节点还有其它程序引用，则不能释放，说明内核有问题，死机。
    if (inode->i_count>1) {
        printk("trying to free inode with count=%d\n",inode->i_count);
        panic("free_inode");
    }
// 如果文件目录项连接数不为0，则表示还有其它文件目录项在使用该节点，
// 不应释放，而应该放回等。
    if (inode->i_nlinks)
        panic("trying to free inode with links");
// 取i 节点所在设备的超级块，测试设备是否存在。
    if (!(sb = get_super(inode->i_dev)))
        panic("trying to free inode on nonexistent device");
// 如果i 节点号=0 或大于该设备上i 节点总数，则出错（0 号i 节点保留没有使用）。
    if (inode->i_num < 1 || inode->i_num > sb->s_ninodes)
        panic("trying to free inode 0 or nonexistant inode");
// 如果该i 节点对应的节点位图不存在，则出错。
    if (!(bh=sb->s_imap[inode->i_num>>13]))
        panic("nonexistent imap in superblock");
// 复位i 节点对应的节点位图中的比特位，如果该比特位已经等于0，则出错。
    if (clear_bit(inode->i_num&8191,bh->b_data))
        printk("free_inode: bit already cleared.\n\r");
// 置i 节点位图所在缓冲区已修改标志，并清空该i 节点结构所占内存区。
    bh->b_dirt = 1;
    memset(inode,0,sizeof(*inode));
}
```