#1.do_rd_request

```cpp
// 执行虚拟盘(ramdisk)读写操作。程序结构与do_hd_request()类似(kernel/blk_drv/hd.c,294)。
do_rd_request
--INIT_REQUEST;           // 检测请求的合法性(参见kernel/blk_drv/blk.h,127)。
// 下面语句取得ramdisk 的起始扇区对应的内存起始位置和内存长度。
// 其中sector << 9 表示sector * 512，CURRENT 定义为(blk_dev[MAJOR_NR].current_request)。
--addr = rd_start + (CURRENT->sector << 9);
--len = CURRENT->nr_sectors << 9;
// 如果子设备号不为1 或者对应内存起始位置>虚拟盘末尾，则结束该请求，并跳转到repeat 处
// （定义在28 行的INIT_REQUEST 内开始处）。
--if ((MINOR (CURRENT->dev) != 1) || (addr + len > rd_start + rd_length))
--{
        end_request (0);
        goto repeat;
--}
// 如果是写命令(WRITE)，则将请求项中缓冲区的内容复制到addr 处，长度为len 字节。
--if (CURRENT->cmd == WRITE)
--{
        (void) memcpy (addr, CURRENT->buffer, len);
        // 如果是读命令(READ)，则将addr 开始的内容复制到请求项中缓冲区中，长度为len 字节。
--}
--else if (CURRENT->cmd == READ)
--{
        (void) memcpy (CURRENT->buffer, addr, len);
        // 否则显示命令不存在，死机。
--}
--else
        panic ("unknown ramdisk-command");
// 请求项成功后处理，置更新标志。并继续处理本设备的下一请求项。
--end_request (1);
--goto repeat;
```

#2.