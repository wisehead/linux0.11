#1.do_tty_interrupt

```cpp

/*
* 呵，有时我是真得很喜欢386。该子程序是从一个中断处理程序中调用的，即使在
* 中断处理程序中睡眠也应该绝对没有问题(我希望如此)。当然，如果有人证明我是
* 错的，那么我将憎恨intel 一辈子?。但是我们必须小心，在调用该子程序之前需
* 要恢复中断。
*
* 我不认为在通常环境下会处在这里睡眠，这样很好，因为任务睡眠是完全任意的。
*/
//// tty 中断处理调用函数 - 执行tty 中断处理。
// 参数：tty - 指定的tty 终端号（0，1 或2）。
// 将指定tty 终端队列缓冲区中的字符复制成规范(熟)模式字符并存放在辅助队列(规范模式队列)中。
// 在串口读字符中断(rs_io.s, 109)和键盘中断(kerboard.S, 69)中调用。
void do_tty_interrupt (int tty)
{
    copy_to_cooked (tty_table + tty);
}
```

#2.caller
rs_io.s

```
read_char:
    in al,dx ;// 读取字符->al。
    mov edx,ecx ;// 当前串口缓冲队列指针地址??edx。
    sub edx,_table_list ;// 缓冲队列指针表首址 - 当前串口队列指针地址??edx，
    shr edx,3 ;// 差值/8。对于串口1 是1，对于串口2 是2。
    mov ecx,[ecx] ;// read-queue # 取读缓冲队列结构地址??ecx。
    mov ebx,head[ecx] ;// 取读队列中缓冲头指针??ebx。
    mov buf[ebx+ecx],al ;// 将字符放在缓冲区中头指针所指的位置。
    inc ebx ;// 将头指针前移一字节。
    and ebx,bsize-1 ;// 用缓冲区大小对头指针进行模操作。指针不能超过缓冲区大小。
    cmp ebx,tail[ecx] ;// 缓冲区头指针与尾指针比较。
    je l1 ;// 若相等，表示缓冲区满，跳转到标号1 处。
    mov head[ecx],ebx ;// 保存修改过的头指针。
l1: push edx ;// 将串口号压入堆栈(1- 串口1，2 - 串口2)，作为参数，
    call _do_tty_interrupt ;// 调用tty 中断处理C 函数（。
    add esp,4 ;// 丢弃入栈参数，并返回。
    ret
```