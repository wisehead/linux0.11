#1.copy_page_tables

```cpp
/*
 * 好了，下面是内存管理mm 中最为复杂的程序之一。它通过只复制内存页面
 * 来拷贝一定范围内线性地址中的内容。希望代码中没有错误，因为我不想
 * 再调试这块代码了 :-)
 *
 * 注意！我们并不是仅复制任何内存块- 内存块的地址需要是4Mb 的倍数（正好
 * 一个页目录项对应的内存大小），因为这样处理可使函数很简单。不管怎样，
 * 它仅被fork()使用（fork.c）
 *
 * 注意!!当from==0 时，是在为第一次fork()调用复制内核空间。此时我们
 * 不想复制整个页目录项对应的内存，因为这样做会导致内存严重的浪费- 我们
 * 只复制头160 个页面- 对应640kB。即使是复制这些页面也已经超出我们的需求，
 * 但这不会占用更多的内存- 在低1Mb 内存范围内我们不执行写时复制操作，所以
 * 这些页面可以与内核共享。因此这是nr=xxxx 的特殊情况（nr 在程序中指页面数）。
 */
//// 复制指定线性地址和长度（页表个数）内存对应的页目录项和页表，从而被复制的页目录和
//// 页表对应的原物理内存区被共享使用。
// 复制指定地址和长度的内存对应的页目录项和页表项。需申请页面来存放新页表，原内存区被共享；
// 此后两个进程将共享内存区，直到有一个进程执行写操作时，才分配新的内存页（写时复制机制）。

copy_page_tables
// 取得源地址和目的地址的目录项(from_dir 和to_dir)。参见对115 句的注释。                          
--from_dir = (unsigned long *) ((from>>20) & 0xffc); /* _pg_dir = 0 */    
--to_dir = (unsigned long *) ((to>>20) & 0xffc);                          
// 计算要复制的内存块占用的页表数（也即目录项数）。                                               
--size = ((unsigned) (size+0x3fffff)) >> 22;  

// 下面开始对每个占用的页表依次进行复制操作。
--for( ; size-->0 ; from_dir++,to_dir++) {
        if (1 & *to_dir)// 如果目的目录项指定的页表已经存在(P=1)，则出错，死机。
            panic("copy_page_tables: already exist");
        if (!(1 & *from_dir))// 如果此源目录项未被使用，则不用复制对应页表，跳过。
            continue;
        // 取当前源目录项中页表的地址 -> from_page_table。
        from_page_table = (unsigned long *) (0xfffff000 & *from_dir);
// 为目的页表取一页空闲内存，如果返回是0 则说明没有申请到空闲内存页面。返回值=-1，退出。
        if (!(to_page_table = (unsigned long *) get_free_page()))
            return -1;  /* Out of memory, see freeing */
        // 设置目的目录项信息。7 是标志信息，表示(Usr, R/W, Present)。
        *to_dir = ((unsigned long) to_page_table) | 7;
        // 针对当前处理的页表，设置需复制的页面数。如果是在内核空间，则仅需复制头160 页，
        // 否则需要复制1 个页表中的所有1024 页面。
        nr = (from==0)?0xA0:1024;
        // 对于当前页表，开始复制指定数目nr 个内存页面。
        for ( ; nr-- > 0 ; from_page_table++,to_page_table++) {
            this_page = *from_page_table;// 取源页表项内容。
            if (!(1 & this_page))// 如果当前源页面没有使用，则不用复制。
                continue;
// 复位页表项中R/W 标志(置0)。(如果U/S 位是0，则R/W 就没有作用。如果U/S 是1，而R/W 是0，
// 那么运行在用户层的代码就只能读页面。如果U/S 和R/W 都置位，则就有写的权限。)
            this_page &= ~2;
            *to_page_table = this_page;// 将该页表项复制到目的页表中。
// 如果该页表项所指页面的地址在1M 以上，则需要设置内存页面映射数组mem_map[]，于是计算
// 页面号，并以它为索引在页面映射数组相应项中增加引用次数。
            if (this_page > LOW_MEM) {
// 下面这句的含义是令源页表项所指内存页也为只读。因为现在开始有两个进程共用内存区了。
// 若其中一个内存需要进行写操作，则可以通过页异常的写保护处理，为执行写操作的进程分配
// 一页新的空闲页面，也即进行写时复制的操作。
                *from_page_table = this_page;// 令源页表项也只读。
                this_page -= LOW_MEM;
                this_page >>= 12;
                mem_map[this_page]++;
            }
        }
--}//end for
--invalidate();// 刷新页变换高速缓冲。
--return 0;                                                            
```


#2.notes

```
caller is :copy_mem

    if (copy_page_tables (old_data_base, new_data_base, data_limit))
    {               // 复制代码和数据段。
        free_page_tables (new_data_base, data_limit);   // 如果出错则释放申请的内存。
        return -ENOMEM;
    }

```

输入参数都是线性地址。
所以，页目录只有一个，就是 page 0.

但是每个用户进程都会有自己的页表，就是从内核页表中拷贝出来的，但是页目录是共用的。

后面需要考虑，如何保证内核页表和用户页表的一致性。




