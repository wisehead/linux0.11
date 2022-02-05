#1.scrup

```cpp
//// 向上卷动一行（屏幕窗口向下移动）。
// 将屏幕窗口向下移动一行。参见程序列表后说明。

scrup
// 如果显示类型是EGA，则执行以下操作。
--if (video_type == VIDEO_TYPE_EGAC || video_type == VIDEO_TYPE_EGAM)
--{
        // 如果移动起始行top=0，移动最底行bottom=video_num_lines=25，则表示整屏窗口向下移动。
----if (!top && bottom == video_num_lines)
----{
------// 调整屏幕显示对应内存的起始位置指针origin 为向下移一行屏幕字符对应的内存位置，同时也调整                                            
------// 当前光标对应的内存位置以及屏幕末行末端字符指针scr_end 的位置。                                                        
------origin += video_size_row;                                                                     
------pos += video_size_row;                                                                        
------scr_end += video_size_row;                                                                    
                                                                                                    // 如果屏幕末端最后一个显示字符所对应的显示内存指针scr_end 超出了实际显示内存的末端，则将
// 屏幕内容内存数据移动到显示内存的起始位置video_mem_start 处，并在出现的新行上填入空格字符。
------if (scr_end > video_mem_end)
------{
--------// %0 - eax(擦除字符+属性)；%1 - ecx((显示器字符行数-1)所对应的字符数/2，是以长字移动)；                 
--------// %2 - edi(显示内存起始位置video_mem_start)；%3 - esi(屏幕内容对应的内存起始位置origin)。         
--------// 移动方向：[edi]->[esi]，移动ecx 个长字。                                             
--------t1 = (video_num_lines - 1) * video_num_columns >> 1;                        
--------_asm {                                                                      
            pushf                                                                   
            mov ecx,t1;                                                             
            //            mov ecx,((video_num_lines - 1) * video_num_columns >> 1); 
            mov ax,video_erase_char;                                                
            mov edi,video_mem_start;                                                
            mov esi,origin;                                                         
            cld;    // 清方向位。                                                        
            rep movsd;  // 重复操作，将当前屏幕内存数据移动到显示内存起始处。                                
            mov ecx,video_num_columns;  // ecx=1 行字符数。                              
            rep stosw;  // 在新行上填入空格字符。                                              
            popf                                                                    
--------}                                                                           
--------// 根据屏幕内存数据移动后的情况，重新调整当前屏幕对应内存的起始指针、光标位置指针和屏幕末端                                  
--------// 对应内存指针scr_end。                                                                
--------scr_end -= origin - video_mem_start;                                             
--------pos -= origin - video_mem_start;                                                 
--------origin = video_mem_start;                                                                                                                    
------}//end if (scr_end > video_mem_end)
----}//end if (!top && bottom == video_num_lines)
----else

--}
// 如果显示类型不是EGA(是MDA)，则执行下面移动操作。因为MDA 显示控制卡会自动调整超出显示范围
// 的情况，也即会自动翻卷指针，所以这里不对屏幕内容对应内存超出显示内存的情况单独处理。处理
// 方法与EGA 非整屏移动情况完全一样。
--else                /* Not EGA/VGA */
--{
--}
```