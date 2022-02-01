#1.rw_port

```cpp
// 端口读写操作函数。
// 参数：rw - 读写命令；buf - 缓冲区；cout - 读写字节数；pos - 端口地址。
// 返回：实际读写的字节数。
static int rw_port(int rw,char * buf, int count, off_t * pos)
{
    int i=*pos;

// 对于所要求读写的字节数，并且端口地址小于64k 时，循环执行单个字节的读写操作。
    while (count-->0 && i<65536) {
// 若是读命令，则从端口i 中读取一字节内容并放到用户缓冲区中。
        if (rw==READ)
            put_fs_byte(inb(i),buf++);
// 若是写命令，则从用户数据缓冲区中取一字节输出到端口i。
        else
            outb(get_fs_byte(buf++),i);
// 前移一个端口。[??]
        i++;
    }
// 计算读/写的字节数，并相应调整读写指针。
    i -= *pos;
    *pos += i;
// 返回读/写的字节数。
    return i;
}
```

#2.caller


```cpp
rw_memory
--rw_port

```
