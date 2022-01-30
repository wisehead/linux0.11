#1.verify_area

```cpp
//// 进程空间区域写前验证函数。
// 对当前进程的地址addr 到addr+size 这一段进程空间以页为单位执行写操作前的检测操作。
// 若页面是只读的，则执行共享检验和复制页面操作（写时复制）。

verify_area
// 将起始地址start 调整为其所在页的左边界开始位置，同时相应地调整验证区域大小。
// 此时start 是当前进程空间中的线性地址。
--size += start & 0xfff;//size加上addr的页内偏移
--start &= 0xfffff000;//获取页号，但是没有移位
--start += get_base (current->ldt[2]);// 此时start 变成系统整个线性空间中的地址位置。
--while (size > 0)
----size -= 4096;
// 写页面验证。若页面不可写，则复制页面。（mm/memory.c，261 行）
//以页为单位验证
----write_verify (start);
----start += 4096;

```