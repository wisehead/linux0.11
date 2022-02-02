#1.put_page

```cpp
/*
 * 下面函数将一内存页面放置在指定地址处。它返回页面的物理地址，如果
 * 内存不够(在访问页表或页面时)，则返回0。
 */
//// 把一物理内存页面映射到指定的线性地址处。
// 主要工作是在页目录和页表中设置指定页面的信息。若成功则返回页面地址。

put_page
// 计算指定地址在页目录表中对应的目录项指针。应该是dir_entry
--page_table = (unsigned long *) ((address>>20) & 0xffc);
// 如果该目录项有效(P=1)(也即指定的页表在内存中)，则从中取得指定页表的地址 -> page_table。                      
--if ((*page_table)&1)                                                         
----page_table = (unsigned long *) (0xfffff000 & *page_table);                 
--else {                                                                       
// 否则，申请空闲页面给页表使用，并在对应目录项中置相应标志7（User, U/S, R/W）。然后将                           
// 该页表的地址 -> page_table。                                                       
----if (!(tmp=get_free_page()))                                                
------return 0;                                                                
----*page_table = tmp|7;                                                       
----page_table = (unsigned long *) tmp;                                        
--}                                                                            
// 在页表中设置指定地址的物理内存页面的页表项内容。每个页表共可有1024 项(0x3ff)。                               
--page_table[(address>>12) & 0x3ff] = page | 7;                                
/* 不需要刷新页变换高速缓冲 */                                                             
--return page;// 返回页面地址。                                                       
```