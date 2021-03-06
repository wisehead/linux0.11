#1. _get_base

```cpp
213 
    // 从地址 addr 处描述符中取段基地址。功能与_set_base()正好相反。
    // edx - 存放基地址(__base);%1 - 地址 addr 偏移 2;%2 - 地址 addr 偏移 4;%3 - addr 偏移 7。
214 #define _get_base(addr) ({\
215 unsigned long __base; \
216 __asm__("movb %3,%%dh\n\t" \
217         "movb %2,%%dl\n\t" \
218 "shll $16,%%edx\n\t" \
219 "movw %1,%%dx" \
220 :"=d" (__base) \
221         :"m" (*((addr)+2)), \
222          "m" (*((addr)+4)), \
223          "m" (*((addr)+7))); \
224 __base;})
225
```
#2. get_base
```asm
// 取局部描述符表中 ldt 所指段描述符中的基地址。
226 #define get_base(ldt) _get_base( ((char *)&(ldt)) )
```