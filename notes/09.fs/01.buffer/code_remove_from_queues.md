#1.remove_from_queues


```cpp
//// 从hash 队列和空闲缓冲队列中移走指定的缓冲块。
static _inline void remove_from_queues(struct buffer_head * bh)
{
/* 从hash 队列中移除缓冲块 */
    if (bh->b_next)
        bh->b_next->b_prev = bh->b_prev;
    if (bh->b_prev)
        bh->b_prev->b_next = bh->b_next;
// 如果该缓冲区是该队列的头一个块，则让hash 表的对应项指向本队列中的下一个缓冲区。
    if (hash(bh->b_dev,bh->b_blocknr) == bh)
        hash(bh->b_dev,bh->b_blocknr) = bh->b_next;
/* 从空闲缓冲区表中移除缓冲块 */
    if (!(bh->b_prev_free) || !(bh->b_next_free))
        panic("Free block list corrupted");
    bh->b_prev_free->b_next_free = bh->b_next_free;
    bh->b_next_free->b_prev_free = bh->b_prev_free;
// 如果空闲链表头指向本缓冲区，则让其指向下一缓冲区。
    if (free_list == bh)
        free_list = bh->b_next_free;
}
```