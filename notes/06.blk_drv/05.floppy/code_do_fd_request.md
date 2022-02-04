#1.do_fd_request

```cpp
//// 软盘读写请求项处理函数。

do_fd_request
// 如果复位标志已置位，则执行软盘复位操作，并返回。
--if (reset)
    {
        reset_floppy ();
        return;
    }
// 如果重新校正标志已置位，则执行软盘重新校正操作，并返回。
--if (recalibrate)
    {
        recalibrate_floppy ();
        return;
    }
// 检测请求项的合法性(参见kernel/blk_drv/blk.h,127)。
--INIT_REQUEST;
// 将请求项结构中软盘设备号中的软盘类型(MINOR(CURRENT->dev)>>2)作为索引取得软盘参数块。
--floppy = (MINOR (CURRENT->dev) >> 2) + floppy_type;    
// 设置读写起始扇区。因为每次读写是以块为单位（1 块2 个扇区），所以起始扇区需要起码比
// 磁盘总扇区数小2 个扇区。否则结束该次软盘请求项，执行下一个请求项。
--block = CURRENT->sector;    // 取当前软盘请求项中起始扇区号??block。
 
 // 求对应在磁道上的扇区号，磁头号，磁道号，搜寻磁道号（对于软驱读不同格式的盘）。                                             
--sector = block % floppy->sect;  // 起始扇区对每磁道扇区数取模，得磁道上扇区号。                            
--block /= floppy->sect;  // 起始扇区对每磁道扇区数取整，得起始磁道数。                                     
--head = block % floppy->head;    // 起始磁道数对磁头数取模，得操作的磁头号。                              
--track = block / floppy->head;   // 起始磁道数对磁头数取整，得操作的磁道号。                              
--seek_track = track << floppy->stretch;  // 相应于驱动器中盘类型进行调整，得寻道号。                      
// 如果寻道号与当前磁头所在磁道不同，则置需要寻道标志seek。                                                      
--if (seek_track != current_track)                                                     
----seek = 1;                                                                          
--sector++;           // 磁盘上实际扇区计数是从1 算起。                                              
--if (CURRENT->cmd == READ)   // 如果请求项中是读操作，则置软盘读命令码。                                  
----command = FD_READ;                                                                 
--else if (CURRENT->cmd == WRITE) // 如果请求项中是写操作，则置软盘写命令码。                              
----command = FD_WRITE;                                                                
--else                                                                                 
--panic ("do_fd_request: unknown command");                                            
// 添加定时器，用于指定驱动器到能正常运行所需延迟的时间（滴答数），当定时时间到时就调用                                          
// 函数floppy_on_interrupt()，                                                            
--add_timer (ticks_to_floppy_on (current_drive), &floppy_on_interrupt);                
```