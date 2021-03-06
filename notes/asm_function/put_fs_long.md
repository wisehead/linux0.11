#1.put_fs_long

```cpp
//// 将一长字存放在 fs 段中指定内存地址处。
// 参数:val - 长字值;addr - 内存地址。
// %0 - 寄存器(长字值 val);%1 - (内存地址 addr)。

extern inline void put_fs_long(unsigned long val,unsigned long * addr)
{
__asm__ ("movl %0,%%fs:%1"::"r" (val),"m" (*addr));
}
```