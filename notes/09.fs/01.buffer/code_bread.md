#1.bread

```cpp
/*
 * 从设备上读取指定的数据块并返回含有数据的缓冲区。如果指定的块不存在
 * 则返回NULL。
 */
//// 从指定设备上读取指定的数据块。
struct buffer_head * bread(int dev,int block)
{
    struct buffer_head * bh;

// 在高速缓冲中申请一块缓冲区。如果返回值是NULL 指针，表示内核出错，死机。
    if (!(bh=getblk(dev,block)))
        panic("bread: getblk returned NULL\n");
// 如果该缓冲区中的数据是有效的（已更新的）可以直接使用，则返回。
    if (bh->b_uptodate)
        return bh;
// 否则调用ll_rw_block()函数，产生读设备块请求。并等待缓冲区解锁。
    ll_rw_block(READ,bh);
    wait_on_buffer(bh);
// 如果该缓冲区已更新，则返回缓冲区头指针，退出。
    if (bh->b_uptodate)
        return bh;
// 否则表明读设备操作失败，释放该缓冲区，返回NULL 指针，退出。
    brelse(bh);
    return NULL;
}
```