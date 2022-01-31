#1.struct dir_entry

```cpp
// 文件目录项结构。
struct dir_entry
{
  unsigned short inode;     // i 节点。
  char name[NAME_LEN];      // 文件名。
};
```