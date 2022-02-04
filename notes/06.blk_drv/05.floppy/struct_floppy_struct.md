#1.struct floppy_struct

```cpp
/*
* 下面的软盘结构定义了不同的软盘类型。与minix 不同的是，linux 没有
* "搜索正确的类型"-类型，因为对其处理的代码令人费解且怪怪的。本程序
* 已经让我遇到了许多的问题了。
*
* 对某些类型的软盘（例如在1.2MB 驱动器中的360kB 软盘等），'stretch'用于
* 检测磁道是否需要特殊处理。其它参数应该是自明的。
*/
// 软盘参数有：
// size 大小(扇区数)；
// sect 每磁道扇区数；
// head 磁头数；
// track 磁道数；
// stretch 对磁道是否要特殊处理（标志）；
// gap 扇区间隙长度(字节数)；
// rate 数据传输速率；
// spec1 参数（高4 位步进速率，低四位磁头卸载时间）。
static struct floppy_struct
{
    unsigned int size, sect, head, track, stretch;
    unsigned char gap, rate, spec1;
}

```