#1.struct file

```cpp
// 文件结构（用于在文件句柄与i 节点之间建立关系）
struct file
{
  unsigned short f_mode;    // 文件操作模式（RW 位）
  unsigned short f_flags;   // 文件打开和控制的标志。
  unsigned short f_count;   // 对应文件句柄（文件描述符）数。
  struct m_inode *f_inode;  // 指向对应i 节点。
  off_t f_pos;          // 文件位置（读写偏移值）。
};

```