#1.permission

```cpp
/*
 *  permission()
 *
 * 该函数用于检测一个文件的读/写/执行权限。我不知道是否只需检查euid，还是
 * 需要检查euid 和uid 两者，不过这很容易修改。
 */
//// 检测文件访问许可权限。
// 参数：inode - 文件对应的i 节点；mask - 访问属性屏蔽码。
// 返回：访问许可返回1，否则返回0。
static int permission(struct m_inode * inode,int mask)
{
    int mode = inode->i_mode;// 文件访问属性

/* 特殊情况：即使是超级用户(root)也不能读/写一个已被删除的文件 */
// 如果i 节点有对应的设备，但该i 节点的连接数等于0，则返回。
    if (inode->i_dev && !inode->i_nlinks)
        return 0;
// 否则，如果进程的有效用户id(euid)与i 节点的用户id 相同，则取文件宿主的用户访问权限。
    else if (current->euid==inode->i_uid)
        mode >>= 6;
// 否则，如果进程的有效组id(egid)与i 节点的组id 相同，则取组用户的访问权限。
    else if (current->egid==inode->i_gid)
        mode >>= 3;
// 如果上面所取的的访问权限与屏蔽码相同，或者是超级用户，则返回1，否则返回0。
    if (((mode & mask & 0007) == mask) || suser()) // suser()在linux/kernel.h。
        return 1;
    return 0;
}

```