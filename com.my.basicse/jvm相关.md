#类加载
 ##loading:
        双亲委托机制（PDM）。PDM更好的保证了Java平台的安全性，在该机制中，JVM自带的Bootstrap是根加载器，其他的加载器都有且仅有一个父类加载器。类的加载首先请求父类加载器加载，父类加载器无能为力时才由其子类加载器自行加载
        双亲委派，主要出于安全考虑
         Bootstrap ClassLoader)用来加载java核心类库，无法被java程序直接引用
         extensions class loader):它用来加载 Java 的扩展库。Java 虚拟机的实现会提供一个扩展库目录。该类加载器在此目录里面查找并加载 Java 类
         system class loader）：它根据 Java 应用的类路径（CLASSPATH）来加载 Java类。一般来说，Java 应用的类都是由它来完成加载的。可以通过ClassLoader.getSystemClassLoader()来获取它
         自定义类加载器，通过继承 java.lang.ClassLoader类的方式实现
        lazyLoading五种情况
        ClassLoader的源码：findinCache-->parent.loadClass-->findClass()
        自定义类加载器：
            extends ClassLoader
            overwrite findClass()-->defineClass(byte[]->Class clazz)
            加密
 ##对象的生命周期：
                 加载：查找并加载类的二进制数据，在Java堆中也创建一个java.lang.Class类的对象连接，连接又包含三块内容：验证、准备、初始化。 1）验证，文件格式、元数据、字节码、符号引用验证； 2）准备，为类的静态变量分配内存，并将其初始化为默认值； 3）解析，把类中的符号引用转换为直接引用
                初始化：为类的静态变量赋予正确的初始值
                使用：new出对象程序中使用
                卸载：执行垃圾回收
 ##对象结构：
             对象头：由两部分组成，第一部分存储对象自身的运行时数据：哈希码、GC分代年龄、锁标识状态、线程持有的锁、偏向线程ID（一般占32/64 bit）。第二部分是指针类型，指向对象的类元数据类型（即对象代表哪个类）。如果是数组对象，则对象头中还有一部分用来记录数组长度。
            实例数据：用来存储对象真正的有效信息（包括父类继承下来的和自己定义的）
            对齐填充：JVM要求对象起始地址必须是8字节的整数倍（8字节对齐）

 ##Linking：
        verification 验证文件是否符合jvm规定
        preparation 静态成员变量赋默认值
        resolution 将类方法属性等符号引用解析为直接引用 常量池中各种符号解析为指针偏移量等内存地址的直接引用
    Initializing: 调用类初始化代码，给静态成员变量赋初始值

#运行时数据区(Runtime data area)
        pc程序计数器：存放指令的位置(该内存区域是唯一一个java虚拟机规范没有规定任何OOM情况的区域)
        jvm stack栈帧：
            Frame 每个方法对应一个栈帧
            Local Variable Table
        Heap堆
        Meathod area方法区(元数据可以限制大小也可以不限制，受限于物理内存大小)：
            Perm Space (<1.8) 字符串常量位于PermSpace FGC不会清理 大小启动的时候指定，不能变
            Meta Space (>=1.8) 字符串常量位于堆 会触发FGC清理 不设定的话，最大就是物理内存
        Runtime Constant Pool运行时常量池
        Native Method Stack本地方法区
        Direct Memory：
            JVM可以直接访问的内核空间的内存 (OS 管理的内存)
            NIO ， 提高效率，实现zero copy
            
 ##如何定位垃圾：
        引用计数
        根可达算法            
            GC root可达性算法根：线程栈变量 静态变量 常量池 JNI指针
 ##常见的垃圾回收算法：
        标记清除(mark sweep) 位置不连续 产生碎片 效率偏低 两遍扫描(标记清除mark-sweep 存活对象比较多时效率高，两遍扫描，容易产生碎片)
        拷贝算法(coping) 没有碎片 浪费空间(拷贝coping 移动复制对象，需要调整对象引用 空间浪费。适用于存活少的)
        标记压缩压缩(mark compact) 没有碎片 效率偏低(两遍扫描 指针需要调整)
 ##JVM内存分代模型(用于分代垃圾回收算法)
        部分垃圾回收器使用的模型
            除epsilon ZGC shenandoah之外的GC都是使用逻辑分代模型
            G1是逻辑分代，物理不分代(G1逻辑分代，物理部不分代，其他的逻辑分代物理也分代)
            除此之外不仅逻辑分代，而且物理分代
        新生代 老年代 永久带1.7/元数据区1.8
            永久代 元数据-->class
            永久代必须指定大小限制。元数据可以设置，也可以不设置，无上限(仅受限于物理内存)
            字符串常量1.7永久代 1.8堆
            MethodArea方法区逻辑概念永久代 元数据
        新生代=Eden+2suvivor区
            YGC回收之后，大多数的对象会被回收，活着的进入s0
            再次YGC,活着的对象eden+s0-->s1
            再次YGC,eden+s1-->s0
            年龄足够-->老年代（15 CMS6）
            s区装不下-->老年代
        老年代
            顽固分子
            老年代满了FGC full gc
        GC tuning
            尽量减少FGC
            MinorGC=YGC
            MajorGC=FGC
 ##常见垃圾回收器
        jdk诞生serial追随 为提高效率诞生ps，为配合CMS诞生pn。cms开启了并发回收，但毛病较多目前没有任何一个jdk版本默认是cms
        serial年轻代串行回收
        ps年轻代并行回收(parallel scavenge +parallel old [最常用])
        parnew年轻代配合cms的并行回收
        serialold
        parallelold
        cms(concurrentMarkSweep)老年代并发的，垃圾回收和应用程序同时运行。有碎片化，浮动垃圾问题。达到一定程度时，会启用serialold单线回收(parNew+CMS(concurrent mark sweep)[初始标记 并发标记 重新标记 并发清理]-内存碎片化 浮动垃圾---》引入serial old进行标记压缩)
        G1(10ms)算法:三色标记+satb[内存上百G] 高吞吐 没有内存碎片收集时间可控
            新老年带比例一般5%-60%不手动指定，因为这是g1预测停顿时间的基准
            也会产生fgc：扩大内存，提升cpu配置，降低MixedGC触发的阈值，让MixedGC提早发生(默认是45%)【mixedgc跟cms一样也是四个过程混合回收，哪个region满了就回收哪个不论是o s 还是e】
            漏标解决：
                增量更新，关注应用量的增加，把黑色重新标记为灰色，下次重新扫描属性cms使用
                satb snapshot at the beginning 关注引用的删除，当B>D消失时，要把这个引用推到gc的堆栈。保证d还能被gc扫描到，g1使用
        zgc(1ms)coloredPointers+loadBarrier【内存上T】
        1.8默认是ps+parallelold
        指定垃圾回收器组合：-XX:+UseParallelOldGC
                        -XX:+UseG1GC
 ##JVM常用命令行参数：
        Hotspot参数分类：
            标准：-开头，所有hotspot都支持
            非标准：-X开头，特定版本Hotspot支持特定命令
            不稳定：-XX开头，下个版本可能取消
        参数设定：
            java -Xmn10M -Xms40M -Xmx60M -XX:+PrintCommandLineFlags -XX:+PrintGC HelloGC PrintGCDetails PrintGCTimeStamps PrintGCCauses
            设置日志参数：-Xloggc:/opt/xxx/logs/xxx-xxx-gc-%t.log -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=20M -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCCause
            -Xmn -Xms -Xmx -Xss 年轻代 最小堆 最大堆 栈空间
            -verbose:class 类加载详细过程
            -XX:+PrintFlagsFinal -XX:+PrintFlagsInitial 必须会用
            -XX:MaxTenuringThreshold 升代年龄，最大值15
#调优：
        吞吐量：用户代码时间/(用户代码执行时间+垃圾回收时间)
        响应时间：STW越短，响应时间越好
 ##问题：
        cpu100%的调优：找出那个进程cpu高(top) 命令：top
                      该进程中那个线程cpu高(top -Hp pid进程id) 命令：top -Hp pid 【观察进程中的哪个线程较高】
                      导出该线程的堆栈jstack 命令：jstack pid  [把进程中所有线程都列出来，重点关注线程的状态waiting blocked。假如很多线程都在等待一把锁waiting on<0x000000088ce>要找到哪个线程持有这把锁。搜索jstack dump的信息，找<0x00000888ce> 看哪个线程持有这把锁的runnable 对应到这段代码]
                      查找哪个方法(栈帧)消耗时间
                      查看对象堆的信息：jmap -histo pid | head -20 【前20个堆内存占用较多的对象。jmap执行期间会对进程产生很大影响。很多服务器备份高可用，可用，把要测的主机隔离开】
                                      jmap -dump：format=b，file=xxx pid 转储命令影响较大
          系统内存飚高：
                  导出堆内存jmap
                  分析jhat jvisualvm mat jprofiler
          如何监控jvm：
                  jstat jvisualvm mat jprofiler arthas top
          远程监控java：jmx 在监控程序时，配置相关监控参数。远程jconsole监控或者Java visualVM监控
                  图形化界面，在测试时用。线上cmd命令行
          线上问题定位：阿里arthas
                  启动 ：java -jar arthas-boot.jar  挂到要监控的进程上
                      jvm 查看jvm的情况
                      thread 查看线程状况
                      dashboard 查看整体内存cpu情况 同top
                      deapdump 导出堆情况
                      jhat进行dump文件分析
                      redefine 热替换文件
          oom产生的原因：有些程序未必会产生oom，但不断fgc(cpu标高，但内存回收特别少)
                       线程池不当运用产生oom问题
                       不断往list里加对象(比较low)
                       tomcat http-header-size过大的问题
                       lambda表达式方法区溢出问题（每一个labmda表达式对象会生成一个class对象）
                       直接内存溢出 使用unsafe分配直接内存或者使用nio的问题
                       栈溢出问题：-Xss设定太小
                       重写finalize引发频繁gc