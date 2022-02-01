#1.free_ind

```cpp
//// 释放一次间接块。

free_ind
// 读取一次间接块，并释放其上表明使用的所有逻辑块，然后释放该一次间接块的缓冲区。
--if (bh = bread (dev, block))
----p = (unsigned short *) bh->b_data;    // 指向数据缓冲区。
----for (i = 0; i < 512; i++, p++)    // 每个逻辑块上可有512 个块号。
------if (*p)
--------free_block (dev, *p); // 释放指定的逻辑块。
----brelse (bh);      // 释放缓冲区。
--free_block (dev, block);
```