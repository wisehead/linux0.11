#1.main

```cpp
main
--mem_init
----HIGH_MEMORY = end_mem;
----for (i=0 ; i<PAGING_PAGES ; i++)
------mem_map[i] = USED;
--trap_init
--blk_dev_init
--chr_dev_init
--tty_init
----rs_init
----con_init
--time_init
--sched_init
--buffer_init
--hd_init
--floppy_init
--sti
--move_to_user_mode
--if (!fork())
----init
--for(;;) pause();
```

#2.init

```cpp
init
--_syscall1(int,setup,void *,BIOS)
--open("/dev/tty0",O_RDWR,0);
--if (!(pid=fork()))//child
----open("/etc/rc",O_RDONLY,0)
----execve("/bin/sh",argv_rc,envp_rc)
--else//father
----while (pid != wait(&i))
--while (1)
----fork
----if (!pid) //child
------setsid
------open("/dev/tty0",O_RDWR,0)
------_exit(execve("/bin/sh",argv,envp));
----while (1)
------if (pid == wait(&i)) break;
----sync
--//end while
--_exit(0)
```