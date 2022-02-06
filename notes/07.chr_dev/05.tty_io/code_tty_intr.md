#1.tty_intr

```cpp
//// tty 键盘终端字符处理函数。
// 参数：tty - 相应tty 终端结构指针；mask - 信号屏蔽位。
void tty_intr (struct tty_struct *tty, int mask)
{
    int i;

// 如果tty 所属组号小于等于0，则退出。
    if (tty->pgrp <= 0)
        return;
// 扫描任务数组，向tty 相应组的所有任务发送指定的信号。
    for (i = 0; i < NR_TASKS; i++) {
        // 如果该项任务指针不为空，并且其组号等于tty 组号，则设置该任务指定的信号mask。
        if (task[i] && task[i]->pgrp == tty->pgrp)
            task[i]->signal |= mask;
    }
}
```

#2.caller

copy_to_cooked