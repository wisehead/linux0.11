#1.get_limit

```x86asm
227
// 取段选择符 segment 的段长值。
// %0 - 存放段长值(字节数);%1 - 段选择符 segment。
228 #define get_limit(segment) ({ \
229 unsigned long __limit; \
230 __asm__("lsll %1,%0\n\tincl %0":"=r" (__limit):"r" (segment)); \ 231 __limit;})
```

