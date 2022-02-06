#1.change_speed

```cpp

//// 修改传输速率。
// 参数：tty - 终端对应的tty 数据结构。
// 在除数锁存标志DLAB(线路控制寄存器位7)置位情况下，通过端口0x3f8 和0x3f9 向UART 分别写入
// 波特率因子低字节和高字节。
static void
change_speed (struct tty_struct *tty)
{
    unsigned short port, quot;

// 对于串口终端，其tty 结构的读缓冲队列data 字段存放的是串行端口号(0x3f8 或0x2f8)。
    if (!(port = tty->read_q.data))
        return;
// 从tty 的termios 结构控制模式标志集中取得设置的波特率索引号，据此从波特率因子数组中取得
// 对应的波特率因子值。CBAUD 是控制模式标志集中波特率位屏蔽码。
    quot = quotient[tty->termios.c_cflag & CBAUD];
    cli ();         // 关中断。
    outb_p (0x80, port + 3);    /* set DLAB */// 首先设置除数锁定标志DLAB。
    outb_p (quot & 0xff, port); /* LS of divisor */// 输出因子低字节。
    outb_p (quot >> 8, port + 1);   /* MS of divisor */// 输出因子高字节。
    outb (0x03, port + 3);  /* reset DLAB */// 复位DLAB。
    sti ();         // 开中断。
}
```

#2.caller

```
- set_termios
- set_termio
```