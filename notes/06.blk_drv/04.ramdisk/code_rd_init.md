#1.rd_init

```cpp
/* 返回内存虚拟盘ramdisk 所需的内存量 */
// 虚拟盘初始化函数。确定虚拟盘在内存中的起始地址，长度。并对整个虚拟盘区清零。
long
rd_init (long mem_start, int length)
{
    int i;
    char *cp;

    blk_dev[MAJOR_NR].request_fn = DEVICE_REQUEST;  // do_rd_request()。
    rd_start = (char *) mem_start;
    rd_length = length;
    cp = rd_start;
    for (i = 0; i < length; i++)
        *cp++ = '\0';
    return (length);
}

```

#2.caller

```
main
--main_memory_start += rd_init(main_memory_start, RAMDISK*1024);

main_memory_start = 1*1024*1024;
```