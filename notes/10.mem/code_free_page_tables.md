#1.free_page_tables

```cpp
free_page_tables
// 计算所占页目录项数(4M 的进位整数倍)，也即所占页表数。
--size = (size + 0x3fffff) >> 22;
// 下面一句计算起始目录项。对应的目录项号=from>>22，因每项占4 字节，并且由于页目录是从
// 物理地址0 开始，因此实际的目录项指针=目录项号<<2，也即(from>>20)。与上0xffc 确保
// 目录项指针范围有效。
--dir = (unsigned long *) ((from>>20) & 0xffc); /* _pg_dir = 0 */
--for ( ; size-->0 ; dir++) {// size 现在是需要被释放内存的目录项数。
        if (!(1 & *dir))// 如果该目录项无效(P 位=0)，则继续。
            continue;// 目录项的位0(P 位)表示对应页表是否存在。
        pg_table = (unsigned long *) (0xfffff000 & *dir);// 取目录项中页表地址。
        for (nr=0 ; nr<1024 ; nr++) {// 每个页表有1024 个页项。
            if (1 & *pg_table)// 若该页表项有效(P 位=1)，则释放对应内存页。
                free_page(0xfffff000 & *pg_table);
            *pg_table = 0;// 该页表项内容清零。
            pg_table++;// 指向页表中下一项。
        }
        free_page(0xfffff000 & *dir);// 释放该页表所占内存页面。但由于页表在
                                        // 物理地址1M 以内，所以这句什么都不做。
        *dir = 0;// 对相应页表的目录项清零。
--}//end for
--invalidate();// 刷新页变换高速缓冲。
```

#2.invalidate

```cpp
// 刷新页变换高速缓冲宏函数。
// 为了提高地址转换的效率，CPU 将最近使用的页表数据存放在芯片中高速缓冲中。
// 在修改过页表信息之后，就需要刷新该缓冲区。这里使用重新加载页目录基址
// 寄存器cr3 的方法来进行刷新。下面eax = 0，是页目录的基址。
//#define invalidate() \
//__asm__("movl %%eax,%%cr3"::"a" (0))
#define invalidate() \
_asm{_asm xor eax,eax _asm mov cr3,eax}
```

#3.caller

```
- do_execve
- do_exit
- copy_mem
```