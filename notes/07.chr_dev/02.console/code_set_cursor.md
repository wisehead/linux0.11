#1.set_cursor

```cpp
//// 根据设置显示光标。
// 根据显示内存光标对应位置pos，设置显示控制器光标的显示位置。
static _inline void
set_cursor (void)
{
    cli ();
// 首先使用索引寄存器端口选择显示控制数据寄存器r14(光标当前显示位置高字节)，然后写入光标
// 当前位置高字节(向右移动9 位表示高字节移到低字节再除以2)。是相对于默认显示内存操作的。
    outb_p (14, video_port_reg);
    outb_p ((unsigned char)(0xff & ((pos - video_mem_start) >> 9)), video_port_val);
// 再使用索引寄存器选择r15，并将光标当前位置低字节写入其中。
    outb_p (15, video_port_reg);
    outb_p ((unsigned char)(0xff & ((pos - video_mem_start) >> 1)), video_port_val);
    sti ();
}

```

#2.caller

con_write