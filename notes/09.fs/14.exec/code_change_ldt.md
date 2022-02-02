#1.change_ldt

```cpp
//// 修改局部描述符表中的描述符基址和段限长，并将参数和环境空间页面放置在数据段末端。
// 参数：text_size - 执行文件头部中a_text 字段给出的代码段长度值；
//       page - 参数和环境空间页面指针数组。
// 返回：数据段限长值(64MB)。
static unsigned long change_ldt(unsigned long text_size,unsigned long * page)
{
    unsigned long code_limit,data_limit,code_base,data_base;
    int i;

// 根据执行文件头部a_text 值，计算以页面长度为边界的代码段限长。并设置数据段长度为64MB。
    code_limit = text_size+PAGE_SIZE -1;
    code_limit &= 0xFFFFF000;
    data_limit = 0x4000000;
// 取当前进程中局部描述符表代码段描述符中代码段基址，代码段基址与数据段基址相同。
    code_base = get_base(current->ldt[1]);
    data_base = code_base;
// 重新设置局部表中代码段和数据段描述符的基址和段限长。
    set_base(current->ldt[1],code_base);
    set_limit(current->ldt[1],code_limit);
    set_base(current->ldt[2],data_base);
    set_limit(current->ldt[2],data_limit);
/* 要确信fs 段寄存器已指向新的数据段 */
// fs 段寄存器中放入局部表数据段描述符的选择符(0x17)。
//  __asm__("pushl $0x17\n\tpop %%fs"::);
    _asm {
        push 0x17
        pop fs
    }
// 将参数和环境空间已存放数据的页面（共可有MAX_ARG_PAGES 页，128kB）放到数据段线性地址的
// 末端。是调用函数put_page()进行操作的（mm/memory.c, 197）。
    data_base += data_limit;
    for (i=MAX_ARG_PAGES-1 ; i>=0 ; i--) {
        data_base -= PAGE_SIZE;
        if (page[i])                        // 如果该页面存在，
            put_page(page[i],data_base);    // 就放置该页面。
    }
    return data_limit;      // 最后返回数据段限长(64MB)。
}

```

#2.caller

```
- do_execve
```