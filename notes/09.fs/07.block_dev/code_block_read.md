#1.block_read

```cpp
//// 数据块读函数- 从指定设备和位置读入指定字节数的数据到高速缓冲中。

block_read
// 由pos 地址换算成开始读写块的块序号block。并求出需读第1 字节在该块中的偏移位置offset。
--int block = *pos >> BLOCK_SIZE_BITS;
--int offset = *pos & (BLOCK_SIZE-1);
// 针对要读入的字节数count，循环执行以下操作，直到全部读入。
--while (count>0) {
// 计算在该块中需读入的字节数。如果需要读入的字节数不满一块，则只需读count 字节。
        chars = BLOCK_SIZE-offset;
        if (chars > count)
            chars = count;
// 读入需要的数据块，并预读下两块数据，如果读操作出错，则返回已读字节数，如果没有读入任何
// 字节，则返回出错号。然后将块号递增1。
        if (!(bh = breada(dev,block,block+1,block+2,-1)))
            return read?read:-EIO;
        block++;
// p 指向从设备读出数据块中需要读取的开始位置。若最后需要读取的数据不足一块，则需从块开始
// 读取所需的字节，因此这里需将offset 置零。
        p = offset + bh->b_data;
        offset = 0;
// 将文件中偏移指针前移已读出字节数chars。累加已读字节数。传送计数值减去此次已传送字节数。
        *pos += chars;
        read += chars;
        count -= chars;
// 从高速缓冲区中p 指向的开始位置复制chars 字节数据到用户缓冲区，并释放该高速缓冲区。
        while (chars-->0)
            put_fs_byte(*(p++),buf++);
        brelse(bh);
--}
--return read;        // 返回已读取的字节数，正常退出。
```