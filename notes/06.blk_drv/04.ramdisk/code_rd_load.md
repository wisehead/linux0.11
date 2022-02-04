#1.rd_load


```cpp
/*
* 如果根文件系统设备(root device)是ramdisk 的话，则尝试加载它。root device 原先是指向
* 软盘的，我们将它改成指向ramdisk。
*/
//// 加载根文件系统到ramdisk。

rd_load
--if (MAJOR (ROOT_DEV) != 2)  // 如果此时根文件设备不是软盘，则退出。
        return;
// 读软盘块256+1,256,256+2。breada()用于读取指定的数据块，并标出还需要读的块，然后返回
// 含有数据块的缓冲区指针。如果返回NULL，则表示数据块不可读(fs/buffer.c,322)。
// 这里block+1 是指磁盘上的超级块。
--bh = breada (ROOT_DEV, block + 1, block, block + 2, -1);

// 将s 指向缓冲区中的磁盘超级块。(d_super_block 磁盘中超级块结构)。
--*((struct d_super_block *) &s) = *((struct d_super_block *) bh->b_data);
--brelse (bh);            // [?? 为什么数据没有复制就立刻释放呢？]

// 块数 = 逻辑块数(区段数) * 2^(每区段块数的次方)。
// 如果数据块数大于内存中虚拟盘所能容纳的块数，则不能加载，显示出错信息并返回。否则显示
// 加载数据块信息。
--nblocks = s.s_nzones << s.s_log_zone_size;

// cp 指向虚拟盘起始处，然后将磁盘上的根文件系统映象文件复制到虚拟盘上。                                                    
--cp = rd_start;                                                                           
--while (nblocks)                                                                          
--{                                                                                        
        if (nblocks > 2)        // 如果需读取的块数多于3 快则采用超前预读方式读数据块。                             
            bh = breada (ROOT_DEV, block, block + 1, block + 2, -1);                       
        else            // 否则就单块读取。                                                        
            bh = bread (ROOT_DEV, block);                                                  
        if (!bh)                                                                           
        {                                                                                  
            printk ("I/O error on block %d, aborting load\n", block);                      
            return;                                                                        
        }                                                                                  
        (void) memcpy (cp, bh->b_data, BLOCK_SIZE); // 将缓冲区中的数据复制到cp 处。                    
        brelse (bh);        // 释放缓冲区。                                                      
        printk ("\010\010\010\010\010%4dk", i); // 打印加载块计数值。                               
        cp += BLOCK_SIZE;       // 虚拟盘指针前移。                                                
        block++;                                                                           
        nblocks--;                                                                         
        i++;                                                                               
----}                                                                                      
----printk ("\010\010\010\010\010done \n");                                                
----ROOT_DEV = 0x0101;      // 修改ROOT_DEV 使其指向虚拟盘ramdisk。                                                                                                                        
```

#2.caller

```
sys_setup
```