# CM 性能分享

推荐改动对象

* Dalvik heap
* dex mmap

前期不推荐动 GL mtrack、Native Heap

## 推荐改动的方向

* 优化包体积——收效极大
  * 删除无用代码（不要依赖minifyEnable——没用）
  * 直接影响的是dex mmap
* 不要写 try/catch OutOfMemoryException/Exception/Throwable，尽量暴露问题
* 差异化UI
  * 多打Drawable-xxxxxxx
  * 分包——Google Play 支持的分辨率分包
* 后台进程——想办法合并
  * cm常驻进程只有一个，大部分进程生命大概几十秒（InstanceService——退出时系统自动回收）——主要是耗用zygote
* 可以的话，不要用EventBus_ 所有用EventBus的类 都会持有class（引用谨慎情况下可以使用）

## 解决方案

* dexCache

  * 进程所用class，放在一个统一的dex

  * 碎化dex

    * redex——Facebook tool
    * 服务进程占用内存优化——有多少个class（class.txt）-> 输入到redex-> 生成class集中的dex
* FinalizeReference

   * 别写finalize
   * **FinalizeHelper**
      * 主动删除Reference（使用反射remove Reference）——FileOutputStream、即使close之后，FinalizeReference会依然持有引用
      * 一定要熟悉要所处理的类(不能直接移除FileOutputStream.class)——CMFileInputStream extends FileInputStream
* zygote优化
   * release zygote 资源（删除所有的Drawable）——CleanMaster  **RomBugSolvUtil**  （小米的zygote会很大）
* WeakReference
  * get之后需要clear——get之后会变为强引用？？？

## heap profile 解析

### hprof on AS

* heap

   * app heap
     * FinalizeReference  一旦override finalize()，对象即被挂载到FinalizeReference（每个进程一个）
     * 创建对象速度比finalize速度还要开——disaster；例子：FileInputStream——不要实现finalize
     * dexCache
       * 优秀范例：微信拆dex——每个进程一个dex，需要具体业务时会去唤起具体的dex
       * 插件化价值在哪里？——带来维护困难的问题（除非功能去耦合性特别强）
     * String/String[]/char[]
       * buffer[]
     * Object[]
       * **魔方——（占用4-5M）——删云端的配置文件（~1M）**
         * static，内存会一直增长
   * image heap
      * framework分配的内存，gc-root 由framework控制（比如广播、ActivityManager）
      * Android的method不是分配在Stack上，而是在heap上
   * zygote heap
      * 每开一个进程，都会有一个zygote（小的zygote 5~6M）
      * 会有系统共用的UI

