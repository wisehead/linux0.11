#1.struct tty_queue

```
// tty 等待队列数据结构。
struct tty_queue
{
  unsigned long data;       // 等待队列缓冲区中当前数据指针字符数[??]）。
// 对于串口终端，则存放串行端口地址。
  unsigned long head;       // 缓冲区中数据头指针。
  unsigned long tail;       // 缓冲区中数据尾指针。
  struct task_struct *proc_list;    // 等待进程列表。
  char buf[TTY_BUF_SIZE];   // 队列的缓冲区。
};
```