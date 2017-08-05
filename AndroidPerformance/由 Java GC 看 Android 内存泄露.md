# 由 _Java GC_ 看 _Android_ 内存泄露

>**标题**：_《由 Java GC 看 Android 内存泄露》_
>**作者**：_KyleCe_
>**关键词**：_JavaGC、Android、内存泄露_
>
>@Email: chengong90@gmail.com
>@date: 2017年07月31日

内存泄露，一个老生常谈的话题，本文将从Java GC的角度出发，着眼JavaGC收集器，一探Android内存泄露的究竟，最后总结实战经验，希望能给读者带来些许启发。


*本篇涵盖：*

- _**准备**——基础知识要求及名字解释_
- _**背景**——Android内存泄露的本质与危害_
- _**原因**——为什么会产生泄露_
- _**基础**——Java内存分配与回收_
- _**引申**——JVM与Android虚拟机_
- _**实战**——内存泄露攻防_

## 零、准备
### 0.0 要求：
  阅读本文，需要读者具有一定的JAVA基础与Android基础
### 0.1 名词解释：
* GC——Garbage Collector垃圾收集器
* MAT——Eclipse  Memory Analyzer Tool 内存分析工具
* LeakCanary——第三方内存泄露监测工具
* StrictMode——Android严格模式，调优时可以参考
* HotSpot——Sun公司开发的Java虚拟机类别，现属Oracle
* 堆、栈——Heap、Stack，一般指内存堆，方法栈
* finalize——Object在被GC回收时可能会被调用的方法
* GC Roots——GC引用路径，通常有好几种类别，是一个集合
* StrongReference/SoftReference/WeakReference/PhantomReference——强/弱/软/虚引用
* STW——StopTheWorld GC时需要暂停所有用户线程
* SafePoint & SafeRegion——安全点/安全区，分别为内存/线程在GC 欲Stop The World时可以停留的点
* OopMap——Ordinary Object Pointer Map，HotSpot实现准备式GC的基础
* 年轻代、老年代、永久代、MetaSpace
* Eden/Survivor——年轻代的区域划分
* MinorGC & FullGC/MajorGC——轻量GC/Full GC
* Age——对象年龄，每活过一次GC，Age+1
* JVM/DVM/ART——Java虚拟机/Android Dalvik/ Android Runtime

### 0.2 演示环境：
* Android Studio——3.0 Canary 8
* Eclipse MAT——V1.7.0
* LeakCanary——V1.5.1
* HotSpot——Java Hotspot JVM(SE6/7)

## 一、本质与危害
### 1.1 何谓内存泄露
在计算机科学中，内存泄漏指由于疏忽或错误造成程序未能释放已经不再使用的内存。内存泄漏并非指内存在物理上的消失，而是应用程序分配某段内存后，由于设计错误，导致在释放该段内存之前就失去了对该段内存的控制，从而造成了内存的浪费。

在安卓中，内存泄露主要是指应用程序进程在运行过程中有不能释放而不再使用的内存，占用了比实际需要多的空间。

图1.1.1是使用MAT分析手机内存快照得到的OverView结果：

![图1.1.1 CMCM某款应用的Debug版内存泄露OverView](http://upload-images.jianshu.io/upload_images/1481332-ca4c6177eb55ae69.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图1.1.1 CMCM某款应用的Debug版内存泄露OverView


### 1.2 恶劣影响
内存泄漏会因为减少可用内存的数量从而降低计算机的性能。最终，在最糟糕的情况下，过多的可用内存被分配掉导致全部或部分设备停止正常工作，或者应用程序崩溃。在以下情況，内存泄漏导致较嚴重的后果：
* 程序运行后置之不理，消耗越来越多的内存（比如服务器上的后台任务，尤其是嵌入式系统中的后台任务，这些任务可能被运行后很多年内都置之不理）；
* 频繁分配新内存；
* 程序能够请求未被释放的内存（比如共享内存）；
* 内存非常有限，比如在嵌入式系统或便携设备中；
* ...

针对安卓，内存泄露轻则导致应用占用内存虚高、增加CPU占用、耗电，重则导致应用程序无法开辟所需大小的内存，引发OOM，触发崩溃，这在内存小的机器上尤为明显（我们平时在测试应用内存占用表现时，可以多使用低端机）。
结合上一节所举例子，由图1.1.1可见该应用的泄露足有35M之多，这一内存结果还是应用刚启动时的情况，随着用户使用时间加长，泄露只会越来越多，直到用户杀死应用或者应用主动崩溃（如图1.2.1）。

![图1.2.1 AndroidStudio 某OutOfMemory 堆栈](http://upload-images.jianshu.io/upload_images/1481332-b460f38cafbd927b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图1.2.1 AndroidStudio 某OutOfMemory 堆栈


## 二、产生缘由

内存泄露诱因有很多，安卓中比较常见的有：

* 静态变量持有引用(集合类、单例造成的内存泄漏)
* 匿名内部类/非静态内部类和异步线程
* Handler 、UI线程的post、AnimatorListener等使用不当
* 资源未关闭(或在finalize中关闭)
* 监听器的使用，在释放对象的同时没有相应删除监听器
* ...

下面针对部分诱因进行说明，具体解决办法此处按下不表。
### 2.1 静态变量导致的泄露
静态集合导致的泄露可以分析为：长生命周期的对象，持有了短生命周期对象的引用，在后者生命周期结束时未释放长周期对象对它的引用，导致对象无法被GC回收。

以如下代码为示例：即使在循环内有设置集合对象为null，但集合中的对象还是存在，GC并不能回收它（这种在集合中不断创建新对象的写法也是极其臭名昭著的）。
```java
    public class ExampleUnitTest {
        private static Vector sVector = new Vector();

        @Test
        public void name() throws Exception {
            for (int i = 1; i < 1000; i++) {
                Object o = new Object();
                sVector.add(o);
                o = null;
            }
        }
    }
```


图2.1为某APP静态集合泄露的对象汇总，可以看到总大小有11.7M之大。
![图2.1 某静态泄露的汇总结果](http://upload-images.jianshu.io/upload_images/1481332-5a2e015b3d5de3b3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600/h/600)
图2.1 某静态泄露的汇总结果

### 2.2 匿名内部类引发的内存泄露
匿名内部类极易引发内存泄露，纵使这样的写法在代码层面会简洁很多，但在涉及到匿名内部类生命周期不依附于外部类时，需要我们谨慎处理，不然就很有可能引发泄露。

如下代码为AppsFlayer SDK 4.7.1 Foreground.java中某段代码的近似版本（该泄露在SDK v4.7.4中已修复，图2.2.1 为两版本代码的对比图，中间截图为监测工具上报的泄露路径）：

```Java
    private void async(final Activity activity) {
        new AsyncTask<Void, Void, Void>() {
            protected Void doInBackground(Void... params) {
                try {
                    Thread.sleep(500L);
                } catch (InterruptedException var4) {
                }
                try {
                    WeakReference<Activity> weakActivity = new WeakReference(activity);
                    weakActivity.get().setContentView(null);
                    weakActivity.clear();
                } catch (Exception var3) {
                    this.cancel(true);
                }
                return null;
            }
        }.execute();
    }
```

![图2.2.1 AppsFlyer leak and resolution](http://upload-images.jianshu.io/upload_images/1481332-783169a523910316.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图2.2.1 AppsFlyer leak and resolution

如下代码展示了常见的Handler写法可能引发的内存泄露：
```Java
	protected Handler mHandler = new Handler() {
		@Override
		public void handleMessage(Message msg) {
			if (!onMessage(msg)) {
				super.handleMessage(msg);
			}
		}
	};
```

如下代码展示了内部类持有外部类的成员变量，存在泄露隐患
```Java
    private class ImageContext implements ThemeDataManager.Context<Pair<String, Bitmap>> {

        @Override
        public void onSucc(final JSONObject extendData, Pair<String, Bitmap> p) {
            if(p == null) {
                return;
            }
            mCoverView.setImageBitmap(p.second);
        }

        @Override
        public void onFail(final JSONObject extendData, int result
                                        , Pair<String, Bitmap> cache) {
        }
    }
```

一般在一个质量欠佳的工程中，匿名内部类或异步线程操作导致的内存泄露随处可见。

### 2.3 Handler任务管理不当
Handler、AnimationListener、AnimatorUpdateListener使用不当也极易导致泄露:
```Java
    mHandler.postDelayed(new Runnable() {
        @Override
        public void run() {
            System.out.println(TAG + TAG + "  running" + parent);
            mHandler.postDelayed(this, 0);
        }
    }, 200);
```

### 2.4 资源未及时关闭
Android资源不及时关闭会出现内存泄露的地方有很多，诸如在使用I/O流、Cursor（图2.4.1展示了在APP开启StrictMode时会收到的FileIO未close的异常Throwable）

![图2.4.1 closable close未调用](http://upload-images.jianshu.io/upload_images/1481332-cbf5aa5529865f5f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图2.4.1 closable close未调用

### 2.5 绑定/解绑、注册/反注册未成对调用
绑定/解绑、注册/反注册同时出现这一点毋庸置疑，但实际工程中发现有开发者对于成对调用的理解不够透彻，会有前后条件不一致的情况，导致内存泄露（如注册时无条件注册，反注册时加入不能100%保证成立的判定条件）

## 三、Java内存分配与垃圾回收策略
### 3.0 虚拟机架构（HotSpot）
图3.0.1为HotSpot虚拟机架构，具体的划分将在下文中描述，这里只需要了解大致概念。
![图3.0.1 HotSpot Structure](http://upload-images.jianshu.io/upload_images/1481332-1bf9a615abb8e4d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图3.0.1 HotSpot Structure

本文所谈论的GC，处理的内存区块针对的主要是虚拟机的Heap，亦即堆。
![图3.0.2 HotSpot JVM components](http://upload-images.jianshu.io/upload_images/1481332-7061db0196475abb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图3.0.2 HotSpot JVM components


### 3.1 内存分配
#### 3.1.1 对象生命周期
* 至少两次标记——一次标记、筛选是否finalize
* finalize()——不承诺完成、一次机会、不承诺调 用顺序——避免无良操作引发JVM崩溃

#### 3.1.2 引用计数收集器
* 为对象添加引用计数器
* 弊端:无法解决对象相互循环引用的问题

#### 3.1.3 可达性分析（Reachability Analysis）
* GC-Roots是否可达
* 枚举 GC Roots

可达性分析，可以解决对象循环引用的问题

如图3.1.1所示的对象中，ObjD、ObjE、ObjF均为GC不可达，可以被GC回收掉
![图3.1.1 GC Roots reachable analyze](http://upload-images.jianshu.io/upload_images/1481332-473f69299d0029a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图3.1.1 GC Roots reachable analyze

#### 3.1.4 引用方式

引用方式的回收时机强调的是该引用方式为对象仅存的形式。
* （**强**）**StrongReference**—protects the referred object from collection by GC
* （**软**）**SoftReference**—won't be collected until its memory is needed
* （**弱**）**WeakReference**—garbage collects when no Strong or Soft refs
* （**虚**）**PhantomReference**—after finalized, before reclaimed

强引用即为一般引用，软引用会在内存不足时回收，弱引用则是在GC时立即回收，虚引用一般用于标记GC对对象内存的操作。

#### 3.1.5 内存回收方式
* GC、收集算法、收集器种类 枚举根节点
* SafePoint & SafeRegion（只有到达SafePoint，非运行状态的用户线程处于SafeRegion时才可以STW）
* 分代回收?

GC运行时，需要Stop The World，HotSpot中，利用OopMap存储对象引用

图3.1.5.1展示了HotSpot的堆结构，可以看到整个堆内存分为三代（年轻代、老年代、永久代"JAVA 8已放弃永久代"）

其中年轻代又分为三个区域（一个Eden，两个Survivor，如此划分是为GC收集算法所做的准备，后面的篇幅有具体介绍）
![图3.1.5.1 HotSpot Heap Structure](http://upload-images.jianshu.io/upload_images/1481332-0cc80185075e1e36.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图3.1.5.1 HotSpot Heap Structure

图3.1.5.2 为本文2.1中示例代码的样式堆占用情况（具体数据依机器而变，参考价值有限）
![图3.1.5.2 Heap Usage ratio](http://upload-images.jianshu.io/upload_images/1481332-5def65145209ff4c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图3.1.5.2 Heap Usage ratio

#### 3.1.6 Java 的 GC Roots
可以作为GC Roots的有：
* JVM Stack(Stack Frame 的本地变量表)中引用的对象
* Native Method Stack 中
* JNI(Native 方法)引用 的对象
* Method Area 中类静态属性引用的对象
* Method Area 中常量引用的对象

#### 3.1.7 SafePoint & SafeRegion
* 只有SafePoint才能STW(方法调用、循环跳 转、异常跳转)(抢先中断、主动中断)
* SafeRegion 无CPU时间程序——扩展的 SafePoint、离开时检查是否在进行GC
* -XX:SurvivorRatio
* 可以把SafeRegion看成是扩大了的SafePoint

### 3.2 内存空间划分

在上一节中，我们可以了解到，虚拟机可以分为如下五个部分：
* 方法区
* 堆
* 虚拟机栈
* 程序计数器 
* 本地方法栈

其中，方法区与堆是进程之间共享的，剩余的三个区块，都是线程级别做出的划分。方法区的回收管理相比于GC原理更为复杂，我们不作介绍。

针对每个线程，后三个区块的协作机理是：
程序计数器记录当前栈帧，在线程运行本地方法时计数器不做记录。

### 3.3 垃圾回收
#### 3.3.1 分代回收
配置参数：
* -XX:NewRatio(Client/ Server差异化配置)
* -XX:PermSize
* ...

分代回收主要是为了提升效率，减少不不要的GC（对于需要长时间保留在内存中的对象进行频次更低的GC扫描）

#### 3.3.2 部分JVM配置参数

图3.3.2.1为部分JVM常规参数一览表，JVM的可配置参数极其丰富，有兴趣读者可查阅其它资料
![图3.3.2.1 部分JVM常规参数一览表](http://upload-images.jianshu.io/upload_images/1481332-c09be6dd982fbba3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图3.3.2.1 部分JVM常规参数一览表

#### 3.3.3 内存分配与回收策略
* Eden为主，TLAB为辅
* 直接进入Old-generation的情况、OOM
* MinorGC & FullGC/MajorGC（图3.3.3.3 MinorGC & MajorGC）

在年轻代内存够用的情况下，内存会被直接分配到年轻代中的Eden区域。（图3.3.3.1 写入年轻代Eden区）
![图3.3.3.1 写入年轻代Eden区](http://upload-images.jianshu.io/upload_images/1481332-242a868f2c4d6c45.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图3.3.3.1 写入年轻代Eden区

在多次GC后，仍然存活的内存会在满足虚拟机配置参数的条件下被晋升到老年区。（图3.3.3.1 展示了年代之间的晋升）
![图3.3.3.2 晋升概览图](http://upload-images.jianshu.io/upload_images/1481332-2540999a7248fa74.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图3.3.3.2 晋升概览图

针对不同年代，虚拟机会采用不同的收集器分时机进行收集，总结来看，年轻代的GC会比老年代的GC **更频繁，效率也更高**。（图3.3.3.3 展示了MinorGC 与 MajorGC的回收成效示例）
![图3.3.3.3 MinorGC & MajorGC](http://upload-images.jianshu.io/upload_images/1481332-facf61f82cf12b67.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图3.3.3.3 MinorGC & MajorGC

#### 3.3.4 内存分配
* 优先分配Eden（图3.3.4.1 展示直接分配至Eden区的情况）
* 大对象直接进入old-generation
* 长期存活对象进入old-generation(年龄判定)
* 空间分配担保（风险、担保失败、OOM）

![图3.3.4.1 对象空间分配](http://upload-images.jianshu.io/upload_images/1481332-09da3e69c1e7de12.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图3.3.4.1 对象空间分配

图3.3.4.2-图3.3.4.4展示了对象年龄的计算方法
![图3.3.4.2 对象年龄](http://upload-images.jianshu.io/upload_images/1481332-099ba10858c1daac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图3.3.4.2 对象年龄

![图3.3.4.3 对象年龄的增长示例](http://upload-images.jianshu.io/upload_images/1481332-f1b583b94f371c44.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图3.3.4.3 对象年龄的增长示例

除了满足晋升年龄限制条件的对象外，当年轻代的Survivor中对象平均年龄超过一定限度时，有可能会被整体直接晋升到老年区，而不用等到累加到限定的年龄。
![图3.3.4.4 从年轻代晋升到老年代](http://upload-images.jianshu.io/upload_images/1481332-65c85ce0940865cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图3.3.4.4 从年轻代晋升到老年代

在Minor GC进行之前，如果年轻代的内存之和超过了老年代可用内存大小，会涉及到一个担保的概念，如果不允许老年代担保，会直接抛出OOM异常。
图3.3.4.5 描述了整个内存分配的概览
![图3.3.4.5 直接分配至Eden区](http://upload-images.jianshu.io/upload_images/1481332-a9b46c9c1ef158eb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图3.3.4.5 直接分配至Eden区

#### 3.3.5 垃圾收集算法
* 标记-清除
* 复制
* 标记-整理

标记清除是最简单的办法，但是它有一个弊端：会产生内存碎片（当可用内存不够时，会提前触发FullGC）。

复制算法能够解决这一问题，但是会造成空间的浪费，之所以HotSpot年轻代GC能够采用复制算法，是因为临时变量都比较"短命"（MinorGC回收效率基本都能达到80%以上），这样一来就可以考虑使用复制算法，"浪费"掉的内存处于可接受范围内。（Eden:Survivor1:Survivor2 常见比例会维持在 8:1:1左右）


##### 3.3.5.1 标记整理

需要先标记，而后对标记可清楚的内存进行清理：

图3.3.5.1.1 为标记过程，图3.3.5.1.2 为直接清除后的内存情况，可以看到未被回收的内存之间会有大小不等的间隔，这就是"内存碎片"。
![图3.3.5.1.1 Marking](http://upload-images.jianshu.io/upload_images/1481332-dffef12d27e0440c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图3.3.5.1.1 Marking

![图3.3.5.1.2 Normal Deletion](http://upload-images.jianshu.io/upload_images/1481332-7d913bf3a474dcd0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图3.3.5.1.2 Normal Deletion

##### 3.3.5.2 复制算法

年轻代一般会采用的算法，简单高效。

图3.3.5.2 为HotSpot年轻代复制算法示意图，GC时，会把无法回收的内存对象"复制"至Survivor中的一块区域——Survivor 0/1会在两次相邻MinorGC之间来回切换 Copy的 From/To角色
![图3.3.5.2 Coping Referenced Object](http://upload-images.jianshu.io/upload_images/1481332-50f18bb588311844.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图3.3.5.2 Coping Referenced Object

##### 3.3.5.3 标记整理算法

标记整理与标记清理的不同点在于，多了一步整理的操作，而不是直接的清除可清除内存（图3.3.5.3 展示了操作过程，可以看到内存碎片不存在了）
![图3.3.5.3 删除整理](http://upload-images.jianshu.io/upload_images/1481332-e57595176a60153f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图3.3.5.3 删除整理


#### 3.3.6 垃圾收集器种类
垃圾收集器从运行方式上来分，主要分串行、并行两类。

![图3.3.6.0.1 串行-并行收集器](http://upload-images.jianshu.io/upload_images/1481332-d44647bb58bf7c2b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图3.3.6.0.1 串行-并行收集器

由图3.3.6.1可见两种方式的主要区别是在STW时运行GC的线程数量不一样，然而并不能简单得理解"pause 时间越短"越好，在低性能的Client上需要考虑多线程切换的消耗。

>Stop the world 会暂停所有用户线程

具体细化，HotSpot中的收集器有如下几种（JDK1.7U14）：
* Serial(串行)
* ParNew
* Parallel Scavenge
* Serial Old(串行)
* Parallel Old
* CMS
* G1—Garbage First(JDK 7U14)

可以依照各收集器的试用内存年代做划分：
![图3.3.6.0.2 GC收集器](http://upload-images.jianshu.io/upload_images/1481332-a5848042f2a13510.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图3.3.6.0.2 GC收集器

**下面就各收集器特点做要点说明：**


##### 3.3.6.1 Serial
* 简单高效——没有线程交互的开销
* 常用于Client
![图3.3.6.1 Serial运行示意图](http://upload-images.jianshu.io/upload_images/1481332-26a178b15d61c0d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图3.3.6.1 Serial运行示意图

##### 3.3.6.2 ParNew
* 常用于Server模式
* 只能与CMS配合工作
* 单CPU效果不如Serial
* -XX:ParallelGCThreads限制线程数量
* 关注吞吐量
![图3.3.6.2 ParNew运行示意图](http://upload-images.jianshu.io/upload_images/1481332-0ba80f3d07039584.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图3.3.6.2 ParNew运行示意图

##### 3.3.6.3 Parallel Scavenge
* -XX:MaxGCPauseMillis——停顿时间以牺牲吞吐量和新生代空间为代价
* -XX:GCTimeRatio
* 自适应调节策略——无需手动设置-Xmn、SurvivorRatio、晋升 OldGeneration大小等参数
![图3.3.6.3 Parallel Scavenge运行示意图](http://upload-images.jianshu.io/upload_images/1481332-8c509f9d210e2339.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图3.3.6.3 Parallel Scavenge运行示意图

##### 3.3.6.4 Serial Old
* 主要用于Client
* 可作为CMS的后备选项
![图3.3.6.4 Serial Old运行示意图](http://upload-images.jianshu.io/upload_images/1481332-8927a9cb7340b35b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图3.3.6.4 Serial Old运行示意图

##### 3.3.6.5 Parallel Old
* Parallel Scavenge的Old generation版本
* 多线程 标记-整理
* 配合ps使用在注重吞吐量及CPU敏感场合
![图3.3.6.5 Parallel Old运行示意图](http://upload-images.jianshu.io/upload_images/1481332-3d1db4efd09b12db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图3.3.6.5 Parallel Old运行示意图

##### 3.3.6.6 CMS
* 目标:获取最短回收停顿时间(标记-清除算法)
* 步骤:初始标记-并发标记-重新标记-并发清除
* 缺点:CPU资源敏感、无法处理浮动垃圾(OG68%启用 CMS，失败时启用Serial Old)、产生大量碎片
![图3.3.6.6.1 CMS运行示意图](http://upload-images.jianshu.io/upload_images/1481332-962ab459822c3a93.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图3.3.6.6.1 CMS运行示意图

![图3.3.6.6.2 Serial收集器与CMS收集器操作线程比较](http://upload-images.jianshu.io/upload_images/1481332-0d5f648b819bd823.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图3.3.6.6.2 Serial收集器与CMS收集器操作线程比较

##### 3.3.6.7 G1
G1 相比于前面所述的收集器格外不一样，因为G1在内存的划分上，将内存平分为大小相等的几个区域，年轻代与老年代之间不在是物理隔离，总结起来，G1特点有：

* 并行与并发
* 分代收集
* 空间整合
* 可预测停顿
* 将堆划分为多个大小相等的Region
* Y/O不再物理隔离
* 有计划避免全盘GC
![图3.3.6.7.1 G1 运行示意图](http://upload-images.jianshu.io/upload_images/1481332-d3f472e8f57f4507.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图3.3.6.7.1 G1 运行示意图

G1会维护一个Region回收优先级列表（根据Region回收价值排序），RememberedSet可保证不全堆扫描也不会遗漏，线程要保证暂停在安全点，就需要维护一个RememberSetLog，这些列表需要在最终标记时与RememberedSet合并，确保不会出错。


#### 3.3.7 性能指标
 * 吞吐量(Throughput):在一段长时间内，没有花费在垃圾收集上的时间所 占的比例。
 * 垃圾收集代价(Garbage Collection Overhead):垃圾收集时间所占的比例。
 * 暂停时间(Pause Time): 当执行垃圾收集时，程序被迫暂停的时间长度。
 * 垃圾收集频率(Frequency Of Collection):相对于程序的执行，垃圾收集执 行的频率。
 * 覆盖区(Footprint): 大小度量，如堆的大小。
 * 敏捷度(Promptness): 从一个对象成为垃圾时到内存被回收时之间的时间 长度。

#### 3.3.8 收集器策略
* Server:
    * 低停顿：ParNew + CMS(Serial Old)
    * 高吞吐/低CPU：Parallel Scavenge + Parallel Old
* Client:
    * Serial + Serial Old

## 四、JVM与Android虚拟机的异同
### 4.1 JVM
JAVA虚拟机运行的是JAVA字节码,JVM基于栈
* JAVA程序经过编译，生成JAVA字节码保存在class文件中，JVM通过解码class文件中的内容来运行程序。

关于栈式虚拟机：
* 代码必须使用这些指令来移动变量(即push和pop)
* 代码尺寸小和解码效率会更高些
* 堆栈虚拟机指令有隐含的操作数。

### 4.2 安卓虚拟机
#### 4.2.1 DVM与JVM：
* JVM基于栈，Dalvik基于寄存器。Dalvik运行dex文件，而JVM运行java字节码自Android 2.2开始，Dalvik支持JIT（just-in-time，即时编译技术）
* 与一般标准Java虚拟机不同在于：
    * 占用更少空间
    * 为简化翻译，常量池只使用32位索引
    * 标准Java字节码实行8位堆栈指令,Dalvik使用16位指令集直接作用于局部变量。局部变量通常来自4位的“虚拟寄存器”区。这样减少了Dalvik的指令计数，提高了翻译速度。
* Dalvik虚拟机运行的是Dalvik字节码
    * DVM运行的是Dalvik字节码，所有的Dalvik字节码由JAVA字节码转换而来，并被打包到一个DEX（Dalvik Executable）可执行文件中，DVM通过解释DEX文件来执行这些字节码。
* Dalvik可执行文件体积更小

#### 4.2.2 DVM与ART
* 区别：Dalvik虚拟机执行的是dex字节码，ART虚拟机执行的是本地机器码
* ART相比于DVM（空间换时间）：
    * (优点)系统性能显著提升
    * (优点)应用启动更快、运行更快、体验更流畅、触感反馈更及时
    * (优点)续航能力提升
    * (优点)支持更低的硬件
    * (缺点)更大的存储空间占用，可能增加10%-20%
    * (缺点)更长的应用安装时间


## 五、如何“防治”
### 5.1 攻
Android内存泄露判定-解决方法
* 工具
    * IDE自带工具（图5.1.1为AS分析出的引用路径）
    * MAT/Android Studio Profiler/第三方(图5.1.2为MAT分析出的某例泄露)
    * LeakCanary 等监测工具(图5.1.3为LeakCanary监测出的未反注册及未取消动画更新监听的泄露GC-Root引用路径)

_各大工具的使用方法因篇幅所限，就不具体介绍，相关资料亦是汗牛充犊。_

![图5.1.1 AS分析出的泄露现场](http://upload-images.jianshu.io/upload_images/1481332-ca53c4d2e0fd72d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图5.1.1 AS分析出的泄露现场

![图5.1.2 Leak by MAT](http://upload-images.jianshu.io/upload_images/1481332-c29f25476bbacbe2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图5.1.2 Leak by MAT

![图5.1.3 (by LeakCanary) 未反注册及未取消动画更新监听的泄露GC-Root引用路径](http://upload-images.jianshu.io/upload_images/1481332-97d976558b387970.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
图5.1.3 (by LeakCanary

* 查找内存泄露步骤:
    * 了解OutOfMemoryError 情况
    * 重现
        * Heap Dump
        * 计算这个对象到 GC roots 的最短强引用路径。
        * 剔除无用引用
### 5.2 防
#### 5.2.1 Java:
* 减少临时对象的使用
* 尽量使用StringBuffer,而不用String来累加字符串
* 尽量避免使用 static 成员变量
* 分散对象创建或删除的时间
* 勿override finalize()

#### 5.2.2 Android:
* 生命周期最小化(如用ApplicationContext替代Activity)
* 不要在静态变量或者静态内部类中使用非静态外部成员变量，实在要用——
    * 将内部类改为静态内部类
    * 静态内部类中使用弱引用来引用外部类的成员变量
* Handler 尽量不持有对象、依赖场景结束时cancelMessageAndCallback
* 关闭资源(集合清除、Bitmap.recycle、Closable.close、Register.unregister、 Observer.remove、WebView处理)
    及时终止或取消异步任务
* 时刻记得不要加载过大的Bitmap对象；(BitmapFactory.Options)
* 优化界面交互过程中频繁的内存使用；
* 有些地方避免使用强引用，替换为弱引用等操作。
* 对批量加载等操作进行缓存设计，譬如列表图片显示，Adapter的convertView缓存等。
* 尽可能的复用资源；譬如系统本身有很多字符串、颜色、图片、动画、样式，尽量复用style以节约内存。
* 对于有缓存等存在的应用尽量实现onLowMemory()和onTrimMemory()方法。
* 尽量使用线程池替代多线程操作，这样可以节约内存及CPU占用率。
* 尽量管理好自己的Service、Thread等后台的生命周期，不要浪费内存占用。
* 尽量的优化自己的代码，减少冗余，进行编译打包等优化对齐处理，避免类加载时浪费内存。

>参考：
* [内存泄漏Wiki词条](https://zh.wikipedia.org/wiki/内存泄漏)
* [《深入理解Java虚拟机:JVM高级特性与最佳实践(第2版)》周志明著](https://www.amazon.cn/gp/product/B00D2ID4PK/ref=od_aui_detailpages00?ie=UTF8&psc=1)
* [Java Garbage Collection Basics](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html)
* [System.gc() should indicate that we don't recommend use and don't
   guarantee behaviour](http://bugs.java.com/view_bug.do?bug_id=6668279)
* [Why is it bad practice to call System.gc()?](https://stackoverflow.com/questions/2414105/why-is-it-bad-practice-to-call-system-gc)
* [JAVA虚拟机与Android虚拟机的区别](http://www.jianshu.com/p/8edac8e09b3e)
* [Android优化改动——java代码](http://blog.csdn.net/u010299178/article/details/52059998)
* [Android DVM/ART 区分](http://www.jb51.net/article/88708.htm)

>拍砖：
>chengong90@gmail.com
