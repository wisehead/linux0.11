#1._bmap

```cpp
//// 文件数据块映射到盘块的处理操作。(block 位图处理函数，bmap - block map)
// 参数：inode – 文件的i 节点；block – 文件中的数据块号；create - 创建标志。
// 如果创建标志置位，则在对应逻辑块不存在时就申请新磁盘块。
// 返回block 数据块对应在设备上的逻辑块号（盘块号）。
_bmap()

// Part 1:===================================================================
// 如果该块号小于7，则使用直接块表示。
--if (block<7) 
// 如果创建标志置位，并且i 节点中对应该块的逻辑块（区段）字段为0，则向相应设备申请一磁盘
// 块（逻辑块，区块），并将盘上逻辑块号（盘块号）填入逻辑块字段中。然后设置i 节点修改时间，
// 置i 节点已修改标志。最后返回逻辑块号。
----if (create && !inode->i_zone[block])
------if (inode->i_zone[block]=new_block(inode->i_dev)) 
--------inode->i_ctime=CURRENT_TIME;
--------inode->i_dirt=1;
----return inode->i_zone[block];


// Part 2:===================================================================
// 如果该块号>=7，并且小于7+512，则说明是一次间接块。下面对一次间接块进行处理。
--block -= 7;
--if (block<512) {
// 如果是创建，并且该i 节点中对应间接块字段为0，表明文件是首次使用间接块，则需申请
// 一磁盘块用于存放间接块信息，并将此实际磁盘块号填入间接块字段中。然后设置i 节点
// 已修改标志和修改时间。
        if (create && !inode->i_zone[7])
            if (inode->i_zone[7]=new_block(inode->i_dev)) {
                inode->i_dirt=1;
                inode->i_ctime=CURRENT_TIME;
            }
// 若此时i 节点间接块字段中为0，表明申请磁盘块失败，返回0 退出。
        if (!inode->i_zone[7])
            return 0;
// 读取设备上的一次间接块。
        if (!(bh = bread(inode->i_dev,inode->i_zone[7])))
            return 0;
// 取该间接块上第block 项中的逻辑块号（盘块号）。
        i = ((unsigned short *) (bh->b_data))[block];
// 如果是创建并且间接块的第block 项中的逻辑块号为0 的话，则申请一磁盘块（逻辑块），并让
// 间接块中的第block 项等于该新逻辑块块号。然后置位间接块的已修改标志。
        if (create && !i)
            if (i=new_block(inode->i_dev)) {
                ((unsigned short *) (bh->b_data))[block]=i;
                bh->b_dirt=1;
            }
// 最后释放该间接块，返回磁盘上新申请的对应block 的逻辑块的块号。
        brelse(bh);
        return i;
--}//if (block<512)

// Part 3:===================================================================
// 程序运行到此，表明数据块是二次间接块，处理过程与一次间接块类似。下面是对二次间接块的处理。
// 将block 再减去间接块所容纳的块数(512)。
    block -= 512;
// 如果是新创建并且i 节点的二次间接块字段为0，则需申请一磁盘块用于存放二次间接块的一级块
// 信息，并将此实际磁盘块号填入二次间接块字段中。之后，置i 节点已修改编制和修改时间。
    if (create && !inode->i_zone[8])
        if (inode->i_zone[8]=new_block(inode->i_dev)) {
            inode->i_dirt=1;
            inode->i_ctime=CURRENT_TIME;
        }
// 若此时i 节点二次间接块字段为0，表明申请磁盘块失败，返回0 退出。
    if (!inode->i_zone[8])
        return 0;
// 读取该二次间接块的一级块。
    if (!(bh=bread(inode->i_dev,inode->i_zone[8])))
        return 0;
// 取该二次间接块的一级块上第(block/512)项中的逻辑块号。
    i = ((unsigned short *)bh->b_data)[block>>9];
// 如果是创建并且二次间接块的一级块上第(block/512)项中的逻辑块号为0 的话，则需申请一磁盘
// 块（逻辑块）作为二次间接块的二级块，并让二次间接块的一级块中第(block/512)项等于该二级
// 块的块号。然后置位二次间接块的一级块已修改标志。并释放二次间接块的一级块。
    if (create && !i)
        if (i=new_block(inode->i_dev)) {
            ((unsigned short *) (bh->b_data))[block>>9]=i;
            bh->b_dirt=1;
        }
    brelse(bh);
// 如果二次间接块的二级块块号为0，表示申请磁盘块失败，返回0 退出。
    if (!i)
        return 0;
// 读取二次间接块的二级块。
    if (!(bh=bread(inode->i_dev,i)))
        return 0;
// 取该二级块上第block 项中的逻辑块号。(与上511 是为了限定block 值不超过511)
    i = ((unsigned short *)bh->b_data)[block&511];
// 如果是创建并且二级块的第block 项中的逻辑块号为0 的话，则申请一磁盘块（逻辑块），作为
// 最终存放数据信息的块。并让二级块中的第block 项等于该新逻辑块块号(i)。然后置位二级块的
// 已修改标志。
    if (create && !i)
        if (i=new_block(inode->i_dev)) {
            ((unsigned short *) (bh->b_data))[block&511]=i;
            bh->b_dirt=1;
        }
// 最后释放该二次间接块的二级块，返回磁盘上新申请的对应block 的逻辑块的块号。
    brelse(bh);
    return i;
```