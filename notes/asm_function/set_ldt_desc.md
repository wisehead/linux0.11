#1.set_ldt_desc

```cpp
#define set_ldt_desc(n,addr) _set_tssldt_desc(((char *) (n)),addr,"0x82")
```

LDT和 TSS descriptor格式应该是一样的，只是type/flag不一样。