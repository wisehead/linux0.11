#1.free_page

```cpp

/*
 * 释放物理地址'addr'开始的一页内存。用于函数'free_page_tables()'。
 */
//// 释放物理地址addr 开始的一页面内存。
// 1MB 以下的内存空间用于内核程序和缓冲，不作为分配页面的内存空间。
void free_page(unsigned long addr)
{
    if (addr < LOW_MEM) return;// 如果物理地址addr 小于内存低端（1MB），则返回。
    if (addr >= HIGH_MEMORY)// 如果物理地址addr>=内存最高端，则显示出错信息。
        panic("trying to free nonexistent page");
    addr -= LOW_MEM;// 物理地址减去低端内存位置，再除以4KB，得页面号。
    addr >>= 12;
    if (mem_map[addr]--) return;// 如果对应内存页面映射字节不等于0，则减1 返回。
    mem_map[addr]=0;// 否则置对应页面映射字节为0，并显示出错信息，死机。
    panic("trying to free free page");
}
```