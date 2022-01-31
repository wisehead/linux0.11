#1.new_block

```cpp
////向设备dev 申请一个逻辑块（盘块，区块）。返回逻辑块号（盘块号）。
// 置位指定逻辑块block 的逻辑块位图比特位。
int new_block(int dev)
{
    struct buffer_head * bh;
    struct super_block * sb;
    int i,j;

// 从设备dev 取超级块，如果指定设备不存在，则出错死机。
    if (!(sb = get_super(dev)))
        panic("trying to get new block from nonexistant device");
// 扫描逻辑块位图，寻找首个0 比特位，寻找空闲逻辑块，获取放置该逻辑块的块号。
    j = 8192;
    for (i=0 ; i<8 ; i++)
        if (bh=sb->s_zmap[i])
            if ((j=find_first_zero(bh->b_data))<8192)
                break;
// 如果全部扫描完还没找到(i>=8 或j>=8192)或者位图所在的缓冲块无效(bh=NULL)则返回0，
// 退出（没有空闲逻辑块）。
    if (i>=8 || !bh || j>=8192)
        return 0;
// 设置新逻辑块对应逻辑块位图中的比特位，若对应比特位已经置位，则出错，死机。
    if (set_bit(j,bh->b_data))
        panic("new_block: bit already set");
// 置对应缓冲区块的已修改标志。如果新逻辑块大于该设备上的总逻辑块数，则说明指定逻辑块在
// 对应设备上不存在。申请失败，返回0，退出。
    bh->b_dirt = 1;
    j += i*8192 + sb->s_firstdatazone-1;
    if (j >= sb->s_nzones)
        return 0;
// 读取设备上的该新逻辑块数据（验证）。如果失败则死机。
    if (!(bh=getblk(dev,j)))
        panic("new_block: cannot get block");
// 新块的引用计数应为1。否则死机。
    if (bh->b_count != 1)
        panic("new block: count is != 1");
// 将该新逻辑块清零，并置位更新标志和已修改标志。然后释放对应缓冲区，返回逻辑块号。
    clear_block(bh->b_data);
    bh->b_uptodate = 1;
    bh->b_dirt = 1;
    brelse(bh);
    return j;
}
```

#2.note

```
注意这里new_block最后返回的bh，count = 0；
这里将磁盘位图置位，然后，获取了一个bh header，ready。可以进行写入了。
但是不明白，为什么要把count=0.
```