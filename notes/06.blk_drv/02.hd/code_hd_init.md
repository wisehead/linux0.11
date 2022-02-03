#1.hd_init

```cpp
// 硬盘系统初始化。
void hd_init (void)
{
    blk_dev[MAJOR_NR].request_fn = DEVICE_REQUEST;  // do_hd_request()。
    set_intr_gate (0x2E, &hd_interrupt);    // 设置硬盘中断门向量 int 0x2E(46)。
// hd_interrupt 在(kernel/system_call.s,221)。
    outb_p (inb_p (0x21) & 0xfb, 0x21); // 复位接联的主8259A int2 的屏蔽位，允许从片
// 发出中断请求信号。
    outb (inb_p (0xA1) & 0xbf, 0xA1);   // 复位硬盘的中断请求屏蔽位（在从片上），允许
// 硬盘控制器发送中断请求信号。
}

```