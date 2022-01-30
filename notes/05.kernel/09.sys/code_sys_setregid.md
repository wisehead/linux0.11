#1.sys_setregid

```cpp
// 设置当前任务的实际以及/或者有效组ID（gid）。如果任务没有超级用户特权，
// 那么只能互换其实际组ID 和有效组ID。如果任务具有超级用户特权，就能任意设置有效的和实际
// 的组ID。保留的gid（saved gid）被设置成与有效gid 同值。
int sys_setregid (int rgid, int egid)
{
    if (rgid > 0)
    {
        if ((current->gid == rgid) || suser ())
            current->gid = rgid;
        else
            return (-EPERM);
    }
    if (egid > 0)
    {
        if ((current->gid == egid) || (current->egid == egid) ||
                                (current->sgid == egid) || suser ())
            current->egid = egid;
        else
            return (-EPERM);
    }
    return 0;
}

```