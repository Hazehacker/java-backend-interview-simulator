# 技术知识库

> 源自 JavaGuide 高频面试题整理，聚焦中国大厂（字节、腾讯、阿里、美团、快手）常考问题。
> 每个分类下列出面试最常问的问题及答案要点，面试官可按候选人背景灵活选题。

---

## 目录

- [1. Java 基础](#1-java-基础)
  - [1.1 String/StringBuilder/StringBuffer](#11-stringstringbuilderstringbuffer)
  - [1.2 集合框架](#12-集合框架)
  - [1.3 泛型](#13-泛型)
  - [1.4 其他基础](#14-其他基础)
- [2. 并发编程](#2-并发编程)
  - [2.1 synchronized](#21-synchronized)
  - [2.2 volatile](#22-volatile)
  - [2.3 JUC 并发工具](#23-juc-并发工具)
  - [2.4 线程池](#24-线程池)
  - [2.5 AQS](#25-aqs)
  - [2.6 CAS](#26-cas)
  - [2.7 死锁](#27-死锁)
- [3. JVM](#3-jvm)
  - [3.1 内存结构](#31-内存结构)
  - [3.2 垃圾回收](#32-垃圾回收)
  - [3.3 垃圾收集器](#33-垃圾收集器)
  - [3.4 类加载](#34-类加载)
  - [3.5 性能调优](#35-性能调优)
- [4. MySQL](#4-mysql)
  - [4.1 索引结构](#41-索引结构)
  - [4.2 事务](#42-事务)
  - [4.3 锁](#43-锁)
  - [4.4 SQL 优化](#44-sql-优化)
  - [4.5 主从复制](#45-主从复制)
  - [4.6 其他](#46-其他)
- [5. Redis](#5-redis)
  - [5.1 数据结构](#51-数据结构)
  - [5.2 持久化](#52-持久化)
  - [5.3 缓存问题](#53-缓存问题)
  - [5.4 分布式](#54-分布式)
  - [5.5 过期与淘汰](#55-过期与淘汰)
  - [5.6 Redis 6.0 多线程](#56-redis-60-多线程)
- [6. Spring](#6-spring)
  - [6.1 IoC / DI](#61-ioc--di)
  - [6.2 AOP](#62-aop)
  - [6.3 事务](#63-事务)
  - [6.4 循环依赖](#64-循环依赖)
  - [6.5 其他](#65-其他)
- [7. 分布式系统](#7-分布式系统)
  - [7.1 分布式锁](#71-分布式锁)
  - [7.2 分布式 ID](#72-分布式-id)
  - [7.3 分布式事务](#73-分布式事务)
  - [7.4 一致性hash](#74-一致性hash)
- [8. 系统设计](#8-系统设计)
  - [8.1 高并发基础](#81-高并发基础)
  - [8.2 消息队列](#82-消息队列)
  - [8.3 场景设计高频题](#83-场景设计高频题)
- [9. 网络与操作系统](#9-网络与操作系统)
  - [9.1 计算机网络](#91-计算机网络)
  - [9.2 操作系统](#92-操作系统)

---

## 1. Java 基础

### 1.1 String/StringBuilder/StringBuffer
- **String 为什么不可变？** — final 修饰字符数组 + 私有化 + 类不可继承，防止子类破坏。好处：线程安全、缓存哈希值、安全性（如数据库账号密码）
- **StringBuilder vs StringBuffer** — StringBuffer 内部 synchronized，线程安全但性能低；StringBuilder 非线程安全，性能好。循环内拼接字符串必须用 StringBuilder，否则每次创建新对象
- **字符串拼接 + 和 StringBuilder** — `+` 在循环内会创建多个 StringBuilder 对象（JDK9 后有优化但仍有差异），应显式使用 StringBuilder

### 1.2 集合框架
- **ArrayList vs LinkedList** — ArrayList 基于数组，随机访问 O(1)；LinkedList 基于双向链表，头尾操作 O(1)；一般场景 ArrayList 更优
- **ArrayList 扩容机制** — 默认容量 10，扩容 1.5 倍（`oldCapacity + (oldCapacity >> 1)`），使用 `Arrays.copyOf` 迁移数据
- **HashMap 底层结构（JDK 1.8+）** — 数组 + 链表 → 数组 + 链表 + 红黑树。链表长度 > 8 且数组 >= 64 时转红黑树；树节点数 < 6 时退回链表
- **HashMap 长度为什么是 2 的幂次方？** — `hash % length` 等价于 `hash & (length - 1)`，位运算效率高；且扩容时元素迁移均匀
- **HashMap 线程不安全原因** — JDK7 头插法多线程扩容会死循环；JDK8 尾插法，但仍会有数据覆盖和 size 计算错误
- **ConcurrentHashMap 线程安全实现** — JDK7 分段锁（Segment）；JDK8 使用 CAS + synchronized，锁粒度细化到单个桶节点
- **fail-fast vs fail-safe** — fail-fast 通过 modCount 检测并发修改并抛出 ConcurrentModificationException；fail-safe 复制副本操作（CopyOnWriteArrayList）

### 1.3 泛型
- **泛型擦除机制** — 编译后泛型信息被擦除（替换为上限或 Object），类型转换需要显式处理；带来的问题：无法直接创建泛型数组、无法使用 instanceof T
- **PECS 原则** — Producer Extends（生产用 extends）、Consumer Super（消费用 super）。如 `List<? extends Number>` 可读不可写

### 1.4 其他基础
- **Object 通用方法** — equals、hashCode、toString、clone、finalize（已废弃）
- **重写 equals 必须重写 hashCode** — 两个对象 equals 则 hashCode 必须相同，否则放入 HashMap/HashSet 会出问题
- **深拷贝 vs 浅拷贝** — 浅拷贝引用共享内部对象；深拷贝递归复制所有嵌套对象

---

## 2. 并发编程

### 2.1 synchronized
- **synchronized 底层原理** — 同步语句块用 `monitorenter`/`monitorexit` 指令；同步方法用 `ACC_SYNCHRONIZED` 标识。底层依赖对象监视器（monitor），依赖 OS 的 Mutex Lock 实现
- **synchronized 锁的膨胀过程** — 无锁 → 偏向锁（第一次 CAS 加锁）→ 轻量级锁（CAS 自旋）→ 重量级锁（阻塞）。对象头 Mark Word 记录锁状态
- **synchronized 和 volatile 的区别** — volatile 仅保证可见性和有序性，不保证原子性，只能修饰变量；synchronized 三者都保证，可修饰方法和代码块

### 2.2 volatile
- **volatile 如何保证可见性** — 写操作后插入 StoreStore + StoreLoad 屏障，读操作后插入 LoadLoad + LoadStore 屏障，强制 CPU 从主存读写而非缓存
- **volatile 能保证原子性吗？** — 不能。以 `i++` 为例，包含"读取-修改-写入"三步，volatile 只保证可见性，无法保证这三步的原子性

### 2.3 JUC 并发工具
- **synchronized 和 ReentrantLock 区别** — ReentrantLock 可中断（lockInterruptibly）、可设置公平/非公平锁、支持超时、支持多 Condition 分组唤醒；两者都是可重入锁
- **ReentrantLock 可重入原理** — 内部 state 计数。获取锁 state+1，释放-1，同一线程可重复获取，需释放相同次数
- **CountDownLatch vs CyclicBarrier** — CountDownLatch 倒计数，不可复用（一次性）；CyclicBarrier 可复用，等所有线程都到达屏障点再一起放行
- **Semaphore 用途** — 信号量，控制同时访问资源的线程数。如连接池大小限制
- **ThreadLocal 内存泄漏原因** — Entry 的 key（ThreadLocal）是弱引用，可被 GC 回收；但 value 是强引用，如果线程持续存活（线程池），value 无法回收导致泄漏。使用完务必调用 remove()

### 2.4 线程池
- **线程池七大参数** — corePoolSize、maximumPoolSize、keepAliveTime、unit、workQueue、threadFactory、handler
- **线程池工作流程** — 核心线程未满 → 创建核心线程；满了则入队列；队列满则创建非核心线程；达到最大线程数则执行拒绝策略
- **线程池拒绝策略** — AbortPolicy（抛异常）、CallerRunsPolicy（调用者执行）、DiscardPolicy（丢弃）、DiscardOldestPolicy（丢弃最老）。不允许丢弃选 CallerRunsPolicy
- **线程池分类** — FixedThreadPool（固定大小）、CachedThreadPool（弹性伸缩）、ScheduledThreadPool（定时任务）、SingleThreadExecutor（单线程）

### 2.5 AQS
- **AQS 原理** — 核心是 CLH 锁队列变体（双向队列 + 自旋阻塞）。state 表示同步状态（volatile），通过 CAS 原子操作管理。模板方法模式定义 acquire/release 流程
- **独占模式 vs 共享模式** — 独占（如 ReentrantLock）一次只一个线程获取锁；共享（如 CountDownLatch、Semaphore）多个线程可同时获取

### 2.6 CAS
- **CAS 原理** — 三个操作数：变量 V、预期值 E、新值 N。当 V=E 时原子更新为 N，否则重试。依赖 CPU 原子指令，Java 通过 Unsafe 类实现
- **CAS 的 ABA 问题及解决** — 变量被改成 B 又改回 A，CAS 误认为未被修改。解决：AtomicStampedReference 携带版本号
- **CAS 的缺点** — 循环时间长开销大；只能保证单一变量的原子性

### 2.7 死锁
- **死锁四个必要条件** — 互斥、请求与保持、不剥夺、循环等待
- **如何预防死锁** — 一次性申请所有资源（破坏请求保持）；主动释放资源（破坏不剥夺）；按序申请资源（破坏循环等待）
- **如何排查死锁** — jstack、jconsole、VisualVM；查看线程栈中 blocked 状态的线程

---

## 3. JVM

### 3.1 内存结构
- **运行时数据区** — 线程私有：程序计数器、虚拟机栈、本地方法栈；线程共享：堆、方法区（元空间）。JDK1.8 方法区实现为元空间（直接内存）
- **Java 对象创建过程** — 类加载检查 → 分配内存（指针碰撞/空闲列表 + TLAB/CAS）→ 初始化零值 → 设置对象头 → 执行构造方法

### 3.2 垃圾回收
- **如何判断对象已死亡** — 引用计数法（难以解决循环引用）；可达性分析法（从 GC Roots 向下搜索）
- **GC Roots 有哪些** — 虚拟机栈/本地方法栈引用的对象、方法区静态属性/常量引用的对象、同步锁持有的对象、JNI 引用的对象
- **四种引用类型** — 强引用（绝不回收）、软引用（内存不足时回收）、弱引用（下一次 GC 即回收）、虚引用（不影响生命周期，用于跟踪 GC）
- **垃圾回收算法** — 标记-清除（效率低、碎片）、复制算法（适合新生代，可用内存减半）、标记-整理（适合老年代）
- **分代收集理论** — 新生代对象存活率低用复制算法；老年代用标记-清除/整理。HotSpot 分 Eden、S0、S1、Old 区
- **Minor GC 和 Full GC** — Minor GC 在新生代，频繁但快；Full GC 在老年代或整堆，频率低但停顿长
- **对象何时进入老年代** — 年龄达到 MaxTenuringThreshold（默认 15）；Survivor 区同年龄对象大小超过 50% 时取较小值晋升；大对象直接进入老年代

### 3.3 垃圾收集器
- **常见收集器** — Serial（单线程）、ParNew（多线程版 Serial）、Parallel Scavenge（吞吐量优先）、CMS（并发收集低停顿）、G1（JDK9+ 默认）、ZGC（低延迟，Java21 分代 ZGC）
- **CMS 收集器原理与缺点** — 四步：初始标记(STW) → 并发标记 → 重新标记(STW) → 并发清除。缺点：对 CPU 敏感、无法处理浮动垃圾、产生空间碎片。Java14 已移除
- **G1 收集器原理** — 基于 Region 的分区收集，将堆划分为多个等大 Region，优先回收价值最大（垃圾量最大）的区域。特点：并行并发、空间整合、可预测停顿
- **ZGC 原理** — 标记-复制算法，暂停时间控制在毫秒级且不受堆大小影响，Java15 正式可用

### 3.4 类加载
- **类加载过程** — 加载 → 验证 → 准备（分配内存初始化零值）→ 解析（符号引用转直接引用）→ 初始化（执行 clinit）
- **双亲委派模型及原因** — 类加载器先委托父类处理，父类无法处理才自己加载。好处：防止类被重复加载，防止核心类被篡改。打破方式：自定义类加载器重写 loadClass
- **方法区演变** — JDK1.8 前用永久代（受堆大小限制）；JDK1.8 起改为元空间（本地内存，不受堆大小限制）

### 3.5 性能调优
- **OOM 排查思路** — 常见类型：heap space、GC overhead limit、MetaSpace、StackOverflow。工具：jmap -heap、jmap -dump + MAT、GC 日志
- **常用调优参数** — -Xms/-Xmx（堆大小）、-Xmn（新生代）、-XX:MetaspaceSize（元空间）、-XX:+UseG1GC、-XX:MaxTenuringThreshold、-XX:+PrintGCDetails

---

## 4. MySQL

### 4.1 索引结构
- **B+ Tree 结构特点** — 多路平衡查找树，非叶子节点只存索引键，叶子节点存数据且用双向链表相连。相比 B 树：IO 次数更少、查询稳定、范围查询友好
- **MySQL 为什么用 B+ 树而不是其他结构** — 相比哈希：支持范围查询和排序；相比红黑树/AVL：树高更低（矮胖），IO 次数少
- **聚簇索引 vs 非聚簇索引** — 聚簇索引：数据与索引一起存放（InnoDB 主键索引），叶子节点存完整数据。非聚簇索引：数据分开存放（二级索引），叶子节点存主键值，需要回表
- **覆盖索引与如何避免回表** — 查询字段正好是索引字段，无需回表。建立联合索引时考虑覆盖所需字段
- **最左前缀原则及失效场景** — 从最左侧开始匹配，直到遇到范围查询为止。失效：`LIKE '%abc'`、跳过最左列、对索引列做计算/函数/类型转换、OR 有一侧无索引、隐式类型转换、IN 取值范围过大
- **索引下推（ICP）** — MySQL 5.6 引入，将 Server 层部分条件下推到存储引擎层执行，减少回表次数

### 4.2 事务
- **MySQL 默认隔离级别** — REPEATABLE-READ（可重复读）
- **MVCC 原理** — 隐藏字段：trx_id（事务ID）、roll_pointer（回滚指针）+ ReadView（活跃事务ID列表等）+ undo log 版本链。RC 下每次快照读生成新 ReadView；RR 下只在第一次快照读时生成
- **事务隔离级别解决的问题** — READ UNCOMMITTED（脏读）、READ COMMITTED（不可重复读）、REPEATABLE-READ（InnoDB 通过 MVCC+Next-Key Lock 阻止幻读）、SERIALIZABLE（完全串行）
- **脏读、不可重复读、幻读区别** — 脏读：读未提交；不可重复读：同一事务内同一数据内容被修改；幻读：同一事务内记录数变化（多了或少了）

### 4.3 锁
- **表级锁 vs 行级锁** — 表级锁粒度大开销小；行级锁粒度小开销大，InnoDB 默认行级锁。行锁针对索引字段，非索引字段会锁整表
- **InnoDB 三类行锁** — 记录锁（Record Lock）锁单行；间隙锁（Gap Lock）锁范围不含记录；临键锁（Next-Key Lock）锁范围含记录。唯一索引/主键时 Next-Key Lock 降级为 Record Lock
- **共享锁 vs 排他锁** — S 锁（读锁）多事务可同时获取；X 锁（写锁）获取后不允许任何锁。意向锁（IS/IX）是表级锁，帮助快速判断是否有行锁
- **死锁及避免策略** — 多个事务互相持有对方需要的锁。避免：按固定顺序访问资源；大事务拆小；减少锁持有时间

### 4.4 SQL 优化
- **慢 SQL 定位** — 慢查询日志（slow_query_log）+ long_query_time；EXPLAIN 分析执行计划；SHOW PROCESSLIST
- **EXPLAIN 关键字段** — type（扫描方式，从好到差：const/eq_ref/ref/range/index/all）、key（实际用到的索引）、rows（扫描行数）、Extra（Using filesort/Using index 等）
- **常见 SQL 优化手段** — 避免 SELECT *；少用 OR；用 JOIN 代替子查询；批量操作；创建合适索引；避免索引失效
- **JOIN 算法** — Nested Loop Join（小表驱动大表）、Hash Join（等价连接）、Sort Merge Join（排序合并）

### 4.5 主从复制
- **主从复制原理** — 基于 binlog，主库记录写操作，从库 IO Thread 拉取 binlog 写入 Relay Log，SQL Thread 重放
- **读写分离及好处** — 读操作路由到从库，写操作路由到主库。减轻主库压力、提升并发能力
- **分库分表时机** — 单表数据量过大（超过千万级）、单机 IO 瓶颈、并发瓶颈

### 4.6 其他
- **一条 SQL 执行流程** — 连接器（认证权限）→ 分析器（词法语法）→ 优化器（执行计划/索引选择）→ 执行器（调用存储引擎）→ 存储引擎（InnoDB）
- **MyISAM vs InnoDB** — InnoDB 支持事务、行锁、外键、崩溃恢复、MVCC；MyISAM 都不支持。InnoDB 是聚簇索引，MyISAM 是非聚簇索引

---

## 5. Redis

### 5.1 数据结构
- **Redis 为什么这么快？** — 纯内存操作、单线程 + IO 多路复用（高效 I/O 模型）、简洁的 RESP 协议、优化的内部数据结构（SDS/跳表/压缩列表）
- **String 底层实现 SDS** — 简单动态字符串，5 种实现（sdshdr8/16/32/64）。相比 C 字符串：防缓冲区溢出、O(1) 长度获取、减少内存分配次数、二进制安全
- **有序集合为什么用跳表而不是平衡树？** — 实现简单；插入删除只需简单操作不需要旋转；范围查询效率高；内存效率高（概率平衡）

### 5.2 持久化
- **RDB 原理** — 定时快照，通过 bgsave fork 子进程执行，COW 机制。THP 大页问题可能导致内存放大
- **AOF 三种 fsync 策略** — always（每次写同步，最安全但最慢）、everysec（每秒同步，最多丢 1 秒）、no（OS 决定，最快但最不安全）
- **混合持久化** — Redis 4.0+，文件结构：RDB 头部 + AOF 增量命令。结合快速恢复和数据完整性

### 5.3 缓存问题
- **缓存穿透及解决方案** — 请求数据既不在缓存也不在数据库。方案：布隆过滤器（判断不存在一定准确）、缓存空值、接口限流
- **缓存击穿及解决方案** — 热点 key 过期瞬间大量请求打到数据库。方案：永不过期 + 异步重建、互斥锁、提前预热
- **缓存雪崩及解决方案** — 大量缓存同时过期或 Redis 宕机。方案：Redis 高可用集群、多级缓存、随机失效时间、提前预热
- **如何保证缓存和数据库一致性？** — Cache Aside Pattern（先更数据库再删缓存）；失败时用消息队列异步重试

### 5.4 分布式
- **Redis 分布式锁实现** — `SET key value EX 3 NX` 原子加锁 + 过期时间；value 要唯一（防误删）；释放锁用 Lua 脚本（先比较值再删除保证原子性）。推荐 Redisson
- **Redisson 看门狗原理** — 每隔 lockWatchdogTimeout/3 检测持锁线程是否存活，是则异步续期 30 秒，防止业务未完成锁提前过期
- **Redis 哨兵模式** — 监控节点状态（主观下线/客观下线）、自动故障转移、选举新 master、多个 sentinel 避免单点
- **Redis Cluster 哈希槽** — 16384 个槽位，key 通过 CRC16 取模分配，支持动态扩容，节点间 Gossip 协议通信

### 5.5 过期与淘汰
- **过期 key 删除策略** — 惰性删除（查询时检查）+ 定期删除（hz 每秒 10 次，随机抽查，取出过期则删除）
- **内存淘汰策略** — 6 种基础：volatile-lru/ttl/random、allkeys-lru/random、no-eviction；4.0+ 新增 volatile-lfu、allkeys-lfu

### 5.6 Redis 6.0 多线程
- **为什么要引入多线程？** — 针对网络 IO 读写瓶颈，提高网络性能；执行命令仍为单线程

---

## 6. Spring

### 6.1 IoC / DI
- **IoC/DI 是什么，解决了什么问题** — 控制反转，将对象创建和管理权交给 Spring 容器。降低耦合度，便于资源管理
- **依赖注入方式** — 构造函数注入（推荐，保证完整性和不可变性）、Setter 注入（可选依赖）、Field 注入（不推荐，隐藏依赖）
- **Bean 生命周期** — 实例化 → 属性填充 → 初始化（Aware 接口 → BeanPostProcessor → InitializingBean → init-method）→ 销毁（DisposableBean → destroy-method）

### 6.2 AOP
- **AOP 原理** — 动态代理：实现接口用 JDK Proxy，未实现接口用 CGLIB 生成子类代理。核心概念：切点、通知、织入
- **JDK 代理 vs CGLIB** — JDK 代理是接口运行时创建，只能代理接口方法；CGLIB 是编译时生成子类代理，可代理类但无法代理 final 方法/类
- **Spring AOP vs AspectJ** — Spring AOP 运行时增强，AspectJ 编译时/类加载时增强；AspectJ 支持更多切入点类型，性能更优

### 6.3 事务
- **@Transactional 失效场景** — 同一个类内部方法调用（不走代理）；非 public 方法；rollbackFor 配置不当；异常被 catch 吞掉； propagation 配置不当（REQUIRES_NEW）；父子事务问题
- **事务传播行为** — REQUIRED（默认，加入当前事务）、REQUIRES_NEW（挂起当前事务新建独立事务）、NESTED（嵌套事务，Savepoint）、SUPPORTS/MANDATORY/NOT_SUPPORTED/NEVER

### 6.4 循环依赖
- **Spring 三级缓存解决循环依赖** — 一级缓存存成品 Bean、二级缓存存早期暴露对象（提前暴露，解决循环依赖）、三级缓存存 ObjectFactory（解决 AOP 代理对象重复生成问题）。结合时二级缓存存代理对象
- **@Lazy 如何解决循环依赖** — 注入代理对象而非真实 Bean，延迟初始化，打破循环调用链

### 6.5 其他
- **@Autowired vs @Resource** — @Autowired 按类型匹配，@Resource 按名称匹配；多实现时 @Qualifier 指定
- **Bean 作用域** — singleton（默认）、prototype（每次创建）、request、session 等。prototype 不纳入 Spring 生命周期管理
- **Spring Boot 自动配置原理** — @EnableAutoConfiguration + META-INF/spring.factories 文件，加载 AutoConfigurationImportSelector

---

## 7. 分布式系统

### 7.1 分布式锁
- **Redis 分布式锁** — `SETNX + EX` 原子加锁；Lua 脚本原子释放；看门狗自动续期。推荐 Redisson
- **ZooKeeper 分布式锁** — 基于临时顺序节点，判断是否为最小节点，是则获锁；否则在前一节点注册删除事件监听（Watcher），避免无效自旋
- **Redis vs ZooKeeper 分布式锁** — Redis 性能高（万级 QPS）但可靠性略差（AP）；ZooKeeper 可靠性高（CP）但性能较低（千级 QPS）

### 7.2 分布式 ID
- **常见方案对比** — UUID（无序、存储开销大）、数据库自增（依赖 DB 单点）、Redis 自增（高性能但依赖 Redis）、Snowflake（时间戳+机器ID+序列号，趋势递增但依赖时钟）
- **Snowflake 的时钟回拨处理** — 拒绝服务、等待回拨恢复、使用zookeeper 分配 workerID 并记录时间戳

### 7.3 分布式事务
- **CAP 理论** — C（一致性）、A（可用性）、P（分区容错性）。分布式系统 P 必须满足，只能在 C 和 A 之间权衡
- **BASE 理论** — Basically Available（基本可用）、Soft State（软状态，允许中间态）、Eventually Consistent（最终一致性）
- **2PC/3PC** — 两阶段提交：Prepare → Commit；缺点：同步阻塞、单点故障、数据不一致。3PC 增加预提交阶段改善
- **TCC 补偿事务** — Try（预留资源）、Confirm（确认执行）、Cancel（取消回滚）。需要业务实现 try/confirm/cancel，空回滚、悬挂问题需处理

### 7.4 一致性hash
- **原理** — 将 hash 空间组织成环（0 ~ 2^32），服务器映射到环上，数据 key 顺时针找最近服务器。虚拟节点解决负载不均

---

## 8. 系统设计

### 8.1 高并发基础
- **水平扩展 vs 垂直扩展** — 水平：增加机器数量；垂直：增加单机硬件资源（CPU/内存/磁盘）
- **线程池调优** — CPU 密集型：corePoolSize = CPU核数 + 1；IO 密集型：corePoolSize = CPU核数 × CPU利用率 × (1 + 平均等待时间/平均工作时间)
- **读写分离** — 写主库，读从库。解决思路：主从延迟（强制读主库、延迟感知）、数据一致性

### 8.2 消息队列
- **Kafka vs RocketMQ vs RabbitMQ** — Kafka（高吞吐日志系统）、RocketMQ（金融级可靠）、RabbitMQ（中小型，Erlang 实现）。选型看吞吐量、可靠性、延迟需求
- **顺序消息** — Kafka 单 Partition 内有序；RocketMQ 支持消息顺序；需要生产者、消费者、Broker 三者共同保证
- **消息丢失场景** — 生产者发送失败、Broker 刷盘未完成、消费者自动提交 offset。解决：同步发送 + ACK、重试、幂等消费者

### 8.3 场景设计高频题
- **秒杀系统设计** — 静态资源 CDN、请求分层（网关→服务→数据）、库存预扣减（Redis DECR）、消息队列异步下单、限流熔断
- **短链系统设计** — 哈希算法（冲突处理）或自增 ID + base62 编码，重定向302 vs 307
- **分布式 Session** — 黏性 Session（同一用户路由到同一机器）、Session 集中存储（Redis）、JWT 无状态

---

## 9. 网络与操作系统

### 9.1 计算机网络
- **TCP 三次握手** — SYN → SYN+ACK → ACK。为什么要三次：同步初始序列号，确认双方收发能力都正常
- **TCP 四次挥手** — FIN → ACK → FIN → ACK。为什么要四次：被动关闭方需完成数据发送才能发 FIN
- **TCP 滑动窗口** — 解决可靠传输与效率矛盾。发送方维持发送窗口，接收方反馈 ACK 和可用窗口大小
- **HTTP/HTTPS** — HTTPS = HTTP + TLS。TLS 1.2/1.3 三次握手：TCP → TLS ClientHello → TLS ServerHello/Certificate/ServerKeyExchange → TLS ClientKeyExchange/ChangeCipherSpec → TLS Finished

### 9.2 操作系统
- **进程 vs 线程** — 进程是资源分配基本单位，线程是CPU调度基本单位。线程共享进程地址空间，通信效率高
- **进程间通信方式** — 管道、消息队列、共享内存、信号量、Socket
- **页面置换算法** — LRU（最近最少使用）、LFU（最不经常使用）、FIFO、Clock
