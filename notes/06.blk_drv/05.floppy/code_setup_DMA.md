#1.setup_DMA


```cpp
//// 设置（初始化）软盘DMA 通道。
static void
setup_DMA (void)
{
    long addr = (long) CURRENT->buffer; // 当前请求项缓冲区所处内存中位置（地址）。

    cli ();
// 如果缓冲区处于内存1M 以上的地方，则将DMA 缓冲区设在临时缓冲区域(tmp_floppy_area 数组)
// (因为8237A 芯片只能在1M 地址范围内寻址)。如果是写盘命令，则还需将数据复制到该临时区域。
    if (addr >= 0x100000)
    {
        addr = (long) tmp_floppy_area;
        if (command == FD_WRITE)
            copy_buffer (CURRENT->buffer, tmp_floppy_area);
    }
/* mask DMA 2 *//* 屏蔽DMA 通道2 */
// 单通道屏蔽寄存器端口为0x10。位0-1 指定DMA 通道(0--3)，位2：1 表示屏蔽，0 表示允许请求。
    immoutb_p (4 | 2, 10);
/* 输出命令字节。我是不知道为什么，但是每个人（minix，*/
/* sanches 和canton）都输出两次，首先是12 口，然后是11 口 */
// 下面嵌入汇编代码向DMA 控制器端口12 和11 写方式字（读盘0x46，写盘0x4A）。
    if (command == FD_READ)
        _asm mov al,DMA_READ;
    else
        _asm mov al,DMA_WRITE;
    _asm {
        out 12,al
        jmp l1
    l1: jmp l2
    l2: out 11,al
        jmp l3
    l3: jmp l4
    l4:
    }
//  __asm__ ("outb %%al,$12\n\tjmp 1f\n1:\tjmp 1f\n1:\t"
//     "outb %%al,$11\n\tjmp 1f\n1:\tjmp 1f\n1:"::
//     "a" ((char) ((command == FD_READ) ? DMA_READ : DMA_WRITE)));
/* 8 low bits of addr *//* 地址低0-7 位 */
// 向DMA 通道2 写入基/当前地址寄存器（端口4）。
    immoutb_p ((unsigned char)addr, 4);
    addr >>= 8;
/* bits 8-15 of addr *//* 地址高8-15 位 */
    immoutb_p ((unsigned char)addr, 4);
    addr >>= 8;
/* bits 16-19 of addr *//* 地址16-19 位 */
// DMA 只可以在1M 内存空间内寻址，其高16-19 位地址需放入页面寄存器(端口0x81)。
    immoutb_p ((unsigned char)addr, 0x81);
/* low 8 bits of count-1 (1024-1=0x3ff) *//* 计数器低8 位(1024-1=0x3ff) */
// 向DMA 通道2 写入基/当前字节计数器值（端口5）。
    immoutb_p (0xff, 5);
/* high 8 bits of count-1 *//* 计数器高8 位 */
// 一次共传输1024 字节（两个扇区）。
    immoutb_p (3, 5);
/* activate DMA 2 *//* 开启DMA 通道2 的请求 */
// 复位对DMA 通道2 的屏蔽，开放DMA2 请求DREQ 信号。
    immoutb_p (0 | 2, 10);
    sti ();
}    
```

#2.caller

```
setup_rw_floppy
```