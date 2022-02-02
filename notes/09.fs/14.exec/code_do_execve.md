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

// 释放该缓冲区。
--brelse(bh);
// 下面对执行头信息进行处理。
// 对于下列情况，将不执行程序：如果执行文件不是需求页可执行文件(ZMAGIC)、或者代码重定位部分
// 长度a_trsize 不等于0、或者数据重定位信息长度不等于0、或者代码段+数据段+堆段长度超过50MB、
// 或者i 节点表明的该执行文件长度小于代码段+数据段+符号表长度+执行头部分长度的总和。
--if (N_MAGIC(ex) != ZMAGIC || ex.a_trsize || ex.a_drsize ||
        ex.a_text+ex.a_data+ex.a_bss>0x3000000 ||
        inode->i_size < ex.a_text+ex.a_data+ex.a_syms+N_TXTOFF(ex)) {
        retval = -ENOEXEC;
        goto exec_error2;
 --}
 
 // 如果sh_bang 标志没有设置，则复制指定个数的环境变量字符串和参数到参数和环境空间中。
// 若sh_bang 标志已经设置，则表明是将运行脚本程序，此时环境变量页面已经复制，无须再复制。
--if (!sh_bang) 
----p = copy_strings(envc,envp,page,p,0);
----p = copy_strings(argc,argv,page,p,0);

/* OK，下面开始就没有返回的地方了 */
// 如果原程序也是一个执行程序，则释放其i 节点，并让进程executable 字段指向新程序i 节点。
--if (current->executable)
----iput(current->executable);
--current->executable = inode;

// 清复位所有信号处理句柄。但对于SIG_IGN 句柄不能复位，因此在322 与323 行之间需添加一条
// if 语句：if (current->sa[I].sa_handler != SIG_IGN)。这是源代码中的一个bug。
--for (i=0 ; i<32 ; i++)
        current->sigaction[i].sa_handler = NULL;
// 根据执行时关闭(close_on_exec)文件句柄位图标志，关闭指定的打开文件，并复位该标志。
--for (i=0 ; i<NR_OPEN ; i++)
        if ((current->close_on_exec>>i)&1)
            sys_close(i);
--current->close_on_exec = 0;    

// 根据指定的基地址和限长，释放原来程序代码段和数据段所对应的内存页表指定的内存块及页表本身。
--free_page_tables(get_base(current->ldt[1]),get_limit(0x0f));
--free_page_tables(get_base(current->ldt[2]),get_limit(0x17));

// 根据a_text 修改局部表中描述符基址和段限长，并将参数和环境空间页面放置在数据段末端。
// 执行下面语句之后，p 此时是以数据段起始处为原点的偏移值，仍指向参数和环境空间数据开始处，
// 也即转换成为堆栈的指针。
--p += change_ldt(ex.a_text,page)-MAX_ARG_PAGES*PAGE_SIZE;    
// create_tables()在新用户堆栈中创建环境和参数变量指针表，并返回该堆栈指针。
--p = (unsigned long) create_tables((char *)p,argc,envc);
// 修改当前进程各字段为新执行程序的信息。令进程代码段尾值字段end_code = a_text；令进程数据
// 段尾字段end_data = a_data + a_text；令进程堆结尾字段brk = a_text + a_data + a_bss。
--current->brk = ex.a_bss +(current->end_data = ex.a_data +(current->end_code = ex.a_text));
// 设置进程堆栈开始字段为堆栈指针所在的页面，并重新设置进程的用户id 和组id。
--current->start_stack = p & 0xfffff000;
--current->euid = e_uid;
--current->egid = e_gid;

// 初始化一页bss 段数据，全为零。                                                                  
--i = ex.a_text+ex.a_data;                                                            
--while (i&0xfff)                                                                     
----put_fs_byte(0,(char *) (i++));                                                    
// 将原调用系统中断的程序在堆栈上的代码指针替换为指向新执行程序的入口点，并将堆栈指针替换                                        
// 为新执行程序的堆栈指针。返回指令将弹出这些堆栈数据并使得CPU 去执行新的执行程序，因此不会                                     
// 返回到原调用系统中断的程序中去了。                                                                  
--eip[0] = ex.a_entry;        /* eip，魔法起作用了 :-) */                                    
--eip[3] = p;         /* esp，堆栈指针 */                                                  
--return 0;                                                                           
                                                                                      
                                                                                   
```

#2.stack map

```cpp
----
orig ss
----
orig ESP
----
eflags
----
cs
----
eip
----
```