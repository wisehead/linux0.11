#1.sys_setup

```cpp
/* This may be used only once, enforced by 'static int callable' */
/* 下面该函数只在初始化时被调用一次。用静态变量callable 作为可调用标志。*/
// 该函数的参数由初始化程序init/main.c 的init 子程序设置为指向0x90080 处，此处存放着setup.s
// 程序从BIOS 取得的2 个硬盘的基本参数表(32 字节)。硬盘参数表信息参见下面列表后的说明。
// 本函数主要功能是读取CMOS 和硬盘参数表信息，用于设置硬盘分区结构hd，并加载RAM 虚拟盘和
// 根文件系统。

sys_setup
// 如果没有在config.h 中定义硬盘参数，就从0x90080 处读入。
#ifndef HD_TYPE
--for (drive = 0; drive < 2; drive++)
 --{
        hd_info[drive].cyl = *(unsigned short *) BIOS;  // 柱面数。
        hd_info[drive].head = *(unsigned char *) (2 + BIOS);    // 磁头数。
        hd_info[drive].wpcom = *(unsigned short *) (5 + BIOS);  // 写前预补偿柱面号。
        hd_info[drive].ctl = *(unsigned char *) (8 + BIOS); // 控制字节。
        hd_info[drive].lzone = *(unsigned short *) (12 + BIOS); // 磁头着陆区柱面号。
        hd_info[drive].sect = *(unsigned char *) (14 + BIOS);   // 每磁道扇区数。
        BIOS += 16;     // 每个硬盘的参数表长16 字节，这里BIOS 指向下一个表。
--}

// 设置每个硬盘的起始扇区号和扇区总数。其中编号i*5 含义参见本程序后的有关说明。
--for (i = 0; i < NR_HD; i++)
--{
        hd[i * 5].start_sect = 0;   // 硬盘起始扇区号。
        hd[i * 5].nr_sects = hd_info[i].head * hd_info[i].sect * hd_info[i].cyl;    // 硬盘总扇区数。
--}

// 读取每一个硬盘上第1 块数据（第1 个扇区有用），获取其中的分区表信息。
// 首先利用函数bread()读硬盘第1 块数据(fs/buffer.c,267)，参数中的0x300 是硬盘的主设备号
// (参见列表后的说明)。然后根据硬盘头1 个扇区位置0x1fe 处的两个字节是否为'55AA'来判断
// 该扇区中位于0x1BE 开始的分区表是否有效。最后将分区表信息放入硬盘分区数据结构hd 中。
--for (drive = 0; drive < NR_HD; drive++)
--{
        if (!(bh = bread (0x300 + drive * 5, 0)))
        {           // 0x300, 0x305 逻辑设备号。
            printk ("Unable to read partition table of drive %d\n\r", drive);
            panic ("");
        }
        if (bh->b_data[510] != 0x55 || (unsigned char) bh->b_data[511] != 0xAA)
        {           // 判断硬盘信息有效标志'55AA'。
            printk ("Bad partition table on drive %d\n\r", drive);
            panic ("");
        }
        p = (struct partition *)(0x1BE + bh->b_data);   // 分区表位于硬盘第1 扇区的0x1BE 处。
        for (i = 1; i < 5; i++, p++)
        {
            hd[i + 5 * drive].start_sect = p->start_sect;
            hd[i + 5 * drive].nr_sects = p->nr_sects;
        }
        brelse (bh);        // 释放为存放硬盘块而申请的内存缓冲区页。
--}
--rd_load ();         // 加载（创建）RAMDISK(kernel/blk_drv/ramdisk.c,71)。
--mount_root ();      // 安装根文件系统(fs/super.c,242)。    
```