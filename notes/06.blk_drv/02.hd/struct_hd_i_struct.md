#1.struct hd_i_struct


```cpp
/*
* This struct defines the HD's and their types.
*/
/* 下面结构定义了硬盘参数及类型 */
// 各字段分别是磁头数、每磁道扇区数、柱面数、写前预补偿柱面号、磁头着陆区柱面号、控制字节。
struct hd_i_struct
{
    int head, sect, cyl, wpcom, lzone, ctl;
};

```