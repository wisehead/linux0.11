#1.transfer

```cpp
/*
* 该函数是在传输操作的所有信息都正确设置好后被调用的（也即软驱马达已开启
* 并且已选择了正确的软盘（软驱）。
*/
//// 读写数据传输函数。
static void
transfer (void)
{
// 首先看当前驱动器参数是否就是指定驱动器的参数，若不是就发送设置驱动器参数命令及相应
// 参数（参数1：高4 位步进速率，低四位磁头卸载时间；参数2：磁头加载时间）。
    if (cur_spec1 != floppy->spec1)
    {
        cur_spec1 = floppy->spec1;
        output_byte (FD_SPECIFY);   // 发送设置磁盘参数命令。
        output_byte (cur_spec1);    /* hut etc */// 发送参数。
        output_byte (6);        /* Head load time =6ms, DMA */
    }
// 判断当前数据传输速率是否与指定驱动器的一致，若不是就发送指定软驱的速率值到数据传输
// 速率控制寄存器(FD_DCR)。
    if (cur_rate != floppy->rate)
        outb_p (cur_rate = floppy->rate, FD_DCR);
// 若返回结果信息表明出错，则再调用软盘请求函数，并返回。
    if (reset)
    {
        do_fd_request ();
        return;
    }
// 若寻道标志为零（不需要寻道），则设置DMA 并发送相应读写操作命令和参数，然后返回。
    if (!seek)
    {
        setup_rw_floppy ();
        return;
    }
// 否则执行寻道处理。置软盘中断处理调用函数为寻道中断函数。
    do_floppy = seek_interrupt;
// 如果器始磁道号不等于零则发送磁头寻道命令和参数
    if (seek_track)
    {
        output_byte (FD_SEEK);  // 发送磁头寻道命令。
        output_byte (head << 2 | current_drive);    //发送参数：磁头号+当前软驱号。
        output_byte (seek_track);   // 发送参数：磁道号。
    }
    else
    {
        output_byte (FD_RECALIBRATE);   // 发送重新校正命令。
        output_byte (head << 2 | current_drive);    //发送参数：磁头号+当前软驱号。
    }
// 如果复位标志已置位，则继续执行软盘请求项。
    if (reset)
        do_fd_request ();
}

```

#2.caller

```
do_fd_request
--floppy_on_interrupt
```