#1.reset_interrupt

```cpp
//// 软盘控制器FDC 复位中断调用函数。在软盘中断处理程序中调用。
// 首先发送检测中断状态命令（无参数），然后读出返回的结果字节。接着发送设定软驱参数命令
// 和相关参数，最后再次调用执行软盘请求。
static void
reset_interrupt (void)
{
    output_byte (FD_SENSEI);    // 发送检测中断状态命令。
    (void) result ();       // 读取命令执行结果字节。
    output_byte (FD_SPECIFY);   // 发送设定软驱参数命令。
    output_byte (cur_spec1);    /* hut etc */// 发送参数。
    output_byte (6);        /* Head load time =6ms, DMA */
    do_fd_request ();       // 调用执行软盘请求。
}
```

#2.reset_floppy

```cpp
/* FDC 复位是通过将数字输出寄存器(DOR)位2 置0 一会儿实现的 */
//// 复位软盘控制器。
static void
reset_floppy (void)
{
    int i;

    reset = 0;          // 复位标志置0。
    cur_spec1 = -1;
    cur_rate = -1;
    recalibrate = 1;        // 重新校正标志置位。
    printk ("Reset-floppy called\n\r"); // 显示执行软盘复位操作信息。
    cli ();         // 关中断。
    do_floppy = reset_interrupt;    // 设置在软盘中断处理程序中调用的函数。
    outb_p (current_DOR & ~0x04, FD_DOR);   // 对软盘控制器FDC 执行复位操作。
    for (i = 0; i < 100; i++)   // 空操作，延迟。
        _asm nop;
    outb (current_DOR, FD_DOR); // 再启动软盘控制器。
    sti ();         // 开中断。
}

```

#3.caller

```cpp
do_fd_request
--// 如果复位标志已置位，则执行软盘复位操作，并返回。
    if (reset)
    {
        reset_floppy ();
        return;
    }
```