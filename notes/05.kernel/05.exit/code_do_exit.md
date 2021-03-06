#1.do_exit

```cpp
do_exit
--free_page_tables(get_base(current->ldt[1]),get_limit(0x0f));
--free_page_tables(get_base(current->ldt[2]),get_limit(0x17))
--其它的以后再说，很多都没有接触到。
```

