#1.file_write

```cpp
//// 文件写函数- 根据i 节点和文件结构信息，将用户数据写入指定设备。
// 由i 节点可以知道设备号，由filp 结构可以知道文件中当前读写指针位置。buf 指定用户态中
// 缓冲区的位置，count 为需要写入的字节数。返回值是实际写入的字节数，或出错号(小于0)。

file_write
// 如果是要向文件后添加数据，则将文件读写指针移到文件尾部。否则就将在文件读写指针处写入。
--if (filp->f_flags & O_APPEND)
----pos = inode->i_size;
--else
----pos = filp->f_pos;
// 若已写入字节数i 小于需要写入的字节数count，则循环执行以下操作。
--while (i<count) {
// 创建数据块号(pos/BLOCK_SIZE)在设备上对应的逻辑块，并返回在设备上的逻辑块号。如果逻辑
// 块号=0，则表示创建失败，退出循环。
        if (!(block = create_block(inode,pos/BLOCK_SIZE)))
            break;
// 根据该逻辑块号读取设备上的相应数据块，若出错则退出循环。
        if (!(bh=bread(inode->i_dev,block)))
            break;
// 求出文件读写指针在数据块中的偏移值c，将p 指向读出数据块缓冲区中开始读取的位置。置该
// 缓冲区已修改标志。
        c = pos % BLOCK_SIZE;
        p = c + bh->b_data;
        bh->b_dirt = 1;
// 从开始读写位置到块末共可写入c=(BLOCK_SIZE-c)个字节。若c 大于剩余还需写入的字节数
// (count-i)，则此次只需再写入c=(count-i)即可。
        c = BLOCK_SIZE-c;
        if (c > count-i) c = count-i;
// 文件读写指针前移此次需写入的字节数。如果当前文件读写指针位置值超过了文件的大小，则
// 修改i 节点中文件大小字段，并置i 节点已修改标志。
        pos += c;
        if (pos > inode->i_size) {
            inode->i_size = pos;
            inode->i_dirt = 1;
        }
// 已写入字节计数累加此次写入的字节数c。从用户缓冲区buf 中复制c 个字节到高速缓冲区中p
// 指向开始的位置处。然后释放该缓冲区。
        i += c;
        while (c-->0)
            *(p++) = get_fs_byte(buf++);
        brelse(bh);
--}//while

// 更改文件修改时间为当前时间。
    inode->i_mtime = CURRENT_TIME;
// 如果此次操作不是在文件尾添加数据，则把文件读写指针调整到当前读写位置，并更改i 节点修改
// 时间为当前时间。
--if (!(filp->f_flags & O_APPEND)) {
        filp->f_pos = pos;
        inode->i_ctime = CURRENT_TIME;
--}
    
```