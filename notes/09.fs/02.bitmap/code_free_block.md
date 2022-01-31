#1.free_block


```
//// 释放设备dev 上数据区中的逻辑块block。
// 复位指定逻辑块block 的逻辑块位图比特位。
// 参数：dev 是设备号，block 是逻辑块号（盘块号）。
void free_block(int dev, int block)
{
    struct super_block * sb;
    struct buffer_head * bh;

// 取指定设备dev 的超级块，如果指定设备不存在，则出错死机。
    if (!(sb = get_super(dev)))
        panic("trying to free block on nonexistent device");
// 若逻辑块号小于首个逻辑块号或者大于设备上总逻辑块数，则出错，死机。
    if (block < sb->s_firstdatazone || block >= sb->s_nzones)
        panic("trying to free block not in datazone");
// 从hash 表中寻找该块数据。若找到了则判断其有效性，并清已修改和更新标志，释放该数据块。
// 该段代码的主要用途是如果该逻辑块当前存在于高速缓冲中，就释放对应的缓冲块。
    bh = get_hash_table(dev,block);
    if (bh) {
        if (bh->b_count != 1) {
            printk("trying to free block (%04x:%d), count=%d\n",
                dev,block,bh->b_count);
            return;
        }
        bh->b_dirt=0;       // 复位脏（已修改）标志位。
        bh->b_uptodate=0;   // 复位更新标志。
        brelse(bh);
    }
// 计算block 在数据区开始算起的数据逻辑块号（从1 开始计数）。然后对逻辑块(区块)位图进行操作，
// 复位对应的比特位。若对应比特位原来即是0，则出错，死机。
    block -= sb->s_firstdatazone - 1 ;
    if (clear_bit(block&8191,sb->s_zmap[block/8192]->b_data)) {
        printk("block (%04x:%d) ",dev,block+sb->s_firstdatazone-1);
        panic("free_block: bit already cleared");
    }
    // 置相应逻辑块位图所在缓冲区已修改标志。
    sb->s_zmap[block/8192]->b_dirt = 1;
}
```

#2.note

由于函数free_block中没有将count--
因此caller需要配合调用brelse，做count--操作。