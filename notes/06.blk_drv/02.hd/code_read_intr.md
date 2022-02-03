#1.read_intr

```cpp
//// 读操作中断调用函数。将在执行硬盘中断处理程序中被调用。

read_intr
--if (win_result ())
--{               // 若控制器忙、读写错或命令执行错，
        bad_rw_intr ();     // 则进行读写硬盘失败处理
        do_hd_request ();       // 然后再次请求硬盘作相应(复位)处理。
        return;
--}
--port_read (HD_DATA, CURRENT->buffer, 256);  // 将数据从数据寄存器口读到请求结构缓冲区。            
--CURRENT->errors = 0;        // 清出错次数。                                          
--CURRENT->buffer += 512; // 调整缓冲区指针，指向新的空区。                                     
--CURRENT->sector++;      // 起始扇区号加1，                                            
--if (--CURRENT->nr_sectors)                                                     
--{               // 如果所需读出的扇区数还没有读完，则                                           
----do_hd = &read_intr; // 再次置硬盘调用C 函数指针为read_intr()                             
----return;         // 因为硬盘中断处理程序每次调用do_hd 时                                     
--}               // 都会将该函数指针置空。参见system_call.s                                  
--end_request (1);        // 若全部扇区数据已经读完，则处理请求结束事宜，                              
--do_hd_request ();       // 执行其它硬盘请求操作。                                                                                                                     
```

#2.caller

```
do_hd_request
```