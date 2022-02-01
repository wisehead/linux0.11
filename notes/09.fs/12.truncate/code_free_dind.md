#1.free_dind

```cpp
//// 释放二次间接块。

free_dind (int dev, int block)

// 读取二次间接块的一级块，并释放其上表明使用的所有逻辑块，然后释放该一级块的缓冲区。
--if (bh = bread (dev, block))
----p = (unsigned short *) bh->b_data;    // 指向数据缓冲区。
----for (i = 0; i < 512; i++, p++)    // 每个逻辑块上可连接512 个二级块。
------if (*p)
--------free_ind (dev, *p);   // 释放所有一次间接块。
----brelse (bh);      // 释放缓冲区。
--//end if
// 最后释放设备上的二次间接块。
--free_block (dev, block);
}

```