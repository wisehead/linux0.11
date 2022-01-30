#1.breada

```cpp
/*
 * OK，breada 可以象bread 一样使用，但会另外预读一些块。该函数参数列表
 * 需要使用一个负数来表明参数列表的结束。
 */
//// 从指定设备读取指定的一些块。
// 成功时返回第1 块的缓冲区头指针，否则返回NULL。
struct buffer_head * breada(int dev,int first, ...)
{
    va_list args;
    struct buffer_head * bh, *tmp;

// 取可变参数表中第1 个参数（块号）。
    va_start(args,first);
// 取高速缓冲中指定设备和块号的缓冲区。如果该缓冲区数据无效，则发出读设备数据块请求。
    if (!(bh=getblk(dev,first)))
        panic("bread: getblk returned NULL\n");
    if (!bh->b_uptodate)
        ll_rw_block(READ,bh);
// 然后顺序取可变参数表中其它预读块号，并作与上面同样处理，但不引用。
    while ((first=va_arg(args,int))>=0) {
        tmp=getblk(dev,first);
        if (tmp) {
            if (!tmp->b_uptodate)
                ll_rw_block(READA,bh);
            tmp->b_count--;
        }
    }
// 可变参数表中所有参数处理完毕。等待第1 个缓冲区解锁（如果已被上锁）。
    va_end(args);
    wait_on_buffer(bh);
// 如果缓冲区中数据有效，则返回缓冲区头指针，退出。否则释放该缓冲区，返回NULL，退出。
    if (bh->b_uptodate)
        return bh;
    brelse(bh);
    return (NULL);
}
```