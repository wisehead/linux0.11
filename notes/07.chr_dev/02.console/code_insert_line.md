#1.insert_line

```cpp
//// 在光标处插入一行（则光标将处在新的空行上）。
// 将屏幕从光标所在行到屏幕底向下卷动一行。
static void
insert_line (void)
{
    int oldtop, oldbottom;

    oldtop = top;           // 保存原top，bottom 值。
    oldbottom = bottom;
    top = y;            // 设置屏幕卷动开始行。
    bottom = video_num_lines;   // 设置屏幕卷动最后行。
    scrdown ();         // 从光标开始处，屏幕内容向下滚动一行。
    top = oldtop;           // 恢复原top，bottom 值。
    bottom = oldbottom;
}
```