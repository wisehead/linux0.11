#1.ri

```cpp
//// 光标上移一行(ri - reverse line feed 反向换行)。
static void
ri (void)
{
// 如果光标不在第1 行上，则直接修改光标当前行标量y--，并调整光标对应显示内存位置pos，减去
// 屏幕上一行字符所对应的内存长度字节数。
    if (y > top)
    {
        y--;
        pos -= video_size_row;
        return;
    }
// 否则需要将屏幕内容下移一行。
    scrdown ();
}
```