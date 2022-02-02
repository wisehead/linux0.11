#1.struct exec

```cpp

// 执行文件结构。
// =============================
// unsigned long a_magic // 执行文件魔数。使用N_MAGIC 等宏访问。
// unsigned a_text // 代码长度，字节数。
// unsigned a_data // 数据长度，字节数。
// unsigned a_bss // 文件中的未初始化数据区长度，字节数。
// unsigned a_syms // 文件中的符号表长度，字节数。
// unsigned a_entry // 执行开始地址。
// unsigned a_trsize // 代码重定位信息长度，字节数。
// unsigned a_drsize // 数据重定位信息长度，字节数。
// -----------------------------
struct exec
{
  unsigned long a_magic;    /* Use macros N_MAGIC, etc for access */
  unsigned a_text;      /* length of text, in bytes */
  unsigned a_data;      /* length of data, in bytes */
  unsigned a_bss;       /* length of uninitialized data area for file, in bytes */
  unsigned a_syms;      /* length of symbol table data in file, in bytes */
  unsigned a_entry;     /* start address */
  unsigned a_trsize;        /* length of relocation info for text, in bytes */
  unsigned a_drsize;        /* length of relocation info for data, in bytes */
};
```