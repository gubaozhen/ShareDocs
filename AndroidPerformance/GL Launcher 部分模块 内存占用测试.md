# GL 占用测试

> trunk 3.0 #90770
>
> 测试机型：N5 6.0

### dump场景：

1. 正常运行——静置3min
2. 进入负一屏
3. 进入3D时钟
4. 进入AllApps
5. 进入天气
6. 进入搜索
7. 应用3D主题（）



## 测试结论

> 以下结论基于N5 6.0机型，结果未与其它机器、安卓版本做横向比较

* launcher进程内，各模块切换时未出现太大波动
* 但launcher起始值（Gfx dev）在 40000KB（~40M）左右，考虑这一点是否有改进必要、空间？
* 3D主题相较2D，内存占用升高~17M左右（具体值未严格测试）



## 实验数据

### 正常运行——静置3min

~~~java
** MEMINFO in pid 16257 [com.ksmobile.launcher] **
                   Pss  Private  Private  Swapped     Heap     Heap     Heap
                 Total    Dirty    Clean    Dirty     Size    Alloc     Free
                ------   ------   ------   ------   ------   ------   ------
  Native Heap    25108    25080        0        0    43264    36859     6404
  Dalvik Heap    42114    42040        0        0    65438    50717    14721
 Dalvik Other     3860     3860        0        0
        Stack     1516     1516        0        0
       Ashmem      176      140        0        0
      Gfx dev    41004    41004        0        0
    Other dev      136        0      136        0
     .so mmap     8610      320      812        0
    .apk mmap     2555        0     2100        0
    .ttf mmap      173        0      128        0
    .dex mmap    29908       48    23812        0
    .oat mmap     2767        0      900        0
    .art mmap     1705     1528        4        0
   Other mmap     1078        8      608        0
   EGL mtrack    44880    44880        0        0
      Unknown     4924     4924        0        0
        TOTAL   210514   165348    28500        0   108702    87576    21125

 App Summary
                       Pss(KB)
                        ------
           Java Heap:    43572
         Native Heap:    25080
                Code:    28120
               Stack:     1516
            Graphics:    85884
       Private Other:     9676
              System:    16666

               TOTAL:   210514      TOTAL SWAP (KB):        0
~~~



### 进入负一屏

> 未点击触发任何动画

```Java
** MEMINFO in pid 16257 [com.ksmobile.launcher] **
                   Pss  Private  Private  Swapped     Heap     Heap     Heap
                 Total    Dirty    Clean    Dirty     Size    Alloc     Free
                ------   ------   ------   ------   ------   ------   ------
  Native Heap    25583    25548        0        0    43264    37267     5996
  Dalvik Heap    45619    45532        0        0    65017    57890     7127
 Dalvik Other     3900     3900        0        0
        Stack     1520     1520        0        0
       Ashmem      176      140        0        0
      Gfx dev    42456    42456        0        0
    Other dev      136        0      136        0
     .so mmap     9323      320      960        0
    .apk mmap     2583        0     2104        0
    .ttf mmap      173        0      128        0
    .dex mmap    30308       48    24112        0
    .oat mmap     3297        0     1284        0
    .art mmap     2015     1540      272        0
   Other mmap     1223        8      696        0
   EGL mtrack    53520    53520        0        0
      Unknown     4924     4924        0        0
        TOTAL   226756   179456    29692        0   108281    95157    13123

 App Summary
                       Pss(KB)
                        ------
           Java Heap:    47344
         Native Heap:    25548
                Code:    28956
               Stack:     1520
            Graphics:    95976
       Private Other:     9804
              System:    17608

               TOTAL:   226756      TOTAL SWAP (KB):        0
```



### 进入3D时钟
```Java
** MEMINFO in pid 16257 [com.ksmobile.launcher] **
                   Pss  Private  Private  Swapped     Heap     Heap     Heap
                 Total    Dirty    Clean    Dirty     Size    Alloc     Free
                ------   ------   ------   ------   ------   ------   ------
  Native Heap    25243    25216        0        0    43776    36699     7076
  Dalvik Heap    42553    42480        0        0    66333    50306    16027
 Dalvik Other     3692     3692        0        0
        Stack     1532     1532        0        0
       Ashmem      176      140        0        0
      Gfx dev    42884    42884        0        0
    Other dev      136        0      136        0
     .so mmap     8622      320      836        0
    .apk mmap     2549        0     2108        0
    .ttf mmap      173        0      128        0
    .dex mmap    29687       48    23884        0
    .oat mmap     3646        0     1660        0
    .art mmap     2539     1564      792        0
   Other mmap     1077        8      608        0
   EGL mtrack    36240    36240        0        0
      Unknown     4924     4924        0        0
        TOTAL   205673   159048    30152        0   110109    87005    23103

 App Summary
                       Pss(KB)
                        ------
           Java Heap:    44836
         Native Heap:    25216
                Code:    28984
               Stack:     1532
            Graphics:    79124
       Private Other:     9508
              System:    16473

               TOTAL:   205673      TOTAL SWAP (KB):        0
```



###  进入AllApps
```Java
** MEMINFO in pid 16257 [com.ksmobile.launcher] **
                   Pss  Private  Private  Swapped     Heap     Heap     Heap
                 Total    Dirty    Clean    Dirty     Size    Alloc     Free
                ------   ------   ------   ------   ------   ------   ------
  Native Heap    25539    25512        0        0    43776    37873     5902
  Dalvik Heap    45413    45340        0        0    66146    57815     8331
 Dalvik Other     3960     3960        0        0
        Stack     1532     1532        0        0
       Ashmem      176      140        0        0
      Gfx dev    39380    39380        0        0
    Other dev      136        0      136        0
     .so mmap     8620      320      836        0
    .apk mmap     2549        0     2108        0
    .ttf mmap      173        0      128        0
    .dex mmap    29887       48    24052        0
    .oat mmap     3646        0     1648        0
    .art mmap     2558     1584      792        0
   Other mmap     1127        8      636        0
   EGL mtrack    44880    44880        0        0
      Unknown     4924     4924        0        0
        TOTAL   214500   167628    30336        0   109922    95688    14233

 App Summary
                       Pss(KB)
                        ------
           Java Heap:    47716
         Native Heap:    25512
                Code:    29140
               Stack:     1532
            Graphics:    84260
       Private Other:     9804
              System:    16536

               TOTAL:   214500      TOTAL SWAP (KB):        0
```



###  进入天气
```Java
** MEMINFO in pid 16257 [com.ksmobile.launcher] **
                   Pss  Private  Private  Swapped     Heap     Heap     Heap
                 Total    Dirty    Clean    Dirty     Size    Alloc     Free
                ------   ------   ------   ------   ------   ------   ------
  Native Heap    30251    30224        0        0    50176    46472     3703
  Dalvik Heap    56889    56816        0        0    82047    68387    13660
 Dalvik Other     4124     4124        0        0
        Stack     1872     1872        0        0
       Ashmem      228      144        0        0
      Gfx dev    48684    48684        0        0
    Other dev      136        0      136        0
     .so mmap     9119      320     1216        0
    .apk mmap     2537        0     2096        0
    .ttf mmap      261        0      216        0
    .dex mmap    31424       48    25576        0
    .oat mmap     3819        0     1784        0
    .art mmap     2593     1620      792        0
   Other mmap     1397        8      852        0
   EGL mtrack    39552    39552        0        0
      Unknown     9044     9044        0        0
        TOTAL   241930   192456    32668        0   132223   114859    17363

 App Summary
                       Pss(KB)
                        ------
           Java Heap:    59228
         Native Heap:    30224
                Code:    31256
               Stack:     1872
            Graphics:    88236
       Private Other:    14308
              System:    16806

               TOTAL:   241930      TOTAL SWAP (KB):        0
```



###  进入搜索
```Java
** MEMINFO in pid 16257 [com.ksmobile.launcher] **
                   Pss  Private  Private  Swapped     Heap     Heap     Heap
                 Total    Dirty    Clean    Dirty     Size    Alloc     Free
                ------   ------   ------   ------   ------   ------   ------
  Native Heap    30979    30952        0        0    51712    46697     5014
  Dalvik Heap    53559    53488        0        0    78029    62140    15889
 Dalvik Other     4084     4084        0        0
        Stack     1864     1864        0        0
       Ashmem      224      152        0        0
      Gfx dev    44948    44948        0        0
    Other dev      136        0      136        0
     .so mmap     9092      320     1184        0
    .apk mmap     1214        0      812        0
    .ttf mmap      224        0      204        0
    .dex mmap    31530       48    25696        0
    .oat mmap     4048        0     2056        0
    .art mmap     2619     1704      732        0
   Other mmap     1449        8      812        0
   EGL mtrack    53520    53520        0        0
      Unknown     8700     8700        0        0
        TOTAL   248190   199788    31632        0   129741   108837    20903

 App Summary
                       Pss(KB)
                        ------
           Java Heap:    55924
         Native Heap:    30952
                Code:    30320
               Stack:     1864
            Graphics:    98468
       Private Other:    13892
              System:    16770

               TOTAL:   248190      TOTAL SWAP (KB):        0
```

###  应用3D主题
```Java
** MEMINFO in pid 23770 [com.ksmobile.launcher] **
                   Pss  Private  Private  Swapped     Heap     Heap     Heap
                 Total    Dirty    Clean    Dirty     Size    Alloc     Free
                ------   ------   ------   ------   ------   ------   ------
  Native Heap    16450    16424        0        0    24832    22375     2456
  Dalvik Heap    32283    32212        0        0    57652    46603    11049
 Dalvik Other     3352     3352        0        0
        Stack      864      864        0        0
       Ashmem      130      128        0        0
      Gfx dev    58872    58872        0        0
    Other dev        4        0        4        0
     .so mmap     3563      268     1692        0
    .apk mmap     1359        0      896        0
    .ttf mmap      114        0      112        0
    .dex mmap    24990       32    20756        0
    .oat mmap     3203        0     1672        0
    .art mmap     1352     1152       16        0
   Other mmap      913        8      716        0
   EGL mtrack    44880    44880        0        0
      Unknown      300      300        0        0
        TOTAL   192629   158492    25864        0    82484    68978    13505

 App Summary
                       Pss(KB)
                        ------
           Java Heap:    33380
         Native Heap:    16424
                Code:    25428
               Stack:      864
            Graphics:   103752
       Private Other:     4508
              System:     8273

               TOTAL:   192629      TOTAL SWAP (KB):        0
```