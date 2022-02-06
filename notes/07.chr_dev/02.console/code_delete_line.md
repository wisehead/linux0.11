#1.delete_line


```cpp

//// 删除光标所在行。
// 从光标所在行开始屏幕内容上卷一行。
static void
delete_line (void)
{
    int oldtop, oldbottom;

    oldtop = top;           // 保存原top，bottom 值。
    oldbottom = bottom;
    top = y;            // 设置屏幕卷动开始行。
    bottom = video_num_lines;   // 设置屏幕卷动最后行。
    scrup ();           // 从光标开始处，屏幕内容向上滚动一行。
    top = oldtop;           // 恢复原top，bottom 值。
    bottom = oldbottom;
}
```