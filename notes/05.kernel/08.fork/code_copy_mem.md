#1.copy_mem

```cpp
// 设置新任务的代码和数据段基址、限长并复制页表。
// nr 为新任务号；p 是新任务数据结构的指针。
int copy_mem (int nr, struct task_struct *p)
{
    unsigned long old_data_base, new_data_base, data_limit;
    unsigned long old_code_base, new_code_base, code_limit;

    code_limit = get_limit (0x0f);  // 取局部描述符表中代码段描述符项中段限长。
    data_limit = get_limit (0x17);  // 取局部描述符表中数据段描述符项中段限长。
    old_code_base = get_base (current->ldt[1]); // 取原代码段基址。
    old_data_base = get_base (current->ldt[2]); // 取原数据段基址。
    if (old_data_base != old_code_base) // 0.11 版不支持代码和数据段分立的情况。
        panic ("We don't support separate I&D");
    if (data_limit < code_limit)    // 如果数据段长度 < 代码段长度也不对。
        panic ("Bad data_limit");
    new_data_base = new_code_base = nr * 0x4000000; // 新基址=任务号*64Mb(任务大小)。
    p->start_code = new_code_base;
    set_base (p->ldt[1], new_code_base);    // 设置代码段描述符中基址域。
    set_base (p->ldt[2], new_data_base);    // 设置数据段描述符中基址域。
    if (copy_page_tables (old_data_base, new_data_base, data_limit))
    {               // 复制代码和数据段。
        free_page_tables (new_data_base, data_limit);   // 如果出错则释放申请的内存。
        return -ENOMEM;
    }
    return 0;
}
```

#2.note

```
 新基址=任务号*64Mb(任务大小)。
 
 limit = 0x3ff * 4KB = 1KB * 4KB = 4MB
```