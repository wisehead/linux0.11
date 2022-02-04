#1. floppy_on_interrupt


```cpp
//// 软驱启动定时中断调用函数。
// 首先检查数字输出寄存器(DOR)，使其选择当前指定的驱动器。然后调用执行软盘读写传输
// 函数transfer()。
static void
floppy_on_interrupt (void)
{
/* 我们不能任意设置选择的软驱，因为这样做可能会引起进程睡眠。我们只是迫使它自己选择 */
  selected = 1;         // 置已选择当前驱动器标志。
// 如果当前驱动器号与数字输出寄存器DOR 中的不同，则重新设置DOR 为当前驱动器current_drive。
// 定时延迟2 个滴答时间，然后调用软盘读写传输函数transfer()。否则直接调用软盘读写传输函数。
    if (current_drive != (current_DOR & 3))
    {
        current_DOR &= 0xFC;
        current_DOR |= current_drive;
        outb (current_DOR, FD_DOR); // 向数字输出寄存器输出当前DOR。
        add_timer (2, &transfer);   // 添加定时器并执行传输函数。
    }
    else
        transfer ();        // 执行软盘读写传输函数。
}
```

#2.caller

```cpp
do_fd_request
// 添加定时器，用于指定驱动器到能正常运行所需延迟的时间（滴答数），当定时时间到时就调用
// 函数floppy_on_interrupt()，
--add_timer (ticks_to_floppy_on (current_drive), &floppy_on_interrupt);
```