#1.setup_rw_floppy


```cpp
//// 设置DMA 并输出软盘操作命令和参数（输出1 字节命令+ 0~7 字节参数）。
_inline void
setup_rw_floppy (void)
{
    setup_DMA ();           // 初始化软盘DMA 通道。
    do_floppy = rw_interrupt;   // 置软盘中断调用函数指针。
    output_byte (command);  // 发送命令字节。
    output_byte (head << 2 | current_drive);    // 发送参数（磁头号+驱动器号）。
    output_byte (track);        // 发送参数（磁道号）。
    output_byte (head);     // 发送参数（磁头号）。
    output_byte (sector);       // 发送参数（起始扇区号）。
    output_byte (2);        /* sector size = 512 */// 发送参数(字节数(N=2)512 字节)。
    output_byte (floppy->sect); // 发送参数（每磁道扇区数）。
    output_byte (floppy->gap);  // 发送参数（扇区间隔长度）。
    output_byte ((char)0xFF);       /* sector size (0xff when n!=0 ?) */
// 发送参数（当N=0 时，扇区定义的字节长度），这里无用。
// 若在发送命令和参数时发生错误，则继续执行下一软盘操作请求。
    if (reset)
        do_fd_request ();
}
```

#2.caller

```
- seek_interrupt
- transfer
```

#3.notes

读写软盘操作委托给DMA，减轻CPU负担，只需要注册回调函数。