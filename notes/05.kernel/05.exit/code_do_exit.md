#1.do_exit

```cpp
do_exit
--free_page_tables(get_base(current->ldt[1]),get_limit(0x0f));
--free_page_tables(get_base(current->ldt[2]),get_limit(0x17))
// 如果当前进程有子进程，就将子进程的father 置为1(其父进程改为进程1)。如果该子进程已经
// 处于僵死(ZOMBIE)状态，则向进程1 发送子进程终止信号SIGCHLD。
--for (i = 0; i < NR_TASKS; i++)
----if (task[i] && task[i]->father == current->pid)
------task[i]->father = 1;
------if (task[i]->state == TASK_ZOMBIE)
--------(void) send_sig (SIGCHLD, task[1], 1);
// 关闭当前进程打开着的所有文件。
--for (i = 0; i < NR_OPEN; i++)
----if (current->filp[i])
------sys_close (i);
// 对当前进程工作目录pwd、根目录root 以及运行程序的i 节点进行同步操作，并分别置空。
--iput (current->pwd);
--current->pwd = NULL;
--iput (current->root);
--current->root = NULL;
--iput (current->executable);
--current->executable = NULL;
// 如果当前进程是领头(leader)进程并且其有控制的终端，则释放该终端。
  if (current->leader && current->tty >= 0)
    tty_table[current->tty].pgrp = 0;
// 如果当前进程上次使用过协处理器，则将last_task_used_math 置空。
  if (last_task_used_math == current)
    last_task_used_math = NULL;
// 如果当前进程是leader 进程，则终止所有相关进程。
--if (current->leader)
----kill_session ();
// 把当前进程置为僵死状态，并设置退出码。
--current->state = TASK_ZOMBIE;
--current->exit_code = code;
// 通知父进程，也即向父进程发送信号SIGCHLD -- 子进程将停止或终止。
--tell_father (current->father);
--schedule ();          // 重新调度进程的运行。
```

#2.caller sys_exit

```
sys_exit
--do_exit ((error_code & 0xff) << 8);s
```
