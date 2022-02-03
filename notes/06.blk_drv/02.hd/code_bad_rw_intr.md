#1.bad_rw_intr

```cpp
//// 读写硬盘失败处理调用函数。
static void bad_rw_intr (void)
{
    if (++CURRENT->errors >= MAX_ERRORS)    // 如果读扇区时的出错次数大于或等于7 次时，
        end_request (0);        // 则结束请求并唤醒等待该请求的进程，而且
// 对应缓冲区更新标志复位（没有更新）。
    if (CURRENT->errors > MAX_ERRORS / 2)   // 如果读一扇区时的出错次数已经大于3 次，
        reset = 1;          // 则要求执行复位硬盘控制器操作。
}

```

#2.caller

```
- read_intr
- write_intr
- recal_intr
- do_hd_request
```