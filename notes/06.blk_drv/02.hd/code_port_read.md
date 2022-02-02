#1.port_read

```cpp
// 读端口port，共读nr 字，保存在buf 中。
//#define port_read(port,buf,nr) \
//__asm__( "cld;rep;insw":: "d" (port), "D" (buf), "c" (nr): "cx", "di")
_inline void port_read(unsigned short port, void* buf,unsigned long nr)
{_asm{
    pushf
    mov dx,port
    mov edi,buf
    mov ecx,nr
    cld
    rep insw
    popf
}}

```