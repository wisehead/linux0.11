#1.bread_page

```cpp
/*
 * bread_page 一次读四个缓冲块内容读到内存指定的地址。它是一个完整的函数，
 * 因为同时读取四块可以获得速度上的好处，不用等着读一块，再读一块了。
 */
//// 读设备上一个页面（4 个缓冲块）的内容到内存指定的地址。
void bread_page(unsigned long address,int dev,int b[4])
{
    struct buffer_head * bh[4];
    int i;

// 循环执行4 次，读一页内容。
    for (i=0 ; i<4 ; i++)
        if (b[i]) {
// 取高速缓冲中指定设备和块号的缓冲区，如果该缓冲区数据无效则产生读设备请求。
            if (bh[i] = getblk(dev,b[i]))
                if (!bh[i]->b_uptodate)
                    ll_rw_block(READ,bh[i]);
        } else
            bh[i] = NULL;
// 将4 块缓冲区上的内容顺序复制到指定地址处。
    for (i=0 ; i<4 ; i++,address += BLOCK_SIZE)
        if (bh[i]) {
            wait_on_buffer(bh[i]);  // 等待缓冲区解锁(如果已被上锁的话)。
            if (bh[i]->b_uptodate)  // 如果该缓冲区中数据有效的话，则复制。
                COPYBLK(bh[i]->b_data,(char *)address);
            brelse(bh[i]);      // 释放该缓冲区。
        }
}

```