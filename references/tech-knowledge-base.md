# 技术知识库

面试中可能涉及的技术领域及考察要点。面试官应根据候选人的简历和回答动态选择相关领域深入追问。

---

## 1. Java 基础

### 1.1 核心特性
- 封装、继承、多态的实现原理
- 抽象类与接口的区别和应用场景
- 异常处理机制（checked vs unchecked）

### 1.2 集合框架
- ArrayList vs LinkedList vs Vector（线程安全、扩容机制、使用场景）
- HashMap 底层结构（JDK 1.7 数组+链表 → JDK 1.8 数组+链表+红黑树）
- ConcurrentHashMap 分段锁原理（JDK 1.7）vs CAS+synchronized（JDK 1.8）
- HashSet 底层依赖 HashMap 的原理
- CopyOnWriteArrayList 的适用场景

### 1.3 String/StringBuilder/StringBuffer
- String 为什么不可变（内存、缓存、安全性）
- StringBuilder vs StringBuffer（线程安全、性能）

### 1.4 泛型
- 泛型擦除机制及类型擦除带来的问题
- PECS 原则（Producer Extends, Consumer Super）

---

## 2. 并发编程

### 2.1 线程基础
- 线程的创建方式（Thread、Runnable、Callable、线程池）
- 线程状态转换
- 守护线程 vs 用户线程

### 2.2 Synchronized
- 锁的膨胀过程（无锁 → 偏向锁 → 轻量级锁 → 重量级锁）
- 对象头结构（Mark Word）
- 偏向锁的撤销条件

### 2.3 volatile
- 可见性保证（缓存行、MESI 协议）
- 禁止指令重排序（JMM 规则）
- 不能保证原子性

### 2.4 JUC 并发工具
- ReentrantLock vs synchronized（可中断、可超时、公平锁）
- CountDownLatch vs CyclicBarrier vs Semaphore
- ConcurrentHashMap 原理
- ThreadLocal 内存泄漏原因

### 2.5 线程池
- 线程池参数（corePoolSize、maxPoolSize、queue、rejectPolicy）
- 线程池分类（Fixed、Cached、Scheduled、Single）
- 线程池拒绝策略

### 2.6 AQS
- AbstractQueuedSynchronizer 工作原理
- 独占模式 vs 共享模式

---

## 3. JVM

### 3.1 内存结构
- 堆（新生代 Eden/Survivor、老年代）/ 栈/本地方法栈/方法区/程序计数器
- OOM 场景及原因

### 3.2 垃圾回收
- 垃圾回收算法（标记-清除、复制、标记-整理）
- 分代收集理论
- 常见垃圾收集器（Serial、ParNew、Parallel Scavenge、CMS、G1、ZGC）
- CMS 收集器工作流程（初始标记、并发标记、重新标记、并发清除）
- G1 收集器（三色标记、Remembered Set）

### 3.3 类加载
- 类加载过程（加载→验证→准备→解析→初始化）
- 双亲委派模型及原因
- 破坏双亲委派的场景

### 3.4 性能调优
- Arthas / jstack / jmap / jstat 使用
- 常见 OOM 类型分析
- JIT 编译优化（逃逸分析、锁消除、标量替换）

---

## 4. MySQL

### 4.1 索引结构
- B+ Tree 原理及与 B-Tree 的区别
- 聚簇索引 vs 非聚簇索引
- 主键索引 vs 唯一索引 vs 普通索引
- 最左前缀原则

### 4.2 事务
- ACID 特性及实现原理
- 隔离级别（Read Uncommitted / Read Committed / Repeatable Read / Serializable）
- MVCC 原理（Read View、undo log 版本链）
- 脏读、不可重复读、幻读

### 4.3 锁
- 行锁 vs 表锁
- 排他锁 vs 共享锁
- Next-Key Lock（记录锁+间隙锁）
- 死锁及避免策略

### 4.4 SQL 优化
- EXPLAIN 分析
- 索引失效场景
- JOIN 原理（Nested Loop / Hash Join / Sort Merge Join）
- 分页优化（延迟关联、游标分页）

### 4.5 主从复制
- 异步复制原理
- 半同步复制
- Binlog 格式（Statement / Row / Mixed）

---

## 5. Redis

### 5.1 数据结构
- String / List / Hash / Set / ZSet 底层实现
- 压缩列表（ziplist）vs 快速列表（quicklist）
- 跳表（skiplist）原理

### 5.2 持久化
- RDB（定时快照，fork 阻塞风险）
- AOF（命令追加，fsync 策略）
- 混合持久化

### 5.3 缓存
- 缓存雪崩（大量 key 同时过期 / Redis 宕机）
- 缓存穿透（恶意请求不存在的数据，布隆过滤器）
- 缓存击穿（热点 key 过期，互斥锁、永不过期+异步重建）
- 缓存一致性问题（Cache Aside / Read Through / Write Through）

### 5.4 分布式
- Redis Cluster 槽分片
- Codis / Twemproxy 代理分片
- 分布式锁（SETNX + Lua / Redisson）

### 5.5 高可用
- 主从复制 + 哨兵模式
- Cluster 集群模式

---

## 6. Spring

### 6.1 IoC / DI
- 依赖注入方式（Setter / Constructor / Field）
- Bean 生命周期
- 循环依赖解决（三级缓存）

### 6.2 AOP
- 动态代理（JDK 动态代理 / CGLIB）
- 代理创建时机（JDK 代理是接口运行时创建，CGLIB 是子类编译时）
- 切面优先级

### 6.3 事务
- 编程式事务 vs 声明式事务（@Transactional）
- 事务传播行为（REQUIRED / REQUIRES_NEW / NESTED 等）
- 事务失效场景（自调用、私有方法、异常被吞）

### 6.4 Spring Boot
- 自动配置原理（@EnableAutoConfiguration / META-INF/spring.factories）
- Starter 机制

---

## 7. 微服务

### 7.1 注册中心
- Eureka / Nacos / Zookeeper 原理对比
- 心跳机制、服务续约

### 7.2 网关
- Gateway / Zuul 路由、过滤器
- 限流算法（计数器 / 滑动窗口 / 令牌桶 / 漏桶）
- 熔断降级（Hystrix / Sentinel / Resilience4j）

### 7.3 负载均衡
- Ribbon / LoadBalancer 负载均衡策略
- Feign 远程调用原理

### 7.4 分布式事务
- CAP 定理 / BASE 理论
- 2PC / 3PC（TCC / Saga / 本地消息表）

---

## 8. 系统设计

### 8.1 高并发
- 水平扩展 vs 垂直扩展
- 读写分离、分库分表
- 消息队列削峰（Kafka / RocketMQ）
- 线程池调优

### 8.2 分布式 ID
- UUID / Snowflake / Leaf / 美团 Leaf
- 时钟回拨处理

### 8.3 一致性
- 强一致性 vs 最终一致性
- Raft 协议（Paxos 变种）
- 分布式锁实现方案对比

### 8.4 场景设计
- 短链系统设计
- 秒杀系统设计
- 评论系统设计
- Feed 流系统设计

---

## 9. 中间件

### 9.1 消息队列
- Kafka / RocketMQ / RabbitMQ 选型对比
- 顺序消息、消息丢失、消息重复消费
- 事务消息

### 9.2 Nginx
- 反向代理 / 负载均衡
- 动静分离

### 9.3 Elasticsearch
- 倒排索引原理
- 分片、副本机制

---

## 10. 计算机基础

### 10.1 网络
- TCP 三次握手、四次挥手
- HTTP/HTTPS
- TCP 滑动窗口、拥塞控制

### 10.2 操作系统
- 进程 vs 线程
- 进程间通信方式
- 页面置换算法
