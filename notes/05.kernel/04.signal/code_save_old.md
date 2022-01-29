#1.save_old

```
save_old
--verify_area
----size += start & 0xfff;
----start &= 0xfffff000;
----start += get_base (current->ldt[2]);
----while (size > 0)
------write_verify (start);
--for (i = 0; i < sizeof (struct sigaction); i++)
----put_fs_byte (*from, to)
```

#2.caller

```
sys_sigaction
```

#3.put_fs_byte

```
//// 将一字节存放在fs 段中指定内存地址处。
// 参数：val - 字节值；addr - 内存地址。
// %0 - 寄存器(字节值val)；%1 - (内存地址addr)。
extern _inline void
put_fs_byte (char val, char *addr)//passed
{
//  __asm__ ("movb %0,%%fs:%1"::"r" (val), "m" (*addr));
    _asm mov ebx,addr
    _asm mov al,val;
    _asm mov byte ptr fs:[ebx],al;
}
```

#4._get_base

```
// 取局部描述符表中ldt 所指段描述符中的基地址。
#define get_base(ldt) _get_base( ((void *)&(ldt)) )
// 从地址addr 处描述符中取段基地址。功能与_set_base()正好相反。
// edx - 存放基地址(__base)；%1 - 地址addr 偏移2；%2 - 地址addr 偏移4；%3 - addr 偏移7。
extern _inline unsigned long _get_base(void *addr)
{
//  unsigned long __base;
    _asm {
        _asm mov ebx,addr
        _asm mov ah,byte ptr [ebx+7] // 取[addr+7]处基址高16 位的高8 位(位31-24)->dh。
        _asm mov al,byte ptr [ebx+4] // 取[addr+4]处基址高16 位的低8 位(位23-16)->dl。
        _asm shl eax,16 // 基地址高16 位移到edx 中高16 位处。
        _asm mov ax,word ptr [ebx+2] // 取[addr+2]处基址低16 位(位15-0)->dx。
//      _asm mov __base,eax
        }
//  return __base;
}

```