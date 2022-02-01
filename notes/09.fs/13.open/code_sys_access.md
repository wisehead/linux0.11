#1.sys_access

```cpp
/*
 * 文件属性XXX，我们该用真实用户id 还是有效用户id？BSD 系统使用了真实用户id，
 * 以使该调用可以供setuid 程序使用。（注：POSIX 标准建议使用真实用户ID）
 */
//// 检查对文件的访问权限。
// 参数filename 是文件名，mode 是屏蔽码，由R_OK(4)、W_OK(2)、X_OK(1)和F_OK(0)组成。
// 如果请求访问允许的话，则返回0，否则返回出错码。
int sys_access(const char * filename,int mode)


// 屏蔽码由低3 位组成，因此清除所有高比特位。
    mode &= 0007;
// 如果文件名对应的i 节点不存在，则返回出错码。
    if (!(inode=namei(filename)))
        return -EACCES;
// 取文件的属性码，并释放该i 节点。
    i_mode = res = inode->i_mode & 0777;
    iput(inode);
// 如果当前进程是该文件的宿主，则取文件宿主属性。
    if (current->uid == inode->i_uid)
        res >>= 6;
// 否则如果当前进程是与该文件同属一组，则取文件组属性。
    else if (current->gid == inode->i_gid)
        res >>= 6;
// 如果文件属性具有查询的属性位，则访问许可，返回0。
    if ((res & 0007 & mode) == mode)
        return 0;
/*
 * XXX 我们最后才做下面的测试，因为我们实际上需要交换有效用户id 和
 * 真实用户id（临时地），然后才调用suser()函数。如果我们确实要调用
 * suser()函数，则需要最后才被调用。
 */
// 如果当前用户id 为0（超级用户）并且屏蔽码执行位是0 或文件可以被任何人访问，则返回0。
    if ((!current->uid) &&
        (!(mode & 1) || (i_mode & 0111)))
        return 0;
    // 否则返回出错码。
    return -EACCES;
```