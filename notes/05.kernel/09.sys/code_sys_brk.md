#1.sys_brk

```cpp
// 当参数end_data_seg 数值合理，并且系统确实有足够的内存，而且进程没有超越其最大数据段大小
// 时，该函数设置数据段末尾为end_data_seg 指定的值。该值必须大于代码结尾并且要小于堆栈
// 结尾16KB。返回值是数据段的新结尾值（如果返回值与要求值不同，则表明有错发生）。
// 该函数并不被用户直接调用，而由libc 库函数进行包装，并且返回值也不一样。
int sys_brk (unsigned long end_data_seg)
{
    if (end_data_seg >= current->end_code &&    // 如果参数>代码结尾，并且
    end_data_seg < current->start_stack - 16384)    // 小于堆栈-16KB，
        current->brk = end_data_seg;    // 则设置新数据段结尾值。
    return current->brk;        // 返回进程当前的数据段结尾值。
}
```

#2.notes

注意这些有意思的内存使用变量