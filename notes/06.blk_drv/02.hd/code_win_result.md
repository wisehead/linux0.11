#1.win_result

```cpp
//// 检测硬盘执行命令后的状态。(win_表示温切斯特硬盘的缩写)
// 读取状态寄存器中的命令执行结果状态。返回0 表示正常，1 出错。如果执行命令错，
// 则再读错误寄存器HD_ERROR(0x1f1)。
static int win_result (void)
{
    int i = inb_p (HD_STATUS);  // 取状态信息。

    if ((i & (BUSY_STAT | READY_STAT | WRERR_STAT | SEEK_STAT | ERR_STAT))
        == (READY_STAT | SEEK_STAT))
        return (0);     /* ok */
    if (i & 1)
        i = inb (HD_ERROR); // 若ERR_STAT 置位，则读取错误寄存器。
    return (1);
}
```