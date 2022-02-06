#1.struct bucket_desc

```cpp

// 存储桶描述符结构。
struct bucket_desc {    /* 16 bytes */
    void            *page;          // 该桶描述符对应的内存页面指针。
    struct bucket_desc  *next;      // 下一个描述符指针。
    void            *freeptr;       // 指向本桶中空闲内存位置的指针。
    unsigned short      refcnt;     // 引用计数。
    unsigned short      bucket_size;// 本描述符对应存储桶的大小。
};
```