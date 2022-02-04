#1.floppy_change


```cpp
/*
* floppy-change()不是从中断程序中调用的，所以这里我们可以轻松一下，睡觉等。
* 注意floppy-on()会尝试设置current_DOR 指向所需的驱动器，但当同时使用几个
* 软盘时不能睡眠：因此此时只能使用循环方式。
*/
//// 检测指定软驱中软盘更换情况。如果软盘更换了则返回1，否则返回0。

floppy_change
--repeat:
--floppy_on (nr);     // 开启指定软驱nr（kernel/sched.c,251）。
// 如果当前选择的软驱不是指定的软驱nr，并且已经选择其它了软驱，则让当前任务进入可中断
// 等待状态。
--while ((unsigned int)(current_DOR & 3) != nr && selected)
        interruptible_sleep_on (&wait_on_floppy_select);
// 如果当前没有选择其它软驱或者当前任务被唤醒时，当前软驱仍然不是指定的软驱nr，则循环等待。
--if ((unsigned int)(current_DOR & 3) != nr)
        goto repeat;
// 取数字输入寄存器值，如果最高位（位7）置位，则表示软盘已更换，此时关闭马达并退出返回1。
// 否则关闭马达退出返回0。
--if (inb (FD_DIR) & 0x80)
    {
        floppy_off (nr);
        return 1;
    }
--floppy_off (nr);
--return 0;
```

#2.floppy_on

```cpp
// 等待指定软驱马达启动所需时间。
void floppy_on (unsigned int nr)
{
    cli ();         // 关中断。
    while (ticks_to_floppy_on (nr)) // 如果马达启动定时还没到，就一直把当前进程置
        sleep_on (nr + wait_motor); // 为不可中断睡眠状态并放入等待马达运行的队列中。
    sti ();         // 开中断。
}
```

#3. ticks_to_floppy_on

```cpp
// 指定软盘到正常运转状态所需延迟滴答数（时间）。
// nr -- 软驱号(0-3)，返回值为滴答数。
int ticks_to_floppy_on (unsigned int nr)
{
    extern unsigned char selected;  // 当前选中的软盘号(kernel/blk_drv/floppy.c,122)。
    unsigned char mask = 0x10 << nr;    // 所选软驱对应数字输出寄存器中启动马达比特位。

    if (nr > 3)
        panic ("floppy_on: nr>3");  // 最多4 个软驱。
    moff_timer[nr] = 10000; /* 100 s = very big :-) */
    cli ();         /* use floppy_off to turn it off */
    mask |= current_DOR;
// 如果不是当前软驱，则首先复位其它软驱的选择位，然后置对应软驱选择位。
    if (!selected)
    {
        mask &= 0xFC;
        mask |= nr;
    }
// 如果数字输出寄存器的当前值与要求的值不同，则向FDC 数字输出端口输出新值(mask)。并且如果
// 要求启动的马达还没有启动，则置相应软驱的马达启动定时器值(HZ/2 = 0.5 秒或50 个滴答)。
// 此后更新当前数字输出寄存器值current_DOR。
    if (mask != current_DOR)
    {
        outb (mask, FD_DOR);
        if ((mask ^ current_DOR) & 0xf0)
            mon_timer[nr] = HZ / 2;
        else if (mon_timer[nr] < 2)
            mon_timer[nr] = 2;
        current_DOR = mask;
    }
    sti ();
    return mon_timer[nr];
}

```

#4.notes

```
ticks_to_floppy_on

outb (mask, FD_DOR);
//这个命令不知道什么意思？？？？？？
这里边前四个bit是启动电机，后面四个bit是启动驱动FDC。

```

#5.caller

```
sys_open
read_super
--check_disk_change
```

