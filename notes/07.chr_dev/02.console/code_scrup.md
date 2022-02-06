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
------else 
------{
--------// 如果调整后的屏幕末端对应的内存指针scr_end 没有超出显示内存的末端video_mem_end，则只需在               
--------// 新行上填入擦除字符(空格字符)。                                                     
--------// %0 - eax(擦除字符+属性)；%1 - ecx(显示器字符行数)；%2 - edi(屏幕对应内存最后一行开始处)；         
--------t1 = scr_end - video_size_row;                                          
--------_asm {                                                                  
            pushf                                                               
            mov ax,video_erase_char;                                            
            mov ecx,video_num_columns;                                          
            mov edi,t1;                                                         
            //mov edi,(scr_end - video_size_row);                               
            cld;    // 清方向位。                                                    
            rep stosw;  // 重复操作，在新出现行上填入擦除字符(空格字符)。                             
            popf                                                                
--------}                                                                                                                     
------}//end else
// 向显示控制器中写入新的屏幕内容对应的内存起始位置值。
------set_origin ();
----}//end if (!top && bottom == video_num_lines)
// 否则表示不是整屏移动。也即表示从指定行top 开始的所有行向上移动1 行(删除1 行)。此时直接
// 将屏幕从指定行top 到屏幕末端所有行对应的显示内存数据向上移动1 行，并在新出现的行上填入擦
// 除字符。
// %0-eax(擦除字符+属性)；%1-ecx(top 行下1 行开始到屏幕末行的行数所对应的内存长字数)；
// %2-edi(top 行所处的内存位置)；%3-esi(top+1 行所处的内存位置)。
----else
----{
--------t1 = (bottom - top - 1) * video_num_columns >> 1;                         
--------t2 = origin + video_size_row * top;                                       
--------t3 = origin + video_size_row * (top + 1);                                 
--------_asm {                                                                    
            pushf                                                                 
            //mov ecx,((bottom - top - 1) * video_num_columns >> 1);              
            mov ecx,t1;                                                           
            //mov edi,(origin + video_size_row * top);                            
            mov edi,t2;                                                           
            //mov esi,(origin + video_size_row * (top + 1));                      
            mov esi,t3;                                                           
            mov ax,video_erase_char;                                              
            cld;    // 清方向位。                                                      
            rep movsd;// 循环操作，将top+1 到bottom 行 所对应的内存块移到top 行开始处。                 
            mov ecx,video_num_columns;  // ecx = 1 行字符数。                          
            rep stosw;// 在新行上填入擦除字符。                                              
            popf                                                                  
--------}                                                                         
                                                                         
----}//end else
--}//end if (video_type == VIDEO_TYPE_EGAC || video_type == VIDEO_TYPE_EGAM)
// 如果显示类型不是EGA(是MDA)，则执行下面移动操作。因为MDA 显示控制卡会自动调整超出显示范围
// 的情况，也即会自动翻卷指针，所以这里不对屏幕内容对应内存超出显示内存的情况单独处理。处理
// 方法与EGA 非整屏移动情况完全一样。
--else                /* Not EGA/VGA */
--{
--}
```