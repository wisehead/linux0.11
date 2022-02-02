#1.do_execve

```cpp
/*
 * 'do_execve()'函数执行一个新程序。
 */
//// execve()系统中断调用函数。加载并执行子进程（其它程序）。
// 该函数系统中断调用(int 0x80)功能号__NR_execve 调用的函数。
// 参数：eip - 指向堆栈中调用系统中断的程序代码指针eip 处，参见kernel/system_call.s 程序
// 开始部分的说明；tmp - 系统中断调用本函数时的返回地址，无用；
//                 filename - 被执行程序文件名；argv - 命令行参数指针数组；
//                 envp - 环境变量指针数组。
// 返回：如果调用成功，则不返回；否则设置出错号，并返回-1。

int do_execve(unsigned long * eip,long tmp,char * filename,char ** argv, char ** envp)
--inode=namei(filename)
// 计算参数个数和环境变量个数。
--argc = count(argv);
--envc = count(envp);
// 执行文件必须是常规文件。若不是常规文件则置出错返回码，跳转到exec_error2(第347 行)。
--restart_interp:
// 读取执行文件的第一块数据到高速缓冲区，若出错则置出错码，跳转到exec_error2 处去处理。
--bh = bread(inode->i_dev,inode->i_zone[0])
// 下面对执行文件的头结构数据进行处理，首先让ex 指向执行头部分的数据结构。
--ex = *((struct exec *) bh->b_data); /* 读取执行头部分 */
// 如果执行文件开始的两个字节为'#!'，并且sh_bang 标志没有置位，则处理脚本文件的执行。
--if ((bh->b_data[0] == '#') && (bh->b_data[1] == '!') && (!sh_bang)) 
----
--//end if ((bh->b_data[0] == '#') && (bh->b_data[1] == '!') 
```