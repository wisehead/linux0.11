#1.try_to_share

```cpp
/*
 * try_to_share()在任务"p"中检查位于地址"address"处的页面，看页面是否存在，是否干净。
 * 如果是干净的话，就与当前任务共享。
 *
 * 注意！这里我们已假定p !=当前任务，并且它们共享同一个执行程序。
 */
//// 尝试对进程指定地址处的页面进行共享操作。
// 同时还验证指定的地址处是否已经申请了页面，若是则出错，死机。
// 返回1-成功，0-失败。

try_to_share
// 求指定内存地址的页目录项。
--from_page = to_page = ((address>>20) & 0xffc);
// 计算进程p 的代码起始地址所对应的页目录项。
--from_page += ((p->start_code>>20) & 0xffc);
// 计算当前进程中代码起始地址所对应的页目录项。
--to_page += ((current->start_code>>20) & 0xffc);

/* 在from 处是否存在页目录？ */
// *** 对p 进程页面进行操作。
// 取页目录项内容。如果该目录项无效(P=0)，则返回。否则取该目录项对应页表地址-> from。
--from = *(unsigned long *) from_page;
// 计算地址对应的页表项指针值，并取出该页表项内容 -> phys_addr。
--from_page = from + ((address>>10) & 0xffc);
--phys_addr = *(unsigned long *) from_page;

// 取页面的地址 -> phys_addr。如果该页面地址不存在或小于内存低端(1M)也返回退出。
--phys_addr &= 0xfffff000;


// *** 对当前进程页面进行操作。
// 取页目录项内容 -> to。如果该目录项无效(P=0)，则取空闲页面，并更新to_page 所指的目录项。
--to = *(unsigned long *) to_page;
--if (!(to & 1))
        if (to = get_free_page())
            *(unsigned long *) to_page = to | 7;
        else
            oom();
// 取对应页表地址 -> to，页表项地址 to_page。如果对应的页面已经存在，则出错，死机。
--to &= 0xfffff000;
--to_page = to + ((address>>10) & 0xffc);        
/* 对它们进行共享处理：写保护 */                                         
// 对p 进程中页面置写保护标志(置R/W=0 只读)。并且当前进程中的对应页表项指向它。              
--*(unsigned long *) from_page &= ~2;                       
--*(unsigned long *) to_page = *(unsigned long *) from_page;
--// 刷新页变换高速缓冲。                                             
--invalidate();                                             
--// 计算所操作页面的页面号，并将对应页面映射数组项中的引用递增1。                        
--phys_addr -= LOW_MEM;                                     
--phys_addr >>= 12;                                         
--mem_map[phys_addr]++;                                     
--return 1;                                                                                                         
```

#2.caller

```
do_no_page
--share_page
```