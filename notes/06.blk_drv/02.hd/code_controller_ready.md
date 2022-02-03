#1.controller_ready

```cpp

//// 判断并循环等待驱动器就绪。
// 读硬盘控制器状态寄存器端口HD_STATUS(0x1f7)，并循环检测驱动器就绪比特位和控制器忙位。
static int controller_ready (void)
{
    int retries = 10000;

    while (--retries && (inb_p (HD_STATUS) & 0xc0) != 0x40);
        return (retries);       // 返回等待循环的次数。
}
```

#2.caller

```
- hd_out
```