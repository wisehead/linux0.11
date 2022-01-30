#1.buffer_init

```cpp
//// 缓冲区初始化函数。
// 参数buffer_end 是指定的缓冲区内存的末端。对于系统有16MB 内存，则缓冲区末端设置为4MB。
// 对于系统有8MB 内存，缓冲区末端设置为2MB。
void buffer_init(long buffer_end)
{
    struct buffer_head * h = start_buffer;
    void * b;
    int i;

// 如果缓冲区高端等于1Mb，则由于从640KB-1MB 被显示内存和BIOS 占用，因此实际可用缓冲区内存
// 高端应该是640KB。否则内存高端一定大于1MB。
    if (buffer_end == 1<<20)
        b = (void *) (640*1024);
    else
        b = (void *) buffer_end;
// 这段代码用于初始化缓冲区，建立空闲缓冲区环链表，并获取系统中缓冲块的数目。
// 操作的过程是从缓冲区高端开始划分1K 大小的缓冲块，与此同时在缓冲区低端建立描述该缓冲块
// 的结构buffer_head，并将这些buffer_head 组成双向链表。
// h 是指向缓冲头结构的指针，而h+1 是指向内存地址连续的下一个缓冲头地址，也可以说是指向h
// 缓冲头的末端外。为了保证有足够长度的内存来存储一个缓冲头结构，需要b 所指向的内存块
// 地址>= h 缓冲头的末端，也即要>=h+1。
    while ( (b = (char*)b - BLOCK_SIZE) >= ((void *) (h+1)) ) {
        h->b_dev = 0;           // 使用该缓冲区的设备号。
        h->b_dirt = 0;          // 脏标志，也即缓冲区修改标志。
        h->b_count = 0;         // 该缓冲区引用计数。
        h->b_lock = 0;          // 缓冲区锁定标志。
        h->b_uptodate = 0;      // 缓冲区更新标志（或称数据有效标志）。
        h->b_wait = NULL;       // 指向等待该缓冲区解锁的进程。
        h->b_next = NULL;       // 指向具有相同hash 值的下一个缓冲头。
        h->b_prev = NULL;       // 指向具有相同hash 值的前一个缓冲头。
        h->b_data = (char *) b; // 指向对应缓冲区数据块（1024 字节）。
        h->b_prev_free = h-1;   // 指向链表中前一项。
        h->b_next_free = h+1;   // 指向链表中下一项。
        h++;                    // h 指向下一新缓冲头位置。
        NR_BUFFERS++;           // 缓冲区块数累加。
        if (b == (void *) 0x100000)     // 如果地址b 递减到等于1MB，则跳过384KB，
            b = (void *) 0xA0000;       // 让b 指向地址0xA0000(640KB)处。
    }
    h--;            // 让h 指向最后一个有效缓冲头。
    free_list = start_buffer;       // 让空闲链表头指向头一个缓冲区头。
    free_list->b_prev_free = h;     // 链表头的b_prev_free 指向前一项（即最后一项）。
    h->b_next_free = free_list;     // h 的下一项指针指向第一项，形成一个环链。
    // 初始化hash 表（哈希表、散列表），置表中所有的指针为NULL。
    for (i=0;i<NR_HASH;i++)
        hash_table[i]=NULL;
}

```