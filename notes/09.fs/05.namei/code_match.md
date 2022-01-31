#1.match

```cpp
/*
 * ok，我们不能使用strncmp 字符串比较函数，因为名称不在我们的数据空间(不在内核空间)。
 * 因而我们只能使用match()。问题不大。match()同样也处理一些完整的测试。
 *
 * 注意！与strncmp 不同的是match()成功时返回1，失败时返回0。
 */
//// 指定长度字符串比较函数。
// 参数：len - 比较的字符串长度；name - 文件名指针；de - 目录项结构。
// 返回：相同返回1，不同返回0。
static int match(int len,const char * name,struct dir_entry * de)
{
    register int same; //__asm__("ax")
    char *de_name;

// 如果目录项指针空，或者目录项i 节点等于0，或者要比较的字符串长度超过文件名长度，则返回0。
    if (!de || !de->inode || len > NAME_LEN)
        return 0;
// 如果要比较的长度len 小于NAME_LEN，但是目录项中文件名长度超过len，则返回0。
    if (len < NAME_LEN && de->name[len])
        return 0;
// 下面嵌入汇编语句，在用户数据空间(fs)执行字符串的比较操作。
// %0 - eax(比较结果same)；%1 - eax(eax 初值0)；%2 - esi(名字指针)；%3 - edi(目录项名指针)；
// %4 - ecx(比较的字节长度值len)。
/*  __asm__("cld\n\t"               // 清方向位。
        "fs ; repe ; cmpsb\n\t"     // 用户空间执行循环比较[esi++]和[edi++]操作，
        "setz %%al"                 // 若比较结果一样(z=0)则设置al=1(same=eax)。
        :"=a" (same)
        :"0" (0),"S" ((long) name),"D" ((long) de->name),"c" (len)
        :"cx","di","si");*/
    de_name = de->name;
    _asm{
        pushf
        xor eax,eax
        mov esi,name
        mov edi,de_name
        mov ecx,len
        cld     // 清方向位。
        // 用户空间执行循环比较[esi++]和[edi++]操作，
        repe cmps byte ptr fs:[edi],[esi]
        //上语句应该是错误的，但我不知道怎么改。还好系统可以运行:)
        setz al         // 若比较结果一样(z=0)则设置al=1(same=eax)。
        mov same,eax
        popf
    }
    return same;            // 返回比较结果。
}

```