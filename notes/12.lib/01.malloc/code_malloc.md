#1.malloc

```cpp
/*
 * 首先我们搜索存储桶目录bucket_dir 来寻找适合请求的桶大小。
 */
// 搜索存储桶目录，寻找适合申请内存块大小的桶描述符链表。如果目录项的桶字节
// 数大于请求的字节数，就找到了对应的桶目录项。
--for (bdir = bucket_dir; bdir->size; bdir++)
        if (bdir->size >= len)
            break;
// 搜索对应桶目录项中描述符链表，查找具有空闲空间的桶描述符。如果桶描述符的空闲内存指针
// freeptr 不为空，则表示找到了相应的桶描述符。
--for (bdesc = bdir->chain; bdesc; bdesc = bdesc->next)
        if (bdesc->freeptr)
            break;            

            break;
    /*
     * 如果没有找到具有空闲空间的桶描述符，那么我们就要新建立一个该目录项的描述符。
     */
--if (!bdesc) {
        char        *cp;
        int     i;

        // 若free_bucket_desc 还为空时，表示第一次调用该程序，则对描述符链表进行初始化。
        // free_bucket_desc 指向第一个空闲桶描述符。
        if (!free_bucket_desc)
            init_bucket_desc();
        // 取free_bucket_desc 指向的空闲桶描述符，并让free_bucket_desc 指向下一个空闲桶描述符。
        bdesc = free_bucket_desc;
        free_bucket_desc = bdesc->next;
        // 初始化该新的桶描述符。令其引用数量等于0；桶的大小等于对应桶目录的大小；申请一内存页面，
        // 让描述符的页面指针page 指向该页面；空闲内存指针也指向该页开头，因为此时全为空闲。
        bdesc->refcnt = 0;
        bdesc->bucket_size = bdir->size;
        bdesc->page = bdesc->freeptr = (void *) cp = (void *)get_free_page();
        // 如果申请内存页面操作失败，则显示出错信息，死机。
        if (!cp)
            panic("Out of memory in kernel malloc()");
        /* 在该页空闲内存中建立空闲对象链表 */
        // 以该桶目录项指定的桶大小为对象长度，对该页内存进行划分，并使每个对象的开始4 字节设置
        // 成指向下一对象的指针。
        for (i=PAGE_SIZE/bdir->size; i > 1; i--) {
            *((char **) cp) = cp + bdir->size;
            cp += bdir->size;
        }
        // 最后一个对象开始处的指针设置为0(NULL)。
        // 然后让该桶描述符的下一描述符指针字段指向对应桶目录项指针chain 所指的描述符，而桶目录的
        // chain 指向该桶描述符，也即将该描述符插入到描述符链链头处。
        *((char **) cp) = 0;
        bdesc->next = bdir->chain; /* OK, link it in! */
        bdir->chain = bdesc;
--}

// 返回指针即等于该描述符对应页面的当前空闲指针。然后调整该空闲空间指针指向下一个空闲对象，
// 并使描述符中对应页面中对象引用计数增1。
--retval = (void *) bdesc->freeptr;
--bdesc->freeptr = *((void **) retval);
--bdesc->refcnt++;
    
```