#1.set_tss_desc

```cpp
#define set_tss_desc(n,addr) _set_tssldt_desc(((char *) (n)),addr,"0x89")
```

#2.