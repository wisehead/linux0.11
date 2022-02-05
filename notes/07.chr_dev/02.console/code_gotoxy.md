#1.gotoxy

```cpp
/* 注意！gotoxy 函数认为x==video_num_columns，这是正确的 */
//// 跟踪光标当前位置。
// 参数：new_x - 光标所在列号；new_y - 光标所在行号。
// 更新当前光标位置变量x,y，并修正pos 指向光标在显示内存中的对应位置

gotoxy
// 更新当前光标变量；更新光标位置对应的在显示内存中位置变量pos。                          
--x = new_x;                                                 
--y = new_y;                                                 
--pos = origin + y * video_size_row + (x << 1);              
```

#2.caller

```
- restore_cur
- con_write
```