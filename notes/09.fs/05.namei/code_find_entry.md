#1.find_entry

```cpp
/*
 *  find_entry()
 *
 * 在指定的目录中寻找一个与名字匹配的目录项。返回一个含有找到目录项的高速
 * 缓冲区以及目录项本身(作为一个参数- res_dir)。并不读目录项的i 节点- 如
 * 果需要的话需自己操作。
 *
 * '..'目录项，操作期间也会对几种特殊情况分别处理- 比如横越一个伪根目录以
 * 及安装点。
 */
//// 查找指定目录和文件名的目录项。
// 参数：dir - 指定目录i 节点的指针；name - 文件名；namelen - 文件名长度；
// 返回：高速缓冲区指针；res_dir - 返回的目录项结构指针；

find_entry
// 计算本目录中目录项项数entries。置空返回目录项结构指针。
--entries = (*dir)->i_size / (sizeof (struct dir_entry));
/* 检查目录项'..'，因为可能需要对其特别处理 */
--if (namelen==2 && get_fs_byte(name)=='.' && get_fs_byte(name+1)=='.') {
/* 伪根中的'..'如同一个假'.'(只需改变名字长度) */
// 如果当前进程的根节点指针即是指定的目录，则将文件名修改为'.'，
        if ((*dir) == current->root)
            namelen=1;
// 否则如果该目录的i 节点号等于ROOT_INO(1)的话，说明是文件系统根节点。则取文件系统的超级块。
        else if ((*dir)->i_num == ROOT_INO) {
/* 在一个安装点上的'..'将导致目录交换到安装到文件系统的目录i 节点。
   注意！由于设置了mounted 标志，因而我们能够取出该新目录 */
            sb=get_super((*dir)->i_dev);
// 如果被安装到的i 节点存在，则先释放原i 节点，然后对被安装到的i 节点进行处理。
// 让*dir 指向该被安装到的i 节点；该i 节点的引用数加1。
            if (sb->s_imount) {
                iput(*dir);
                (*dir)=sb->s_imount;
                (*dir)->i_count++;
            }
        }
--}
--block = (*dir)->i_zone[0]
--bh = bread((*dir)->i_dev,block)
// 在目录项数据块中搜索匹配指定文件名的目录项，首先让de 指向数据块，并在不超过目录中目录项数
// 的条件下，循环执行搜索。
    i = 0;
    de = (struct dir_entry *) bh->b_data;
    while (i < entries) {
// 如果当前目录项数据块已经搜索完，还没有找到匹配的目录项，则释放当前目录项数据块。
        if ((char *)de >= BLOCK_SIZE+bh->b_data) {
            brelse(bh);
            bh = NULL;
// 在读入下一目录项数据块。若这块为空，则只要还没有搜索完目录中的所有目录项，就跳过该块，
// 继续读下一目录项数据块。若该块不空，就让de 指向该目录项数据块，继续搜索。
            if (!(block = bmap(*dir,i/DIR_ENTRIES_PER_BLOCK)) ||
                !(bh = bread((*dir)->i_dev,block))) {
                i += DIR_ENTRIES_PER_BLOCK;
                continue;
            }
            de = (struct dir_entry *) bh->b_data;
        }
// 如果找到匹配的目录项的话，则返回该目录项结构指针和该目录项数据块指针，退出。
        if (match(namelen,name,de)) {
            *res_dir = de;
            return bh;
        }
// 否则继续在目录项数据块中比较下一个目录项。
        de++;
        i++;
    }
// 若指定目录中的所有目录项都搜索完还没有找到相应的目录项，则释放目录项数据块，返回NULL。
    brelse(bh);
    
```

#2.note

```
安装点和安装文件系统的目录，这两个概念是怎么回事？？？
```