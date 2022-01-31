#1.get_super

```cpp
//// 取指定设备的超级块。返回该超级块结构指针。
struct super_block *
get_super (int dev)
{
    struct super_block *s;

// 如果没有指定设备，则返回空指针。
    if (!dev)
        return NULL;
// s 指向超级块数组开始处。搜索整个超级块数组，寻找指定设备的超级块。
    s = 0 + super_block;
    while (s < NR_SUPER + super_block)
// 如果当前搜索项是指定设备的超级块，则首先等待该超级块解锁（若已经被其它进程上锁的话）。
// 在等待期间，该超级块有可能被其它设备使用，因此此时需再判断一次是否是指定设备的超级块，
// 如果是则返回该超级块的指针。否则就重新对超级块数组再搜索一遍，因此s 重又指向超级块数组
// 开始处。
    if (s->s_dev == dev)
    {
        wait_on_super (s);
        if (s->s_dev == dev)
            return s;
        s = 0 + super_block;
// 如果当前搜索项不是，则检查下一项。如果没有找到指定的超级块，则返回空指针。
    }
    else
        s++;
    return NULL;
}
```