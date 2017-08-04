## 问题 答案

1. 关注哪些指标，Gl 这个值是否要关注 
   * 前期可以不用过分关注，但GL的过度使用会消耗系统RAM，影响用户手机性能
   * 各版本安卓处理机制不一样
2. 系统级别or公摊？
   * 不公摊
   * 有些版本会归到系统（S7 7.0）（系统RAM被用尽时，应用依旧能被找到并被终结）、大部分早期系统归属于APP
3. 申请多了会不会导致手机内存不足？
   * 会逐渐消耗RAM，直到系统发现RAM不够用，这时系统将强制停止应用并重新启动应用

## 概念
没有显存的概念，GPU与CPU用的都是RAM，GPU使用的内存是与CPU共享的内存(省去了copy步骤)，实验发现在读System info时会出现freeRAM为负值的情况（[共享内存被重复计算](https://android.stackexchange.com/questions/63854/what-is-lost-ram-in-dumpsys-meminfo)）；


## 1. 关注哪些指标
1. EGLmtrack
2. GLmtrack
3. Graphics
4. Gfx dev（在N5 6.0上发现的指标）



## 2. GL占用的内存是进程级别的，还是每个app均分的

S7 7.0不再累加至APP，只会显示在系统级的占用中，7.0之前的实验手机会归属APP


## 3. 如果被占满，会影响什么

吃RAM，Killer 会杀应用



###  process 占用对比图

* N5 4.4.4

![N5 4.4 SnappyApp SnappyApp, Today at 5.58.46 PM.png](http://upload-images.jianshu.io/upload_images/1481332-8b818d02c51c72cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* N5 6.0

![N5 6 SnappyApp SnappyApp, Today at 6.00.18 PM.png](http://upload-images.jianshu.io/upload_images/1481332-dcda6522c6726272.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* S7 7.0
  ![S7 7.0 SnappyApp SnappyApp, Today at 6.09.12 PM.png](http://upload-images.jianshu.io/upload_images/1481332-1d8ba32468b154d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 参考

1. [Android Memory Analysis](http://processors.wiki.ti.com/index.php/Android_Memory_Analysis#Heap)
2. ​[Open Source Mali GPUs Android Gralloc Module](https://developer.arm.com/products/software/mali-drivers/android-gralloc-module)
3. [ Android 内存详细分析](http://blog.csdn.net/hnulwt/article/details/44900811)
4. [Android 4.4 meminfo implementation and analysis](http://www.programering.com/a/MDNyEzNwATg.html)

## 附录——具体实验数据

* 包括：实验进程、系统总体meminfo


### N5(6.0.1)

* N5(6.0.1) before — process info
~~~java
** MEMINFO in pid 31360 [com.ksmobile.launcher] **
                   Pss  Private  Private  Swapped     Heap     Heap     Heap
                 Total    Dirty    Clean    Dirty     Size    Alloc     Free
                ------   ------   ------   ------   ------   ------   ------
  Native Heap    16511    16484        0        0    24704    23356     1347
  Dalvik Heap    14613    14540        0        0    34618    31199     3419
 Dalvik Other     1844     1844        0        0
        Stack      700      700        0        0
       Ashmem      130      128        0        0
      Gfx dev      436      436        0        0
    Other dev        4        0        4        0
     .so mmap     2520      240       28        0
    .apk mmap     7116        0       84        0
    .ttf mmap       16        0       16        0
    .dex mmap    12888       12     6564        0
    .oat mmap     1738        0      664        0
    .art mmap     1097      916        4        0
   Other mmap      261        8       80        0
   EGL mtrack    38832    38832        0        0
      Unknown      284      284        0        0
        TOTAL    98990    74424     7444        0    59322    54555     4766

 App Summary
                       Pss(KB)
                        ------
           Java Heap:    15460
         Native Heap:    16484
                Code:     7608
               Stack:      700
            Graphics:    39268
       Private Other:     2348
              System:    17122

               TOTAL:    98990      TOTAL SWAP (KB):        0

 Objects
               Views:        9         ViewRootImpl:        1
         AppContexts:        4           Activities:        1
              Assets:        3        AssetManagers:        2
       Local Binders:       54        Proxy Binders:       36
       Parcel memory:       21         Parcel count:       84
    Death Recipients:        3      OpenSSL Sockets:        2
~~~

* N5(6.0.1) after  — process info
  *  450*800 40KB add ~700 times
~~~java
** MEMINFO in pid 23056 [com.ksmobile.launcher] **
                   Pss  Private  Private  Swapped     Heap     Heap     Heap
                 Total    Dirty    Clean    Dirty     Size    Alloc     Free
                ------   ------   ------   ------   ------   ------   ------
  Native Heap    17810    17684        0        0    30208    27535     2672
  Dalvik Heap    16327    15580        0        0    42264    26733    15531
 Dalvik Other     2153     2152        0        0
        Stack     1228     1228        0        0
       Ashmem      160      136        0        0
      Gfx dev  1050976  1050976        0        0
    Other dev       14        0       12        0
     .so mmap     2789      264     1324        0
    .apk mmap      245        0        0        0
    .dex mmap      902       32      836        0
    .oat mmap     1476        0      168        0
    .art mmap     1878     1248        8        0
   Other mmap       70        8       44        0
   EGL mtrack    38832    38832        0        0
      Unknown     3966     3964        0        0
        TOTAL  1138826  1132104     2392        0    72472    54268    18203

 App Summary
                       Pss(KB)
                        ------
           Java Heap:    16836
         Native Heap:    17684
                Code:     2624
               Stack:     1228
            Graphics:  1089808
       Private Other:     6316
              System:     4330

               TOTAL:  1138826      TOTAL SWAP (KB):        0

 Objects
               Views:       10         ViewRootImpl:        1
         AppContexts:        5           Activities:        1
              Assets:        5        AssetManagers:        4
       Local Binders:       66        Proxy Binders:       37
       Parcel memory:       35         Parcel count:      140
    Death Recipients:        1      OpenSSL Sockets:        5
~~~

* N5(6.0.1) before — system info

~~~java
Total PSS by process:
   145909 kB: system (pid 777)
    87058 kB: com.ksmobile.launcher (pid 32395 / activities)
    75342 kB: com.google.android.gms (pid 24986)
    67422 kB: com.android.systemui (pid 921 / activities)
    56581 kB: com.google.android.googlequicksearchbox (pid 30322 / activities)
    47510 kB: com.ss.android.article.news (pid 31181)
    46092 kB: com.cleanmaster.security:DefendService (pid 30154)
    41012 kB: com.google.android.gms.persistent (pid 30925)
    39052 kB: com.cleanmaster.mguard_x86:service (pid 30278)
    38365 kB: com.google.android.googlequicksearchbox:search (pid 30737)
    37485 kB: net.zedge.android (pid 31658)
    34972 kB: com.cleanmaster.mguard_x86 (pid 31260)
    29638 kB: net.zedge.android:remote (pid 31800)
    24995 kB: com.ksmobile.launcher:locker (pid 32452)
    24710 kB: com.qihoo.daemon (pid 29788)
    ...

Total PSS by OOM adjustment:
    83380 kB: Native
               23315 kB: surfaceflinger (pid 189)
               16087 kB: com.qihoo.appstore:feedback (pid 30674)
                6471 kB: mediaserver (pid 198)
                3805 kB: mm-qcamera-daemon (pid 213)
                3472 kB: logd (pid 147)
                3325 kB: com.qihoo.appstore_CoreDaemon (pid 30675)
                3324 kB: com.qihoo.appstore_CoreDaemon (pid 30676)
                2387 kB: zygote (pid 209)
                2213 kB: sdcard (pid 907)
                1895 kB: rild (pid 195)
                1489 kB: drmserver (pid 196)
                1332 kB: sensors.qcom (pid 205)
                1046 kB: thermal-engine-hh (pid 208)
                 ...
   145909 kB: System
              145909 kB: system (pid 777)
    93092 kB: Persistent
               67422 kB: com.android.systemui (pid 921 / activities)
               14645 kB: com.android.phone (pid 1720)
                7697 kB: com.android.nfc (pid 1694)
                3328 kB: com.redbend.vdmc (pid 1707)
    87058 kB: Foreground
               87058 kB: com.ksmobile.launcher (pid 32395 / activities)
   119505 kB: Visible
               41012 kB: com.google.android.gms.persistent (pid 30925)
               24995 kB: com.ksmobile.launcher:locker (pid 32452)
               24710 kB: com.qihoo.daemon (pid 29788)
               22285 kB: com.qihoo.appstore (pid 25085)
                4220 kB: com.google.android.googlequicksearchbox:interactor (pid 8853)
                2283 kB: com.willme.topactivity (pid 7476)
   148221 kB: Perceptible
               75342 kB: com.google.android.gms (pid 24986)
               20060 kB: panda.keyboard.emoji.theme (pid 27216)
               15631 kB: com.ss.android.article.news:pushservice (pid 27252)
               13739 kB: com.ss.android.article.news:push (pid 27199)
               13012 kB: com.ogqcorp.bgh:optimize (pid 29543)
                6337 kB: org.anothermonitor (pid 28517)
                4100 kB: com.lemon.faceu:QALSERVICE (pid 28529)
   140798 kB: A Services
               46092 kB: com.cleanmaster.security:DefendService (pid 30154)
               39052 kB: com.cleanmaster.mguard_x86:service (pid 30278)
               19652 kB: com.cleanmaster.security:ScanService (pid 30910)
               14428 kB: panda.keyboard.emoji.theme:alive (pid 30211)
               12759 kB: com.qihoo.appstore:PluginP06 (pid 30773)
                8815 kB: net.arraynetworks.mobilenow.browser (pid 31486)
    56581 kB: Home
               56581 kB: com.google.android.googlequicksearchbox (pid 30322 / activities)
   172095 kB: B Services
               47510 kB: com.ss.android.article.news (pid 31181)
               37485 kB: net.zedge.android (pid 31658)
               29638 kB: net.zedge.android:remote (pid 31800)
               20459 kB: com.tencent.mobileqq:MSF (pid 29732)
               17643 kB: com.tencent.mobileqq (pid 30607)
               12427 kB: com.qihoo.appstore:PluginP01 (pid 32293)
                4702 kB: android.process.media (pid 30399)
                2231 kB: com.qualcomm.qcrilmsgtunnel (pid 32205)
   186700 kB: Cached
               38365 kB: com.google.android.googlequicksearchbox:search (pid 30737)
               34972 kB: com.cleanmaster.mguard_x86 (pid 31260)
               21686 kB: com.google.android.apps.docs (pid 31985)
               20105 kB: com.android.vending (pid 30062)
               15185 kB: com.google.android.gm (pid 31671)
               14503 kB: com.cleanmaster.security (pid 32364)
               12226 kB: com.tencent.mobileqq:tool (pid 31814)
                6854 kB: android.process.acore (pid 31566)
                3835 kB: com.google.process.gapps (pid 29804)
                3323 kB: com.android.documentsui (pid 31875)
                3164 kB: com.google.android.partnersetup (pid 30939)
                2784 kB: com.google.process.gapps (pid 31731)
                2780 kB: com.android.defcontainer (pid 30646)
                2389 kB: com.android.musicfx (pid 31592)
                2295 kB: com.android.shell (pid 31957)
                2234 kB: com.android.externalstorage (pid 31911)

Total PSS by category:
   308019 kB: Dalvik
   290637 kB: Native
   209504 kB: .dex mmap
   121488 kB: EGL mtrack
    56387 kB: .so mmap
    45931 kB: .art mmap
    44995 kB: .apk mmap
    41002 kB: Dalvik Other
    37681 kB: .oat mmap
    25746 kB: Unknown
    17816 kB: Stack
    16488 kB: Gfx dev
    10644 kB: Ashmem
     6246 kB: Other mmap
      391 kB: .ttf mmap
      308 kB: Other dev
       56 kB: Cursor
        0 kB: .jar mmap
        0 kB: GL mtrack
        0 kB: Other mtrack

Total RAM: 1899508 kB (status normal)
 Free RAM: 685976 kB (186700 cached pss + 414160 cached kernel + 85116 free)
 Used RAM: 1167267 kB (1046639 used pss + 120628 kernel)
 Lost RAM: 46265 kB
   Tuning: 192 (large 512), oom 184320 kB, restore limit 61440 kB (high-end-gfx)
~~~

* N5(6.0.1) after  — system info
  * 450*800 40KB add ~700 times

~~~java
Total PSS by process:
  1130944 kB: com.ksmobile.launcher (pid 24136 / activities)
   114538 kB: system (pid 777)
    68790 kB: com.android.systemui (pid 921 / activities)
    52071 kB: com.ss.android.article.news (pid 24912)
    26048 kB: net.zedge.android:remote (pid 24882)
    23373 kB: surfaceflinger (pid 189)
    22384 kB: com.qihoo.daemon (pid 25065)
    20916 kB: com.ss.android.article.news:pushservice (pid 24887)
    16828 kB: com.ksmobile.launcher:locker (pid 24194)
    16149 kB: com.qihoo.appstore (pid 25085)
    11888 kB: com.android.phone (pid 1720)
    11313 kB: com.ss.android.article.news:push (pid 7350)
    11173 kB: com.qihoo.appstore:PluginP01 (pid 24925)
    10948 kB: com.google.android.gms.persistent (pid 25139)
    10820 kB: com.google.android.gms (pid 24986)
     ...

Total PSS by OOM adjustment:
    67191 kB: Native
               23373 kB: surfaceflinger (pid 189)
                6477 kB: mediaserver (pid 198)
                6136 kB: net.zedge.android (pid 25341)
                3808 kB: mm-qcamera-daemon (pid 213)
                3396 kB: logd (pid 147)
                2544 kB: zygote (pid 209)
                2218 kB: sdcard (pid 907)
                1822 kB: rild (pid 195)
                1499 kB: drmserver (pid 196)
                1326 kB: app_process (pid 25237)
                1323 kB: sensors.qcom (pid 205)
                1049 kB: thermal-engine-hh (pid 208)
                 ...
   114538 kB: System
              114538 kB: system (pid 777)
    90790 kB: Persistent
               68790 kB: com.android.systemui (pid 921 / activities)
               11888 kB: com.android.phone (pid 1720)
                6926 kB: com.android.nfc (pid 1694)
                3186 kB: com.redbend.vdmc (pid 1707)
  1325419 kB: Foreground
             1130944 kB: com.ksmobile.launcher (pid 24136 / activities)
               52071 kB: com.ss.android.article.news (pid 24912)
               26048 kB: net.zedge.android:remote (pid 24882)
               22384 kB: com.qihoo.daemon (pid 25065)
               20916 kB: com.ss.android.article.news:pushservice (pid 24887)
               16149 kB: com.qihoo.appstore (pid 25085)
               11313 kB: com.ss.android.article.news:push (pid 7350)
               11173 kB: com.qihoo.appstore:PluginP01 (pid 24925)
               10948 kB: com.google.android.gms.persistent (pid 25139)
               10820 kB: com.google.android.gms (pid 24986)
                6642 kB: com.google.process.gapps (pid 25166)
                6011 kB: com.cleanmaster.security:DefendService (pid 25195)
    24877 kB: Visible
               16828 kB: com.ksmobile.launcher:locker (pid 24194)
                5095 kB: com.google.android.googlequicksearchbox:interactor (pid 8853)
                2954 kB: com.willme.topactivity (pid 7476)

Total PSS by category:
  1063088 kB: Gfx dev
   152523 kB: Dalvik
   119378 kB: Native
   106128 kB: EGL mtrack
    44258 kB: .dex mmap
    33132 kB: .so mmap
    28841 kB: .oat mmap
    23106 kB: .art mmap
    17281 kB: Dalvik Other
     8912 kB: Ashmem
     8012 kB: Stack
     7723 kB: Unknown
     6149 kB: .apk mmap
     4004 kB: Other mmap
      248 kB: Other dev
       28 kB: .ttf mmap
        4 kB: Cursor
        0 kB: .jar mmap
        0 kB: GL mtrack
        0 kB: Other mtrack

Total RAM: 1899508 kB (status critical)
 Free RAM: -925232 kB (0 cached pss + -982200 cached kernel + 56968 free)
 Used RAM: 1722755 kB (1622815 used pss + 99940 kernel)
 Lost RAM: 1101985 kB
   Tuning: 192 (large 512), oom 184320 kB, restore limit 61440 kB (high-end-gfx)
~~~



###  N5 (4.4.4)

* N5 (4.4.4) before — process info
~~~java
** MEMINFO in pid 14573 [com.ksmobile.launcher] **
                   Pss  Private  Private  Swapped     Heap     Heap     Heap
                 Total    Dirty    Clean    Dirty     Size    Alloc     Free
                ------   ------   ------   ------   ------   ------   ------
  Native Heap     4254     4240        0        0    17532     8226      161
  Dalvik Heap    29801    29360        0        0    45260    45220       40
 Dalvik Other     5527     5436        0        0
        Stack      404      404        0        0
       Ashmem      128      128        0        0
    Other dev      408      388        4        0
     .so mmap     3352      484     1064        0
    .jar mmap        1        0        0        0
    .apk mmap      734        0       44        0
    .ttf mmap        3        0        0        0
    .dex mmap     8964      908     1936        0
   Other mmap       72        4        8        0
     Graphics    44008    44008        0        0
           GL     4588     4588        0        0
      Unknown      240      240        0        0
        TOTAL   102484    90188     3056        0    62792    53446      201

 Objects
               Views:        7         ViewRootImpl:        1
         AppContexts:        7           Activities:        1
              Assets:        3        AssetManagers:        3
       Local Binders:       53        Proxy Binders:       31
    Death Recipients:        2
     OpenSSL Sockets:        3
~~~

* N5 (4.4.4) after  — process info
  * 450*800 40KB add ~600 times
  * 加至400个左右之后，再创建时有明显卡顿
  * 加至674时，进程被杀
~~~java
** MEMINFO in pid 14573 [com.ksmobile.launcher] **
                   Pss  Private  Private  Swapped     Heap     Heap     Heap
                 Total    Dirty    Clean    Dirty     Size    Alloc     Free
                ------   ------   ------   ------   ------   ------   ------
  Native Heap    10553    10520        0        0    20328    13316      655
  Dalvik Heap    33028    32020        0        0    47768    46375     1393
 Dalvik Other     6871     6672        0        0
        Stack      780      780        0        0
       Ashmem      128      128        0        0
    Other dev   856029   811420       12        0
     .so mmap    11241      592    10176        0
    .apk mmap     1646        0     1544        0
    .dex mmap    10534     1400     8392        0
   Other mmap     1035        4      980        0
     Graphics    35944    35944        0        0
           GL   905840   905840        0        0
      Unknown     3488     3488        0        0
        TOTAL  1877117  1808808    21104        0    68096    59691     2048

 Objects
               Views:       10         ViewRootImpl:        1
         AppContexts:        9           Activities:        1
              Assets:        3        AssetManagers:        3
       Local Binders:       53        Proxy Binders:       37
    Death Recipients:        3
     OpenSSL Sockets:        4
~~~

###  S7 (7.0)
* S7 (7.0) before — process info
~~~java
** MEMINFO in pid 30892 [com.ksmobile.launcher] **
                   Pss  Private  Private  SwapPss     Heap     Heap     Heap
                 Total    Dirty    Clean    Dirty     Size    Alloc     Free
                ------   ------   ------   ------   ------   ------   ------
  Native Heap    14752    14700       28      129    22016    17597     4418
  Dalvik Heap     9456     9056      340      115    16068     9641     6427
 Dalvik Other     1638     1636        0        0
        Stack      832      832        0        0
       Ashmem      132      132        0        0
    Other dev       11        4        4        0
     .so mmap    10540      208     3176       49
    .apk mmap     1124        0      108        0
    .ttf mmap       39        0        0        0
    .dex mmap    11385       24     1692        0
    .oat mmap     6937        0     1880        0
    .art mmap     2158     1488      152       11
   Other mmap      501        8       20        0
   EGL mtrack    36360    36360        0        0
      Unknown     2233     2228        4       41
        TOTAL    98443    66676     7404      345    38084    27238    10845

 App Summary
                       Pss(KB)
                        ------
           Java Heap:    10696
         Native Heap:    14700
                Code:     7088
               Stack:      832
            Graphics:    36360
       Private Other:     4404
              System:    24363

               TOTAL:    98443       TOTAL SWAP PSS:      345

 Objects
               Views:        7         ViewRootImpl:        1
         AppContexts:        4           Activities:        1
              Assets:        4        AssetManagers:        2
       Local Binders:       55        Proxy Binders:       36
       Parcel memory:       19         Parcel count:       74
    Death Recipients:        5      OpenSSL Sockets:        3
~~~

* S7 (7.0) after — process info
  * 450*800 40KB add ~700 times
  * add ~1639times crash
~~~java
~700 times
** MEMINFO in pid 30892 [com.ksmobile.launcher] **
                   Pss  Private  Private  SwapPss     Heap     Heap     Heap
                 Total    Dirty    Clean    Dirty     Size    Alloc     Free
                ------   ------   ------   ------   ------   ------   ------
  Native Heap    18639    18596       20      137    26112    22285     3826
  Dalvik Heap     9654     9260      340      132    16742    10045     6697
 Dalvik Other     1808     1808        0        2
        Stack      840      840        0        0
       Ashmem      132      132        0        0
    Other dev       11        4        4        0
     .so mmap    10617      220     3180       57
    .apk mmap     1164        0       88        0
    .ttf mmap       39        0        0        0
    .dex mmap    13693       24     1756        0
    .oat mmap     6643        0     1480        0
    .art mmap     2093     1524       64       12
   Other mmap      502        8       20        0
   EGL mtrack    36360    36360        0        0
      Unknown     2237     2232        4       45
        TOTAL   104817    71008     6956      385    42854    32330    10523

 App Summary
                       Pss(KB)
                        ------
           Java Heap:    10848
         Native Heap:    18596
                Code:     6748
               Stack:      840
            Graphics:    36360
       Private Other:     4572
              System:    26853

               TOTAL:   104817       TOTAL SWAP PSS:      385

 Objects
               Views:        7         ViewRootImpl:        1
         AppContexts:        4           Activities:        1
              Assets:        4        AssetManagers:        2
       Local Binders:       52        Proxy Binders:       38
       Parcel memory:       20         Parcel count:       76
    Death Recipients:        4      OpenSSL Sockets:        3
~~~

~~~java
~1600 times
** MEMINFO in pid 30892 [com.ksmobile.launcher] **
                   Pss  Private  Private  SwapPss     Heap     Heap     Heap
                 Total    Dirty    Clean    Dirty     Size    Alloc     Free
                ------   ------   ------   ------   ------   ------   ------
  Native Heap     1673     1276      392    22773    34304    27502     6801
  Dalvik Heap     1244      732      488    11170    19996    11998     7998
 Dalvik Other      276      228       48     2263
        Stack       92       72       20      972
       Ashmem        0        0        0        0
    Other dev        8        0        8        0
     .so mmap      361       32       60      434
    .dex mmap        0        0        0     2292
    .oat mmap      317        0        0        0
    .art mmap      625       96      276     1423
   Other mmap       17        0        8        6
   EGL mtrack    36360    36360        0        0
      Unknown      268       36      232     2993
        TOTAL    85567    38832     1532    44326    54300    39500    14799

 App Summary
                       Pss(KB)
                        ------
           Java Heap:     1104
         Native Heap:     1276
                Code:       92
               Stack:       72
            Graphics:    36360
       Private Other:     1460
              System:    45203

               TOTAL:    85567       TOTAL SWAP PSS:    44326
~~~



* S7 (7.0) before-- system meminfo
~~~java
adb shell dumpsys meminfo
Applications Memory Usage (in Kilobytes):
Uptime: 159462877 Realtime: 621816430

Total PSS by process:
    243,555K: system (pid 3733)
    113,495K: com.sec.android.app.launcher (pid 28430)
    109,894K: com.sec.android.inputmethod (pid 28546)
    100,240K: com.ksmobile.launcher (pid 30892 / activities)
     99,829K: com.android.systemui (pid 4168)
     89,197K: com.google.android.gms.persistent (pid 28583)
     83,407K: com.facebook.orca (pid 28720)
     78,232K: com.facebook.katana (pid 29536)
     63,246K: com.google.android.gms (pid 28485)
     59,776K: com.viber.voip (pid 29818)
     59,206K: com.android.phone (pid 4076)
     55,803K: com.tencent.mm (pid 31437)
     54,263K: com.google.android.googlequicksearchbox:search (pid 28500)
     48,905K: com.android.vending (pid 30256)
     43,592K: com.google.android.music:main (pid 31250)
     42,261K: com.ksmobile.launcher:locker (pid 30973)
     40,438K: com.kakao.talk (pid 29189)
     31,482K: com.sec.android.app.shealth (pid 30611)
     30,149K: cameraserver (pid 3287)
     29,733K: com.sec.android.app.shealth:remote (pid 30588)
     28,458K: com.sec.imsservice (pid 4544)
     25,821K: com.tencent.mm:push (pid 31342)
     23,715K: com.instagram.android:mqtt (pid 29523)
     23,401K: com.sec.android.app.sbrowser (pid 30570)
     22,883K: android.process.acore (pid 29770)
     20,899K: com.samsung.voiceserviceplatform (pid 31095)
     19,720K: com.facebook.katana:notification (pid 29393)
     19,100K: perfd (pid 13648)
     18,813K: com.android.settings (pid 30865)
     18,486K: com.microsoft.office.word (pid 30451)
     18,472K: com.sec.android.daemonapp (pid 28669)
     18,023K: surfaceflinger (pid 3113)
     17,759K: com.sec.android.app.samsungapps (pid 31264)
     17,327K: audioserver (pid 3286)
     17,167K: com.microsoft.office.powerpoint (pid 30419)
     17,106K: com.microsoft.office.excel (pid 30370)
     15,187K: com.enhance.gameservice (pid 30316)
     14,868K: com.samsung.android.sm.provider (pid 29999)
     14,651K: com.android.nfc (pid 4851)
     14,615K: com.facebook.appmanager (pid 30333)
     14,613K: com.android.bluetooth (pid 4097)
     14,569K: logd (pid 3030)
     14,375K: com.tencent.mm:recovery (pid 31371)
     14,194K: com.sec.spp.push (pid 29362)
     14,055K: com.samsung.android.providers.context (pid 4864)
     13,799K: com.sohu.inputmethod.sogou (pid 31311)
     13,563K: com.samsung.SMT (pid 31222)
     13,479K: com.samsung.android.scloud (pid 30963)
     12,884K: com.samsung.android.SettingsReceiver (pid 29297)
     11,854K: android.process.media (pid 29549)
     10,980K: com.sohu.inputmethod.sogou:classic (pid 30680)
     10,006K: com.google.process.gapps (pid 28561)
      ...

Total PSS by OOM adjustment:
    198,985K: Native
         30,149K: cameraserver (pid 3287)
         19,100K: perfd (pid 13648)
         18,023K: surfaceflinger (pid 3113)
         17,327K: audioserver (pid 3286)
         14,569K: logd (pid 3030)
          9,202K: mediaserver (pid 3296)
          7,600K: rild (pid 3288)
          5,756K: fingerprintd (pid 3301)
          4,903K: zygote (pid 3276)
          4,579K: rild (pid 3329)
          3,962K: media.codec (pid 3293)
          3,896K: gpsd (pid 3271)
          3,550K: media.extractor (pid 3295)
          3,403K: vold (pid 3097)
          3,301K: zygote64 (pid 3275)
          3,048K: mediadrmserver (pid 3294)
          2,924K: drmserver (pid 3254)
          2,825K: imsd (pid 3262)
          2,640K: wpa_supplicant (pid 4126)
          2,479K: secure_storage_daemon (pid 3114)
          2,251K: netd (pid 3299)
          2,122K: installd (pid 3289)
          1,812K: bintvoutservice (pid 3261)
          1,719K: keystore (pid 3290)
          1,431K: lhd (pid 3270)
          1,370K: adbd (pid 17944)
          1,292K: /init (pid 1)
          1,234K: ueventd (pid 2006)
          1,153K: sensorhubservice (pid 3260)
          1,133K: jackservice (pid 3258)
          1,123K: sdp_cryptod (pid 3279)
          1,011K: smdexe (pid 3268)
            ...
    243,555K: System
        243,555K: system (pid 3733)
    234,512K: Persistent
         99,829K: com.android.systemui (pid 4168)
         59,206K: com.android.phone (pid 4076)
         28,458K: com.sec.imsservice (pid 4544)
         14,651K: com.android.nfc (pid 4851)
         14,055K: com.samsung.android.providers.context (pid 4864)
          6,728K: com.sec.epdg (pid 4557)
          6,213K: com.sec.sve (pid 4537)
          5,372K: system (pid 4872)
     14,613K: Persistent Service
         14,613K: com.android.bluetooth (pid 4097)
    100,240K: Foreground
        100,240K: com.ksmobile.launcher (pid 30892 / activities)
    146,027K: Visible
         89,197K: com.google.android.gms.persistent (pid 28583)
         42,261K: com.ksmobile.launcher:locker (pid 30973)
          7,762K: com.google.android.webview:sandboxed_process0 (pid 31117)
          6,807K: com.sec.phone (pid 29453)
    126,518K: Perceptible
        109,894K: com.sec.android.inputmethod (pid 28546)
          8,749K: com.samsung.android.beaconmanager (pid 30806)
          7,875K: com.samsung.android.MtpApplication (pid 31532)
    252,016K: A Services
         63,246K: com.google.android.gms (pid 28485)
         59,776K: com.viber.voip (pid 29818)
         55,803K: com.tencent.mm (pid 31437)
         40,438K: com.kakao.talk (pid 29189)
         20,899K: com.samsung.voiceserviceplatform (pid 31095)
         11,854K: android.process.media (pid 29549)
    113,495K: Home
        113,495K: com.sec.android.app.launcher (pid 28430)
    272,366K: B Services
         83,407K: com.facebook.orca (pid 28720)
         43,592K: com.google.android.music:main (pid 31250)
         25,821K: com.tencent.mm:push (pid 31342)
         23,715K: com.instagram.android:mqtt (pid 29523)
         19,720K: com.facebook.katana:notification (pid 29393)
         18,472K: com.sec.android.daemonapp (pid 28669)
         14,194K: com.sec.spp.push (pid 29362)
         13,563K: com.samsung.SMT (pid 31222)
         12,884K: com.samsung.android.SettingsReceiver (pid 29297)
          6,176K: com.google.android.webview:sandboxed_process0 (pid 29122)
          5,546K: com.sec.bcservice (pid 29440)
          5,276K: com.trustonic.tuiservice (pid 29346)
    575,453K: Cached
         78,232K: com.facebook.katana (pid 29536)
         54,263K: com.google.android.googlequicksearchbox:search (pid 28500)
         48,905K: com.android.vending (pid 30256)
         31,482K: com.sec.android.app.shealth (pid 30611)
         29,733K: com.sec.android.app.shealth:remote (pid 30588)
         23,401K: com.sec.android.app.sbrowser (pid 30570)
         22,883K: android.process.acore (pid 29770)
         18,813K: com.android.settings (pid 30865)
         18,486K: com.microsoft.office.word (pid 30451)
         17,759K: com.sec.android.app.samsungapps (pid 31264)
         17,167K: com.microsoft.office.powerpoint (pid 30419)
         17,106K: com.microsoft.office.excel (pid 30370)
         15,187K: com.enhance.gameservice (pid 30316)
         14,868K: com.samsung.android.sm.provider (pid 29999)
         14,615K: com.facebook.appmanager (pid 30333)
         14,375K: com.tencent.mm:recovery (pid 31371)
         13,799K: com.sohu.inputmethod.sogou (pid 31311)
         13,479K: com.samsung.android.scloud (pid 30963)
         10,980K: com.sohu.inputmethod.sogou:classic (pid 30680)
         10,006K: com.google.process.gapps (pid 28561)
          8,068K: com.samsung.android.app.mirrorlink (pid 30925)
          7,946K: com.sec.android.widgetapp.samsungapps (pid 28757)
          7,709K: com.samsung.svoice.sync (pid 31177)
          7,611K: com.samsung.android.app.watchmanager:contentprovider (pid 30508)
          7,262K: com.google.android.partnersetup (pid 28614)
          6,998K: com.android.printspooler (pid 30273)
          6,861K: com.samsung.android.bbc.bbcagent (pid 30214)
          6,736K: com.samsung.android.sm.devicesecurity (pid 30555)
          6,440K: com.samsung.android.themecenter (pid 31046)
          6,350K: com.wssnps (pid 30664)
          6,218K: com.android.defcontainer (pid 30200)
          5,971K: com.facebook.system (pid 30721)
          5,744K: com.sec.android.provider.badge (pid 28449)

Total PSS by category:
    563,955K: .dex mmap
    323,481K: Native
    316,582K: Dalvik
    159,410K: .apk mmap
    135,693K: .so mmap
    133,259K: .oat mmap
     86,833K: .art mmap
     85,866K: Unknown
     72,356K: Dalvik Other
     37,392K: EGL mtrack
     25,414K: GL mtrack
     23,008K: Other mmap
     21,424K: Stack
      2,624K: Ashmem
      1,300K: .ttf mmap
        690K: Other dev
          0K: Cursor
          0K: Gfx dev
          0K: .jar mmap
          0K: Other mtrack

Total RAM: 3,618,628K (status normal)
 Free RAM: 1,415,025K (  575,453K cached pss +   719,768K cached kernel +   119,804K free)
 Used RAM: 2,249,471K (1,702,327K used pss +   547,144K kernel)
 Lost RAM:   103,474K
   Tuning: 256 (large 512), oom   325,000K, restore limit   108,333K (high-end-gfx)
~~~

* S7 (7.0) add to 700 times—system

~~~java
adb shell dumpsys meminfo
Applications Memory Usage (in Kilobytes):
Uptime: 159462877 Realtime: 621816430

Total PSS by process:
    243,555K: system (pid 3733)
    113,495K: com.sec.android.app.launcher (pid 28430)
    109,894K: com.sec.android.inputmethod (pid 28546)
    100,240K: com.ksmobile.launcher (pid 30892 / activities)
     99,829K: com.android.systemui (pid 4168)
     89,197K: com.google.android.gms.persistent (pid 28583)
     83,407K: com.facebook.orca (pid 28720)
     78,232K: com.facebook.katana (pid 29536)
     63,246K: com.google.android.gms (pid 28485)
     59,776K: com.viber.voip (pid 29818)
     59,206K: com.android.phone (pid 4076)
     55,803K: com.tencent.mm (pid 31437)
     54,263K: com.google.android.googlequicksearchbox:search (pid 28500)
     48,905K: com.android.vending (pid 30256)
     43,592K: com.google.android.music:main (pid 31250)
     42,261K: com.ksmobile.launcher:locker (pid 30973)
     40,438K: com.kakao.talk (pid 29189)
     31,482K: com.sec.android.app.shealth (pid 30611)
     30,149K: cameraserver (pid 3287)
     29,733K: com.sec.android.app.shealth:remote (pid 30588)
     28,458K: com.sec.imsservice (pid 4544)
     25,821K: com.tencent.mm:push (pid 31342)
     23,715K: com.instagram.android:mqtt (pid 29523)
     23,401K: com.sec.android.app.sbrowser (pid 30570)
     22,883K: android.process.acore (pid 29770)
     20,899K: com.samsung.voiceserviceplatform (pid 31095)
     19,720K: com.facebook.katana:notification (pid 29393)
     19,100K: perfd (pid 13648)
     18,813K: com.android.settings (pid 30865)
     18,486K: com.microsoft.office.word (pid 30451)
     18,472K: com.sec.android.daemonapp (pid 28669)
     18,023K: surfaceflinger (pid 3113)
     17,759K: com.sec.android.app.samsungapps (pid 31264)
     17,327K: audioserver (pid 3286)
     17,167K: com.microsoft.office.powerpoint (pid 30419)
     17,106K: com.microsoft.office.excel (pid 30370)
     15,187K: com.enhance.gameservice (pid 30316)
     14,868K: com.samsung.android.sm.provider (pid 29999)
     14,651K: com.android.nfc (pid 4851)
     14,615K: com.facebook.appmanager (pid 30333)
     14,613K: com.android.bluetooth (pid 4097)
     14,569K: logd (pid 3030)
     14,375K: com.tencent.mm:recovery (pid 31371)
     14,194K: com.sec.spp.push (pid 29362)
     14,055K: com.samsung.android.providers.context (pid 4864)
     13,799K: com.sohu.inputmethod.sogou (pid 31311)
     13,563K: com.samsung.SMT (pid 31222)
     13,479K: com.samsung.android.scloud (pid 30963)
     12,884K: com.samsung.android.SettingsReceiver (pid 29297)
     11,854K: android.process.media (pid 29549)
     10,980K: com.sohu.inputmethod.sogou:classic (pid 30680)
     10,006K: com.google.process.gapps (pid 28561)
      ...

Total PSS by OOM adjustment:
    198,985K: Native
         30,149K: cameraserver (pid 3287)
         19,100K: perfd (pid 13648)
         18,023K: surfaceflinger (pid 3113)
         17,327K: audioserver (pid 3286)
         14,569K: logd (pid 3030)
          9,202K: mediaserver (pid 3296)
          7,600K: rild (pid 3288)
          5,756K: fingerprintd (pid 3301)
          4,903K: zygote (pid 3276)
          4,579K: rild (pid 3329)
          3,962K: media.codec (pid 3293)
          3,896K: gpsd (pid 3271)
          3,550K: media.extractor (pid 3295)
          3,403K: vold (pid 3097)
          3,301K: zygote64 (pid 3275)
          3,048K: mediadrmserver (pid 3294)
          2,924K: drmserver (pid 3254)
          2,825K: imsd (pid 3262)
          2,640K: wpa_supplicant (pid 4126)
          2,479K: secure_storage_daemon (pid 3114)
          2,251K: netd (pid 3299)
          2,122K: installd (pid 3289)
          1,812K: bintvoutservice (pid 3261)
          1,719K: keystore (pid 3290)
          1,431K: lhd (pid 3270)
          1,370K: adbd (pid 17944)
          1,292K: /init (pid 1)
          1,234K: ueventd (pid 2006)
          1,153K: sensorhubservice (pid 3260)
          1,133K: jackservice (pid 3258)
          1,123K: sdp_cryptod (pid 3279)
          1,011K: smdexe (pid 3268)
            ...
    243,555K: System
        243,555K: system (pid 3733)
    234,512K: Persistent
         99,829K: com.android.systemui (pid 4168)
         59,206K: com.android.phone (pid 4076)
         28,458K: com.sec.imsservice (pid 4544)
         14,651K: com.android.nfc (pid 4851)
         14,055K: com.samsung.android.providers.context (pid 4864)
          6,728K: com.sec.epdg (pid 4557)
          6,213K: com.sec.sve (pid 4537)
          5,372K: system (pid 4872)
     14,613K: Persistent Service
         14,613K: com.android.bluetooth (pid 4097)
    100,240K: Foreground
        100,240K: com.ksmobile.launcher (pid 30892 / activities)
    146,027K: Visible
         89,197K: com.google.android.gms.persistent (pid 28583)
         42,261K: com.ksmobile.launcher:locker (pid 30973)
          7,762K: com.google.android.webview:sandboxed_process0 (pid 31117)
          6,807K: com.sec.phone (pid 29453)
    126,518K: Perceptible
        109,894K: com.sec.android.inputmethod (pid 28546)
          8,749K: com.samsung.android.beaconmanager (pid 30806)
          7,875K: com.samsung.android.MtpApplication (pid 31532)
    252,016K: A Services
         63,246K: com.google.android.gms (pid 28485)
         59,776K: com.viber.voip (pid 29818)
         55,803K: com.tencent.mm (pid 31437)
         40,438K: com.kakao.talk (pid 29189)
         20,899K: com.samsung.voiceserviceplatform (pid 31095)
         11,854K: android.process.media (pid 29549)
    113,495K: Home
        113,495K: com.sec.android.app.launcher (pid 28430)
    272,366K: B Services
         83,407K: com.facebook.orca (pid 28720)
         43,592K: com.google.android.music:main (pid 31250)
         25,821K: com.tencent.mm:push (pid 31342)
         23,715K: com.instagram.android:mqtt (pid 29523)
         19,720K: com.facebook.katana:notification (pid 29393)
         18,472K: com.sec.android.daemonapp (pid 28669)
         14,194K: com.sec.spp.push (pid 29362)
         13,563K: com.samsung.SMT (pid 31222)
         12,884K: com.samsung.android.SettingsReceiver (pid 29297)
          6,176K: com.google.android.webview:sandboxed_process0 (pid 29122)
          5,546K: com.sec.bcservice (pid 29440)
          5,276K: com.trustonic.tuiservice (pid 29346)
    575,453K: Cached
         78,232K: com.facebook.katana (pid 29536)
         54,263K: com.google.android.googlequicksearchbox:search (pid 28500)
         48,905K: com.android.vending (pid 30256)
         31,482K: com.sec.android.app.shealth (pid 30611)
         29,733K: com.sec.android.app.shealth:remote (pid 30588)
         23,401K: com.sec.android.app.sbrowser (pid 30570)
         22,883K: android.process.acore (pid 29770)
         18,813K: com.android.settings (pid 30865)
         18,486K: com.microsoft.office.word (pid 30451)
         17,759K: com.sec.android.app.samsungapps (pid 31264)
         17,167K: com.microsoft.office.powerpoint (pid 30419)
         17,106K: com.microsoft.office.excel (pid 30370)
         15,187K: com.enhance.gameservice (pid 30316)
         14,868K: com.samsung.android.sm.provider (pid 29999)
         14,615K: com.facebook.appmanager (pid 30333)
         14,375K: com.tencent.mm:recovery (pid 31371)
         13,799K: com.sohu.inputmethod.sogou (pid 31311)
         13,479K: com.samsung.android.scloud (pid 30963)
         10,980K: com.sohu.inputmethod.sogou:classic (pid 30680)
         10,006K: com.google.process.gapps (pid 28561)
          8,068K: com.samsung.android.app.mirrorlink (pid 30925)
          7,946K: com.sec.android.widgetapp.samsungapps (pid 28757)
          7,709K: com.samsung.svoice.sync (pid 31177)
          7,611K: com.samsung.android.app.watchmanager:contentprovider (pid 30508)
          7,262K: com.google.android.partnersetup (pid 28614)
          6,998K: com.android.printspooler (pid 30273)
          6,861K: com.samsung.android.bbc.bbcagent (pid 30214)
          6,736K: com.samsung.android.sm.devicesecurity (pid 30555)
          6,440K: com.samsung.android.themecenter (pid 31046)
          6,350K: com.wssnps (pid 30664)
          6,218K: com.android.defcontainer (pid 30200)
          5,971K: com.facebook.system (pid 30721)
          5,744K: com.sec.android.provider.badge (pid 28449)

Total PSS by category:
    563,955K: .dex mmap
    323,481K: Native
    316,582K: Dalvik
    159,410K: .apk mmap
    135,693K: .so mmap
    133,259K: .oat mmap
     86,833K: .art mmap
     85,866K: Unknown
     72,356K: Dalvik Other
     37,392K: EGL mtrack
     25,414K: GL mtrack
     23,008K: Other mmap
     21,424K: Stack
      2,624K: Ashmem
      1,300K: .ttf mmap
        690K: Other dev
          0K: Cursor
          0K: Gfx dev
          0K: .jar mmap
          0K: Other mtrack

Total RAM: 3,618,628K (status normal)
 Free RAM: 1,415,025K (  575,453K cached pss +   719,768K cached kernel +   119,804K free)
 Used RAM: 2,249,471K (1,702,327K used pss +   547,144K kernel)
 Lost RAM:   103,474K
   Tuning: 256 (large 512), oom   325,000K, restore limit   108,333K (high-end-gfx)
~~~



* S7 (7.0) add to 1600times-- system meminfo
~~~java
adb shell dumpsys meminfo
Applications Memory Usage (in Kilobytes):
Uptime: 159822185 Realtime: 622175738

Total PSS by process:
    195,004K: system (pid 3733)
     93,971K: com.android.systemui (pid 4168)
     90,102K: com.ksmobile.launcher (pid 30892 / activities)
     56,990K: com.sec.android.inputmethod (pid 1645)
     44,428K: com.android.phone (pid 4076)
     32,911K: com.samsung.android.SettingsReceiver (pid 1668)
     30,107K: cameraserver (pid 3287)
     26,831K: com.sec.imsservice (pid 4544)
     18,216K: perfd (pid 13648)
     18,113K: surfaceflinger (pid 3113)
     17,984K: com.samsung.android.providers.context (pid 4864)
     15,564K: audioserver (pid 3286)
     14,876K: com.android.bluetooth (pid 4097)
     14,558K: logd (pid 3030)
     13,383K: com.android.nfc (pid 4851)
      ...

Total PSS by OOM adjustment:
    225,396K: Native
         32,911K: com.samsung.android.SettingsReceiver (pid 1668)
         30,107K: cameraserver (pid 3287)
         18,216K: perfd (pid 13648)
         18,113K: surfaceflinger (pid 3113)
         15,564K: audioserver (pid 3286)
         14,558K: logd (pid 3030)
          7,763K: mediaserver (pid 3296)
          7,166K: zygote (pid 3276)
          6,762K: rild (pid 3288)
          5,760K: fingerprintd (pid 3301)
          5,075K: zygote64 (pid 3275)
          4,375K: rild (pid 3329)
          3,900K: gpsd (pid 3271)
          3,239K: media.codec (pid 3293)
          3,181K: media.extractor (pid 3295)
          3,049K: mediadrmserver (pid 3294)
          2,834K: imsd (pid 3262)
          2,813K: vold (pid 3097)
          2,492K: secure_storage_daemon (pid 3114)
          2,013K: drmserver (pid 3254)
          ...
    195,004K: System
        195,004K: system (pid 3733)
    216,904K: Persistent
         93,971K: com.android.systemui (pid 4168)
         44,428K: com.android.phone (pid 4076)
         26,831K: com.sec.imsservice (pid 4544)
         17,984K: com.samsung.android.providers.context (pid 4864)
         13,383K: com.android.nfc (pid 4851)
          7,387K: com.sec.epdg (pid 4557)
          6,879K: com.sec.sve (pid 4537)
          6,041K: system (pid 4872)
     14,876K: Persistent Service
         14,876K: com.android.bluetooth (pid 4097)
    147,092K: Foreground
         90,102K: com.ksmobile.launcher (pid 30892 / activities)
         56,990K: com.sec.android.inputmethod (pid 1645)
     31,385K: Visible
          8,638K: com.google.android.webview:sandboxed_process0 (pid 31117)
          6,181K: com.samsung.android.app.aodservice (pid 31831)
          5,567K: com.sec.phone (pid 29453)
          5,518K: org.simalliance.openmobileapi.service:remote (pid 31989)
          5,481K: com.google.android.ext.services (pid 31975)

Total PSS by category:
     38,712K: Dalvik
     37,494K: .oat mmap
     37,392K: EGL mtrack
     32,933K: .so mmap
     25,415K: GL mtrack
     21,688K: .dex mmap
     21,641K: Native
     17,508K: .art mmap
     13,483K: .apk mmap
      5,765K: Dalvik Other
      3,258K: Unknown
      3,111K: Other mmap
      2,024K: Stack
        435K: Other dev
          4K: Ashmem
          0K: Cursor
          0K: Gfx dev
          0K: .jar mmap
          0K: .ttf mmap
          0K: Other mtrack

Total RAM: 3,618,628K (status critical)
 Free RAM:   303,876K (        0K cached pss +    48,660K cached kernel +   255,216K free)
 Used RAM: 1,263,977K (  830,657K used pss +   433,320K kernel)
 Lost RAM: 2,451,008K
   Tuning: 256 (large 512), oom   325,000K, restore limit   108,333K (high-end-gfx)
~~~
N5X (6.0)

* before — process info

~~~java
adb shell dumpsys meminfo com.ksmobile.launcher
Applications Memory Usage (kB):
Uptime: 74645983 Realtime: 77651699

** MEMINFO in pid 1781 [com.ksmobile.launcher] **
                   Pss  Private  Private  Swapped     Heap     Heap     Heap
                 Total    Dirty    Clean    Dirty     Size    Alloc     Free
                ------   ------   ------   ------   ------   ------   ------
  Native Heap    17183    17140        0    13420    44160    41566     2593
  Dalvik Heap    19441    19408        0     7936    29041    26703     2338
 Dalvik Other     2580     2580        0        0
        Stack     1736     1736        0        0
       Ashmem      160      136        0        0
      Gfx dev      332      332        0        0
    Other dev       16        0       16        0
     .so mmap    14643      404     9964     2648
    .apk mmap      690        0      328        0
    .ttf mmap        3        0        0        0
    .dex mmap    12559     2292     6724        0
    .oat mmap     3576        0     1432        4
    .art mmap     1786     1248       12       36
   Other mmap      626        8      432        0
   EGL mtrack    24808    24808        0        0
      Unknown     4511     4496        0       64
        TOTAL   104650    74588    18908    24108    73201    68269     4931

 App Summary
                       Pss(KB)
                        ------
           Java Heap:    20668
         Native Heap:    17140
                Code:    21144
               Stack:     1736
            Graphics:    25140
       Private Other:     7668
              System:    11154

               TOTAL:   104650      TOTAL SWAP (KB):    24108

 Objects
               Views:       16         ViewRootImpl:        1
         AppContexts:        5           Activities:        1
              Assets:        5        AssetManagers:        4
       Local Binders:       80        Proxy Binders:       40
       Parcel memory:       38         Parcel count:      152
    Death Recipients:        3      OpenSSL Sockets:        3

 SQL
         MEMORY_USED:      494
  PAGECACHE_OVERFLOW:      172          MALLOC_SIZE:       62

 DATABASES
      pgsz     dbsz   Lookaside(b)          cache  Dbname
         4       20             37        15/30/6  /data/user/0/com.ksmobile.launcher/databases/du_ad_ts.db
         4       44             45         5/44/8  /data/user/0/com.ksmobile.launcher/databases/launcher.db
         4       28             20         1/32/4  /data/user/0/com.ksmobile.launcher/databases/du_ad_cache.db
         4       20             33        39/30/6  /data/user/0/com.ksmobile.launcher/databases/UnreadNotification.db
~~~

* after — process info
~~~java
** MEMINFO in pid 7723 [com.ksmobile.launcher] **
                   Pss  Private  Private  Swapped     Heap     Heap     Heap
                 Total    Dirty    Clean    Dirty     Size    Alloc     Free
                ------   ------   ------   ------   ------   ------   ------
  Native Heap    12123    12076        0    13488    33792    30210     3581
  Dalvik Heap    10791    10728        0     7992    28068    20057     8011
 Dalvik Other     1479     1472        0        0
        Stack      580      580        0        0
       Ashmem      132      132        0        0
      Gfx dev   899942   899612        0        0
    Other dev        6        0        4        0
     .so mmap     1056      492      116     2840
    .dex mmap       12       12        0        0
    .oat mmap      449        0        8        4
    .art mmap     1162      956        0       88
   Other mmap       12        8        0        0
   EGL mtrack    24808    24808        0        0
      Unknown      313      308        0       68
        TOTAL   952865   951184      128    24480    61860    50267    11592

 App Summary
                       Pss(KB)
                        ------
           Java Heap:    11684
         Native Heap:    12076
                Code:      628
               Stack:      580
            Graphics:   924420
       Private Other:     1924
              System:     1553

               TOTAL:   952865      TOTAL SWAP (KB):    24480

 Objects
               Views:        7         ViewRootImpl:        1
         AppContexts:        4           Activities:        1
              Assets:        3        AssetManagers:        2
       Local Binders:       49        Proxy Binders:       27
       Parcel memory:       18         Parcel count:       72
    Death Recipients:        0      OpenSSL Sockets:        2
~~~


* before — system info

~~~java
adb shell dumpsys meminfo
Applications Memory Usage (kB):
Uptime: 74450605 Realtime: 77456321

Total PSS by process:
    91848 kB: com.android.systemui (pid 16734 / activities)
    79340 kB: com.ksmobile.launcher (pid 1781 / activities)
    78185 kB: system (pid 956)
    57571 kB: com.google.android.gms (pid 31743)
    47665 kB: com.tsf.shell (pid 22586 / activities)
    43203 kB: com.google.android.gms.persistent (pid 4084)
    42226 kB: com.ijinshan.browser_fast:cheetah_push_fast (pid 4800)
    38801 kB: com.google.android.googlequicksearchbox:search (pid 2618)
    35783 kB: com.google.android.googlequicksearchbox (pid 29005 / activities)
    35071 kB: com.apusapps.launcher (pid 7653)
    34969 kB: com.cmcm.locker:locker (pid 2494)
    27941 kB: com.facebook.katana (pid 25369)
    27527 kB: com.tencent.android.qqdownloader:daemon (pid 30013)
    25022 kB: com.google.android.inputmethod.japanese (pid 5081)
    23698 kB: surfaceflinger (pid 370)
    22381 kB: com.ksmobile.launcher:locker (pid 1676)
    21393 kB: com.willme.topactivity (pid 18529)
    18879 kB: com.tencent.android.qqdownloader (pid 30164)
    17408 kB: com.tencent.android.qqdownloader:connect (pid 2375)
    17118 kB: com.tencent.mtt:service (pid 15423)
    17105 kB: com.android.vending (pid 1338)
    17037 kB: perfd (pid 30233)
    16106 kB: com.cleanmaster.security:DefendService (pid 24514)
    15263 kB: com.tencent.mobileqq (pid 28428)
    13743 kB: com.tencent.mtt (pid 29232)
    13693 kB: com.tencent.mobileqq:MSF (pid 25867)
    13496 kB: com.cleanmaster.security:ScanService (pid 1606)
    12887 kB: com.tencent.mm (pid 20715)
    10793 kB: com.tencent.mm:push (pid 11273)
    10537 kB: com.android.phone (pid 3642)
    10377 kB: com.hola.launcher:boost (pid 11469)
     9861 kB: com.ijinshan.browser_fast (pid 25107)
     9758 kB: com.meitu.meipaimv:pushservice (pid 24369)
     9661 kB: com.tsq.tongshi:push (pid 25578)
     9185 kB: com.ksmobile.launcher:crash_service (pid 2708)
     8667 kB: android.process.acore (pid 2636)
     7542 kB: com.hola.launcher (pid 7985)
     7511 kB: com.cmcm.locker:service (pid 1093)
     7099 kB: com.google.process.gapps (pid 2648)
     6327 kB: com.qiyi.video (pid 24604)
     6227 kB: com.android.nfc (pid 3584)
     5738 kB: com.tencent.android.qqdownloader:tools (pid 1428)
     5655 kB: mediaserver (pid 487)
     5514 kB: com.ijinshan.browser_fast:sync (pid 2572)
     5268 kB: com.google.android.apps.messaging:rcs (pid 25605)
     5230 kB: com.sohu.inputmethod.sogou:classic (pid 2459)
     4879 kB: com.sohu.inputmethod.sogou (pid 28670)
     4755 kB: android.process.media (pid 30159)
     4533 kB: logd (pid 335)
     4359 kB: com.google.android.partnersetup (pid 2689)
     4117 kB: com.sohu.inputmethod.status (pid 1304)
     3791 kB: com.android.bluetooth (pid 12136)
     3751 kB: eu.chainfire.supersu (pid 2287)
     3414 kB: com.facebook.katana:notification (pid 25275)
     3291 kB: com.quicinc.cne.CNEService (pid 3572)
     3157 kB: com.tsf.shell:Service (pid 28899)
     3135 kB: com.wandoujia.phoenix2.usbproxy (pid 2432)
     3086 kB: com.google.android.googlequicksearchbox:interactor (pid 3486)
     2918 kB: com.qiyi.video:downloader (pid 25032)
     2835 kB: hd.backgrounds.wallpapers.theme (pid 24986)
     2811 kB: com.meitu.meipaimv (pid 20164)
     2562 kB: com.qualcomm.qcrilmsgtunnel (pid 32737)
     2530 kB: com.qualcomm.telephony (pid 1011)
     2488 kB: .DaemonService (pid 24390)
     1937 kB: zygote (pid 500)
     1776 kB: com.qualcomm.qti.rcsbootstraputil (pid 3593)
     1739 kB: zygote64 (pid 499)
     1654 kB: com.qualcomm.qti.rcsimsbootstraputil (pid 3618)
     1619 kB: com.ijinshan.browser_fast:service (pid 29497)
     1491 kB: hd.backgrounds.wallpapers.theme:alive (pid 24972)
     1475 kB: com.ijinshan.browser_fast:player (pid 9295)
     1005 kB: dumpsys (pid 2753)
      937 kB: rild (pid 485)
      724 kB: sdcard (pid 3162)
      692 kB: adbd (pid 476)
      674 kB: netd (pid 482)
      534 kB: /init (pid 1)
      519 kB: keystore (pid 490)
      463 kB: logcat (pid 30118)
      434 kB: ueventd (pid 308)
      432 kB: healthd (pid 367)
      430 kB: wpa_supplicant (pid 11758)
      378 kB: drmserver (pid 486)
      345 kB: sh (pid 30230)
      328 kB: thermal-engine (pid 496)
      300 kB: installd (pid 488)
      296 kB: cnd (pid 508)
      240 kB: qmuxd (pid 492)
      230 kB: servicemanager (pid 369)
      227 kB: location-mq (pid 515)
      225 kB: perfd (pid 493)
      201 kB: fingerprintd (pid 510)
      198 kB: gatekeeperd (pid 501)
      179 kB: vold (pid 337)
      177 kB: lowi-server (pid 516)
      168 kB: time_daemon (pid 505)
      152 kB: msm_irqbalance (pid 374)
      147 kB: wcnss_filter (pid 4746)
      134 kB: qseecomd (pid 382)
       98 kB: slim_daemon (pid 517)
       92 kB: rmt_storage (pid 371)
       89 kB: mm-qcamera-daemon (pid 506)
       79 kB: libdaemon.so (pid 8152)
       71 kB: lmkd (pid 368)
       63 kB: imsdatadaemon (pid 512)
       56 kB: netmgrd (pid 495)
       51 kB: kbmonitor (pid 5098)
       41 kB: pm-service (pid 372)
       40 kB: cnss-daemon (pid 504)
       16 kB: kwatchdog (pid 24574)
       13 kB: pm-proxy (pid 410)
       13 kB: ATFWD-daemon (pid 507)
       12 kB: debuggerd64 (pid 484)
       12 kB: imscmservice (pid 498)
       11 kB: qti (pid 494)
        8 kB: debuggerd (pid 483)
        7 kB: qseecomd (pid 373)
        7 kB: imsqmidaemon (pid 497)
        7 kB: loc_launcher (pid 503)
        4 kB: sysbe (pid 16143)

Total PSS by OOM adjustment:
    66158 kB: Native
               23698 kB: surfaceflinger (pid 370)
               17037 kB: perfd (pid 30233)
                5655 kB: mediaserver (pid 487)
                4533 kB: logd (pid 335)
                1937 kB: zygote (pid 500)
                1739 kB: zygote64 (pid 499)
                1005 kB: dumpsys (pid 2753)
                 937 kB: rild (pid 485)
                 724 kB: sdcard (pid 3162)
                 692 kB: adbd (pid 476)
                 674 kB: netd (pid 482)
                 534 kB: /init (pid 1)
                 519 kB: keystore (pid 490)
                 463 kB: logcat (pid 30118)
                 434 kB: ueventd (pid 308)
                 432 kB: healthd (pid 367)
                 430 kB: wpa_supplicant (pid 11758)
                 378 kB: drmserver (pid 486)
                 345 kB: sh (pid 30230)
                 328 kB: thermal-engine (pid 496)
                 300 kB: installd (pid 488)
                 296 kB: cnd (pid 508)
                 240 kB: qmuxd (pid 492)
                 230 kB: servicemanager (pid 369)
                 227 kB: location-mq (pid 515)
                 225 kB: perfd (pid 493)
                 201 kB: fingerprintd (pid 510)
                 198 kB: gatekeeperd (pid 501)
                 179 kB: vold (pid 337)
                 177 kB: lowi-server (pid 516)
                 168 kB: time_daemon (pid 505)
                 152 kB: msm_irqbalance (pid 374)
                 147 kB: wcnss_filter (pid 4746)
                 134 kB: qseecomd (pid 382)
                  98 kB: slim_daemon (pid 517)
                  92 kB: rmt_storage (pid 371)
                  89 kB: mm-qcamera-daemon (pid 506)
                  79 kB: libdaemon.so (pid 8152)
                  71 kB: lmkd (pid 368)
                  63 kB: imsdatadaemon (pid 512)
                  56 kB: netmgrd (pid 495)
                  51 kB: kbmonitor (pid 5098)
                  41 kB: pm-service (pid 372)
                  40 kB: cnss-daemon (pid 504)
                  16 kB: kwatchdog (pid 24574)
                  13 kB: pm-proxy (pid 410)
                  13 kB: ATFWD-daemon (pid 507)
                  12 kB: debuggerd64 (pid 484)
                  12 kB: imscmservice (pid 498)
                  11 kB: qti (pid 494)
                   8 kB: debuggerd (pid 483)
                   7 kB: qseecomd (pid 373)
                   7 kB: imsqmidaemon (pid 497)
                   7 kB: loc_launcher (pid 503)
                   4 kB: sysbe (pid 16143)
    78185 kB: System
               78185 kB: system (pid 956)
   115333 kB: Persistent
               91848 kB: com.android.systemui (pid 16734 / activities)
               10537 kB: com.android.phone (pid 3642)
                6227 kB: com.android.nfc (pid 3584)
                3291 kB: com.quicinc.cne.CNEService (pid 3572)
                1776 kB: com.qualcomm.qti.rcsbootstraputil (pid 3593)
                1654 kB: com.qualcomm.qti.rcsimsbootstraputil (pid 3618)
     3791 kB: Persistent Service
                3791 kB: com.android.bluetooth (pid 12136)
    79340 kB: Foreground
               79340 kB: com.ksmobile.launcher (pid 1781 / activities)
   107168 kB: Visible
               43203 kB: com.google.android.gms.persistent (pid 4084)
               22381 kB: com.ksmobile.launcher:locker (pid 1676)
               21393 kB: com.willme.topactivity (pid 18529)
               17105 kB: com.android.vending (pid 1338)
                3086 kB: com.google.android.googlequicksearchbox:interactor (pid 3486)
   261087 kB: Perceptible
               57571 kB: com.google.android.gms (pid 31743)
               42226 kB: com.ijinshan.browser_fast:cheetah_push_fast (pid 4800)
               35071 kB: com.apusapps.launcher (pid 7653)
               34969 kB: com.cmcm.locker:locker (pid 2494)
               25022 kB: com.google.android.inputmethod.japanese (pid 5081)
               17118 kB: com.tencent.mtt:service (pid 15423)
               12887 kB: com.tencent.mm (pid 20715)
               10793 kB: com.tencent.mm:push (pid 11273)
               10377 kB: com.hola.launcher:boost (pid 11469)
                7542 kB: com.hola.launcher (pid 7985)
                7511 kB: com.cmcm.locker:service (pid 1093)
   135933 kB: A Services
               27527 kB: com.tencent.android.qqdownloader:daemon (pid 30013)
               18879 kB: com.tencent.android.qqdownloader (pid 30164)
               17408 kB: com.tencent.android.qqdownloader:connect (pid 2375)
               16106 kB: com.cleanmaster.security:DefendService (pid 24514)
               15263 kB: com.tencent.mobileqq (pid 28428)
               13743 kB: com.tencent.mtt (pid 29232)
               13496 kB: com.cleanmaster.security:ScanService (pid 1606)
                9185 kB: com.ksmobile.launcher:crash_service (pid 2708)
                2835 kB: hd.backgrounds.wallpapers.theme (pid 24986)
                1491 kB: hd.backgrounds.wallpapers.theme:alive (pid 24972)
    35783 kB: Home
               35783 kB: com.google.android.googlequicksearchbox (pid 29005 / activities)
   163742 kB: B Services
               47665 kB: com.tsf.shell (pid 22586 / activities)
               27941 kB: com.facebook.katana (pid 25369)
               13693 kB: com.tencent.mobileqq:MSF (pid 25867)
                9861 kB: com.ijinshan.browser_fast (pid 25107)
                9758 kB: com.meitu.meipaimv:pushservice (pid 24369)
                9661 kB: com.tsq.tongshi:push (pid 25578)
                6327 kB: com.qiyi.video (pid 24604)
                5268 kB: com.google.android.apps.messaging:rcs (pid 25605)
                4879 kB: com.sohu.inputmethod.sogou (pid 28670)
                4755 kB: android.process.media (pid 30159)
                4117 kB: com.sohu.inputmethod.status (pid 1304)
                3414 kB: com.facebook.katana:notification (pid 25275)
                2918 kB: com.qiyi.video:downloader (pid 25032)
                2811 kB: com.meitu.meipaimv (pid 20164)
                2562 kB: com.qualcomm.qcrilmsgtunnel (pid 32737)
                2530 kB: com.qualcomm.telephony (pid 1011)
                2488 kB: .DaemonService (pid 24390)
                1619 kB: com.ijinshan.browser_fast:service (pid 29497)
                1475 kB: com.ijinshan.browser_fast:player (pid 9295)
    85451 kB: Cached
               38801 kB: com.google.android.googlequicksearchbox:search (pid 2618)
                8667 kB: android.process.acore (pid 2636)
                7099 kB: com.google.process.gapps (pid 2648)
                5738 kB: com.tencent.android.qqdownloader:tools (pid 1428)
                5514 kB: com.ijinshan.browser_fast:sync (pid 2572)
                5230 kB: com.sohu.inputmethod.sogou:classic (pid 2459)
                4359 kB: com.google.android.partnersetup (pid 2689)
                3751 kB: eu.chainfire.supersu (pid 2287)
                3157 kB: com.tsf.shell:Service (pid 28899)
                3135 kB: com.wandoujia.phoenix2.usbproxy (pid 2432)

Total PSS by category:
   326678 kB: Dalvik
   312366 kB: Native
   110348 kB: EGL mtrack
    72257 kB: Dalvik Other
    70164 kB: .dex mmap
    58165 kB: .art mmap
    44261 kB: .so mmap
    43276 kB: Gfx dev
    30254 kB: .oat mmap
    19852 kB: Stack
    18357 kB: .apk mmap
    14146 kB: Unknown
     5981 kB: Other mmap
     5356 kB: Ashmem
      420 kB: Other dev
       62 kB: .ttf mmap
       28 kB: Cursor
        0 kB: .jar mmap
        0 kB: GL mtrack
        0 kB: Other mtrack

Total RAM: 1857352 kB (status moderate)
 Free RAM: 317179 kB (85451 cached pss + 118152 cached kernel + 113576 free)
 Used RAM: 1292912 kB (1046520 used pss + 246392 kernel)
 Lost RAM: 247261 kB
     ZRAM: 123836 kB physical used for 442256 kB in swap (520908 kB total swap)
   Tuning: 192 (large 512), oom 322560 kB, restore limit 107520 kB (high-end-gfx)
~~~

*  after — systeminfo
~~~java
adb shell dumpsys meminfo
Applications Memory Usage (kB):
Uptime: 75090373 Realtime: 78096089

Total PSS by process:
   665973 kB: com.ksmobile.launcher (pid 10462 / activities)
    69582 kB: system (pid 956)
    65839 kB: com.android.systemui (pid 16734 / activities)
    56779 kB: com.apusapps.launcher (pid 8734)
    41436 kB: com.google.android.gms (pid 8728)
    36731 kB: com.ijinshan.browser_fast (pid 8873)
    35125 kB: com.google.android.gms.persistent (pid 9253)
    31103 kB: com.google.android.googlequicksearchbox (pid 8854 / activities)
    25180 kB: com.qiyi.video (pid 9215)
    23759 kB: surfaceflinger (pid 370)
    19387 kB: com.facebook.katana (pid 9800)
    19028 kB: com.tencent.android.qqdownloader:daemon (pid 8714)
    16181 kB: com.hola.launcher:boost (pid 9661)
    16153 kB: com.ijinshan.browser_fast:cheetah_push_fast (pid 8395)
    15966 kB: com.cleanmaster.security:DefendService (pid 10816)
    14720 kB: .DaemonService (pid 10872)
    14550 kB: system:ui (pid 10935 / activities)
    12847 kB: com.whatsapp (pid 9481)
    12154 kB: com.meitu.meipaimv:pushservice (pid 9228)
    11331 kB: com.tencent.android.qqdownloader:wns (pid 9724)
    10962 kB: com.qiyi.video:downloader (pid 10424)
    10726 kB: com.ksmobile.launcher:locker (pid 10545)
     9243 kB: com.meitu.meipaimv (pid 9629)
     9130 kB: com.hola.launcher (pid 9876)
     8969 kB: com.android.phone (pid 3642)
     8226 kB: com.google.android.googlequicksearchbox:search (pid 11031)
     8121 kB: com.google.android.inputmethod.japanese (pid 9153)
     7308 kB: com.tencent.android.qqdownloader (pid 11077)
     5043 kB: com.ksmobile.launcher:crash_service (pid 11089)
     4986 kB: com.ijinshan.browser_fast:service (pid 8820)
     4820 kB: com.ijinshan.browser_fast:player (pid 8975)
     4784 kB: com.android.nfc (pid 3584)
     4608 kB: android.process.media (pid 10087)
     4137 kB: logd (pid 335)
     3675 kB: com.tencent.mtt:service (pid 11001)
     3590 kB: com.quicinc.cne.CNEService (pid 3572)
     3585 kB: com.android.bluetooth (pid 12136)
     2517 kB: com.qualcomm.qti.rcsbootstraputil (pid 3593)
     2418 kB: mediaserver (pid 487)
     2277 kB: com.qualcomm.qti.rcsimsbootstraputil (pid 3618)
     1208 kB: zygote64 (pid 499)
     1005 kB: perfd (pid 30233)
      951 kB: zygote (pid 500)
      894 kB: dumpsys (pid 11059)
      793 kB: rild (pid 485)
      684 kB: adbd (pid 476)
      619 kB: netd (pid 482)
      534 kB: /init (pid 1)
      434 kB: ueventd (pid 308)
      432 kB: healthd (pid 367)
      419 kB: debuggerd64 (pid 484)
      412 kB: wpa_supplicant (pid 11758)
      408 kB: sdcard (pid 3162)
      352 kB: debuggerd (pid 483)
      325 kB: thermal-engine (pid 496)
      273 kB: libdaemon.so (pid 10289)
      255 kB: kbmonitor (pid 8538)
      236 kB: qmuxd (pid 492)
      233 kB: perfd (pid 493)
      221 kB: servicemanager (pid 369)
      201 kB: cnd (pid 508)
      193 kB: logcat (pid 30118)
      184 kB: location-mq (pid 515)
      161 kB: slim_daemon (pid 517)
      153 kB: msm_irqbalance (pid 374)
      146 kB: vold (pid 337)
      144 kB: wcnss_filter (pid 4746)
      124 kB: rmt_storage (pid 371)
      114 kB: lowi-server (pid 516)
      102 kB: keystore (pid 490)
      100 kB: netmgrd (pid 495)
       97 kB: time_daemon (pid 505)
       92 kB: mm-qcamera-daemon (pid 506)
       83 kB: qseecomd (pid 382)
       76 kB: lmkd (pid 368)
       64 kB: imsdatadaemon (pid 512)
       50 kB: pm-service (pid 372)
       44 kB: cnss-daemon (pid 504)
       34 kB: gatekeeperd (pid 501)
       30 kB: installd (pid 488)
       20 kB: drmserver (pid 486)
       20 kB: qti (pid 494)
       16 kB: imscmservice (pid 498)
       14 kB: pm-proxy (pid 410)
       14 kB: ATFWD-daemon (pid 507)
       14 kB: fingerprintd (pid 510)
        8 kB: qseecomd (pid 373)
        8 kB: imsqmidaemon (pid 497)
        8 kB: loc_launcher (pid 503)
        8 kB: sh (pid 30230)
        0 kB: com.tencent.android.qqdownloader (pid 10949)

Total PSS by OOM adjustment:
    55675 kB: Native
               23759 kB: surfaceflinger (pid 370)
                7308 kB: com.tencent.android.qqdownloader (pid 11077)
                5043 kB: com.ksmobile.launcher:crash_service (pid 11089)
                4137 kB: logd (pid 335)
                2418 kB: mediaserver (pid 487)
                1208 kB: zygote64 (pid 499)
                1005 kB: perfd (pid 30233)
                 951 kB: zygote (pid 500)
                 894 kB: dumpsys (pid 11059)
                 793 kB: rild (pid 485)
                 684 kB: adbd (pid 476)
                 619 kB: netd (pid 482)
                 534 kB: /init (pid 1)
                 434 kB: ueventd (pid 308)
                 432 kB: healthd (pid 367)
                 419 kB: debuggerd64 (pid 484)
                 412 kB: wpa_supplicant (pid 11758)
                 408 kB: sdcard (pid 3162)
                 352 kB: debuggerd (pid 483)
                 325 kB: thermal-engine (pid 496)
                 273 kB: libdaemon.so (pid 10289)
                 255 kB: kbmonitor (pid 8538)
                 236 kB: qmuxd (pid 492)
                 233 kB: perfd (pid 493)
                 221 kB: servicemanager (pid 369)
                 201 kB: cnd (pid 508)
                 193 kB: logcat (pid 30118)
                 184 kB: location-mq (pid 515)
                 161 kB: slim_daemon (pid 517)
                 153 kB: msm_irqbalance (pid 374)
                 146 kB: vold (pid 337)
                 144 kB: wcnss_filter (pid 4746)
                 124 kB: rmt_storage (pid 371)
                 114 kB: lowi-server (pid 516)
                 102 kB: keystore (pid 490)
                 100 kB: netmgrd (pid 495)
                  97 kB: time_daemon (pid 505)
                  92 kB: mm-qcamera-daemon (pid 506)
                  83 kB: qseecomd (pid 382)
                  76 kB: lmkd (pid 368)
                  64 kB: imsdatadaemon (pid 512)
                  50 kB: pm-service (pid 372)
                  44 kB: cnss-daemon (pid 504)
                  34 kB: gatekeeperd (pid 501)
                  30 kB: installd (pid 488)
                  20 kB: drmserver (pid 486)
                  20 kB: qti (pid 494)
                  16 kB: imscmservice (pid 498)
                  14 kB: pm-proxy (pid 410)
                  14 kB: ATFWD-daemon (pid 507)
                  14 kB: fingerprintd (pid 510)
                   8 kB: qseecomd (pid 373)
                   8 kB: imsqmidaemon (pid 497)
                   8 kB: loc_launcher (pid 503)
                   8 kB: sh (pid 30230)
    69582 kB: System
               69582 kB: system (pid 956)
    87976 kB: Persistent
               65839 kB: com.android.systemui (pid 16734 / activities)
                8969 kB: com.android.phone (pid 3642)
                4784 kB: com.android.nfc (pid 3584)
                3590 kB: com.quicinc.cne.CNEService (pid 3572)
                2517 kB: com.qualcomm.qti.rcsbootstraputil (pid 3593)
                2277 kB: com.qualcomm.qti.rcsimsbootstraputil (pid 3618)
     3585 kB: Persistent Service
                3585 kB: com.android.bluetooth (pid 12136)
    61804 kB: Foreground
               19387 kB: com.facebook.katana (pid 9800)
               15966 kB: com.cleanmaster.security:DefendService (pid 10816)
               14550 kB: system:ui (pid 10935 / activities)
                8226 kB: com.google.android.googlequicksearchbox:search (pid 11031)
                3675 kB: com.tencent.mtt:service (pid 11001)
   753260 kB: Visible
              665973 kB: com.ksmobile.launcher (pid 10462 / activities)
               41436 kB: com.google.android.gms (pid 8728)
               35125 kB: com.google.android.gms.persistent (pid 9253)
               10726 kB: com.ksmobile.launcher:locker (pid 10545)
   106364 kB: Perceptible
               56779 kB: com.apusapps.launcher (pid 8734)
               16181 kB: com.hola.launcher:boost (pid 9661)
               16153 kB: com.ijinshan.browser_fast:cheetah_push_fast (pid 8395)
                9130 kB: com.hola.launcher (pid 9876)
                8121 kB: com.google.android.inputmethod.japanese (pid 9153)
    70080 kB: A Services
               19028 kB: com.tencent.android.qqdownloader:daemon (pid 8714)
               14720 kB: .DaemonService (pid 10872)
               12847 kB: com.whatsapp (pid 9481)
               12154 kB: com.meitu.meipaimv:pushservice (pid 9228)
               11331 kB: com.tencent.android.qqdownloader:wns (pid 9724)
                   0 kB: com.tencent.android.qqdownloader (pid 10949)
    31103 kB: Home
               31103 kB: com.google.android.googlequicksearchbox (pid 8854 / activities)
    96530 kB: B Services
               36731 kB: com.ijinshan.browser_fast (pid 8873)
               25180 kB: com.qiyi.video (pid 9215)
               10962 kB: com.qiyi.video:downloader (pid 10424)
                9243 kB: com.meitu.meipaimv (pid 9629)
                4986 kB: com.ijinshan.browser_fast:service (pid 8820)
                4820 kB: com.ijinshan.browser_fast:player (pid 8975)
                4608 kB: android.process.media (pid 10087)

Total PSS by category:
   605900 kB: Gfx dev
   216376 kB: Native
   216027 kB: Dalvik
    58088 kB: EGL mtrack
    55370 kB: Dalvik Other
    41388 kB: .art mmap
    36466 kB: .dex mmap
    29690 kB: .so mmap
    21092 kB: Unknown
    20420 kB: .oat mmap
    16980 kB: Stack
     8743 kB: .apk mmap
     5432 kB: Ashmem
     3654 kB: Other mmap
      325 kB: Other dev
        8 kB: Cursor
        0 kB: .jar mmap
        0 kB: .ttf mmap
        0 kB: GL mtrack
        0 kB: Other mtrack

Total RAM: 1857352 kB (status critical)
 Free RAM: -471604 kB (0 cached pss + -514364 cached kernel + 42760 free)
 Used RAM: 1540335 kB (1335959 used pss + 204376 kernel)
 Lost RAM: 788621 kB
     ZRAM: 80308 kB physical used for 264720 kB in swap (520908 kB total swap)
   Tuning: 192 (large 512), oom 322560 kB, restore limit 107520 kB (high-end-gfx)
~~~

###  Huawei P6 (7.0)

> 900 times之后，系统无明显卡顿
> 1400 times之后，点按home键，系统明显反应变慢，系统Launcher重启
> 回到桌面，系统无明显卡顿； 当开启大内存APP时会引发launcher被杀

* before — process info  P6
```Java
** MEMINFO in pid 4843 [com.ksmobile.launcher] **
                   Pss  Private  Private  SwapPss     Heap     Heap     Heap
                 Total    Dirty    Clean    Dirty     Size    Alloc     Free
                ------   ------   ------   ------   ------   ------   ------
  Native Heap    12447    12428        0        0    22016    16749     5266
  Dalvik Heap     8831     8792        0        0     8353     5012     3341
 Dalvik Other     1495     1492        0        0
        Stack      796      796        0        0
       Ashmem      132      132        0        0
    Other dev        4        0        4        0
     .so mmap     4186      192      964       89
    .apk mmap     4695      104     1108        0
    .ttf mmap        7        0        0        0
    .dex mmap     7648       24     1988        0
    .oat mmap     4481        0     1404        0
    .art mmap     1914     1104       52        5
   Other mmap      179        4       20        0
    GL mtrack    18304    18304        0        0
      Unknown     2316     2316        0        2
        TOTAL    67531    45688     5540       96    30369    21761     8607

 App Summary
                       Pss(KB)
                        ------
           Java Heap:     9948
         Native Heap:    12428
                Code:     5784
               Stack:      796
            Graphics:    18304
       Private Other:     3968
              System:    16303

               TOTAL:    67531       TOTAL SWAP PSS:       96
```
* after — process info P6
```Java
1300 times
** MEMINFO in pid 9127 [com.ksmobile.launcher] **
                   Pss  Private  Private  SwapPss     Heap     Heap     Heap
                 Total    Dirty    Clean    Dirty     Size    Alloc     Free
                ------   ------   ------   ------   ------   ------   ------
  Native Heap    17486    17428       20        0    24064    20711     3352
  Dalvik Heap     6901     6556      272        0     7984     4791     3193
 Dalvik Other     1314     1304        0        0
        Stack      572      572        0        0
       Ashmem        8        4        0        0
    Other dev        8        4        4        0
     .so mmap     7905      200     6988      231
    .apk mmap        6        0        0        0
    .ttf mmap        1        0        0        0
    .dex mmap     4114       20      560        0
    .oat mmap     1570        0      744        0
    .art mmap     1370      980       40       17
   Other mmap       42        4       16        1
    GL mtrack  1947212  1947212        0        0
      Unknown     2062     2052        8        6
        TOTAL  1990826  1976336     8652      255    32048    25502     6545

 App Summary
                       Pss(KB)
                        ------
           Java Heap:     7576
         Native Heap:    17428
                Code:     8512
               Stack:      572
            Graphics:  1947212
       Private Other:     3688
              System:     5838

               TOTAL:  1990826       TOTAL SWAP PSS:      255

 Objects
               Views:        7         ViewRootImpl:        1
         AppContexts:        5           Activities:        1
              Assets:        4        AssetManagers:        2
       Local Binders:       46        Proxy Binders:       33
       Parcel memory:       15         Parcel count:       60
    Death Recipients:        2      OpenSSL Sockets:        1

```
* before — system info P6
```Java
Total PSS by process:
    189,883K: system (pid 1300)
    163,800K: com.android.systemui (pid 1700)
    122,828K: com.huawei.android.launcher (pid 3949 / activities)
    116,247K: com.google.android.gms (pid 23035)
    108,920K: surfaceflinger (pid 493)
     97,248K: com.android.mms (pid 4270 / activities)
     93,070K: com.android.settings (pid 1577 / activities)
     81,845K: com.tencent.mm (pid 23577 / activities)
     69,182K: com.android.contacts (pid 1248 / activities)
     68,012K: com.huawei.systemmanager:service (pid 1937)
     65,761K: com.ksmobile.launcher (pid 4843 / activities)
     57,707K: com.android.phone (pid 1990)
     56,123K: com.sohu.inputmethod.sogou (pid 4331)
     55,349K: com.google.android.gms.persistent (pid 13795)
     49,012K: com.android.incallui (pid 2493 / activities)
     46,293K: com.ksmobile.launcher:locker (pid 4933)
     39,852K: com.huawei.systemmanager (pid 1669)
     33,109K: com.tencent.wework (pid 3817)
     32,240K: com.huawei.android.FloatTasks (pid 4316)
     28,995K: com.android.vending (pid 28699)
     27,186K: com.huawei.imonitor (pid 2411)
     26,858K: com.iflytek.speechsuite (pid 2883)
     24,799K: com.tencent.mm:push (pid 23528)
     24,511K: com.tencent.mm:exdevice (pid 23641)
     22,047K: android.process.acore (pid 3761)
     21,967K: com.huawei.health:DaemonService (pid 22960)
     20,068K: com.huawei.powergenie (pid 1964)
     19,123K: perfd (pid 3896)
     18,619K: com.huawei.android.chr (pid 1951)
     18,414K: com.huawei.bd (pid 2827)
     16,915K: com.amap.android.ams (pid 3930)
     16,585K: HwCamCfgSvr (pid 520)
     14,578K: com.huawei.hwid (pid 4417)
     12,821K: com.tencent.wework:pushservice (pid 5107)
     12,088K: com.huawei.hidisk (pid 23106)
     11,682K: com.huawei.android.totemweather (pid 2022)
     11,671K: com.android.nfc (pid 3867)
     11,294K: com.tencent.wework:push (pid 4643)
     10,972K: com.android.browser (pid 4676)
     10,808K: perfhub (pid 488)
     10,323K: com.huawei.android.ds (pid 23125)
      9,050K: sogou.mobile.explorer.hotwords (pid 27923)
      9,034K: com.huawei.lbs:remote (pid 5840)
      8,877K: com.sohu.inputmethod.sogou:push_service (pid 10853)
      8,721K: com.huawei.hwid:pushservice (pid 4822)
      8,700K: com.huawei.android.pushagent.PushService (pid 2961)
      7,933K: aptouch_daemon (pid 498)
      7,838K: com.huawei.ihealth (pid 1977)
      7,714K: com.huawei.lbs (pid 3900)
      7,146K: com.huawei.android.hwouc (pid 3617)
      7,007K: com.google.process.gapps (pid 4749)
      6,981K: com.sohu.inputmethod.sogou:classic (pid 5030)
      6,400K: com.huawei.remoteassistant:pushservice (pid 4876)
      6,158K: glgps4774 (pid 1287)
      6,107K: com.google.android.ext.services (pid 2421)
      5,899K: com.huawei.trustspace (pid 4692)
      5,706K: imonitor (pid 535)
      5,640K: com.huawei.geofence (pid 2843)
      5,428K: com.huawei.android.findmyphone:pushservice (pid 4772)
      5,350K: com.huawei.indexsearch.observer (pid 3872)
      4,642K: com.huawei.android.pushagent (pid 4796)
      4,491K: com.huawei.securitymgr (pid 1684)
      4,402K: zygote (pid 515)
      4,402K: audioserver (pid 571)
      4,269K: com.huawei.cryptosms.service (pid 4391)
      4,240K: com.android.printspooler (pid 5367)
      4,220K: com.google.android.partnersetup (pid 4723)
      4,062K: com.huawei.vassistant:interactor (pid 3849)
      4,028K: com.android.defcontainer (pid 4607)
      3,981K: zygote64 (pid 514)
      3,724K: com.android.supl (pid 3893)
      3,659K: mediaserver (pid 528)
      3,513K: com.svox.pico (pid 4988)
      3,466K: logd (pid 392)
      3,194K: cameraserver (pid 522)
      3,047K: netd (pid 530)
      2,969K: rild (pid 754)
      2,564K: inv_ipld (pid 758)
      2,399K: mediadrmserver (pid 526)
      2,092K: powerlogd (pid 491)
      2,074K: vold (pid 428)
      1,962K: wpa_supplicant (pid 1936)
      1,880K: teecd (pid 424)
      1,785K: lmkd (pid 494)
      1,741K: drmserver (pid 523)
      1,726K: /init (pid 1)
      1,414K: media.extractor (pid 527)
      1,378K: fingerprintd (pid 531)
      1,334K: keystore (pid 529)
      1,196K: adbd (pid 3807)
      1,183K: media.codec (pid 525)
      1,162K: hwpged (pid 519)
      1,110K: bastetd (pid 505)
      1,107K: lhd4774 (pid 1285)
      1,067K: chargemonitor (pid 537)
      1,011K: installd (pid 524)
      1,005K: media.log (pid 521)
        980K: gatekeeperd (pid 541)
        974K: accproxy (pid 533)
        970K: ueventd (pid 280)
        924K: thermal-daemon (pid 518)
        811K: dumpsys (pid 5913)
        761K: defragd (pid 532)
        698K: servicemanager (pid 492)
        663K: vr_daemon (pid 512)
        649K: fusion_daemon (pid 540)
        638K: sh (pid 3894)
        620K: logcat (pid 3813)
        610K: tui_daemon (pid 511)
        589K: hwnffserver (pid 517)
        584K: hivwserver (pid 513)
        560K: filebackup (pid 4203)
        488K: chargelogcat (pid 3615)
        432K: healthd (pid 487)
        414K: hw_ueventd (pid 510)
        317K: debuggerd64 (pid 426)
        309K: wwdaemonservice (pid 4681)
        302K: debuggerd64:signaller (pid 435)
        294K: debuggerd (pid 425)
        284K: oeminfo_nvm_server (pid 306)
        264K: debuggerd:signaller (pid 436)

Total PSS by OOM adjustment:
    249,678K: Native
        108,920K: surfaceflinger (pid 493)
         19,123K: perfd (pid 3896)
         16,585K: HwCamCfgSvr (pid 520)
         10,808K: perfhub (pid 488)
          7,933K: aptouch_daemon (pid 498)
          6,158K: glgps4774 (pid 1287)
          5,706K: imonitor (pid 535)
          4,402K: zygote (pid 515)
          4,402K: audioserver (pid 571)
          3,981K: zygote64 (pid 514)
          3,659K: mediaserver (pid 528)
          3,466K: logd (pid 392)
          3,194K: cameraserver (pid 522)
          3,047K: netd (pid 530)
          2,969K: rild (pid 754)
          2,564K: inv_ipld (pid 758)
          2,399K: mediadrmserver (pid 526)
          2,092K: powerlogd (pid 491)
          2,074K: vold (pid 428)
          1,962K: wpa_supplicant (pid 1936)
          1,880K: teecd (pid 424)
          1,785K: lmkd (pid 494)
          1,741K: drmserver (pid 523)
          1,726K: /init (pid 1)
          1,414K: media.extractor (pid 527)
          1,378K: fingerprintd (pid 531)
          1,334K: keystore (pid 529)
          1,196K: adbd (pid 3807)
          1,183K: media.codec (pid 525)
          1,162K: hwpged (pid 519)
          1,110K: bastetd (pid 505)
          1,107K: lhd4774 (pid 1285)
          1,067K: chargemonitor (pid 537)
          1,011K: installd (pid 524)
          1,005K: media.log (pid 521)
            980K: gatekeeperd (pid 541)
            974K: accproxy (pid 533)
            970K: ueventd (pid 280)
            924K: thermal-daemon (pid 518)
            811K: dumpsys (pid 5913)
            761K: defragd (pid 532)
            698K: servicemanager (pid 492)
            663K: vr_daemon (pid 512)
            649K: fusion_daemon (pid 540)
            638K: sh (pid 3894)
            620K: logcat (pid 3813)
            610K: tui_daemon (pid 511)
            589K: hwnffserver (pid 517)
            584K: hivwserver (pid 513)
            560K: filebackup (pid 4203)
            488K: chargelogcat (pid 3615)
            432K: healthd (pid 487)
            414K: hw_ueventd (pid 510)
            317K: debuggerd64 (pid 426)
            309K: wwdaemonservice (pid 4681)
            302K: debuggerd64:signaller (pid 435)
            294K: debuggerd (pid 425)
            284K: oeminfo_nvm_server (pid 306)
            264K: debuggerd:signaller (pid 436)
    189,883K: System
        189,883K: system (pid 1300)
    306,622K: Persistent
        163,800K: com.android.systemui (pid 1700)
         57,707K: com.android.phone (pid 1990)
         20,068K: com.huawei.powergenie (pid 1964)
         18,619K: com.huawei.android.chr (pid 1951)
         11,671K: com.android.nfc (pid 3867)
          7,838K: com.huawei.ihealth (pid 1977)
          7,714K: com.huawei.lbs (pid 3900)
          5,640K: com.huawei.geofence (pid 2843)
          5,350K: com.huawei.indexsearch.observer (pid 3872)
          4,491K: com.huawei.securitymgr (pid 1684)
          3,724K: com.android.supl (pid 3893)
      9,034K: Persistent Service
          9,034K: com.huawei.lbs:remote (pid 5840)
    133,773K: Foreground
         68,012K: com.huawei.systemmanager:service (pid 1937)
         65,761K: com.ksmobile.launcher (pid 4843 / activities)
    338,681K: Visible
         97,248K: com.android.mms (pid 4270 / activities)
         55,349K: com.google.android.gms.persistent (pid 13795)
         49,012K: com.android.incallui (pid 2493 / activities)
         46,293K: com.ksmobile.launcher:locker (pid 4933)
         32,240K: com.huawei.android.FloatTasks (pid 4316)
         27,186K: com.huawei.imonitor (pid 2411)
         16,915K: com.amap.android.ams (pid 3930)
          6,107K: com.google.android.ext.services (pid 2421)
          4,269K: com.huawei.cryptosms.service (pid 4391)
          4,062K: com.huawei.vassistant:interactor (pid 3849)
    323,283K: Perceptible
        122,828K: com.huawei.android.launcher (pid 3949 / activities)
         69,182K: com.android.contacts (pid 1248 / activities)
         56,123K: com.sohu.inputmethod.sogou (pid 4331)
         33,109K: com.tencent.wework (pid 3817)
         22,047K: android.process.acore (pid 3761)
         11,294K: com.tencent.wework:push (pid 4643)
          8,700K: com.huawei.android.pushagent.PushService (pid 2961)
     60,913K: A Services
         26,858K: com.iflytek.speechsuite (pid 2883)
         21,967K: com.huawei.health:DaemonService (pid 22960)
         12,088K: com.huawei.hidisk (pid 23106)
     93,070K: Previous
         93,070K: com.android.settings (pid 1577 / activities)
    196,647K: B Services
         81,845K: com.tencent.mm (pid 23577 / activities)
         24,799K: com.tencent.mm:push (pid 23528)
         24,511K: com.tencent.mm:exdevice (pid 23641)
         18,414K: com.huawei.bd (pid 2827)
         11,682K: com.huawei.android.totemweather (pid 2022)
         10,323K: com.huawei.android.ds (pid 23125)
          9,050K: sogou.mobile.explorer.hotwords (pid 27923)
          8,877K: com.sohu.inputmethod.sogou:push_service (pid 10853)
          7,146K: com.huawei.android.hwouc (pid 3617)
    284,544K: Cached
        116,247K: com.google.android.gms (pid 23035)
         39,852K: com.huawei.systemmanager (pid 1669)
         28,995K: com.android.vending (pid 28699)
         14,578K: com.huawei.hwid (pid 4417)
         12,821K: com.tencent.wework:pushservice (pid 5107)
         10,972K: com.android.browser (pid 4676)
          8,721K: com.huawei.hwid:pushservice (pid 4822)
          7,007K: com.google.process.gapps (pid 4749)
          6,981K: com.sohu.inputmethod.sogou:classic (pid 5030)
          6,400K: com.huawei.remoteassistant:pushservice (pid 4876)
          5,899K: com.huawei.trustspace (pid 4692)
          5,428K: com.huawei.android.findmyphone:pushservice (pid 4772)
          4,642K: com.huawei.android.pushagent (pid 4796)
          4,240K: com.android.printspooler (pid 5367)
          4,220K: com.google.android.partnersetup (pid 4723)
          4,028K: com.android.defcontainer (pid 4607)
          3,513K: com.svox.pico (pid 4988)

Total PSS by category:
    477,913K: Native
    428,686K: Dalvik
    233,521K: .dex mmap
    163,652K: GL mtrack
     96,633K: .apk mmap
     85,180K: .art mmap
     83,615K: .oat mmap
     80,856K: EGL mtrack
     78,898K: .so mmap
     69,944K: Unknown
     60,228K: Dalvik Other
     26,156K: Stack
     16,292K: Other mmap
      2,701K: .ttf mmap
      1,040K: Other dev
        695K: Ashmem
         32K: Cursor
         30K: .jar mmap
          0K: Gfx dev
          0K: Other mtrack

Total RAM: 3,811,064K (status normal)
 Free RAM: 2,081,068K (  284,544K cached pss + 1,644,696K cached kernel +   151,828K free)
 Used RAM: 2,357,248K (1,901,584K used pss +   455,664K kernel)
 Lost RAM:  -524,974K
     ZRAM:   113,112K physical used for   379,464K in swap (1,572,860K total swap)
   Tuning: 384 (large 512), oom   322,560K, restore limit   107,520K (high-end-gfx)
```

* after — system info P6
```Java
1300times
Total PSS by process:
  1,977,732K: com.ksmobile.launcher (pid 9127 / activities)
    159,580K: system (pid 1300)
    159,364K: com.android.systemui (pid 1700)
    116,754K: surfaceflinger (pid 493)
     62,987K: com.tencent.wework (pid 9986)
     62,686K: com.huawei.systemmanager:service (pid 1937)
     49,298K: com.huawei.android.launcher (pid 8436 / activities)
     42,673K: com.google.android.gms.persistent (pid 13795)
     36,862K: com.android.incallui (pid 2493 / activities)
     35,711K: com.android.phone (pid 1990)
     30,544K: com.huawei.android.FloatTasks (pid 4316)
     27,002K: com.tencent.mm (pid 8040)
     23,768K: com.huawei.imonitor (pid 2411)
     23,176K: com.sohu.inputmethod.sogou (pid 8648)
     20,599K: com.tencent.wework:push (pid 10027)
     19,970K: com.android.mms (pid 8488)
     19,423K: com.android.contacts (pid 8549)
     18,117K: perfd (pid 3896)
     17,995K: android.process.acore (pid 8689)
     16,391K: com.android.vending (pid 9610)
     16,162K: com.huawei.powergenie (pid 1964)
     15,497K: HwCamCfgSvr (pid 520)
     14,283K: com.amap.android.ams (pid 3930)
     12,212K: com.tencent.mm:push (pid 9047)
     12,019K: com.google.android.gms:car (pid 9678)
     11,551K: com.tencent.mm:exdevice (pid 9096)
     11,188K: com.huawei.health:DaemonService (pid 9513)
     10,241K: com.ksmobile.launcher:locker (pid 9181)
     10,183K: perfhub (pid 488)
      9,368K: com.huawei.android.chr (pid 1951)
      8,616K: com.android.nfc (pid 3867)
      8,611K: com.google.android.gms (pid 9164)
      7,977K: com.huawei.hidisk (pid 8787)
      7,818K: aptouch_daemon (pid 498)
      7,322K: com.huawei.lbs:remote (pid 5840)
      7,181K: com.sohu.inputmethod.sogou:push_service (pid 9353)
      6,855K: com.huawei.lbs (pid 3900)
      6,421K: com.google.android.ext.services (pid 2421)
      6,319K: com.huawei.android.ds (pid 8816)
      5,532K: com.huawei.ihealth (pid 1977)
      5,302K: imonitor (pid 535)
      5,246K: com.huawei.bd (pid 8923)
      5,233K: com.google.process.gapps (pid 9245)
      5,146K: com.huawei.android.totemweather (pid 8725)
      4,870K: com.huawei.geofence (pid 2843)
      4,848K: com.huawei.android.pushagent.PushService (pid 8290)
      4,792K: com.huawei.indexsearch.observer (pid 3872)
      4,724K: com.iflytek.speechsuite (pid 9579)
      4,405K: com.android.settings (pid 8595)
      4,169K: com.huawei.cryptosms.service (pid 8617)
      4,103K: com.huawei.vassistant:interactor (pid 3849)
      3,751K: com.huawei.android.hwouc (pid 9414)
      3,563K: zygote (pid 515)
      3,479K: logd (pid 392)
      3,436K: com.huawei.securitymgr (pid 1684)
      3,238K: com.android.supl (pid 3893)
      3,146K: audioserver (pid 571)
      3,125K: glgps4774 (pid 1287)
      2,914K: netd (pid 530)
      2,809K: mediaserver (pid 528)
      2,519K: zygote64 (pid 514)
      2,433K: cameraserver (pid 522)
      2,425K: inv_ipld (pid 758)
      2,352K: rild (pid 754)
      2,241K: mediadrmserver (pid 526)
      2,116K: powerlogd (pid 491)
      1,916K: teecd (pid 424)
      1,874K: vold (pid 428)
      1,777K: lmkd (pid 494)
      1,593K: dumpsys (pid 10113)
      1,532K: /init (pid 1)
      1,398K: wpa_supplicant (pid 1936)
      1,372K: drmserver (pid 523)
      1,353K: media.extractor (pid 527)
      1,214K: fingerprintd (pid 531)
      1,137K: keystore (pid 529)
      1,133K: media.codec (pid 525)
      1,042K: hwpged (pid 519)
      1,024K: ueventd (pid 280)
        997K: chargemonitor (pid 537)
        986K: adbd (pid 3807)
        953K: lhd4774 (pid 1285)
        941K: accproxy (pid 533)
        932K: thermal-daemon (pid 518)
        929K: gatekeeperd (pid 541)
        923K: media.log (pid 521)
        879K: bastetd (pid 505)
        873K: installd (pid 524)
        702K: defragd (pid 532)
        690K: servicemanager (pid 492)
        645K: vr_daemon (pid 512)
        597K: fusion_daemon (pid 540)
        593K: tui_daemon (pid 511)
        584K: logcat (pid 3813)
        557K: hivwserver (pid 513)
        557K: hwnffserver (pid 517)
        529K: filebackup (pid 4203)
        474K: chargelogcat (pid 3615)
        441K: sh (pid 3894)
        401K: hw_ueventd (pid 510)
        396K: healthd (pid 487)
        334K: wwdaemonservice (pid 10060)
        303K: debuggerd64 (pid 426)
        302K: debuggerd64:signaller (pid 435)
        239K: debuggerd (pid 425)
        239K: debuggerd:signaller (pid 436)
        164K: oeminfo_nvm_server (pid 306)

Total PSS by OOM adjustment:
    242,318K: Native
        116,754K: surfaceflinger (pid 493)
         18,117K: perfd (pid 3896)
         15,497K: HwCamCfgSvr (pid 520)
         10,183K: perfhub (pid 488)
          7,818K: aptouch_daemon (pid 498)
          5,302K: imonitor (pid 535)
          3,563K: zygote (pid 515)
          3,479K: logd (pid 392)
          3,146K: audioserver (pid 571)
          3,125K: glgps4774 (pid 1287)
          2,914K: netd (pid 530)
          2,809K: mediaserver (pid 528)
          2,519K: zygote64 (pid 514)
          2,433K: cameraserver (pid 522)
          2,425K: inv_ipld (pid 758)
          2,352K: rild (pid 754)
          2,241K: mediadrmserver (pid 526)
          2,116K: powerlogd (pid 491)
          1,916K: teecd (pid 424)
          1,874K: vold (pid 428)
          1,777K: lmkd (pid 494)
          1,593K: dumpsys (pid 10113)
          1,532K: /init (pid 1)
          1,398K: wpa_supplicant (pid 1936)
          1,372K: drmserver (pid 523)
          1,353K: media.extractor (pid 527)
          1,214K: fingerprintd (pid 531)
          1,137K: keystore (pid 529)
          1,133K: media.codec (pid 525)
          1,042K: hwpged (pid 519)
          1,024K: ueventd (pid 280)
            997K: chargemonitor (pid 537)
            986K: adbd (pid 3807)
            953K: lhd4774 (pid 1285)
            941K: accproxy (pid 533)
            932K: thermal-daemon (pid 518)
            929K: gatekeeperd (pid 541)
            923K: media.log (pid 521)
            879K: bastetd (pid 505)
            873K: installd (pid 524)
            702K: defragd (pid 532)
            690K: servicemanager (pid 492)
            645K: vr_daemon (pid 512)
            597K: fusion_daemon (pid 540)
            593K: tui_daemon (pid 511)
            584K: logcat (pid 3813)
            557K: hivwserver (pid 513)
            557K: hwnffserver (pid 517)
            529K: filebackup (pid 4203)
            474K: chargelogcat (pid 3615)
            441K: sh (pid 3894)
            401K: hw_ueventd (pid 510)
            396K: healthd (pid 487)
            334K: wwdaemonservice (pid 10060)
            303K: debuggerd64 (pid 426)
            302K: debuggerd64:signaller (pid 435)
            239K: debuggerd (pid 425)
            239K: debuggerd:signaller (pid 436)
            164K: oeminfo_nvm_server (pid 306)
    159,580K: System
        159,580K: system (pid 1300)
    257,944K: Persistent
        159,364K: com.android.systemui (pid 1700)
         35,711K: com.android.phone (pid 1990)
         16,162K: com.huawei.powergenie (pid 1964)
          9,368K: com.huawei.android.chr (pid 1951)
          8,616K: com.android.nfc (pid 3867)
          6,855K: com.huawei.lbs (pid 3900)
          5,532K: com.huawei.ihealth (pid 1977)
          4,870K: com.huawei.geofence (pid 2843)
          4,792K: com.huawei.indexsearch.observer (pid 3872)
          3,436K: com.huawei.securitymgr (pid 1684)
          3,238K: com.android.supl (pid 3893)
      7,322K: Persistent Service
          7,322K: com.huawei.lbs:remote (pid 5840)
  2,040,418K: Foreground
      1,977,732K: com.ksmobile.launcher (pid 9127 / activities)
         62,686K: com.huawei.systemmanager:service (pid 1937)
    209,425K: Visible
         42,673K: com.google.android.gms.persistent (pid 13795)
         36,862K: com.android.incallui (pid 2493 / activities)
         30,544K: com.huawei.android.FloatTasks (pid 4316)
         23,768K: com.huawei.imonitor (pid 2411)
         19,970K: com.android.mms (pid 8488)
         16,391K: com.android.vending (pid 9610)
         14,283K: com.amap.android.ams (pid 3930)
         10,241K: com.ksmobile.launcher:locker (pid 9181)
          6,421K: com.google.android.ext.services (pid 2421)
          4,169K: com.huawei.cryptosms.service (pid 8617)
          4,103K: com.huawei.vassistant:interactor (pid 3849)
    198,326K: Perceptible
         62,987K: com.tencent.wework (pid 9986)
         49,298K: com.huawei.android.launcher (pid 8436 / activities)
         23,176K: com.sohu.inputmethod.sogou (pid 8648)
         20,599K: com.tencent.wework:push (pid 10027)
         19,423K: com.android.contacts (pid 8549)
         17,995K: android.process.acore (pid 8689)
          4,848K: com.huawei.android.pushagent.PushService (pid 8290)
     34,613K: A Services
         11,188K: com.huawei.health:DaemonService (pid 9513)
          7,977K: com.huawei.hidisk (pid 8787)
          6,319K: com.huawei.android.ds (pid 8816)
          4,724K: com.iflytek.speechsuite (pid 9579)
          4,405K: com.android.settings (pid 8595)
     72,089K: B Services
         27,002K: com.tencent.mm (pid 8040)
         12,212K: com.tencent.mm:push (pid 9047)
         11,551K: com.tencent.mm:exdevice (pid 9096)
          7,181K: com.sohu.inputmethod.sogou:push_service (pid 9353)
          5,246K: com.huawei.bd (pid 8923)
          5,146K: com.huawei.android.totemweather (pid 8725)
          3,751K: com.huawei.android.hwouc (pid 9414)
     25,863K: Cached
         12,019K: com.google.android.gms:car (pid 9678)
          8,611K: com.google.android.gms (pid 9164)
          5,233K: com.google.process.gapps (pid 9245)

Total PSS by category:
  2,016,744K: GL mtrack
    153,009K: Dalvik
    133,150K: Native
     89,144K: EGL mtrack
     40,400K: .art mmap
     35,695K: .oat mmap
     34,463K: .so mmap
     30,164K: .dex mmap
     28,656K: Unknown
     26,166K: Dalvik Other
     11,068K: Stack
      6,642K: .apk mmap
      4,262K: Other mmap
        805K: Other dev
        152K: Ashmem
          0K: Cursor
          0K: Gfx dev
          0K: .jar mmap
          0K: .ttf mmap
          0K: Other mtrack

Total RAM: 3,811,064K (status low)
 Free RAM:   604,059K (   25,863K cached pss +   186,648K cached kernel +   391,548K free)
 Used RAM: 3,592,343K (3,222,035K used pss +   370,308K kernel)
 Lost RAM:   -72,966K
     ZRAM:   211,032K physical used for   738,332K in swap (1,572,860K total swap)
   Tuning: 384 (large 512), oom   322,560K, restore limit   107,520K (high-end-gfx)
```