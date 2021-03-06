#1.release

```cpp
caller:
- tell_father
- sys_waitpid


release
--for (i=1 ; i<NR_TASKS ; i++)
----if (task[i]==p)
------task[i]=NULL
------free_page
------schedule
```