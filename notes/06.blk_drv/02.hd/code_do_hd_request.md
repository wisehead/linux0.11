#1.do_hd_request

```cpp

// 执行硬盘读写请求操作。
void do_hd_request (void)
--block = CURRENT->sector;    // 请求的起始扇区。
--block += hd[dev].start_sect;    // 将所需读的块对应到整个硬盘上的绝对扇区号。
--dev /= 5;           // 此时dev 代表硬盘号（0 或1）。
// 下面嵌入汇编代码用来从硬盘信息结构中根据起始扇区号和每磁道扇区数计算在磁道中的
// 扇区号(sec)、所在柱面号(cyl)和磁头号(head)。
--sec = hd_info[dev].sect;
--_asm {
        mov eax,block
        xor edx,edx
        mov ebx,sec
        div ebx
        mov block,eax
        mov sec,edx
    }
//__asm__ ("divl %4": "=a" (block), "=d" (sec):"" (block), "1" (0),
//     "r" (hd_info[dev].
//      sect));
--head = hd_info[dev].head;
--_asm {
        mov eax,block
        xor edx,edx
        mov ebx,head
        div ebx
        mov cyl,eax
        mov head,edx
    }
    sec++;
    nsect = CURRENT->nr_sectors;    // 欲读/写的扇区数。
// 如果当前请求是写扇区操作，则发送写命令，循环读取状态寄存器信息并判断请求服务标志                                    
// DRQ_STAT 是否置位。DRQ_STAT 是硬盘状态寄存器的请求服务位（include/linux/hdreg.h，27）。            
--if (CURRENT->cmd == WRITE)                                                                                                                              
----hd_out (dev, nsect, sec, head, cyl, WIN_WRITE, &write_intr);                                                                              
----port_write (HD_DATA, CURRENT->buffer, 256);                              
----// 如果当前请求是读硬盘扇区，则向硬盘控制器发送读扇区命令。                                                                                                                     
--else if (CURRENT->cmd == READ)                                                                                                                   
----hd_out (dev, nsect, sec, head, cyl, WIN_READ, &read_intr);                                                                                                
```