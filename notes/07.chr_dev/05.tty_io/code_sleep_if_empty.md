#1.sleep_if_empty

```cpp
//// 如果队列缓冲区空则让进程进入可中断的睡眠状态。
// 参数：queue - 指定队列的指针。
// 进程在取队列缓冲区中字符时调用此函数。
static void
sleep_if_empty (struct tty_queue *queue)
{
    cli ();         // 关中断。
// 若当前进程没有信号要处理并且指定的队列缓冲区空，则让进程进入可中断睡眠状态，并让
// 队列的进程等待指针指向该进程。
    while (!current->signal && EMPTY (*queue))
        interruptible_sleep_on (&queue->proc_list);
    sti ();         // 开中断。
}
```

#2.caller

```
- wait_for_keypress
- tty_read
```
