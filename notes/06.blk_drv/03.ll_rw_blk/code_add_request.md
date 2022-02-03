#1.add_request

```cpp
/*
* add-request adds a request to the linked list.
* It disables interrupts so that it can muck with the
* request-lists in peace.
*/
/*
* add-request()向连表中加入一项请求。它关闭中断，
* 这样就能安全地处理请求连表了
*/
//// 向链表中加入请求项。参数dev 指定块设备，req 是请求的结构信息。
add_request (struct blk_dev_struct *dev, struct request *req)
--cli ();         // 关中断。
--if (req->bh)
        req->bh->b_dirt = 0;    // 清缓冲区“脏”标志。
// 如果dev 的当前请求(current_request)子段为空，则表示目前该设备没有请求项，本次是第1 个
// 请求项，因此可将块设备当前请求指针直接指向请求项，并立刻执行相应设备的请求函数。
--if (!(tmp = dev->current_request))
--{
        dev->current_request = req;
        sti ();         // 开中断。
        (dev->request_fn) ();   // 执行设备请求函数，对于硬盘(3)是do_hd_request()。
        return;
--}
// 如果目前该设备已经有请求项在等待，则首先利用电梯算法搜索最佳位置，然后将当前请求插入
// 请求链表中。
--for (; tmp->next; tmp = tmp->next)
----if ((IN_ORDER (tmp, req) ||
        !IN_ORDER (tmp, tmp->next)) && IN_ORDER (req, tmp->next))
        break;
--req->next = tmp->next;
--tmp->next = req;
--sti ();
}
```

#2.caller


```
ll_rw_block
--make_request

```