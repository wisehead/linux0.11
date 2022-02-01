#1.truncate

```cpp
//// 将节点对应的文件长度截为0，并释放占用的设备空间。

truncate (struct m_inode *inode)
// 释放i 节点的7 个直接逻辑块，并将这7 个逻辑块项全置零。
--for (i = 0; i < 7; i++)                                                  
----if (inode->i_zone[i])                                                  
----{               // 如果块号不为0，则释放之。                                       
------free_block (inode->i_dev, inode->i_zone[i]);                         
------inode->i_zone[i] = 0;                                                
----}                                                                      
--free_ind (inode->i_dev, inode->i_zone[7]);    // 释放一次间接块。                
--free_dind (inode->i_dev, inode->i_zone[8]);   // 释放二次间接块。                
--inode->i_zone[7] = inode->i_zone[8] = 0;  // 逻辑块项7、8 置零。                 
--inode->i_size = 0;        // 文件大小置零。                                     
--inode->i_dirt = 1;        // 置节点已修改标志。                                   
--inode->i_mtime = inode->i_ctime = CURRENT_TIME;   // 重置文件和节点修改时间为当前时间。   

```

#2.caller

```
- iput
- open_namei
```