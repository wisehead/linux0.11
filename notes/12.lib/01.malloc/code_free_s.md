#1.free_s

```cpp
/*
 * 下面是释放子程序。如果你知道释放对象的大小，则free_s()将使用该信息加速
 * 搜寻对应桶描述符的速度。
 *
 * 我们将定义一个宏，使得"free(x)"成为"free_s(x, 0)"。
 */
//// 释放存储桶对象。
// 参数：obj - 对应对象指针；size - 大小。

free_s
    /* 计算该对象所在的页面 */
--page = (void *)  ((unsigned long) obj & 0xfffff000);

    /* 现在搜索存储桶目录项所链接的桶描述符，寻找该页面 */
--for (bdir = bucket_dir; bdir->size; bdir++) {
        prev = 0;
        /* 如果参数size 是0，则下面条件肯定是false */
        if (bdir->size < size)
            continue;
        // 搜索对应目录项中链接的所有描述符，查找对应页面。如果某描述符页面指针等于page 则表示找到
        // 了相应的描述符，跳转到found。如果描述符不含有对应page，则让描述符指针prev 指向该描述符。
        for (bdesc = bdir->chain; bdesc; bdesc = bdesc->next) {
            if (bdesc->page == page)
                goto found;
            prev = bdesc;
        }
--}

// 找到对应的桶描述符后，首先关中断。然后将该对象内存块链入空闲块对象链表中，
// 并使该描述符的对象引用计数减1。
--cli(); /* 为了避免竞争条件 */               
--*((void **)obj) = bdesc->freeptr;   
--bdesc->freeptr = obj;               
--bdesc->refcnt--;      

--// 如果引用计数已等于0，则我们就可以释放对应的内存页面和该桶描述符。                                                 
--if (bdesc->refcnt == 0) {                                                            
        /*                                                                             
         * 我们需要确信prev 仍然是正确的，若某程序粗鲁地中断了我们                                              
         * 就有可能不是了。                                                                    
         */                                                                            
        // 如果prev 已经不是搜索到的描述符的前一个描述符，则重新搜索当前描述符的前一个描述符。                                
        if ((prev && (prev->next != bdesc)) ||                                         
            (!prev && (bdir->chain != bdesc)))                                         
            for (prev = bdir->chain; prev; prev = prev->next)                          
                if (prev->next == bdesc)                                               
                    break;                                                             
        // 如果找到该前一个描述符，则从描述符链中删除当前描述符。                                                 
        if (prev)                                                                      
            prev->next = bdesc->next;                                                  
        // 如果prev==NULL，则说明当前一个描述符是该目录项首个描述符，也即目录项中chain 应该直接                          
        // 指向当前描述符bdesc，否则表示链表有问题，则显示出错信息，死机。因此，为了将当前描述符                               
        // 从链表中删除，应该让chain 指向下一个描述符。                                                   
        else {                                                                         
            if (bdir->chain != bdesc)                                                  
                panic("malloc bucket chains corrupted");                               
            bdir->chain = bdesc->next;                                                 
        }                                                                              
        // 释放当前描述符所操作的内存页面，并将该描述符插入空闲描述符链表开始处。                                         
        free_page((unsigned long) bdesc->page);                                        
        bdesc->next = free_bucket_desc;                                                
        free_bucket_desc = bdesc;                                                      
--}                                                                                    
                                                                                                    
```