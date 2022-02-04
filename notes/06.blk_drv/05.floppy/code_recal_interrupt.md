#1.recal_interrupt

```cpp
/*
* 特殊情况 - 用于意外中断（或复位）处理后。
*/
//// 软驱重新校正中断调用函数。
// 首先发送检测中断状态命令（无参数），如果返回结果表明出错，则置复位标志，否则复位重新
// 校正标志。然后再次执行软盘请求。
static void
recal_interrupt (void)
{
    output_byte (FD_SENSEI);    // 发送检测中断状态命令。
    if (result () != 2 || (ST0 & 0xE0) == 0x60) // 如果返回结果字节数不等于2 或命令
        reset = 1;          // 异常结束，则置复位标志。
    else                // 否则复位重新校正标志。
        recalibrate = 0;
    do_fd_request ();       // 执行软盘请求项。
}

```

#2.caller

```
do_fd_request
--if (recalibrate)
----recalibrate_floppy
```