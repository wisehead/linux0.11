#1.respond

```cpp
//// 发送对终端VT100 的响应序列。
// 将响应序列放入读缓冲队列中。
static void
respond (struct tty_struct *tty)
{
    char *p = RESPONSE;

    cli ();         // 关中断。
    while (*p)
    {               // 将字符序列放入写队列。
        PUTCH (*p, tty->read_q);
        p++;
    }
    sti ();         // 开中断。
    copy_to_cooked (tty);       // 转换成规范模式(放入辅助队列中)。
}
```