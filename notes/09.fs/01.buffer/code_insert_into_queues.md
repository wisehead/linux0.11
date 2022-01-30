#1.insert_into_queues

```cpp
//// 将指定缓冲区插入空闲链表尾并放入hash 队列中。
static _inline void insert_into_queues(struct buffer_head * bh)
{
/* 放在空闲链表末尾处 */
    bh->b_next_free = free_list;
    bh->b_prev_free = free_list->b_prev_free;
    free_list->b_prev_free->b_next_free = bh;
    free_list->b_prev_free = bh;
/* 如果该缓冲块对应一个设备，则将其插入新hash 队列中 */
    bh->b_prev = NULL;
    bh->b_next = NULL;
    if (!bh->b_dev)
        return;
    bh->b_next = hash(bh->b_dev,bh->b_blocknr);
    hash(bh->b_dev,bh->b_blocknr) = bh;
    bh->b_next->b_prev = bh;
}
```