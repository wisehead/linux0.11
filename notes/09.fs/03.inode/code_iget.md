#1.iget

```cpp
iget
--get_empty_inode
--while (inode < NR_INODE+inode_table)
----if (inode->i_dev != dev || inode->i_num != nr)
------continue
----wait_on_inode(inode);
// 将该i 节点引用计数增1。
----inode->i_count++;
----if (inode->i_mount)
// 如果该i 节点是其它文件系统的安装点，则在超级块表中搜寻安装在此i 节点的超级块。如果没有
// 找到，则显示出错信息，并释放函数开始获取的空闲节点，返回该i 节点指针。
------for (i = 0 ; i<NR_SUPER ; i++)
--------if (super_block[i].s_imount==inode)
----------break;
------if (i >= NR_SUPER) {
--------printk("Mounted inode hasn't got sb\n");
--------if (empty)
----------iput(empty);
--------return inode;
// 将该i 节点写盘。从安装在此i 节点文件系统的超级块上取设备号，并令i 节点号为1。然后重新
// 扫描整个i 节点表，取该被安装文件系统的根节点。
------iput(inode);
------dev = super_block[i].s_dev;
------nr = ROOT_INO;
------inode = inode_table;
------continue;
// 已经找到相应的i 节点，因此放弃临时申请的空闲节点，返回该找到的i 节点。
----if (empty)
------iput(empty);
----return inode;

// 如果在i 节点表中没有找到指定的i 节点，则利用前面申请的空闲i 节点在i 节点表中建立该节点。
// 并从相应设备上读取该i 节点信息。返回该i 节点。
--if (!empty)
----return (NULL);
--inode=empty;
--inode->i_dev = dev;
--inode->i_num = nr;
--read_inode(inode);
--return inode;
```