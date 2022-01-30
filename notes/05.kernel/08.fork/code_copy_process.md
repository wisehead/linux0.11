#1.copy_process

```cpp
/*
* OK，下面是主要的fork 子程序。它复制系统进程信息(task[n])并且设置必要的寄存器。
* 它还整个地复制数据段。
*/
// 复制进程。
copy_process
--p = (struct task_struct *) get_free_page ();    // 为新任务数据结构分配内存
--task[nr] = p;           // 将新任务结构指针放入任务数组中。
// 其中nr 为任务号，由前面find_empty_process()返回。
--*p = *current;      /* NOTE! this doesn't copy the supervisor stack */
/* 注意！这样做不会复制超级用户的堆栈 （只复制当前进程内容）。*/
    p->state = TASK_UNINTERRUPTIBLE;    // 将新进程的状态先置为不可中断等待状态。
    p->pid = last_pid;      // 新进程号。由前面调用find_empty_process()得到。
    p->father = current->pid;   // 设置父进程号。
    p->counter = p->priority;
    p->signal = 0;      // 信号位图置0。
    p->alarm = 0;
    p->leader = 0;      /* process leadership doesn't inherit */
/* 进程的领导权是不能继承的 */
    p->utime = p->stime = 0;    // 初始化用户态时间和核心态时间。
    p->cutime = p->cstime = 0;  // 初始化子进程用户态和核心态时间。
    p->start_time = jiffies;    // 当前滴答数时间。
// 以下设置任务状态段TSS 所需的数据（参见列表后说明）。
    p->tss.back_link = 0;
    p->tss.esp0 = PAGE_SIZE + (long) p; // 堆栈指针（由于是给任务结构p 分配了1 页
// 新内存，所以此时esp0 正好指向该页顶端）。
    p->tss.ss0 = 0x10;      // 堆栈段选择符（内核数据段）[??]。
    p->tss.eip = eip;       // 指令代码指针。
    p->tss.eflags = eflags; // 标志寄存器。
    p->tss.eax = 0;
    p->tss.ecx = ecx;
    p->tss.edx = edx;
    p->tss.ebx = ebx;
    p->tss.esp = esp;
    p->tss.ebp = ebp;
    p->tss.esi = esi;
    p->tss.edi = edi;
    p->tss.es = es & 0xffff;    // 段寄存器仅16 位有效。
    p->tss.cs = cs & 0xffff;
    p->tss.ss = ss & 0xffff;
    p->tss.ds = ds & 0xffff;
    p->tss.fs = fs & 0xffff;
    p->tss.gs = gs & 0xffff;
    p->tss.ldt = _LDT (nr); // 该新任务nr 的局部描述符表选择符（LDT 的描述符在GDT 中）。
    p->tss.trace_bitmap = 0x80000000;
// 如果当前任务使用了协处理器，就保存其上下文。
    p_i387 = &p->tss.i387;
    if (last_task_used_math == current)
    _asm{
        mov ebx, p_i387
        clts
        fnsave [p_i387]
    }
//    __asm__ ("clts ; fnsave %0"::"m" (p->tss.i387));
// 设置新任务的代码和数据段基址、限长并复制页表。如果出错（返回值不是0），则复位任务数组中
// 相应项并释放为该新任务分配的内存页。
    if (copy_mem (nr, p))
    {               // 返回不为0 表示出错。
        task[nr] = NULL;
        free_page ((long) p);
        return -EAGAIN;
    }
// 如果父进程中有文件是打开的，则将对应文件的打开次数增1。
    for (i = 0; i < NR_OPEN; i++)
        if (f = p->filp[i])
            f->f_count++;
// 将当前进程（父进程）的pwd, root 和executable 引用次数均增1。
    if (current->pwd)
        current->pwd->i_count++;
    if (current->root)
        current->root->i_count++;
    if (current->executable)
        current->executable->i_count++;
// 在GDT 中设置新任务的TSS 和LDT 描述符项，数据从task 结构中取。
// 在任务切换时，任务寄存器tr 由CPU 自动加载。
    set_tss_desc (gdt + (nr << 1) + FIRST_TSS_ENTRY, &(p->tss));
    set_ldt_desc (gdt + (nr << 1) + FIRST_LDT_ENTRY, &(p->ldt));
    p->state = TASK_RUNNING;    /* do this last, just in case */
/* 最后再将新任务设置成可运行状态，以防万一 */
    return last_pid;        // 返回新进程号（与任务号是不同的）。    


```

#2.设置TSS/LDT
注意：GDT中TSS/LDT的表项中，存的是TSS结构和LDT结构的地址，CPU寄存器TR /LDTR 会自动load，然后再去找内部的信息。

```

// 在GDT 中设置新任务的TSS 和LDT 描述符项，数据从task 结构中取。
// 在任务切换时，任务寄存器tr 由CPU 自动加载。
    set_tss_desc (gdt + (nr << 1) + FIRST_TSS_ENTRY, &(p->tss));
    set_ldt_desc (gdt + (nr << 1) + FIRST_LDT_ENTRY, &(p->ldt));
    
```