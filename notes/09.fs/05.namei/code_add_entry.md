#1.add_entry

```cpp
/*
 *  add_entry()
 *
 * 使用与find_entry()同样的方法，往指定目录中添加一文件目录项。
 * 如果失败则返回NULL。
 *
 * 注意！！'de'(指定目录项结构指针)的i 节点部分被设置为0 - 这表示
 * 在调用该函数和往目录项中添加信息之间不能睡眠，因为若睡眠那么其它
 * 人(进程)可能会已经使用了该目录项。
 */
//// 根据指定的目录和文件名添加目录项。
// 参数：dir - 指定目录的i 节点；name - 文件名；namelen - 文件名长度；
// 返回：高速缓冲区指针；res_dir - 返回的目录项结构指针；

add_entry
--block = dir->i_zone[0]
--bh = bread(dir->i_dev,block)
--de = (struct dir_entry *) bh->b_data;
--while (1) {
// 如果当前判别的目录项已经超出当前数据块，则释放该数据块，重新申请一块磁盘块block。如果
// 申请失败，则返回NULL，退出。
        if ((char *)de >= BLOCK_SIZE+bh->b_data) {
            brelse(bh);
            bh = NULL;
            block = create_block(dir,i/DIR_ENTRIES_PER_BLOCK);
            if (!block)
                return NULL;
// 如果读取磁盘块返回的指针为空，则跳过该块继续。
            if (!(bh = bread(dir->i_dev,block))) {
                i += DIR_ENTRIES_PER_BLOCK;
                continue;
            }
// 否则，让目录项结构指针de 志向该块的高速缓冲数据块开始处。
            de = (struct dir_entry *) bh->b_data;
        }
// 如果当前所操作的目录项序号i*目录结构大小已经超过了该目录所指出的大小i_size，则说明该第i
// 个目录项还未使用，我们可以使用它。于是对该目录项进行设置(置该目录项的i 节点指针为空)。并
// 更新该目录的长度值(加上一个目录项的长度，设置目录的i 节点已修改标志，再更新该目录的改变时
// 间为当前时间。
        if (i*sizeof(struct dir_entry) >= dir->i_size) {
            de->inode=0;
            dir->i_size = (i+1)*sizeof(struct dir_entry);
            dir->i_dirt = 1;
            dir->i_ctime = CURRENT_TIME;
        }
// 若该目录项的i 节点为空，则表示找到一个还未使用的目录项。于是更新目录的修改时间为当前时间。
// 并从用户数据区复制文件名到该目录项的文件名字段，置相应的高速缓冲块已修改标志。返回该目录
// 项的指针以及该高速缓冲区的指针，退出。
        if (!de->inode) {
            dir->i_mtime = CURRENT_TIME;
            for (i=0; i < NAME_LEN ; i++)
                de->name[i]=(i<namelen)?get_fs_byte(name+i):0;
            bh->b_dirt = 1;
            *res_dir = de;
            return bh;
        }
// 如果该目录项已经被使用，则继续检测下一个目录项。
        de++;
        i++;
--}
// 执行不到这里。也许Linus 在写这段代码时是先复制了上面find_entry()的代码，而后修改的:)
    brelse(bh);
    return NULL;
```