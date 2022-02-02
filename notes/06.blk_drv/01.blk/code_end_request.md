#1.end_request

```cpp
// 结束请求。
extern _inline void
end_request (int uptodate)
{
  DEVICE_OFF (CURRENT->dev);    // 关闭设备。
  if (CURRENT->bh)
    {               // CURRENT 为指定主设备号的当前请求结构。
      CURRENT->bh->b_uptodate = uptodate;   // 置更新标志。
      unlock_buffer (CURRENT->bh);  // 解锁缓冲区。
    }
  if (!uptodate)
    {               // 如果更新标志为0 则显示设备错误信息。
      printk (DEVICE_NAME " I/O error\n\r");
      printk ("dev %04x, block %d\n\r", CURRENT->dev, CURRENT->bh->b_blocknr);
    }
  wake_up (&CURRENT->waiting);  // 唤醒等待该请求项的进程。
  wake_up (&wait_for_request);  // 唤醒等待请求的进程。
  CURRENT->dev = -1;        // 释放该请求项。
  CURRENT = CURRENT->next;  // 从请求链表中删除该请求项。
}

```