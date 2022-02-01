#1.empty_dir

```cpp
/*
 * 用于检查指定的目录是否为空的子程序(用于rmdir 系统调用函数)。
 */
//// 检查指定目录是否是空的。
// 参数：inode - 指定目录的i 节点指针。
// 返回：0 - 是空的；1 - 不空。

empty_dir
// 计算指定目录中现有目录项的个数(应该起码有2 个，即"."和".."两个文件目录项)。
--len = inode->i_size / sizeof (struct dir_entry);
// 让de 指向含有读出磁盘块数据的高速缓冲区中第1 项目录项。
--de = (struct dir_entry *) bh->b_data;
// 令nr 等于目录项序号；de 指向第三个目录项。
--nr = 2;
--de += 2;
// 循环检测该目录中所有的目录项(len-2 个)，看有没有目录项的i 节点号字段不为0(被使用)。
--while (nr<len) {
// 如果该块磁盘块中的目录项已经检测完，则释放该磁盘块的高速缓冲区，读取下一块含有目录项的
// 磁盘块。若相应块没有使用(或已经不用，如文件已经删除等)，则继续读下一块，若读不出，则出
// 错，返回0。否则让de 指向读出块的首个目录项。
        if ((void *) de >= (void *) (bh->b_data+BLOCK_SIZE)) {
            brelse(bh);
            block=bmap(inode,nr/DIR_ENTRIES_PER_BLOCK);
            if (!block) {
                nr += DIR_ENTRIES_PER_BLOCK;
                continue;
            }
            if (!(bh=bread(inode->i_dev,block)))
                return 0;
            de = (struct dir_entry *) bh->b_data;
        }
// 如果该目录项的i 节点号字段不等于0，则表示该目录项目前正被使用，则释放该高速缓冲区，
// 返回0，退出。
        if (de->inode) {
            brelse(bh);
            return 0;
        }
// 否则，若还没有查询完该目录中的所有目录项，则继续检测。
        de++;
        nr++;
--}
// 到这里说明该目录中没有找到已用的目录项(当然除了头两个以外)，则返回缓冲区，返回1。
--brelse(bh);
--return 1;
```