#1.struct i387_struct

```cpp
//math协处理器上下文环境
struct i387_struct {
    long    cwd;
    long    swd;
    long    twd;
    long    fip;
    long    fcs;
    long    foo;
    long    fos;
    long    st_space[20];   /* 8*10 bytes for each FP-reg = 80 bytes */
};
```