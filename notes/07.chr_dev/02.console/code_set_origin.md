#1.set_origin

```cpp
//// 设置滚屏起始显示内存地址。
set_origin (void)
// 首先选择显示控制数据寄存器r12，然后写入卷屏起始地址高字节。向右移动9 位，表示向右移动
// 8 位，再除以2(2 字节代表屏幕上1 字符)。是相对于默认显示内存操作的。
--outb_p (12, video_port_reg);
--outb_p ((unsigned char)(0xff & ((origin - video_mem_start) >> 9)), video_port_val);
// 再选择显示控制数据寄存器r13，然后写入卷屏起始地址底字节。向右移动1 位表示除以2。
--outb_p (13, video_port_reg);
--outb_p ((unsigned char)(0xff & ((origin - video_mem_start) >> 1)), video_port_val);

```

#2.caller

scrup