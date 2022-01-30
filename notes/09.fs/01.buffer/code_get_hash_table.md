#1.get_hash_table

```cpp
/*
 * 代码为什么会是这样子的？我听见你问... 原因是竞争条件。由于我们没有对
 * 缓冲区上锁（除非我们正在读取它们中的数据），那么当我们（进程）睡眠时
 * 缓冲区可能会发生一些问题（例如一个读错误将导致该缓冲区出错）。目前
 * 这种情况实际上是不会发生的，但处理的代码已经准备好了。
 */
struct buffer_head * get_hash_table(int dev, int block)
{
    struct buffer_head * bh;

    for (;;) {
        // 在高速缓冲中寻找给定设备和指定块的缓冲区，如果没有找到则返回NULL，退出。
        if (!(bh=find_buffer(dev,block)))
            return NULL;
        // 对该缓冲区增加引用计数，并等待该缓冲区解锁（如果已被上锁）。
        bh->b_count++;
        wait_on_buffer(bh);
        // 由于经过了睡眠状态，因此有必要再验证该缓冲区块的正确性，并返回缓冲区头指针。
        if (bh->b_dev == dev && bh->b_blocknr == block)
            return bh;
// 如果该缓冲区所属的设备号或块号在睡眠时发生了改变，则撤消对它的引用计数，重新寻找。
        bh->b_count--;
    }
}
```