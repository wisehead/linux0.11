#1.sys_lseek

```cpp
//// 重定位文件读写指针系统调用函数。
// 参数fd 是文件句柄，offset 是新的文件读写指针偏移值，origin 是偏移的起始位置，是SEEK_SET
// (0，从文件开始处)、SEEK_CUR(1，从当前读写位置)、SEEK_END(2，从文件尾处)三者之一。

sys_lseek
// 根据设置的定位标志，分别重新定位文件读写指针。
--switch (origin)
// origin = SEEK_SET，要求以文件起始处作为原点设置文件读写指针。若偏移值小于零，则出错返
// 回错误码。否则设置文件读写指针等于offset。
----case 0:
        if (offset < 0)
            return -EINVAL;
        file->f_pos = offset;
        break;
// origin = SEEK_CUR，要求以文件当前读写指针处作为原点重定位读写指针。如果文件当前指针加
// 上偏移值小于0，则返回出错码退出。否则在当前读写指针上加上偏移值。
----case 1:
        if (file->f_pos + offset < 0)
            return -EINVAL;
        file->f_pos += offset;
        break;
// origin = SEEK_END，要求以文件末尾作为原点重定位读写指针。此时若文件大小加上偏移值小于零
// 则返回出错码退出。否则重定位读写指针为文件长度加上偏移值。
----case 2:
        if ((tmp = file->f_inode->i_size + offset) < 0)
            return -EINVAL;
        file->f_pos = tmp;
        break;
// origin 设置出错，返回出错码退出。
----default:
        return -EINVAL;
--return file->f_pos;     // 返回重定位后的文件读写指针值。
```