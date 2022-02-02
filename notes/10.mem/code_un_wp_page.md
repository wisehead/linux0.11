#1.un_wp_page

```cpp
//// 取消写保护页面函数。用于页异常中断过程中写保护异常的处理（写时复制）。
// 输入参数为页表项指针。
// [ un_wp_page 意思是取消页面的写保护：Un-Write Protected。]
void un_wp_page(unsigned long * table_entry)
--old_page = 0xfffff000 & *table_entry;// 取原页面对应的目录项号。
// 如果原页面地址大于内存低端LOW_MEM(1Mb)，并且其在页面映射字节图数组中值为1（表示仅
// 被引用1 次，页面没有被共享），则在该页面的页表项中置R/W 标志（可写），并刷新页变换
// 高速缓冲，然后返回。
--if (old_page >= LOW_MEM && mem_map[MAP_NR(old_page)]==1) {
        *table_entry |= 2;
        invalidate();
        return;
--}
// 否则，在主内存区内申请一页空闲页面。
--if (!(new_page=get_free_page()))
----oom();// Out of Memory。内存不够处理。
// 如果原页面大于内存低端（则意味着mem_map[]>1，页面是共享的），则将原页面的页面映射
// 数组值递减1。然后将指定页表项内容更新为新页面的地址，并置可读写等标志(U/S, R/W, P)。
// 刷新页变换高速缓冲。最后将原页面内容复制到新页面。
--if (old_page >= LOW_MEM)
----mem_map[MAP_NR(old_page)]--;
--*table_entry = new_page | 7;
--invalidate();
--copy_page(old_page,new_page);

```

#2.caller

```
- do_wp_page
- write_verify

```