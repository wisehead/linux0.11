#1.tty_ioctl

```cpp

//// tty 终端设备的ioctl 函数。
// 参数：dev - 设备号；cmd - ioctl 命令；arg - 操作参数指针。
int
tty_ioctl (int dev, int cmd, int arg)
```

#2.caller

```cpp
// ioctl 操作函数指针表。
static ioctl_ptr ioctl_table[]={
    NULL,       /* nodev */
    NULL,       /* /dev/mem */
    NULL,       /* /dev/fd */
    NULL,       /* /dev/hd */
    tty_ioctl,  /* /dev/ttyx */
    tty_ioctl,  /* /dev/tty */
    NULL,       /* /dev/lp */
    NULL        /* named pipes */
};

```

#3.sys_ioctl

```
sys_ioctl
// 否则返回实际ioctl 函数返回码，成功则返回0，否则返回出错码。
    return ioctl_table[MAJOR(dev)](dev,cmd,arg);
```