#1.dev type

```cpp
/* devices are as follows: (same as minix, so we can use the minix
* file system. These are major numbers.)
*
* 0 - unused (nodev)
* 1 - /dev/mem
* 2 - /dev/fd
* 3 - /dev/hd
* 4 - /dev/ttyx
* 5 - /dev/tty
* 6 - /dev/lp
* 7 - unnamed pipes
*/
/*
* 系统所含的设备如下：（与minix 系统的一样，所以我们可以使用minix 的
* 文件系统。以下这些是主设备号。）
*
* 0 - 没有用到（nodev）
* 1 - /dev/mem 内存设备。
* 2 - /dev/fd 软盘设备。
* 3 - /dev/hd 硬盘设备。
* 4 - /dev/ttyx tty 串行终端设备。
* 5 - /dev/tty tty 终端设备。
* 6 - /dev/lp 打印设备。
* 7 - unnamed pipes 没有命名的管道。
*/

```