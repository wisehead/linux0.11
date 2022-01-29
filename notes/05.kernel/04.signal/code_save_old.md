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