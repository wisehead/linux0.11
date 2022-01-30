#1.getblk

```cpp
/*
 * OK，下面是getblk 函数，该函数的逻辑并不是很清晰，同样也是因为要考虑
 * 竞争条件问题。其中大部分代码很少用到，(例如重复操作语句)，因此它应该
 * 比看上去的样子有效得多。
 *
 * 算法已经作了改变：希望能更好，而且一个难以琢磨的错误已经去除。
 */
// 下面宏定义用于同时判断缓冲区的修改标志和锁定标志，并且定义修改标志的权重要比锁定标志大。
#define BADNESS(bh) (((bh)->b_dirt<<1)+(bh)->b_lock)
//// 取高速缓冲中指定的缓冲区。
// 检查所指定的缓冲区是否已经在高速缓冲中，如果不在，就需要在高速缓冲中建立一个对应的新项。
// 返回相应缓冲区头指针。

struct buffer_head * getblk(int dev,int block)
{
    struct buffer_head * tmp, * bh;
repeat:
    // 搜索hash 表，如果指定块已经在高速缓冲中，则返回对应缓冲区头指针，退出。
    if (bh = get_hash_table(dev,block))
        return bh;
// 扫描空闲数据块链表，寻找空闲缓冲区。
// 首先让tmp 指向空闲链表的第一个空闲缓冲区头。
    tmp = free_list;
    do {
// 如果该缓冲区正被使用（引用计数不等于0），则继续扫描下一项。
        if (tmp->b_count)
            continue;
// 如果缓冲头指针bh 为空，或者tmp 所指缓冲头的标志(修改、锁定)权重小于bh 头标志的权重，
// 则让bh 指向该tmp 缓冲区头。如果该tmp 缓冲区头表明缓冲区既没有修改也没有锁定标志置位，
// 则说明已为指定设备上的块取得对应的高速缓冲区，则退出循环。
        if (!bh || BADNESS(tmp)<BADNESS(bh)) {
            bh = tmp;
            if (!BADNESS(tmp))
                break;
        }
/* 重复操作直到找到适合的缓冲区 */
    } while ((tmp = tmp->b_next_free) != free_list);
// 如果所有缓冲区都正被使用（所有缓冲区的头部引用计数都>0），
// 则睡眠，等待有空闲的缓冲区可用。
    if (!bh) {
        sleep_on(&buffer_wait);
        goto repeat;
    }
    // 等待该缓冲区解锁（如果已被上锁的话）。
    wait_on_buffer(bh);
    // 如果该缓冲区又被其它任务使用的话，只好重复上述过程。
    if (bh->b_count)
        goto repeat;
// 如果该缓冲区已被修改，则将数据写盘，并再次等待缓冲区解锁。如果该缓冲区又被其它任务使用
// 的话，只好再重复上述过程。
    while (bh->b_dirt) {
        sync_dev(bh->b_dev);
        wait_on_buffer(bh);
        if (bh->b_count)
            goto repeat;
    }
/* 注意！！当进程为了等待该缓冲块而睡眠时，其它进程可能已经将该缓冲块 */
/* 加入进高速缓冲中，所以要对此进行检查。 */
// 在高速缓冲hash 表中检查指定设备和块的缓冲区是否已经被加入进去。如果是的话，就再次重复
// 上述过程。
    if (find_buffer(dev,block))
        goto repeat;
/* OK，最终我们知道该缓冲区是指定参数的唯一一块， */
/* 而且还没有被使用(b_count=0)，未被上锁(b_lock=0)，并且是干净的（未被修改的） */
// 于是让我们占用此缓冲区。置引用计数为1，复位修改标志和有效(更新)标志。
    bh->b_count=1;
    bh->b_dirt=0;
    bh->b_uptodate=0;
// 从hash 队列和空闲块链表中移出该缓冲区头，让该缓冲区用于指定设备和其上的指定块。
    remove_from_queues(bh);
    bh->b_dev=dev;
    bh->b_blocknr=block;
// 然后根据此新的设备号和块号重新插入空闲链表和hash 队列新位置处。并最终返回缓冲头指针。
    insert_into_queues(bh);
    return bh;
}

```