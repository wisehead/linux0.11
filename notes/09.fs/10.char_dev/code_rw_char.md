#1.rw_char

```cpp
//// 字符设备读写操作函数。
// 参数：rw - 读写命令；dev - 设备号；buf - 缓冲区；count - 读写字节数；pos -读写指针。
// 返回：实际读/写字节数。

rw_char
// 若该设备没有对应的读/写函数，则返回出错码。
--call_addr=crw_table[MAJOR(dev)]
// 调用对应设备的读写操作函数，并返回实际读/写的字节数。
--return call_addr(rw,MINOR(dev),buf,count,pos);
```

#2.caller

```
- sys_read
- sys_write
```