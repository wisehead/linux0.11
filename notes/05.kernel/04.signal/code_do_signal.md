#1.do_signal

```cpp
// 系统调用中断处理程序中真正的信号处理程序（在kernel/system_call.s,119 行）。
// 该段代码的主要作用是将信号的处理句柄插入到用户程序堆栈中，并在本系统调用结束
// 返回后立刻执行信号句柄程序，然后继续执行用户的程序。


do_signal
--struct sigaction *sa = current->sigaction + signr - 1;  //current->sigaction[signu-1]。
--sa_handler = (unsigned long) sa->sa_handler;
    // 如果该信号句柄只需使用一次，则将该句柄置空(该信号句柄已经保存在sa_handler 指针中)。
--if (sa->sa_flags & SA_ONESHOT)
----sa->sa_handler = NULL;
// 下面这段代码将信号处理句柄插入到用户堆栈中，同时也将sa_restorer,signr,进程屏蔽码(如果
// SA_NOMASK 没置位),eax,ecx,edx 作为参数以及原调用系统调用的程序返回指针及标志寄存器值
// 压入堆栈。因此在本次系统调用中断(0x80)返回用户程序时会首先执行用户的信号句柄程序，然后
// 再继续执行用户程序。
// 将用户调用系统调用的代码指针eip 指向该信号处理句柄。
--*(&eip) = sa_handler;
// 将原调用程序的用户的堆栈指针向下扩展7（或8）个长字（用来存放调用信号句柄的参数等），
// 并检查内存使用情况（例如如果内存超界则分配新页等）。
--*(&esp) -= longs;
// 在用户堆栈中从下到上存放sa_restorer, 信号signr, 屏蔽码blocked(如果SA_NOMASK 置位),
// eax, ecx, edx, eflags 和用户程序原代码指针。

    tmp_esp = esp;
    put_fs_long ((long) sa->sa_restorer, tmp_esp++);
    put_fs_long (signr, tmp_esp++);
    if (!(sa->sa_flags & SA_NOMASK))
        put_fs_long (current->blocked, tmp_esp++);
    put_fs_long (eax, tmp_esp++);
    put_fs_long (ecx, tmp_esp++);
    put_fs_long (edx, tmp_esp++);
    put_fs_long (eflags, tmp_esp++);
    put_fs_long (old_eip, tmp_esp++);
    current->blocked |= sa->sa_mask;    // 进程阻塞码(屏蔽码)添上sa_mask 中的码位。
    
```

#2.caller

```
//进行系统调用后，做善后信号处理
_system_call
--call [_sys_call_table+eax*4]
--call _do_signal

```