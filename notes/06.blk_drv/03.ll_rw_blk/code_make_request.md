#1.make_request

```
//// 创建请求项并插入请求队列。参数是：主设备号major，命令rw，存放数据的缓冲区头指针bh。

make_request
--repeat:
/* 我们不能让队列中全都是写请求项：我们需要为读请求保留一些空间：读操作
* 是优先的。请求队列的后三分之一空间是为读准备的。
*/
// 请求项是从请求数组末尾开始搜索空项填入的。根据上述要求，对于读命令请求，可以直接
// 从队列末尾开始操作，而写请求则只能从队列的2/3 处向头上搜索空项填入。
--if (rw == READ)
        req = request + NR_REQUEST; // 对于读请求，将队列指针指向队列尾部。
--else
        req = request + ((NR_REQUEST * 2) / 3); // 对于写请求，队列指针指向队列2/3 处。
/* 搜索一个空请求项 */
// 从后向前搜索，当请求结构request 的dev 字段值=-1 时，表示该项未被占用。
--while (--req >= request)
        if (req->dev < 0)
            break;
            
/* 如果没有找到空闲项，则让该次新请求睡眠：需检查是否提前读/写 */                                        
// 如果没有一项是空闲的（此时request 数组指针已经搜索越过头部），则查看此次请求是否是                            
// 提前读/写（READA 或WRITEA），如果是则放弃此次请求。否则让本次请求睡眠（等待请求队列                         
// 腾出空项），过一会再来搜索请求队列。                                                       
--if (req < request)                                                        
--{               // 如果请求队列中没有空项，则                                          
    if (rw_ahead)                                                           
    {           // 如果是提前读/写请求，则解锁缓冲区，退出。                                    
        unlock_buffer (bh);                                                 
        return;                                                             
    }                                                                       
    sleep_on (&wait_for_request);   // 否则让本次请求睡眠，过会再查看请求队列。                 
    goto repeat;                                                            
--}                                                                         
/* 向空闲请求项中填写请求信息，并将其加入队列中 */                                                
// 请求结构参见（kernel/blk_drv/blk.h,23）。                                         
--req->dev = bh->b_dev;       // 设备号。                                       
--req->cmd = rw;      // 命令(READ/WRITE)。                                    
--req->errors = 0;        // 操作时产生的错误次数。                                    
--req->sector = bh->b_blocknr << 1;   // 起始扇区。(1 块=2 扇区)                    
--req->nr_sectors = 2;        // 读写扇区数。                                     
--req->buffer = bh->b_data;   // 数据缓冲区。                                     
--req->waiting = NULL;        // 任务等待操作执行完成的地方。                             
--req->bh = bh;           // 缓冲区头指针。                                        
--req->next = NULL;       // 指向下一请求项。                                       
--add_request (major + blk_dev, req); // 将请求项加入队列中(blk_dev[major],req)。     

```