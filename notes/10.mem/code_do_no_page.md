#1.do_no_page

```cpp
do_no_page
--address &= 0xfffff000;// 页面地址。
// 首先算出指定线性地址在进程空间中相对于进程基址的偏移长度值。//段内偏移的page number
--tmp = address - current->start_code;
// 若当前进程的executable 空，或者指定地址超出代码+数据长度，则申请一页物理内存，并映射
// 影射到指定的线性地址处。executable 是进程的i 节点结构。该值为0，表明进程刚开始设置，
// 需要内存；而指定的线性地址超出代码加数据长度，表明进程在申请新的内存空间，也需要给予。
// 因此就直接调用get_empty_page()函数，申请一页物理内存并映射到指定线性地址处即可。
// start_code 是进程代码段地址，end_data 是代码加数据长度。对于linux 内核，它的代码段和
// 数据段是起始基址是相同的。
--if (!current->executable || tmp >= current->end_data) {
        get_empty_page(address);
        return;
--}

// 如果尝试共享页面成功，则退出。
--if (share_page(tmp))
        return;
// 取空闲页面，如果内存不够了，则显示内存不够，终止进程。
--page = get_free_page()      

/* 记住，（程序）头要使用1 个数据块 */
// 首先计算缺页所在的数据块项。BLOCK_SIZE = 1024 字节，因此一页内存需要4 个数据块。
--block = 1 + tmp/BLOCK_SIZE;
// 根据i 节点信息，取数据块在设备上的对应的逻辑块号。
--for (i=0 ; i<4 ; block++,i++)
        nr[i] = bmap(current->executable,block);
// 读设备上一个页面的数据（4 个逻辑块）到指定物理地址page 处。
--bread_page(page,current->executable->i_dev,nr);
// 在增加了一页内存后，该页内存的部分可能会超过进程的end_data 位置。下面的循环即是对物理
// 页面超出的部分进行清零处理。
--i = tmp + 4096 - current->end_data;                                          
--tmp = page + 4096;                                                           
--while (i-- > 0) {                                                            
----tmp--;                                                                   
----*(char *)tmp = 0;                                                        
--}                                                                            
// 如果把物理页面映射到指定线性地址的操作成功，就返回。否则就释放内存页，显示内存不够。                                  
--if (put_page(page,address))                                                  
----return;                                                                  
--free_page(page);                                                             
--oom();                                                                       
```