#1.tty_read

```cpp
//// tty 读函数。
// 参数：channel - 子设备号；buf - 缓冲区指针；nr - 欲读字节数。
// 返回已读字节数。

从tty->secondary 读到用户buf，fs段。跨状态。
```

#2.caller

```
sys_read/sys_write
--rw_char                                               
----crw_table//call_addr(rw,MINOR(dev),buf,count,pos)   
------rw_tty                                            
--------rw_ttyx                                         
----------tty_read                                      
```