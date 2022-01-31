#1.mount_root

```cpp
//// 安装根文件系统。
// 该函数是在系统开机初始化设置时(sys_setup())调用的。( kernel/blk_drv/hd.c, 157 )

mount_root
// 初始化文件表数组（共64 项，也即系统同时只能打开64 个文件），将所有文件结构中的引用计数
// 设置为0。[??为什么放在这里初始化？]
--for (i = 0; i < NR_FILE; i++)
----file_table[i].f_count = 0;
// 如果根文件系统所在设备是软盘的话，就提示“插入根文件系统盘，并按回车键”，并等待按键。
--if (MAJOR (ROOT_DEV) == 2)
    {
        printk ("Insert root floppy and press ENTER");
        wait_for_keypress ();
    }
// 初始化超级块数组（共8 项）。
--for (p = &super_block[0]; p < &super_block[NR_SUPER]; p++)
    {
        p->s_dev = 0;
        p->s_lock = 0;
        p->s_wait = NULL;
    }   
// 如果读根设备上超级块失败，则显示信息，并死机。
--read_super (ROOT_DEV)   
//从设备上读取文件系统的根i 节点(1)，如果失败则显示出错信息，死机。
--mi = iget (ROOT_DEV, ROOT_INO)
// 该i 节点引用次数递增3 次。因为下面266-268 行上也引用了该i 节点。
--mi->i_count += 3;       /* NOTE! it is logically used 4 times, not 1 */
/* 注意！从逻辑上讲，它已被引用了4 次，而不是1 次 */
// 置该超级块的被安装文件系统i 节点和被安装到的i 节点为该i 节点。
--p->s_isup = p->s_imount = mi;
// 设置当前进程的当前工作目录和根目录i 节点。此时当前进程是1 号进程。
--current->pwd = mi;
--current->root = mi;
// 统计该设备上空闲块数。首先令i 等于超级块中表明的设备逻辑块总数。
    free = 0;
    i = p->s_nzones;
// 然后根据逻辑块位图中相应比特位的占用情况统计出空闲块数。这里宏函数set_bit()只是在测试
// 比特位，而非设置比特位。"i&8191"用于取得i 节点号在当前块中的偏移值。"i>>13"是将i 除以
// 8192，也即除一个磁盘块包含的比特位数。
    while (--i >= 0)
        if (!set_bit (i & 8191, p->s_zmap[i >> 13]->b_data))
            free++;
// 显示设备上空闲逻辑块数/逻辑块总数。
    printk ("%d/%d free blocks\n\r", free, p->s_nzones);
// 统计设备上空闲i 节点数。首先令i 等于超级块中表明的设备上i 节点总数+1。加1 是将0 节点
// 也统计进去。
    free = 0;
    i = p->s_ninodes + 1;
// 然后根据i 节点位图中相应比特位的占用情况计算出空闲i 节点数。
    while (--i >= 0)
        if (!set_bit (i & 8191, p->s_imap[i >> 13]->b_data))
            free++;
// 显示设备上可用的空闲i 节点数/i 节点总数。
    printk ("%d/%d free inodes\n\r", free, p->s_ninodes);
    
```

#2.caller

```
- sys_setup
```