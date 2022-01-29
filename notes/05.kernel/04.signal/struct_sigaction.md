#1.struct sigaction

```cpp
// 下面是sigaction 的数据结构。
// sa_handler 是对应某信号指定要采取的行动。可以是上面的SIG_DFL，或者是SIG_IGN 来忽略
// 该信号，也可以是指向处理该信号函数的一个指针。
// sa_mask 给出了对信号的屏蔽码，在信号程序执行时将阻塞对这些信号的处理。
// sa_flags 指定改变信号处理过程的信号集。它是由37-39 行的位标志定义的。
// sa_restorer 恢复过程指针，是用于保存原返回的过程指针。
// 另外，引起触发信号处理的信号也将被阻塞，除非使用了SA_NOMASK 标志。
struct sigaction
{
  void (*sa_handler) (int);
  sigset_t sa_mask;
  int sa_flags;
  void (*sa_restorer) (void);
};
```