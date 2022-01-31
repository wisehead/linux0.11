#1.find_first_zero

```
//// 从addr 开始寻找第1 个0 值比特位。
// 输入：%0 - ecx(返回值)；%1 - ecx(0)；%2 - esi(addr)。
// 在addr 指定地址开始的位图中寻找第1 个是0 的比特位，并将其距离addr 的比特位偏移值返回。
extern _inline int find_first_zero(char *addr)
{
//  int __res;
    _asm{
        pushf
        xor ecx,ecx
        mov esi,addr
        cld   /*清方向位。*/
    l1: lodsd   /*取[esi] -> eax。*/
        not eax   /*eax 中每位取反。*/
        bsf edx,eax   /*从位0 扫描eax 中是1 的第1 个位，其偏移值 -> edx。*/
        je l2   /*如果eax 中全是0，则向前跳转到标号2 处(40 行)。*/
        add ecx,edx   /*偏移值加入ecx(ecx 中是位图中首个是0 的比特位的偏移值)*/
        jmp l3   /*向前跳转到标号3 处（结束）。*/
    l2: add ecx,32   /*没有找到0 比特位，则将ecx 加上1 个长字的位偏移量32。*/
        cmp ecx,8192   /*已经扫描了8192 位（1024 字节）了吗？*/
        jl l1  /*若还没有扫描完1 块数据，则向前跳转到标号1 处，继续。*/
//  l3: mov __res,ecx  /*结束。此时ecx 中是位偏移量。*/
    l3: mov eax,ecx
        popf
    }
//  return __res;
}
/*#define find_first_zero(addr) ({ \
int __res; \
__asm__("cld\n" \
    "1:\tlodsl\n\t" \
    "notl %%eax\n\t" \
    "bsfl %%eax,%%edx\n\t" \
    "je 2f\n\t" \
    "addl %%edx,%%ecx\n\t" \
    "jmp 3f\n" \
    "2:\taddl $32,%%ecx\n\t" \
    "cmpl $8192,%%ecx\n\t" \
    "jl 1b\n" \
    "3:" \
    :"=c" (__res):"c" (0),"S" (addr):"ax","dx","si"); \
__res;})*/
```