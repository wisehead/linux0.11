#1.sched_init

```cpp
sched_init
--set_tss_desc
--set_ldt_desc
--__asm__("pushfl ; andl $0xffffbfff,(%esp) ; popfl")
--ltr(0)
--lldt(0)
--set_intr_gate(0x20,&timer_interrupt);
--set_system_gate(0x80,&system_call)
```