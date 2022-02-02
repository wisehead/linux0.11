#1.struct request

```cpp
/*
* OK，下面是request 结构的一个扩展形式，因而当实现以后，我们就可以在分页请求中
* 使用同样的request 结构。在分页处理中，'bh'是NULL，而'waiting'则用于等待读/写的完成。
*/
// 下面是请求队列中项的结构。其中如果dev=-1，则表示该项没有被使用。
struct request
{
  int dev;          /* -1 if no request */// 使用的设备号。
  int cmd;          /* READ or WRITE */// 命令(READ 或WRITE)。
  int errors;           //操作时产生的错误次数。
  unsigned long sector;     // 起始扇区。(1 块=2 扇区)
  unsigned long nr_sectors; // 读/写扇区数。
  char *buffer;         // 数据缓冲区。
  struct task_struct *waiting;  // 任务等待操作执行完成的地方。
  struct buffer_head *bh;   // 缓冲区头指针(include/linux/fs.h,68)。
  struct request *next;     // 指向下一请求项。
};
```