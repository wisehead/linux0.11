#1.floppy_init

```cpp
//// 软盘系统初始化。
// 设置软盘块设备的请求处理函数(do_fd_request())，并设置软盘中断门(int 0x26，对应硬件
// 中断请求信号IRQ6），然后取消对该中断信号的屏蔽，允许软盘控制器FDC 发送中断请求信号。
void
floppy_init (void)
{
    blk_dev[MAJOR_NR].request_fn = DEVICE_REQUEST;  // = do_fd_request()。
    set_trap_gate (0x26, &floppy_interrupt);    //设置软盘中断门 int 0x26(38)。
    outb (inb_p (0x21) & ~0x40, 0x21);  // 复位软盘的中断请求屏蔽位，允许
                                        // 软盘控制器发送中断请求信号。
}
```