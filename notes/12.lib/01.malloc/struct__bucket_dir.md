#1.struct _bucket_dir

```cpp
// 存储桶描述符目录结构。
struct _bucket_dir {    /* 8 bytes */
    int         size;           // 该存储桶的大小(字节数)。
    struct bucket_desc  *chain; // 该存储桶目录项的桶描述符链表指针。
};
```

#2.bucket_dir[]

```cpp
/*
 * 下面是我们存放第一个给定大小存储桶描述符指针的地方。
 *
 * 如果Linux 内核分配了许多指定大小的对象，那么我们就希望将该指定的大小加到
 * 该列表(链表)中，因为这样可以使内存的分配更有效。但是，因为一页完整内存页面
 * 必须用于列表中指定大小的所有对象，所以需要做总数方面的测试操作。
 */
// 存储桶目录列表(数组)。
struct _bucket_dir bucket_dir[] = {
    { 16,   (struct bucket_desc *) 0},// 16 字节长度的内存块。
    { 32,   (struct bucket_desc *) 0},// 32 字节长度的内存块。
    { 64,   (struct bucket_desc *) 0},// 64 字节长度的内存块。
    { 128,  (struct bucket_desc *) 0},// 128 字节长度的内存块。
    { 256,  (struct bucket_desc *) 0},// 256 字节长度的内存块。
    { 512,  (struct bucket_desc *) 0},// 512 字节长度的内存块。
    { 1024, (struct bucket_desc *) 0},// 1024 字节长度的内存块。
    { 2048, (struct bucket_desc *) 0},// 2048 字节长度的内存块。
    { 4096, (struct bucket_desc *) 0},// 4096 字节(1 页)内存。
    { 0,    (struct bucket_desc *) 0}};   /* End of list marker */
```
