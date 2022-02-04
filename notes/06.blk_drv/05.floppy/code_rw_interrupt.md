#1.rw_interrupt

```cpp
/*
* OK，下面该中断处理函数是在DMA 读/写成功后调用的，这样我们就可以检查执行结果，
* 并复制缓冲区中的数据。
*/
//// 软盘读写操作成功中断调用函数。。
static void
rw_interrupt (void)
{
// 如果返回结果字节数不等于7，或者状态字节0、1 或2 中存在出错标志，则若是写保护
// 就显示出错信息，释放当前驱动器，并结束当前请求项。否则就执行出错计数处理。
// 然后继续执行软盘请求操作。
// ( 0xf8 = ST0_INTR | ST0_SE | ST0_ECE | ST0_NR )
// ( 0xbf = ST1_EOC | ST1_CRC | ST1_OR | ST1_ND | ST1_WP | ST1_MAM，应该是0xb7)
// ( 0x73 = ST2_CM | ST2_CRC | ST2_WC | ST2_BC | ST2_MAM )
    if (result () != 7 || (ST0 & 0xf8) || (ST1 & 0xbf) || (ST2 & 0x73))
    {
        if (ST1 & 0x02)
        {           // 0x02 = ST1_WP - Write Protected。
            printk ("Drive %d is write protected\n\r", current_drive);
            floppy_deselect (current_drive);
            end_request (0);
        }
        else
            bad_flp_intr ();
        do_fd_request ();
        return;
    }
// 如果当前请求项的缓冲区位于1M 地址以上，则说明此次软盘读操作的内容还放在临时缓冲区内，
// 需要复制到请求项的缓冲区中（因为DMA 只能在1M 地址范围寻址）。
    if (command == FD_READ && (unsigned long) (CURRENT->buffer) >= 0x100000)
        copy_buffer (tmp_floppy_area, CURRENT->buffer);
// 释放当前软盘，结束当前请求项（置更新标志），再继续执行其它软盘请求项。
    floppy_deselect (current_drive);
    end_request (1);
    do_fd_request ();
}
```

#2.caller

```cpp
setup_rw_floppy
--do_floppy = rw_interrupt;   // 置软盘中断调用函数指针。
```