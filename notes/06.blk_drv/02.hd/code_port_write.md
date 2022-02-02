#1.port_write

```cpp

// 写端口port，共写nr 字，从buf 中取数据。
//#define port_write(port,buf,nr) \
//__asm__( "cld;rep;outsw":: "d" (port), "S" (buf), "c" (nr): "cx", "si")
_inline void port_write(unsigned short port, void* buf,unsigned long nr)
{_asm{
    pushf
    mov dx,port
    mov esi,buf
    mov ecx,nr
    cld
    rep outsw
    popf
}}
```

#2.caller

```
- write_intr
- 	do_hd_request
```