#1.get_free_page

```cpp
/*
 * 获取首个(实际上是最后1 个:-)物理空闲页面，并标记为已使用。如果没有空闲页面，
 * 就返回0。
 */
//// 取物理空闲页面。如果已经没有可用内存了，则返回0。
// 输入：%1(ax=0) - 0；%2(LOW_MEM)；%3(cx=PAGING PAGES)；%4(edi=mem_map+PAGING_PAGES-1)。
// 输出：返回%0(ax=页面起始地址)。
// 上面%4 寄存器实际指向mem_map[]内存字节图的最后一个字节。本函数从字节图末端开始向前扫描
// 所有页面标志（页面总数为PAGING_PAGES），若有页面空闲（其内存映像字节为0）则返回页面地址。
// 注意！本函数只是指出在主内存区的一页空闲页面，但并没有映射到某个进程的线性地址去。后面
// 的put_page()函数就是用来作映射的。
unsigned long get_free_page(void)
{
//  unsigned long __res = mem_map+PAGING_PAGES-1;
    __asm {
        pushf
        xor eax, eax
        mov ecx,PAGING_PAGES
//      mov edi,__res
        mov edi,offset mem_map + PAGING_PAGES - 1
        std
        repne scasb     // 方向位置位，将al(0)与对应(di)每个页面的内容比较，
        jne l1      // 如果没有等于0 的字节，则跳转结束（返回0）。
        mov byte ptr [edi+1],1  // 将对应页面的内存映像位置1。
        sal ecx,12  // 页面数*4K = 相对页面起始地址。
        add ecx,LOW_MEM // 再加上低端内存地址，即获得页面实际物理起始地址。
        mov edx,ecx // 将页面实际起始地址 -> edx 寄存器。
        mov ecx,1024    // 寄存器ecx 置计数值1024。
        lea edi,[edx+4092]// 将4092+edx 的位置 -> edi(该页面的末端)。
        rep stosd   // 将edi 所指内存清零（反方向，也即将该页面清零）。
//      mov __res,edx   // 将页面起始地址 -> __res（返回值）。
        mov eax,edx
    l1: popf
    }
//  return __res;// 返回空闲页面地址（如果无空闲也则返回0）。
}

//unsigned long get_free_page(void)
//{
//register unsigned long __res asm("ax");
//
//__asm__("std ; repne ; scasb\n\t" // 方向位置位，将al(0)与对应每个页面的(di)内容比较，
//  "jne 1f\n\t"    // 如果没有等于0 的字节，则跳转结束（返回0）。
//  "movb $1,1(%%edi)\n\t"// 将对应页面的内存映像位置1。
//  "sall $12,%%ecx\n\t"// 页面数*4K = 相对页面起始地址。
//  "addl %2,%%ecx\n\t"// 再加上低端内存地址，即获得页面实际物理起始地址。
//  "movl %%ecx,%%edx\n\t"// 将页面实际起始地址 -> edx 寄存器。
//  "movl $1024,%%ecx\n\t"// 寄存器ecx 置计数值1024。
//  "leal 4092(%%edx),%%edi\n\t"// 将4092+edx 的位置 -> edi(该页面的末端)。
/// "rep ; stosl\n\t"// 将edi 所指内存清零（反方向，也即将该页面清零）。
//  "movl %%edx,%%eax\n"// 将页面起始地址 -> eax（返回值）。
//  "1:"
//  :"=a" (__res)
//  :"0" (0),"i" (LOW_MEM),"c" (PAGING_PAGES),
//  "D" (mem_map+PAGING_PAGES-1)
//  :"di","cx","dx");
//return __res;// 返回空闲页面地址（如果无空闲也则返回0）。
//}

```