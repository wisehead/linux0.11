#1.immoutb_p

```cpp
// 字节直接输出（嵌入汇编语言宏）。
//#define immoutb_p(val,port) \
//__asm__ ("outb %0,%1\n\tjmp 1f\n1:\tjmp 1f\n1:"::"a" ((char) (val)),"i" (port))
void _inline immoutb_p(unsigned char val,unsigned short port)
{_asm{
    mov al,val
    mov dx,port
    out dx,al
    jmp l1
l1: jmp l2
l2:
}}

```