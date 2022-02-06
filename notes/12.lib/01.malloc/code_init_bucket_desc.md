#1.init_bucket_desc

```cpp

/*
 * 下面的子程序用于初始化一页桶描述符页面。
 */
//// 初始化桶描述符。
// 建立空闲桶描述符链表，并让free_bucket_desc 指向第一个空闲桶描述符。
static _inline void init_bucket_desc()
{
    struct bucket_desc *bdesc, *first;
    int i;

// 申请一页内存，用于存放桶描述符。如果失败，则显示初始化桶描述符时内存不够出错信息，死机。
    first = bdesc = (struct bucket_desc *) get_free_page();
    if (!bdesc)
        panic("Out of memory in init_bucket_desc()");
// 首先计算一页内存中可存放的桶描述符数量，然后对其建立单向连接指针。
    for (i = PAGE_SIZE/sizeof(struct bucket_desc); i > 1; i--) {
        bdesc->next = bdesc+1;
        bdesc++;
    }
    /*
     * 这是在最后处理的，目的是为了避免在get_free_page()睡眠时该子程序又被
     * 调用而引起的竞争条件。
     */
// 将空闲桶描述符指针free_bucket_desc 加入链表中。
    bdesc->next = free_bucket_desc;
    free_bucket_desc = first;
}
```

#2.caller

malloc