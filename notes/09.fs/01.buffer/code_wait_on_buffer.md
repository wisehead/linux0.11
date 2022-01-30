#1.wait_on_buffer

```cpp
//// 等待指定缓冲区解锁。
static _inline void wait_on_buffer(struct buffer_head * bh)
{
    cli();      // 关中断。
    while (bh->b_lock)  // 如果已被上锁，则进程进入睡眠，等待其解锁。
        sleep_on(&bh->b_wait);
    sti();      // 开中断。
}
```