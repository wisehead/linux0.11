#1.get_termio

```cpp
//// 读取termio 结构中的信息。
// 参数：tty - 指定终端的tty 结构指针；termio - 用户数据区termio 结构缓冲区指针。
// 返回0。
static int
get_termio (struct tty_struct *tty, struct termio *termio)
{
    int i;
    struct termio tmp_termio;

// 首先验证一下用户的缓冲区指针所指内存区是否足够，如不够则分配内存。
    verify_area (termio, sizeof (*termio));
// 将termios 结构的信息复制到termio 结构中。目的是为了其中模式标志集的类型进行转换，也即
// 从termios 的长整数类型转换为termio 的短整数类型。
    tmp_termio.c_iflag = tty->termios.c_iflag;
    tmp_termio.c_oflag = tty->termios.c_oflag;
    tmp_termio.c_cflag = tty->termios.c_cflag;
    tmp_termio.c_lflag = tty->termios.c_lflag;
// 两种结构的c_line 和c_cc[]字段是完全相同的。
    tmp_termio.c_line = tty->termios.c_line;
    for (i = 0; i < NCC; i++)
        tmp_termio.c_cc[i] = tty->termios.c_cc[i];
// 最后复制指定tty 结构中的termio 结构信息到用户 termio 结构缓冲区。
    for (i = 0; i < (sizeof (*termio)); i++)
        put_fs_byte (((char *) &tmp_termio)[i], i + (char *) termio);
    return 0;
}

```