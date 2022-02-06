#1.con_write

```cpp
//// 控制台写函数。
// 从终端对应的tty 写缓冲队列中取字符，并显示在屏幕上。
con_write
--
```

#2.caller

```cpp
// tty 数据结构的tty_table 数组。其中包含三个初始化项数据，分别对应控制台、串口终端1 和
// 串口终端2 的初始化数据。
struct tty_struct tty_table[] = {
{
    {//termios
        ICRNL,          /* 将输入的CR 转换为NL */
        OPOST | ONLCR,      /* 将输出的NL 转CRNL */
        0,              // 控制模式标志初始化为0。
        ISIG | ICANON | ECHO | ECHOCTL | ECHOKE,    // 本地模式标志。
        0,              /* 控制台termio。 */
        INIT_C_CC           // 控制字符数组。
    },
    0,              /* 所属初始进程组。 */
    0,              /* 初始停止标志。 */
    con_write,          // tty 写函数指针。
    {0, 0, 0, 0, ""},       /* console read-queue */// tty 控制台读队列。
    {0, 0, 0, 0, ""},       /* console write-queue */// tty 控制台写队列。
    {0, 0, 0, 0, ""}        /* console secondary queue */// tty 控制台辅助(第二)队列。
},
```