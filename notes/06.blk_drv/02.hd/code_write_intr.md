#1.write_intr

```cpp
//// 写扇区中断调用函数。在硬盘中断处理程序中被调用。
// 在写命令执行后，会产生硬盘中断信号，执行硬盘中断处理程序，此时在硬盘中断处理程序中调用的
// C 函数指针do_hd()已经指向write_intr()，因此会在写操作完成（或出错）后，执行该函数。
static void write_intr (void)
{
    if (win_result ())
    {               // 如果硬盘控制器返回错误信息，
        bad_rw_intr ();     // 则首先进行硬盘读写失败处理，
        do_hd_request ();       // 然后再次请求硬盘作相应(复位)处理，
        return;         // 然后返回（也退出了此次硬盘中断）。
    }
    if (--CURRENT->nr_sectors)
    {               // 否则将欲写扇区数减1，若还有扇区要写，则
        CURRENT->sector++;  // 当前请求起始扇区号+1，
        CURRENT->buffer += 512; // 调整请求缓冲区指针，
        do_hd = &write_intr;    // 置硬盘中断程序调用函数指针为write_intr()，
        port_write (HD_DATA, CURRENT->buffer, 256); // 再向数据寄存器端口写256 字节。
        return;         // 返回等待硬盘再次完成写操作后的中断处理。
    }
    end_request (1);        // 若全部扇区数据已经写完，则处理请求结束事宜，
    do_hd_request ();       // 执行其它硬盘请求操作。
}
```

#2.caller

```
do_hd_request
```