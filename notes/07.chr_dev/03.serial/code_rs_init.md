#1.rs_init

```cpp
//// 初始化串行中断程序和串行接口。
void
rs_init (void)
{
    set_intr_gate (0x24, rs1_interrupt);    // 设置串行口1 的中断门向量(硬件IRQ4 信号)。
    set_intr_gate (0x23, rs2_interrupt);    // 设置串行口2 的中断门向量(硬件IRQ3 信号)。
    init (tty_table[1].read_q.data);    // 初始化串行口1(.data 是端口号)。
    init (tty_table[2].read_q.data);    // 初始化串行口2。
    outb (inb_p (0x21) & 0xE7, 0x21);   // 允许主8259A 芯片的IRQ3，IRQ4 中断信号请求。
}
```

#2. init

```cpp
//// 初始化串行端口
// port: 串口1 - 0x3F8，串口2 - 0x2F8。
static void init (int port)
{
    outb_p (0x80, port + 3);    /* set DLAB of line control reg */
/* 设置线路控制寄存器的DLAB 位(位7) */
    outb_p (0x30, port);        /* LS of divisor (48 -> 2400 bps */
/* 发送波特率因子低字节，0x30->2400bps */
    outb_p (0x00, port + 1);    /* MS of divisor */
/* 发送波特率因子高字节，0x00 */
    outb_p (0x03, port + 3);    /* reset DLAB */
/* 复位DLAB 位，数据位为8 位 */
    outb_p (0x0b, port + 4);    /* set DTR,RTS, OUT_2 */
/* 设置DTR，RTS，辅助用户输出2 */
    outb_p (0x0d, port + 1);    /* enable all intrs but writes */
/* 除了写(写保持空)以外，允许所有中断源中断 */
    (void) inb (port);      /* read data port to reset things (?) */
/* 读数据口，以进行复位操作(?) */
}
```