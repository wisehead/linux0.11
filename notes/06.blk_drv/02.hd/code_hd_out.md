#1.hd_out

```cpp
//// 向硬盘控制器发送命令块（参见列表后的说明）。
// 调用参数：drive - 硬盘号(0-1)； nsect - 读写扇区数；
// sect - 起始扇区； head - 磁头号；
// cyl - 柱面号； cmd - 命令码；
// *intr_addr() - 硬盘中断处理程序中将调用的C 处理函数。
static void hd_out (unsigned int drive, unsigned int nsect, unsigned int sect,
            unsigned int head, unsigned int cyl, unsigned int cmd,
            void (*intr_addr) (void))
{
    register int port; //asm ("dx");    // port 变量对应寄存器dx。

    if (drive > 1 || head > 15) // 如果驱动器号(0,1)>1 或磁头号>15，则程序不支持。
        panic ("Trying to write bad sector");
    if (!controller_ready ())   // 如果等待一段时间后仍未就绪则出错，死机。
        panic ("HD controller not ready");
    do_hd = intr_addr;      // do_hd 函数指针将在硬盘中断程序中被调用。
    outb_p (hd_info[drive].ctl, HD_CMD);    // 向控制寄存器(0x3f6)输出控制字节。
    port = HD_DATA;     // 置dx 为数据寄存器端口(0x1f0)。
    outb_p (hd_info[drive].wpcom >> 2, ++port); // 参数：写预补偿柱面号(需除4)。
    outb_p (nsect, ++port); // 参数：读/写扇区总数。
    outb_p (sect, ++port);  // 参数：起始扇区。
    outb_p (cyl, ++port);       // 参数：柱面号低8 位。
    outb_p (cyl >> 8, ++port);  // 参数：柱面号高8 位。
    outb_p (0xA0 | (drive << 4) | head, ++port);    // 参数：驱动器号+磁头号。
    outb (cmd, ++port);     // 命令：硬盘控制命令。
}
--
```

#2.caller

```
- reset_hd
- do_hd_request
```
