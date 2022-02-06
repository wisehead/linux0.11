#1.csi_J

```cpp
//// 删除屏幕上与光标位置相关的部分，以屏幕为单位。csi - 控制序列引导码(Control Sequence
// Introducer)。
// ANSI 转义序列：'ESC [sJ'(s = 0 删除光标到屏幕底端；1 删除屏幕开始到光标处；2 整屏删除)。
// 参数：par - 对应上面s。

csi_J
// 首先根据三种情况分别设置需要删除的字符数和删除开始的显示内存位置。                                                   
--switch (par)                                                                         
--{                                                                                    
--case 0:         /* erase from cursor to end of display *//* 擦除光标到屏幕底端 */             
      count = (scr_end - pos) >> 1;                                                    
      start = pos;                                                                     
      break;                                                                           
--case 1:         /* erase from start to cursor *//* 删除从屏幕开始到光标处的字符 */                 
      count = (pos - origin) >> 1;                                                     
      start = origin;                                                                  
      break;                                                                           
--case 2:         /* erase whole display *//* 删除整个屏幕上的字符 */                            
      count = video_num_columns * video_num_lines;                                     
      start = origin;                                                                  
      break;                                                                           
--default:                                                                             
      return;                                                                          
--}                                                                                    
// 然后使用擦除字符填写删除字符的地方。                                                                  
// %0 - ecx(要删除的字符数count)；%1 - edi(删除操作开始地址)；%2 - eax（填入的擦除字符）。                        
--_asm {                                                                               
      pushf                                                                            
      mov ecx,count;                                                                   
      mov edi,start;                                                                   
      mov ax,video_erase_char;                                                         
      cld;                                                                             
      rep stosw;                                                                       
      popf                                                                             
--}                                                                                    
```

#2.caller

con_write