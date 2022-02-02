#1.struct stat

```cpp

struct stat
{
  dev_t st_dev;         // 含有文件的设备号。
  ino_t st_ino;         // 文件i 节点号。
  umode_t st_mode;      // 文件属性（见下面）。
  nlink_t st_nlink;     // 指定文件的连接数。
  uid_t st_uid;         // 文件的用户(标识)号。
  gid_t st_gid;         // 文件的组号。
  dev_t st_rdev;        // 设备号(如果文件是特殊的字符文件或块文件)。
  off_t st_size;        // 文件大小（字节数）（如果文件是常规文件）。
  time_t st_atime;      // 上次（最后）访问时间。
  time_t st_mtime;      // 最后修改时间。
  time_t st_ctime;      // 最后节点修改时间。
};
```