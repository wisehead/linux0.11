#1.reset_hd

```cpp

//// 复位硬盘nr。首先复位（重新校正）硬盘控制器。然后发送硬盘控制器命令“建立驱动器参数”，
// 其中recal_intr()是在硬盘中断处理程序中调用的重新校正处理函数。
static void reset_hd (int nr)
{
    reset_controller ();
    hd_out (nr, hd_info[nr].sect, hd_info[nr].sect, hd_info[nr].head - 1,
            hd_info[nr].cyl, WIN_SPECIFY, &recal_intr);
}
```

#2.reset_controller

```cpp
//// 诊断复位（重新校正）硬盘控制器。
static void reset_controller (void)
{
    int i;

    outb (4, HD_CMD);       // 向控制寄存器端口发送控制字节(4-复位)。
    for (i = 0; i < 100; i++)
        nop ();         // 等待一段时间（循环空操作）。
    outb (hd_info[0].ctl & 0x0f, HD_CMD);   // 再发送正常的控制字节(不禁止重试、重读)。
    if (drive_busy ())      // 若等待硬盘就绪超时，则显示出错信息。
        printk ("HD-controller still busy\n\r");
    if ((i = inb (HD_ERROR)) != 1)  // 取错误寄存器，若不等于1（无错误）则出错。
        printk ("HD-controller reset failed: %02x\n\r", i);
}
```

#3.recal_intr

```cpp
//// 硬盘重新校正（复位）中断调用函数。在硬盘中断处理程序中被调用。
// 如果硬盘控制器返回错误信息，则首先进行硬盘读写失败处理，然后请求硬盘作相应(复位)处理。
static void recal_intr (void)
{
    if (win_result ())
        bad_rw_intr ();
    do_hd_request ();
}

```