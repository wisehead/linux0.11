#1.clear_bit

```cpp
//// 复位指定地址开始的第nr 位偏移处的比特位。返回原比特位的反码（1 或0）。
// 输入：%0 - eax（返回值)，%1 - eax(0)；%2 - nr，位偏移值；%3 - (addr)，addr 的内容。
extern _inline int clear_bit(unsigned long nr,char* addr)
{
//  volatile register int __res;
    _asm{
        xor eax,eax
        mov ebx,nr
        mov edx,addr
        btr [edx],ebx
        setnb al
//      mov __res,eax
    }
//  return __res;
}
//#define clear_bit(nr,addr) ({\
//register int res __asm__("ax"); \
//__asm__ __volatile__("btrl %2,%3\n\tsetnb %%al": \
//"=a" (res):"0" (0),"r" (nr),"m" (*(addr))); \
//res;})
```