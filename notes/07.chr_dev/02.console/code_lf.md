#1.lf

```cpp
//// 光标位置下移一行(lf - line feed 换行)。
static void
lf (void)
{
// 如果光标没有处在倒数第2 行之后，则直接修改光标当前行变量y++，并调整光标对应显示内存位置
// pos(加上屏幕一行字符所对应的内存长度)。
    if (y + 1 < bottom)
    {
        y++;
        pos += video_size_row;
        return;
    }
// 否则需要将屏幕内容上移一行。
    scrup ();
}
```