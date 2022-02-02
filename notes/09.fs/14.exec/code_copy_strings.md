#1.copy_strings


```cpp
/*
 * 'copy_string()'函数从用户内存空间拷贝参数和环境字符串到内核空闲页面内存中。
 * 这些已具有直接放到新用户内存中的格式。
 *
 * 由TYT(Tytso)于1991.12.24 日修改，增加了from_kmem 参数，该参数指明了字符串或
 * 字符串数组是来自用户段还是内核段。
 *
 * from_kmem     argv *        argv **
 *    0          用户空间      用户空间
 *    1          内核空间      用户空间
 *    2          内核空间      内核空间
 *
 * 我们是通过巧妙处理fs 段寄存器来操作的。由于加载一个段寄存器代价太大，所以
 * 我们尽量避免调用set_fs()，除非实在必要。
 */
//// 复制指定个数的参数字符串到参数和环境空间。
// 参数：argc - 欲添加的参数个数；argv - 参数指针数组；page - 参数和环境空间页面指针数组。
//       p -在参数表空间中的偏移指针，始终指向已复制串的头部；from_kmem - 字符串来源标志。
// 在do_execve()函数中，p 初始化为指向参数表(128kB)空间的最后一个长字处，参数字符串
// 是以堆栈操作方式逆向往其中复制存放的，因此p 指针会始终指向参数字符串的头部。
// 返回：参数和环境空间当前头部指针。

copy_strings
// 取ds 寄存器值到new_fs，并保存原fs 寄存器值到old_fs。
--new_fs = get_ds();
--old_fs = get_fs();
// 如果字符串和字符串数组来自内核空间，则设置fs 段寄存器指向内核数据段（ds）。
    if (from_kmem==2)
        set_fs(new_fs);
// 循环处理各个参数，从最后一个参数逆向开始复制，复制到指定偏移地址处。
    while (argc-- > 0) {
// 如果字符串在用户空间而字符串数组在内核空间，则设置fs 段寄存器指向内核数据段（ds）。
        if (from_kmem == 1)
            set_fs(new_fs);
// 从最后一个参数开始逆向操作，取fs 段中最后一参数指针到tmp，如果为空，则出错死机。
        if (!(tmp = (char *)get_fs_long(((unsigned long *)argv)+argc)))
            panic("argc is wrong");
// 如果字符串在用户空间而字符串数组在内核空间，则恢复fs 段寄存器原值。
        if (from_kmem == 1)
            set_fs(old_fs);
// 计算该参数字符串长度len，并使tmp 指向该参数字符串末端。
        len=0;      /* 我们知道串是以NULL 字节结尾的 */
        do {
            len++;
        } while (get_fs_byte(tmp++));
// 如果该字符串长度超过此时参数和环境空间中还剩余的空闲长度，则恢复fs 段寄存器并返回0。
        if (p-len < 0) {    /* 不会发生-因为有128kB 的空间 */
            set_fs(old_fs);
            return 0;
        }
// 复制fs 段中当前指定的参数字符串，是从该字符串尾逆向开始复制。
        while (len) {
            --p; --tmp; --len;
// 函数刚开始执行时，偏移变量offset 被初始化为0，因此若offset-1<0，说明是首次复制字符串，
// 则令其等于p 指针在页面内的偏移值，并申请空闲页面。
            if (--offset < 0) {
                offset = p % PAGE_SIZE;
// 如果字符串和字符串数组在内核空间，则恢复fs 段寄存器原值。
                if (from_kmem==2)
                    set_fs(old_fs);
// 如果当前偏移值p 所在的串空间页面指针数组项page[p/PAGE_SIZE]==0，表示相应页面还不存在，
// 则需申请新的内存空闲页面，将该页面指针填入指针数组，并且也使pag 指向该新页面，若申请不
// 到空闲页面则返回0。
                if (!(pag = (char *) page[p/PAGE_SIZE]) &&
                    !(pag = (char *) page[p/PAGE_SIZE] =
                      (char *) get_free_page()))
                    return 0;
// 如果字符串和字符串数组来自内核空间，则设置fs 段寄存器指向内核数据段（ds）。
                if (from_kmem==2)
                    set_fs(new_fs);

            }
// 从fs 段中复制参数字符串中一字节到pag+offset 处。
            *(pag + offset) = get_fs_byte(tmp);
        }
    }
// 如果字符串和字符串数组在内核空间，则恢复fs 段寄存器原值。
    if (from_kmem==2)
        set_fs(old_fs);
// 最后，返回参数和环境空间中已复制参数信息的头部偏移值。
    return p;        
```

#2.caller

```
do_execve

```