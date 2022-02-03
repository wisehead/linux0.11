#1.drive_busy

```cpp
//// 等待硬盘就绪。也即循环等待主状态控制器忙标志位复位。若仅有就绪或寻道结束标志
// 置位，则成功，返回0。若经过一段时间仍为忙，则返回1。
static int drive_busy (void)
{
    unsigned int i;

    for (i = 0; i < 10000; i++) // 循环等待就绪标志位置位。
        if (READY_STAT == (inb_p (HD_STATUS) & (BUSY_STAT | READY_STAT)))
            break;
    i = inb (HD_STATUS);    // 再取主控制器状态字节。
    i &= BUSY_STAT | READY_STAT | SEEK_STAT;    // 检测忙位、就绪位和寻道结束位。
    if (i == READY_STAT | SEEK_STAT)    // 若仅有就绪或寻道结束标志，则返回0。
        return (0);
    printk ("HD controller times out\n\r"); // 否则等待超时，显示信息。并返回1。
    return (1);
}

```

#2.caller

```
- reset_controller
```