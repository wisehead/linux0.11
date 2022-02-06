#1.con_init

```cpp
/*
* void con_init(void);
* 这个子程序初始化控制台中断，其它什么都不做。如果你想让屏幕干净的话，就使用
* 适当的转义字符序列调用tty_write()函数。
*
* 读取setup.s 程序保存的信息，用以确定当前显示器类型，并且设置所有相关参数。
*/
void
con_init (void)
{
    register unsigned char a;
    char *display_desc = "????";
    char *display_ptr;

    video_num_columns = ORIG_VIDEO_COLS;    // 显示器显示字符列数。
    video_size_row = video_num_columns * 2; // 每行需使用字节数。
    video_num_lines = ORIG_VIDEO_LINES; // 显示器显示字符行数。
    video_page = (unsigned char)ORIG_VIDEO_PAGE;    // 当前显示页面。
    video_erase_char = 0x0720;  // 擦除字符(0x20 显示字符， 0x07 是属性)。

// 如果原始显示模式等于7，则表示是单色显示器。
    if (ORIG_VIDEO_MODE == 7)   /* Is this a monochrome display? */
    {
        video_mem_start = 0xb0000;  // 设置单显映象内存起始地址。
        video_port_reg = 0x3b4; // 设置单显索引寄存器端口。
        video_port_val = 0x3b5; // 设置单显数据寄存器端口。
// 根据BIOS 中断int 0x10 功能0x12 获得的显示模式信息，判断显示卡单色显示卡还是彩色显示卡。
// 如果使用上述中断功能所得到的BX 寄存器返回值不等于0x10，则说明是EGA 卡。因此初始
// 显示类型为EGA 单色；所使用映象内存末端地址为0xb8000；并置显示器描述字符串为'EGAm'。
// 在系统初始化期间显示器描述字符串将显示在屏幕的右上角。
        if ((ORIG_VIDEO_EGA_BX & 0xff) != 0x10)
        {
            video_type = VIDEO_TYPE_EGAM;   // 设置显示类型(EGA 单色)。
            video_mem_end = 0xb8000;    // 设置显示内存末端地址。
            display_desc = "EGAm";  // 设置显示描述字符串。
        }
// 如果BX 寄存器的值等于0x10，则说明是单色显示卡MDA。则设置相应参数。
        else
        {
            video_type = VIDEO_TYPE_MDA;    // 设置显示类型(MDA 单色)。
            video_mem_end = 0xb2000;    // 设置显示内存末端地址。
            display_desc = "*MDA";  // 设置显示描述字符串。
        }
    }
// 如果显示模式不为7，则为彩色模式。此时所用的显示内存起始地址为0xb800；显示控制索引寄存
// 器端口地址为0x3d4；数据寄存器端口地址为0x3d5。
    else                /* If not, it is color. */
    {
        video_mem_start = 0xb8000;  // 显示内存起始地址。
        video_port_reg = 0x3d4; // 设置彩色显示索引寄存器端口。
        video_port_val = 0x3d5; // 设置彩色显示数据寄存器端口。
// 再判断显示卡类别。如果BX 不等于0x10，则说明是EGA 显示卡。
        if ((ORIG_VIDEO_EGA_BX & 0xff) != 0x10)
        {
            video_type = VIDEO_TYPE_EGAC;   // 设置显示类型(EGA 彩色)。
            video_mem_end = 0xbc000;    // 设置显示内存末端地址。
            display_desc = "EGAc";  // 设置显示描述字符串。
        }
// 如果BX 寄存器的值等于0x10，则说明是CGA 显示卡。则设置相应参数。
        else
        {
            video_type = VIDEO_TYPE_CGA;    // 设置显示类型(CGA)。
            video_mem_end = 0xba000;    // 设置显示内存末端地址。
            display_desc = "*CGA";  // 设置显示描述字符串。
        }
    }

/* Let the user known what kind of display driver we are using */
/* 让用户知道我们正在使用哪一类显示驱动程序 */

// 在屏幕的右上角显示显示描述字符串。采用的方法是直接将字符串写到显示内存的相应位置处。
// 首先将显示指针display_ptr 指到屏幕第一行右端差4 个字符处(每个字符需2 个字节，因此减8)。
    display_ptr = ((char *) video_mem_start) + video_size_row - 8;
// 然后循环复制字符串中的字符，并且每复制一个字符都空开一个属性字节。
    while (*display_desc)
    {
        *display_ptr++ = *display_desc++;   // 复制字符。
        display_ptr++;      // 空开属性字节位置。
    }

/* 初始化用于滚屏的变量(主要用于EGA/VGA) */

    origin = video_mem_start;   // 滚屏起始显示内存地址。
    scr_end = video_mem_start + video_num_lines * video_size_row;   // 滚屏结束内存地址。
    top = 0;            // 最顶行号。
    bottom = video_num_lines;   // 最底行号。

    gotoxy (ORIG_X, ORIG_Y);    // 初始化光标位置x,y 和对应的内存位置pos。
    set_trap_gate (0x21, &keyboard_interrupt);  // 设置键盘中断陷阱门。
    outb_p ((unsigned char)(inb_p (0x21) & 0xfd), 0x21);    // 取消8259A 中对键盘中断的屏蔽，允许IRQ1。
    a = inb_p (0x61);       // 延迟读取键盘端口0x61(8255A 端口PB)。
    outb_p ((unsigned char)(a | 0x80), 0x61);   // 设置禁止键盘工作(位7 置位)，
    outb (a, 0x61);     // 再允许键盘工作，用以复位键盘操作。
}    
```