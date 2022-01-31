#1.iput

```cpp
//// 释放一个i 节点(回写入设备)。
iput
--wait_on_inode(inode);   // 等待inode 节点解锁(如果已上锁的话)。
// 如果是管道i 节点，则唤醒等待该管道的进程，引用次数减1，如果还有引用则返回。否则释放
// 管道占用的内存页面，并复位该节点的引用计数值、已修改标志和管道标志，并返回。
// 对于pipe 节点，inode->i_size 存放着物理内存页地址。参见get_pipe_inode()，228，234 行。
    if (inode->i_pipe) {
        wake_up(&inode->i_wait);
        if (--inode->i_count)
            return;
        free_page(inode->i_size);
        inode->i_count=0;
        inode->i_dirt=0;
        inode->i_pipe=0;
        return;
    }

// 如果是块设备文件的i 节点，此时逻辑块字段0 中是设备号，则刷新该设备。并等待i 节点解锁。 
--if (S_ISBLK(inode->i_mode))
----sync_dev(inode->i_zone[0]);
------for (i=0 ; i<NR_BUFFERS ; i++,bh++)
--------if (bh->b_dev != dev)
----------continue
--------wait_on_buffer(bh);
--------if (bh->b_dev == dev && bh->b_dirt)
----------ll_rw_block(WRITE,bh);
------sync_inodes
--------for(i=0 ; i<NR_INODE ; i++,inode++)
----------if (inode->i_dirt && !inode->i_pipe)    // 如果该i 节点已修改且不是管道节点，
------------write_inode(inode);                 // 则写盘。
--------------lock_inode(inode);
--------------block = 2 + sb->s_imap_blocks + sb->s_zmap_blocks +(inode->i_num-1)/INODES_PER_BLOCK;
--------------bh=bread(inode->i_dev,block)
--------------((struct d_inode *)bh->b_data)[(inode->i_num-1)%INODES_PER_BLOCK] =*(struct d_inode *)inode;
--------------bh->b_dirt=1;
--------------inode->i_dirt=0;
--------------brelse(bh);
--------------unlock_inode(inode);
----wait_on_inode(inode);   
--if (!inode->i_nlinks)
----truncate(inode);
----free_inode(inode);
--if (inode->i_dirt)
----write_inode(inode); /* we can sleep - so do again */
----wait_on_inode(inode);
----goto repeat;
--inode->i_count--;
```