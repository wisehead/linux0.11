#1._copy_page

```cpp
// 复制1 页内存（4K 字节）。
//#define copy_page(from,to) \
//__asm__("cld ; rep ; movsl"::"S" (from),"D" (to),"c" (1024):"cx","di","si")
#define copy_page(from,to) _copy_page((void *)(from),(void *)(to))
_inline void _copy_page(void *from, void *to)
{_asm{
    pushf
    mov ecx,1024
    mov esi,from
    mov edi,to
    cld
    rep movsd
    popf
}}

```

#2.caller

```
- un_wp_page
```