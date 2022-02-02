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
```