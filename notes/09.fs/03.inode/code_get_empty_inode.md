#1.get_empty_inode

```cpp
get_empty_inode
--while (inode->i_count)
----for (i = NR_INODE; i ; i--) 
------if (!last_inode->i_count)
--------inode = last_inode;
--------if (!inode->i_dirt && !inode->i_lock)
----------break;//for (i = NR_INODE; i ; i--) 
----if (!inode)
------panic("No free inodes in mem");
----wait_on_inode(inode);
----while (inode->i_dirt)
------write_inode(inode);
------wait_on_inode(inode);
--memset(inode,0,sizeof(*inode));
--inode->i_count = 1;
--return inode;
```