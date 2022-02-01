#1.block_write

```cpp
//// 数据块写函数- 向指定设备从给定偏移处写入指定长度字节数据。
// 参数：dev - 设备号；pos - 设备文件中偏移量指针；buf - 用户地址空间中缓冲区地址；
//       count - 要传送的字节数。
// 对于内核来说，写操作是向高速缓冲区中写入数据，什么时候数据最终写入设备是由高速缓冲管理
// 程序决定并处理的。另外，因为设备是以块为单位进行读写的，因此对于写开始位置不处于块起始
// 处时，需要先将开始字节所在的整个块读出，然后将需要写的数据从写开始处填写满该块，再将完
// 整的一块数据写盘（即交由高速缓冲程序去处理）。

block_write

// 针对要写入的字节数count，循环执行以下操作，直到全部写入。
--while (count > 0) {
// 计算在该块中可写入的字节数。如果需要写入的字节数填不满一块，则只需写count 字节。
        chars = BLOCK_SIZE - offset;
        if (chars > count)
            chars=count;
// 如果正好要写1 块数据，则直接申请1 块高速缓冲块，否则需要读入将被修改的数据块，并预读
// 下两块数据，然后将块号递增1。
        if (chars == BLOCK_SIZE)
            bh = getblk(dev,block);
        else
            bh = breada(dev,block,block+1,block+2,-1);
        block++;
// 如果缓冲块操作失败，则返回已写字节数，如果没有写入任何字节，则返回出错号（负数）。
        if (!bh)
            return written?written:-EIO;
// p 指向读出数据块中开始写的位置。若最后写入的数据不足一块，则需从块开始填写（修改）所需
// 的字节，因此这里需置offset 为零。
        p = offset + bh->b_data;
        offset = 0;
// 将文件中偏移指针前移已写字节数。累加已写字节数chars。传送计数值减去此次已传送字节数。
        *pos += chars;
        written += chars;
        count -= chars;
// 从用户缓冲区复制chars 字节到p 指向的高速缓冲区中开始写入的位置。
        while (chars-- > 0)
            *(p++) = get_fs_byte(buf++);
// 置该缓冲区块已修改标志，并释放该缓冲区（也即该缓冲区引用计数递减1）。
        bh->b_dirt = 1;
        brelse(bh);
--}
```