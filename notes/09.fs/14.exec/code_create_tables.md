#1.create_tables

```cpp
/*
 * create_tables()函数在新用户内存中解析环境变量和参数字符串，由此
 * 创建指针表，并将它们的地址放到"堆栈"上，然后返回新栈的指针值。
 */
//// 在新用户堆栈中创建环境和参数变量指针表。
// 参数：p - 以数据段为起点的参数和环境信息偏移指针；argc - 参数个数；envc -环境变量数。
// 返回：堆栈指针。


create_tables
// 堆栈指针是以4 字节（1 节）为边界寻址的，因此这里让sp 为4 的整数倍。
--sp = (unsigned long *) (0xfffffffc & (unsigned long) p);
// sp 向下移动，空出环境参数占用的空间个数，并让环境参数指针envp 指向该处。
--sp -= envc+1;
--envp = sp;
// sp 向下移动，空出命令行参数指针占用的空间个数，并让argv 指针指向该处。
// 下面指针加1，sp 将递增指针宽度字节值。
--sp -= argc+1;
--argv = sp;

// 将环境参数指针envp 和命令行参数指针以及命令行参数个数压入堆栈。
--put_fs_long((unsigned long)envp,--sp);
--put_fs_long((unsigned long)argv,--sp);
--put_fs_long((unsigned long)argc,--sp);

// 将命令行各参数指针放入前面空出来的相应地方，最后放置一个NULL 指针。
--while (argc-->0) {
        put_fs_long((unsigned long) p,argv++);
        while (get_fs_byte(p++)) /* nothing */ ;// p 指针前移4 字节。
--}
--put_fs_long(0,argv);
// 将环境变量各指针放入前面空出来的相应地方，最后放置一个NULL 指针。
--while (envc-->0) {
        put_fs_long((unsigned long) p,envp++);
        while (get_fs_byte(p++)) /* nothing */ ;
--}
--put_fs_long(0,envp);
--return sp;      // 返回构造的当前新堆栈指针。
```