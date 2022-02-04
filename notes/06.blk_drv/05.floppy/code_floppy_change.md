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