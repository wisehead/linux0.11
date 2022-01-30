#1.set_bit

```cpp
//// 置位指定地址开始的第nr 个位偏移处的比特位(nr 可以大于32！)。返回原比特位（0 或1）。
// 输入：%0 - eax（返回值)，%1 - eax(0)；%2 - nr，位偏移值；%3 - (addr)，addr 的内容。
extern _inline int set_bit(unsigned long nr,char* addr)
{
//  volatile register int __res;
    _asm{
        xor eax,eax
        mov ebx,nr
        mov edx,addr
        bts [edx],ebx
        setb al
//      mov __res,eax
    }
//  return __res;
}
//#define set_bit(nr,addr) ({\
//register int res __asm__("ax"); \
//__asm__ __volatile__("btsl %2,%3\n\tsetb %%al": \
//"=a" (res):"0" (0),"r" (nr),"m" (*(addr))); \
//res;})
```

