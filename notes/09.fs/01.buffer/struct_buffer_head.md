#1.struct buffer_head

```cpp
// 缓冲区头数据结构。（极为重要！！！）
// 在程序中常用bh 来表示buffer_head 类型的缩写。
struct buffer_head
{
  char *b_data;         /* pointer to data block (1024 bytes) *///指针。
  unsigned long b_blocknr;  /* block number */// 块号。
  unsigned short b_dev;     /* device (0 = free) */// 数据源的设备号。
  unsigned char b_uptodate; // 更新标志：表示数据是否已更新。
  unsigned char b_dirt;     /* 0-clean,1-dirty *///修改标志:0 未修改,1 已修改.
  unsigned char b_count;    /* users using this block */// 使用的用户数。
  unsigned char b_lock;     /* 0 - ok, 1 -locked */// 缓冲区是否被锁定。
  struct task_struct *b_wait;   // 指向等待该缓冲区解锁的任务。
  struct buffer_head *b_prev;   // hash 队列上前一块（这四个指针用于缓冲区的管理）。
  struct buffer_head *b_next;   // hash 队列上下一块。
  struct buffer_head *b_prev_free;  // 空闲表上前一块。
  struct buffer_head *b_next_free;  // 空闲表上下一块。
};
```