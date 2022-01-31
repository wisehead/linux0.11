#1.get_pipe_inode

```cpp
get_pipe_inode
--get_empty_inode
--inode->i_size=get_free_page()// 节点的i_size 字段指向缓冲区。
--inode->i_count = 2; /* 读/写两者总计 */
--PIPE_HEAD(*inode) = PIPE_TAIL(*inode) = 0;// 复位管道头尾指针。
--inode->i_pipe = 1;          // 置节点为管道使用的标志。
```