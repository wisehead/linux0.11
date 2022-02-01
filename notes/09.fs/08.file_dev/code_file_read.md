#1.file_read

```cpp
//// 文件读函数- 根据i 节点和文件结构，读设备数据。
// 由i 节点可以知道设备号，由filp 结构可以知道文件中当前读写指针位置。buf 指定用户态中
// 缓冲区的位置，count 为需要读取的字节数。返回值是实际读取的字节数，或出错号(小于0)。

file_read
// 若还需要读取的字节数不等于0，就循环执行以下操作，直到全部读出。
--while (left) {
// 根据i 节点和文件表结构信息，取数据块文件当前读写位置在设备上对应的逻辑块号nr。若nr 不
// 为0，则从i 节点指定的设备上读取该逻辑块，如果读操作失败则退出循环。若nr 为0，表示指定
// 的数据块不存在，置缓冲块指针为NULL。
        if (nr = bmap(inode,(filp->f_pos)/BLOCK_SIZE)) {
            if (!(bh=bread(inode->i_dev,nr)))
                break;
        } else
            bh = NULL;
// 计算文件读写指针在数据块中的偏移值nr，则该块中可读字节数为(BLOCK_SIZE-nr)，然后与还需
// 读取的字节数left 作比较，其中小值即为本次需读的字节数chars。若(BLOCK_SIZE-nr)大则说明
// 该块是需要读取的最后一块数据，反之则还需要读取一块数据。
        nr = filp->f_pos % BLOCK_SIZE;
        chars = MIN( BLOCK_SIZE-nr , left );
// 调整读写文件指针。指针前移此次将读取的字节数chars。剩余字节计数相应减去chars。
        filp->f_pos += chars;
        left -= chars;
// 若从设备上读到了数据，则将p 指向读出数据块缓冲区中开始读取的位置，并且复制chars 字节
// 到用户缓冲区buf 中。否则往用户缓冲区中填入chars 个0 值字节。
        if (bh) {
            char * p = nr + bh->b_data;
            while (chars-->0)
                put_fs_byte(*(p++),buf++);
            brelse(bh);
        } else {
            while (chars-->0)
                put_fs_byte(0,buf++);
        }
--}
// 修改该i 节点的访问时间为当前时间。返回读取的字节数，若读取字节数为0，则返回出错号。
--inode->i_atime = CURRENT_TIME;
```