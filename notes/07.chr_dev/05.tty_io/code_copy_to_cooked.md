#1.copy_to_cooked

```cpp
//// 复制成规范模式字符序列。
// 将指定tty 终端队列缓冲区中的字符复制成规范(熟)模式字符并存放在辅助队列(规范模式队列)中。
// 参数：tty - 指定终端的tty 结构。

将read_q中的数据放入secondary_q
```

#2.caller

```
- respond
- do_tty_interrupt
```

#3.notes

```
tty_read: 读取second --> buffer
tty_write:buffer --> tty->write_q，然后写入显存屏幕
copy_to_cooked:  read_q --> secondary

中断，keyboard中断处理函数：
read_char（rs_io.s）：
放入read_q

```