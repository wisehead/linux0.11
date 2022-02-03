#1.struct partition

```cpp
// 硬盘分区表结构。参见下面列表后信息。
struct partition
{
  unsigned char boot_ind;   /* 0x80 - active (unused) */
  unsigned char head;       /* ? */
  unsigned char sector;     /* ? */
  unsigned char cyl;        /* ? */
  unsigned char sys_ind;    /* ? */
  unsigned char end_head;   /* ? */
  unsigned char end_sector; /* ? */
  unsigned char end_cyl;    /* ? */
  unsigned int start_sect;  /* starting sector counting from 0 */
  unsigned int nr_sects;    /* nr of sectors in partition */
};
```