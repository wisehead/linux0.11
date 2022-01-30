#1.clear_block

```asm
//// 将指定地址(addr)处的一块内存清零。嵌入汇编程序宏。
// 输入：eax = 0，ecx = 数据块大小BLOCK_SIZE/4，edi = addr。
extern _inline void clear_block(char *addr)
{_asm{
    pushf
    mov edi,addr
    mov ecx,BLOCK_SIZE/4
    xor eax,eax
    cld
    rep stosd
    popf
}}
```