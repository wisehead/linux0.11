#1.seek_interrupt

```cpp
/*
* 该子程序是在每次软盘控制器寻道（或重新校正）中断后被调用的。注意
* "unexpected interrupt"(意外中断)子程序也会执行重新校正操作，但不在此地。
*/
//// 寻道处理中断调用函数。
// 首先发送检测中断状态命令，获得状态信息ST0 和磁头所在磁道信息。若出错则执行错误计数
// 检测处理或取消本次软盘操作请求项。否则根据状态信息设置当前磁道变量，然后调用函数
// setup_rw_floppy()设置DMA 并输出软盘读写命令和参数。
static void
seek_interrupt (void)
{
/* sense drive status *//* 检测中断状态 */
// 发送检测中断状态命令，该命令不带参数。返回结果信息两个字节：ST0 和磁头当前磁道号。
    output_byte (FD_SENSEI);
// 如果返回结果字节数不等于2，或者ST0 不为寻道结束，或者磁头所在磁道(ST1)不等于设定磁道，
// 则说明发生了错误，于是执行检测错误计数处理，然后继续执行软盘请求项，并退出。
    if (result () != 2 || (ST0 & 0xF8) != 0x20 || ST1 != seek_track)
    {
        bad_flp_intr ();
        do_fd_request ();
        return;
    }
    current_track = ST1;        // 设置当前磁道。
    setup_rw_floppy ();     // 设置DMA 并输出软盘操作命令和参数。
}
```