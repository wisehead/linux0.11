#1.struct super_block

```cpp
// 内存中磁盘超级块结构。
struct super_block
{
  unsigned short s_ninodes; // 节点数。
  unsigned short s_nzones;  // 逻辑块数。
  unsigned short s_imap_blocks; // i 节点位图所占用的数据块数。
  unsigned short s_zmap_blocks; // 逻辑块位图所占用的数据块数。
  unsigned short s_firstdatazone;   // 第一个数据逻辑块号。
  unsigned short s_log_zone_size;   // log(数据块数/逻辑块)。（以2 为底）。
  unsigned long s_max_size; // 文件最大长度。
  unsigned short s_magic;   // 文件系统魔数。
/* These are only in memory */
  struct buffer_head *s_imap[8];    // i 节点位图缓冲块指针数组(占用8 块，可表示64M)。
  struct buffer_head *s_zmap[8];    // 逻辑块位图缓冲块指针数组（占用8 块）。
  unsigned short s_dev;     // 超级块所在的设备号。
  struct m_inode *s_isup;   // 被安装的文件系统根目录的i 节点。(isup-super i)
  struct m_inode *s_imount; // 被安装到的i 节点。
  unsigned long s_time;     // 修改时间。
  struct task_struct *s_wait;   // 等待该超级块的进程。
  unsigned char s_lock;     // 被锁定标志。
  unsigned char s_rd_only;  // 只读标志。
  unsigned char s_dirt;     // 已修改(脏)标志。
};

```

#2.struct d_super_block

```cpp
// 磁盘上超级块结构。上面125-132 行完全一样。
struct d_super_block
{
  unsigned short s_ninodes; // 节点数。
  unsigned short s_nzones;  // 逻辑块数。
  unsigned short s_imap_blocks; // i 节点位图所占用的数据块数。
  unsigned short s_zmap_blocks; // 逻辑块位图所占用的数据块数。
  unsigned short s_firstdatazone;   // 第一个数据逻辑块。
  unsigned short s_log_zone_size;   // log(数据块数/逻辑块)。（以2 为底）。
  unsigned long s_max_size; // 文件最大长度。
  unsigned short s_magic;   // 文件系统魔数。
};

```