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

#3. recalibrate_floppy

```cpp
//// 软盘重新校正处理函数。
// 向软盘控制器FDC 发送重新校正命令和参数，并复位重新校正标志。
static void
recalibrate_floppy (void)
{
    recalibrate = 0;        // 复位重新校正标志。
    current_track = 0;      // 当前磁道号归零。
    do_floppy = recal_interrupt;    // 置软盘中断调用函数指针指向重新校正调用函数。
    output_byte (FD_RECALIBRATE);   // 发送命令：重新校正。
    output_byte (head << 2 | current_drive);    // 发送参数：（磁头号加）当前驱动器号。
    if (reset)          // 如果出错(复位标志被置位)则继续执行软盘请求。
        do_fd_request ();
}
```