#1.bad_flp_intr

```cpp

//// 软盘操作出错中断调用函数。由软驱中断处理程序调用。
static void
bad_flp_intr (void)
{
    CURRENT->errors++;      // 当前请求项出错次数增1。
// 如果当前请求项出错次数大于最大允许出错次数，则取消选定当前软驱，并结束该请求项（不更新）。
    if (CURRENT->errors > MAX_ERRORS)
    {
        floppy_deselect (current_drive);
        end_request (0);
    }
// 如果当前请求项出错次数大于最大允许出错次数的一半，则置复位标志，需对软驱进行复位操作，
// 然后再试。否则软驱需重新校正一下，再试。
    if (CURRENT->errors > MAX_ERRORS / 2)
        reset = 1;
    else
        recalibrate = 1;
}
```