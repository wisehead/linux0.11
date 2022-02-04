#1.result

```cpp
//// 读取FDC 执行的结果信息。
// 结果信息最多7 个字节，存放在reply_buffer[]中。返回读入的结果字节数，若返回值=-1
// 表示出错。
static int
result (void)
{
    int i = 0, counter, status;

    if (reset)
        return -1;
    for (counter = 0; counter < 10000; counter++)
    {
        status = inb_p (FD_STATUS) & (STATUS_DIR | STATUS_READY | STATUS_BUSY);
        if (status == STATUS_READY)
            return i;
        if (status == (STATUS_DIR | STATUS_READY | STATUS_BUSY))
        {
            if (i >= MAX_REPLIES)
                break;
            reply_buffer[i++] = inb_p (FD_DATA);
        }
    }
    reset = 1;
    printk ("Getstatus times out\n\r");
    return -1;
}

```


