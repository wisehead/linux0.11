#1.brelse

```cpp
//// 释放指定的缓冲区。
// 等待该缓冲区解锁。引用计数递减1。唤醒等待空闲缓冲区的进程。
void brelse(struct buffer_head * buf)
{
    if (!buf)       // 如果缓冲头指针无效则返回。
        return;
    wait_on_buffer(buf);
    if (!(buf->b_count--))
        panic("Trying to free free buffer");
    wake_up(&buffer_wait);
}
```