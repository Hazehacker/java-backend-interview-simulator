# Java 后端实操题集

> 面试最后环节可选的编码题。难度从简单到中等，覆盖并发编程、数据库、设计模式、框架原理、系统设计等方向。
>
> 面试官可根据候选人水平和剩余时间灵活选题，每题标注了难度和预估用时。

---

## 目录

- [1. 并发编程](#1-并发编程)
  - [1.1 生产者消费者模式](#11-生产者消费者模式)
  - [1.2 线程安全的单例模式](#12-线程安全的单例模式)
  - [1.3 实现一个简易线程池](#13-实现一个简易线程池)
  - [1.4 ConcurrentHashMap 核心操作](#14-concurrenthashmap-核心操作)
- [2. 数据库](#2-数据库)
  - [2.1 SQL 题：连续登录用户](#21-sql-题连续登录用户)
  - [2.2 索引设计题](#22-索引设计题)
  - [2.3 事务隔离级别实现](#23-事务隔离级别实现)
- [3. 设计模式](#3-设计模式)
  - [3.1 工厂模式实现](#31-工厂模式实现)
  - [3.2 策略模式实现](#32-策略模式实现)
- [4. 框架原理](#4-框架原理)
  - [4.1 简版 IOC 容器](#41-简版-ioc-容器)
  - [4.2 拦截器链实现](#42-拦截器链实现)
- [5. 系统设计](#5-系统设计)
  - [5.1 限流器实现](#51-限流器实现)
  - [5.2 分布式 ID 生成器](#52-分布式-id-生成器)
- [6. 数据结构](#6-数据结构)
  - [6.1 LRU 缓存实现](#61-lru-缓存实现)
  - [6.2 跳表实现](#62-跳表实现)

---

## 1. 并发编程

### 1.1 生产者消费者模式

**难度**：中等 | **预估用时**：10-15 分钟 | **高频指数**：⭐⭐⭐⭐⭐

**题目**：实现一个标准的生产者-消费者模式，使用阻塞队列

```java
import java.util.concurrent.*;

public class ProducerConsumerDemo {
    
    public static void main(String[] args) {
        // 使用有界阻塞队列，容量为 10
        BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(10);
        
        // 启动 2 个生产者
        for (int i = 0; i < 2; i++) {
            final int producerId = i;
            new Thread(() -> {
                try {
                    int value = 0;
                    while (true) {
                        // TODO: 生产者逻辑
                        // 生产值并放入队列
                        // 放入成功后打印 "Producer X produced: Y"
                    }
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }, "Producer-" + producerId).start();
        }
        
        // 启动 3 个消费者
        for (int i = 0; i < 3; i++) {
            final int consumerId = i;
            new Thread(() -> {
                try {
                    while (true) {
                        // TODO: 消费者逻辑
                        // 从队列取出值
                        // 消费后打印 "Consumer X consumed: Y"
                    }
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }, "Consumer-" + consumerId).start();
        }
    }
}
```

**参考解答**：

```java
import java.util.concurrent.*;

public class ProducerConsumerDemo {
    
    public static void main(String[] args) {
        BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(10);
        
        // 生产者
        for (int i = 0; i < 2; i++) {
            final int producerId = i;
            new Thread(() -> {
                int value = 0;
                try {
                    while (true) {
                        Thread.sleep((producerId + 1) * 100); // 模拟不同生产速度
                        queue.put(value);  // 阻塞直到队列有空位
                        System.out.println("Producer " + producerId + " produced: " + value);
                        value++;
                    }
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }, "Producer-" + producerId).start();
        }
        
        // 消费者
        for (int i = 0; i < 3; i++) {
            final int consumerId = i;
            new Thread(() -> {
                try {
                    while (true) {
                        int value = queue.take();  // 阻塞直到队列有元素
                        System.out.println("Consumer " + consumerId + " consumed: " + value);
                        Thread.sleep(150);  // 模拟消费处理时间
                    }
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }, "Consumer-" + consumerId).start();
        }
    }
}
```

**追问方向**：
- ArrayBlockingQueue vs LinkedBlockingQueue 的区别？（数组 vs 链表，有界 vs 无界）
- 阻塞队列的 put 和 offer 有什么区别？（后者不阻塞，队列满时抛异常）
- 如何实现一个支持优先级的阻塞队列？

---

### 1.2 线程安全的单例模式

**难度**：简单 | **预估用时**：5-8 分钟 | **高频指数**：⭐⭐⭐⭐⭐

**题目**：写出线程安全的单例模式实现（至少两种）

**参考解答**：

```java
// 方案一：饿汉式（线程安全，但启动即创建，可能浪费资源）
class Singleton1 {
    private static final Singleton1 INSTANCE = new Singleton1();
    
    private Singleton1() {}
    
    public static Singleton1 getInstance() {
        return INSTANCE;
    }
}

// 方案二：懒汉式 + synchronized（线程安全，但性能差）
class Singleton2 {
    private static Singleton2 INSTANCE;
    
    private Singleton2() {}
    
    public static synchronized Singleton2 getInstance() {
        if (INSTANCE == null) {
            INSTANCE = new Singleton2();
        }
        return INSTANCE;
    }
}

// 方案三：双重检查锁定（DCL，推荐）
class Singleton3 {
    // volatile 防止指令重排序
    private static volatile Singleton3 INSTANCE;
    
    private Singleton3() {}
    
    public static Singleton3 getInstance() {
        if (INSTANCE == null) {                 // 第一次检查
            synchronized (Singleton3.class) {
                if (INSTANCE == null) {         // 第二次检查
                    INSTANCE = new Singleton3();
                }
            }
        }
        return INSTANCE;
    }
}

// 方案四：静态内部类（推荐，更简洁）
class Singleton4 {
    private Singleton4() {}
    
    private static class Holder {
        private static final Singleton4 INSTANCE = new Singleton4();
    }
    
    public static Singleton4 getInstance() {
        return Holder.INSTANCE;
    }
}

// 方案五：枚举（Effective Java 作者推荐，最简洁）
enum Singleton5 {
    INSTANCE;
    
    public void doSomething() {
        // 业务方法
    }
}
```

**追问方向**：
- DCL 中为什么要用 volatile？（防止指令重排序）
- 饿汉式和懒汉式的区别？（资源浪费 vs 延迟初始化）
- 枚举单例为什么是线程安全的？（Java 规范保证）

---

### 1.3 实现一个简易线程池

**难度**：中等偏难 | **预估用时**：15-20 分钟 | **高频指数**：⭐⭐⭐⭐

**题目**：实现一个简易的线程池，包含核心线程数、最大线程数、任务队列

```java
import java.util.concurrent.*;
import java.util.*;

public class SimpleThreadPool {
    
    private final int corePoolSize;
    private final int maxPoolSize;
    private final long keepAliveTime;
    private final TimeUnit unit;
    private final BlockingQueue<Runnable> workQueue;
    
    public SimpleThreadPool(int corePoolSize, int maxPoolSize, 
                           long keepAliveTime, TimeUnit unit,
                           BlockingQueue<Runnable> workQueue) {
        this.corePoolSize = corePoolSize;
        this.maxPoolSize = maxPoolSize;
        this.keepAliveTime = keepAliveTime;
        this.unit = unit;
        this.workQueue = workQueue;
    }
    
    public void execute(Runnable task) {
        // TODO: 实现逻辑
    }
    
    public void shutdown() {
        // TODO: 关闭线程池
    }
}
```

**参考解答**：

```java
import java.util.concurrent.*;
import java.util.*;

public class SimpleThreadPool {
    
    private final int corePoolSize;
    private final int maxPoolSize;
    private final long keepAliveTime;
    private final TimeUnit unit;
    private final BlockingQueue<Runnable> workQueue;
    private final Set<Worker> workers = new HashSet<>();
    
    public SimpleThreadPool(int corePoolSize, int maxPoolSize, 
                           long keepAliveTime, TimeUnit unit,
                           BlockingQueue<Runnable> workQueue) {
        this.corePoolSize = corePoolSize;
        this.maxPoolSize = maxPoolSize;
        this.keepAliveTime = keepAliveTime;
        this.unit = unit;
        this.workQueue = workQueue;
    }
    
    public void execute(Runnable task) {
        synchronized (workers) {
            // 活跃线程数
            int activeCount = workers.size();
            
            // 情况1：核心线程未满，创建核心线程
            if (activeCount < corePoolSize) {
                Worker worker = new Worker(task, true);
                workers.add(worker);
                worker.thread.start();
            } 
            // 情况2：核心线程满，队列未满，加入队列
            else if (workQueue.offer(task)) {
                // 成功入队
            }
            // 情况3：队列满，创建临时线程
            else if (activeCount < maxPoolSize) {
                Worker worker = new Worker(task, false);
                workers.add(worker);
                worker.thread.start();
            }
            // 情况4：达到最大线程数，拒绝任务
            else {
                throw new RejectedExecutionException("Thread pool is full");
            }
        }
    }
    
    public void shutdown() {
        synchronized (workers) {
            for (Worker worker : workers) {
                worker.shutdown();
            }
        }
    }
    
    private class Worker implements Runnable {
        final Thread thread;
        final Runnable firstTask;
        boolean isCore;
        
        Worker(Runnable firstTask, boolean isCore) {
            this.firstTask = firstTask;
            this.isCore = isCore;
            this.thread = new Thread(this);
        }
        
        public void run() {
            Runnable task = firstTask;
            while (task != null || (task = getTask()) != null) {
                try {
                    task.run();
                } finally {
                    task = null;
                }
            }
            cleanup();
        }
        
        Runnable getTask() {
            try {
                // 核心线程等待队列，空则阻塞
                // 非核心线程超时等待
                if (isCore) {
                    return workQueue.take();
                } else {
                    return workQueue.poll(keepAliveTime, unit);
                }
            } catch (InterruptedException e) {
                return null;
            }
        }
        
        void shutdown() {
            thread.interrupt();
        }
        
        void cleanup() {
            synchronized (workers) {
                workers.remove(this);
            }
        }
    }
}
```

**追问方向**：
- 线程池的拒绝策略有哪些？（Abort/CallerRunsPolicy/DiscardPolicy/DiscardOldestPolicy）
- 核心线程会超时被回收吗？（默认不会，需要 allowCoreThreadTimeOut）
- 如何实现线程池的监控？（activeCount、queue size 等）

---

### 1.4 ConcurrentHashMap 核心操作

**难度**：中等 | **预估用时**：10-15 分钟 | **高频指数**：⭐⭐⭐⭐⭐

**题目**：实现一个简易的分段锁 HashMap，理解 ConcurrentHashMap 的分段思想

```java
public class SegmentMap<K, V> {
    
    // 分段数
    private static final int SEGMENTS = 16;
    
    // 分段数组
    private final Segment<K, V>[] segments;
    
    public SegmentMap() {
        segments = new Segment[SEGMENTS];
        for (int i = 0; i < SEGMENTS; i++) {
            segments[i] = new Segment<>();
        }
    }
    
    // 计算 key 所在的分段索引
    private int segmentIndex(K key) {
        return Math.abs(key.hashCode() % SEGMENTS);
    }
    
    public V get(K key) {
        // TODO: 实现 get
        return null;
    }
    
    public V put(K key, V value) {
        // TODO: 实现 put
        return null;
    }
    
    // 分段内部类
    private static class Segment<K, V> {
        // 简单用 synchronized 模拟分段锁
        private final Object lock = new Object();
        private Map<K, V> map = new HashMap<>();
        
        V get(K key) {
            synchronized (lock) {
                return map.get(key);
            }
        }
        
        V put(K key, V value) {
            synchronized (lock) {
                return map.put(key, value);
            }
        }
    }
}
```

**参考解答**：

```java
import java.util.*;

public class SegmentMap<K, V> {
    
    private static final int SEGMENTS = 16;
    private final Segment<K, V>[] segments;
    
    public SegmentMap() {
        segments = new Segment[SEGMENTS];
        for (int i = 0; i < SEGMENTS; i++) {
            segments[i] = new Segment<>();
        }
    }
    
    private int segmentIndex(K key) {
        return Math.abs(key.hashCode() % SEGMENTS);
    }
    
    public V get(K key) {
        int index = segmentIndex(key);
        return segments[index].get(key);
    }
    
    public V put(K key, V value) {
        int index = segmentIndex(key);
        return segments[index].put(key, value);
    }
    
    // JDK 8 ConcurrentHashMap 的核心原理（简化的分段锁）
    // 不同分段可以并发访问，提高吞吐量
    
    private static class Segment<K, V> {
        private final Object lock = new Object();
        private volatile Map<K, V> map = new HashMap<>();
        
        V get(K key) {
            synchronized (lock) {
                return map.get(key);
            }
        }
        
        V put(K key, V value) {
            synchronized (lock) {
                return map.put(key, value);
            }
        }
        
        // JDK 8 ConcurrentHashMap 的改进：
        // 1. 取消分段锁，改用 Node + CAS + synchronized
        // 2. 锁的粒度更细，只锁单个桶
        // 3. 使用红黑树优化链表过长的情况
    }
    
    public static void main(String[] args) throws InterruptedException {
        SegmentMap<String, Integer> map = new SegmentMap<>();
        
        // 测试并发put
        List<Thread> threads = new ArrayList<>();
        for (int i = 0; i < 100; i++) {
            final int num = i;
            threads.add(new Thread(() -> {
                map.put("key" + (num % 10), num);
            }));
        }
        
        for (Thread t : threads) {
            t.start();
        }
        
        for (Thread t : threads) {
            t.join();
        }
        
        // 输出每个 key 的值（最后写入的）
        for (int i = 0; i < 10; i++) {
            System.out.println("key" + i + " = " + map.get("key" + i));
        }
    }
}
```

**追问方向**：
- JDK 7 和 JDK 8 的 ConcurrentHashMap 有什么区别？（分段锁 vs CAS + synchronized）
- 为什么要用 volatile 修饰 map？（保证可见性）
- CAS 的 ABA 问题如何解决？（版本号/时间戳）

---

## 2. 数据库

### 2.1 SQL 题：连续登录用户

**难度**：中等 | **预估用时**：10-15 分钟 | **高频指数**：⭐⭐⭐⭐

**题目**：有一个用户登录表 `user_logins(user_id, login_date)`，找出连续登录天数>=3天的用户

```sql
-- 表结构
CREATE TABLE user_logins (
    user_id INT,
    login_date DATE
);

-- 示例数据
-- user_id | login_date
-- 1       | 2024-01-01
-- 1       | 2024-01-02
-- 1       | 2024-01-03  <- 连续3天
-- 1       | 2024-01-05  <- 断开
-- 2       | 2024-01-01
-- 2       | 2024-01-02
-- 2       | 2024-01-04  <- 只有2天连续

-- 请写出 SQL
```

**参考解答**：

```sql
-- 解法1：窗口函数 + 分组
WITH ranked AS (
    SELECT 
        user_id,
        login_date,
        DATE_SUB(login_date, INTERVAL ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date) DAY) AS grp
    FROM user_logins
),
consecutive AS (
    SELECT 
        user_id,
        COUNT(*) AS consecutive_days
    FROM ranked
    GROUP BY user_id, grp
    HAVING COUNT(*) >= 3
)
SELECT DISTINCT user_id FROM consecutive;

-- 核心思路：
-- 1. 按日期排序，用日期减去行号，相同的 grp 表示连续
-- 2. 2024-01-01 - 1 = 2023-12-31
--    2024-01-02 - 2 = 2023-12-31  <- 同一 grp
--    2024-01-03 - 3 = 2023-12-31  <- 同一 grp
--    2024-01-05 - 4 = 2024-01-01  <- 断开，grp 变化
```

**追问方向**：
- 这个解法的时间复杂度是多少？（O(n log n)）
- 如果表很大（1亿行），如何优化？（分区、索引）
- 如何找出每个用户的最长连续登录天数？

---

### 2.2 索引设计题

**难度**：中等 | **预估用时**：8-10 分钟 | **高频指数**：⭐⭐⭐⭐⭐

**题目**：设计一个论坛系统的索引，包含以下查询场景

**场景**：
1. 按时间范围查询帖子 `WHERE create_time BETWEEN ? AND ?`
2. 按板块查询帖子 `WHERE board_id = ?`
3. 按用户查询帖子 `WHERE user_id = ?`
4. 查询某板块下某用户的帖子 `WHERE board_id = ? AND user_id = ?`

**请设计索引并说明理由**

**参考解答**：

```sql
-- 基础表结构
CREATE TABLE posts (
    id BIGINT PRIMARY KEY,
    board_id INT NOT NULL,
    user_id INT NOT NULL,
    title VARCHAR(200),
    content TEXT,
    create_time DATETIME,
    update_time DATETIME
);

-- 索引设计：

-- 1. 单独索引
-- idx_board_id(board_id)           -- 场景2
-- idx_user_id(user_id)             -- 场景3
-- idx_create_time(create_time)     -- 场景1

-- 2. 联合索引（覆盖高频组合查询）
-- idx_board_user(board_id, user_id) -- 场景4
-- idx_board_time(board_id, create_time) -- 场景1+2（范围放最后）

-- 为什么不把全部字段放一个联合索引？
-- 1. 索引不是越多越好，更新成本高
-- 2. 区分度大的字段放前面
-- 3. 范围查询字段放最后（MySQL 索引最左前缀原则）

-- 最优设计（根据查询频率调整）：
-- 高频：查询某板块的帖子 -> (board_id, create_time)
-- 高频：查询某用户的帖子 -> (user_id, create_time)
-- 中频：查询某板块某用户的帖子 -> (board_id, user_id)
-- 低频：按时间范围 -> create_time

-- 覆盖索引：如果经常需要查询 title，可以建立覆盖索引
-- (board_id, create_time) INCLUDE (title)  -- SQL Server
-- 或者 (board_id, create_time, title)       -- MySQL（但会增加索引大小）
```

**追问方向**：
- 什么是索引覆盖？（索引包含所有要查询的字段，无需回表）
- 什么情况下索引会失效？（左模糊、函数、隐式转换）
- 联合索引的最左前缀原则是什么？

---

### 2.3 事务隔离级别实现

**难度**：难 | **预估用时**：15-20 分钟 | **高频指数**：⭐⭐⭐

**题目**：说明 MySQL 四种隔离级别及其实现原理

**参考解答**：

```java
/**
 * MySQL InnoDB 事务隔离级别：
 * 
 * 1. READ UNCOMMITTED（读未提交）
 *    - 最低级别，允许脏读
 *    - 实现：不加锁，直接读取最新数据
 * 
 * 2. READ COMMITTED（读已提交）
 *    - 避免脏读，可能出现不可重复读
 *    - 实现：读取时加共享锁，读取后释放
 *    - 每次读取都重新获取最新数据
 * 
 * 3. REPEATABLE READ（可重复读，MySQL 默认）
 *    - 避免脏读、不可重复读，可能出现幻读
 *    - 实现：
 *      - 读取时加共享锁，事务结束才释放
 *      - 使用 MVCC 快照读，不加锁的 SELECT 读取快照
 * 
 * 4. SERIALIZABLE（串行化）
 *    - 最高级别，完全串行执行
 *    - 实现：所有读都加排他锁
 */

public class TransactionIsolationDemo {
    
    // 模拟 MVCC 的关键概念
    static class MVCC {
        // 每行数据有隐藏的两个字段：
        // 1. DB_TRX_ID：最后修改的事务ID
        // 2. DB_ROLL_PTR：指向undo log的指针
        
        // SELECT 读取版本：
        // - READ COMMITTED：总是读取最新快照
        // - REPEATABLE READ：读取事务开始时的快照
        
        // 示例：
        void demonstrateReadCommitted() {
            // 事务A：插入一条数据，提交
            // transaction_a.execute("INSERT INTO users VALUES (1, 'Alice')");
            // transaction_a.commit();
            
            // 事务B：读取，可以读到事务A提交的数据
            // transaction_b.execute("SELECT * FROM users"); // 看到 Alice
        }
        
        void demonstrateRepeatableRead() {
            // 事务A：插入一条数据，提交
            // transaction_a.execute("INSERT INTO users VALUES (1, 'Alice')");
            // transaction_a.commit();
            
            // 事务B：开启事务时创建快照，不受其他事务影响
            // transaction_b.execute("SELECT * FROM users"); // 看不到 Alice
            // transaction_b.start(); // 创建快照的时间点
        }
    }
    
    // 间隙锁（Next-Key Lock）解决幻读
    // 在 REPEATABLE READ 下，范围查询会锁定区间，防止幻读
    void demonstrateGapLock() {
        // 假设 users 表有 id 1, 3, 5
        // SELECT * FROM users WHERE id > 2 AND id < 5 FOR UPDATE;
        // 会锁定 (2, 5) 这个区间，即使 id=4 不存在也会被锁定
        // 这样其他事务无法插入 id=4 的记录，避免幻读
    }
}
```

**追问方向**：
- 什么是 MVCC？（多版本并发控制）
- 什么是脏读、不可重复读、幻读？
- InnoDB 是如何解决幻读的？（间隙锁/Next-Key Lock）

---

## 3. 设计模式

### 3.1 工厂模式实现

**难度**：简单 | **预估用时**：5-8 分钟 | **高频指数**：⭐⭐⭐⭐

**题目**：实现一个支付工厂，根据支付方式返回对应的支付处理器

```java
// 支付接口
interface Payment {
    void pay(double amount);
    void refund(double amount);
}

// 支付宝实现
class Alipay implements Payment {
    @Override
    public void pay(double amount) {
        System.out.println("支付宝支付：" + amount);
    }
    
    @Override
    public void refund(double amount) {
        System.out.println("支付宝退款：" + amount);
    }
}

// 微信支付实现
class WechatPay implements Payment {
    @Override
    public void pay(double amount) {
        System.out.println("微信支付：" + amount);
    }
    
    @Override
    public void refund(double amount) {
        System.out.println("微信退款：" + amount);
    }
}

// 工厂类
class PaymentFactory {
    // TODO: 实现 getPayment 方法
    public static Payment getPayment(String type) {
        // 根据 type 返回对应的 Payment 实现
        return null;
    }
}
```

**参考解答**：

```java
// 工厂实现
class PaymentFactory {
    
    private static final Map<String, Payment> PAYMENTS = new HashMap<>();
    
    static {
        PAYMENTS.put("alipay", new Alipay());
        PAYMENTS.put("wechat", new WechatPay());
        // 可以继续添加其他支付方式
    }
    
    public static Payment getPayment(String type) {
        Payment payment = PAYMENTS.get(type.toLowerCase());
        if (payment == null) {
            throw new IllegalArgumentException("不支持的支付方式：" + type);
        }
        return payment;
    }
    
    // 扩展：使用反射，支持通过类名动态创建
    public static Payment getPaymentByClass(String className) {
        try {
            Class<?> clazz = Class.forName(className);
            return (Payment) clazz.getDeclaredConstructor().newInstance();
        } catch (Exception e) {
            throw new RuntimeException("创建支付实例失败", e);
        }
    }
}

// 使用
public class Main {
    public static void main(String[] args) {
        Payment alipay = PaymentFactory.getPayment("alipay");
        alipay.pay(100.0);
        
        Payment wechat = PaymentFactory.getPayment("wechat");
        wechat.pay(200.0);
    }
}
```

**追问方向**：
- 工厂模式和 new 创建对象有什么区别？（解耦、便于扩展、隐藏创建细节）
- 简单工厂、工厂方法、抽象工厂的区别？
- Spring 中哪里用到了工厂模式？（BeanFactory、FactoryBean）

---

### 3.2 策略模式实现

**难度**：简单 | **预估用时**：5-8 分钟 | **高频指数**：⭐⭐⭐⭐

**题目**：实现一个订单折扣计算器，使用策略模式

```java
// 折扣策略接口
interface DiscountStrategy {
    double calculate(double originalPrice);
}

// 原价策略
class NoDiscount implements DiscountStrategy {
    @Override
    public double calculate(double originalPrice) {
        return originalPrice;
    }
}

// 满减策略
class FullDiscount implements DiscountStrategy {
    private double threshold;  // 满多少
    private double reduction;  // 减多少
    
    // TODO: 构造函数和实现
    
    @Override
    public double calculate(double originalPrice) {
        // 满 threshold 减 reduction
        return originalPrice;
    }
}

// 折扣策略
class PercentDiscount implements DiscountStrategy {
    private double percent;  // 折扣率，如 0.8 表示 8 折
    
    // TODO: 构造函数和实现
    
    @Override
    public double calculate(double originalPrice) {
        return originalPrice;
    }
}

// 订单类
class Order {
    private DiscountStrategy strategy;
    
    public void setStrategy(DiscountStrategy strategy) {
        this.strategy = strategy;
    }
    
    public double calculatePrice(double originalPrice) {
        return strategy.calculate(originalPrice);
    }
}
```

**参考解答**：

```java
// 满减策略
class FullDiscount implements DiscountStrategy {
    private double threshold;
    private double reduction;
    
    public FullDiscount(double threshold, double reduction) {
        this.threshold = threshold;
        this.reduction = reduction;
    }
    
    @Override
    public double calculate(double originalPrice) {
        if (originalPrice >= threshold) {
            return originalPrice - reduction;
        }
        return originalPrice;
    }
}

// 折扣策略
class PercentDiscount implements DiscountStrategy {
    private double percent;
    
    public PercentDiscount(double percent) {
        this.percent = percent;
    }
    
    @Override
    public double calculate(double originalPrice) {
        return originalPrice * percent;
    }
}

// 使用
public class Main {
    public static void main(String[] args) {
        Order order = new Order();
        
        // 使用满减策略
        order.setStrategy(new FullDiscount(100, 10));
        System.out.println("满减价格：" + order.calculatePrice(150));  // 140
        
        // 切换为折扣策略
        order.setStrategy(new PercentDiscount(0.8));
        System.out.println("8折价格：" + order.calculatePrice(150));  // 120
        
        // 切换为不打折
        order.setStrategy(new NoDiscount());
        System.out.println("原价：" + order.calculatePrice(150));  // 150
    }
}
```

**追问方向**：
- 策略模式和 if-else 相比有什么优势？（扩展性、可读性、单元测试）
- 策略模式和工厂模式有什么区别？（工厂创建对象，策略使用对象）
- Spring 中哪里用到了策略模式？（Resource、PropertyEditor）

---

## 4. 框架原理

### 4.1 简版 IOC 容器

**难度**：难 | **预估用时**：15-20 分钟 | **高频指数**：⭐⭐⭐

**题目**：实现一个简单的 IOC 容器，支持依赖注入

```java
import java.lang.reflect.*;
import java.util.*;

public class SimpleIOC {
    
    private Map<String, Object> beans = new HashMap<>();
    
    // 注册 bean
    public void register(String name, Object obj) {
        beans.put(name, obj);
    }
    
    // 获取 bean
    public Object getBean(String name) {
        return beans.get(name);
    }
    
    // 扫描并注入依赖
    public void autowire(Object obj) {
        // TODO: 实现自动注入
        // 1. 获取类的所有字段
        // 2. 查找 @Autowired 注解的字段
        // 3. 根据字段类型从容器中查找匹配的 bean
        // 4. 通过反射设置字段值
    }
}
```

**参考解答**：

```java
import java.lang.reflect.*;
import java.util.*;

public class SimpleIOC {
    
    private Map<String, Object> beans = new HashMap<>();
    private Map<Class<?>, String> classToName = new HashMap<>();
    
    public void register(Class<?> clazz, Object obj) {
        String name = clazz.getSimpleName();
        // 类名首字母小写作为 bean 名称
        name = name.substring(0, 1).toLowerCase() + name.substring(1);
        beans.put(name, obj);
        classToName.put(clazz, name);
    }
    
    public Object getBean(Class<?> clazz) {
        String name = classToName.get(clazz);
        return name != null ? beans.get(name) : null;
    }
    
    public Object getBean(String name) {
        return beans.get(name);
    }
    
    // 自动注入
    public void autowire(Object obj) {
        Class<?> clazz = obj.getClass();
        
        for (Field field : clazz.getDeclaredFields()) {
            // 检查 @Autowired 注解
            if (field.isAnnotationPresent(Autowired.class)) {
                Class<?> fieldType = field.getType();
                Object dependency = getBean(fieldType);
                
                if (dependency == null) {
                    throw new RuntimeException("找不到依赖：" + fieldType.getName());
                }
                
                try {
                    field.setAccessible(true);
                    field.set(obj, dependency);
                } catch (IllegalAccessException e) {
                    throw new RuntimeException("注入失败", e);
                }
            }
        }
    }
    
    // 组件扫描（简化版）
    public void scan(String basePackage) throws Exception {
        // 实际项目中会用类加载器扫描 basePackage 下的类
        // 这里简化处理，直接注册示例类
    }
    
    public static void main(String[] args) throws Exception {
        SimpleIOC ioc = new SimpleIOC();
        
        // 注册 bean
        UserService userService = new UserServiceImpl();
        OrderService orderService = new OrderServiceImpl();
        
        ioc.register(UserService.class, userService);
        ioc.register(OrderService.class, orderService);
        
        // 注入依赖
        ioc.autowire(userService);
        
        // 验证注入
        UserService us = (UserService) ioc.getBean(UserService.class);
        us.save();
    }
}

// 自定义注解
@Retention(RetentionPolicy.RUNTIME)
@interface Autowired {}

// 示例接口和实现
interface UserService {
    void save();
}

class UserServiceImpl implements UserService {
    @Autowired
    private OrderService orderService;
    
    @Override
    public void save() {
        System.out.println("UserService.save()");
        System.out.println("依赖注入成功：" + orderService);
    }
}

interface OrderService {}
class OrderServiceImpl implements OrderService {}
```

**追问方向**：
- Spring IOC 的初始化过程？（加载 BeanDefinition、实例化、依赖注入）
- BeanFactory 和 FactoryBean 有什么区别？
- 循环依赖如何检测和处理？（三级缓存）

---

### 4.2 拦截器链实现

**难度**：中等 | **预估用时**：10-15 分钟 | **高频指数**：⭐⭐⭐

**题目**：实现一个拦截器链，模拟 Spring MVC 的 HandlerInterceptor

```java
import java.util.*;

public class InterceptorChain {
    
    private List<Interceptor> interceptors = new ArrayList<>();
    
    // 添加拦截器
    public void addInterceptor(Interceptor interceptor) {
        interceptors.add(interceptor);
    }
    
    // 执行拦截器链
    public boolean execute(Object handler) {
        // TODO: 按顺序执行 preHandle
        // 任何一个返回 false，则中断执行
        
        try {
            // 执行目标处理逻辑
            // handler.handle();
            
            // 逆向执行 postHandle
            for (int i = interceptors.size() - 1; i >= 0; i--) {
                interceptors.get(i).postHandle(handler, null, null, null);
            }
        } finally {
            // 始终执行 afterCompletion
            for (int i = interceptors.size() - 1; i >= 0; i--) {
                interceptors.get(i).afterCompletion(handler, null, null, null);
            }
        }
        
        return true;
    }
}

// 拦截器接口
interface Interceptor {
    boolean preHandle(Object handler, Object request, Object response);
    void postHandle(Object handler, Object request, Object response, Object model);
    void afterCompletion(Object handler, Object request, Object response, Exception ex);
}
```

**参考解答**：

```java
import java.util.*;

public class InterceptorChain {
    
    private List<Interceptor> interceptors = new ArrayList<>();
    
    public void addInterceptor(Interceptor interceptor) {
        interceptors.add(interceptor);
    }
    
    public boolean execute(Object handler) {
        // 逆向存储，以便逆向执行 postHandle
        List<Interceptor> reversed = new ArrayList<>(interceptors);
        Collections.reverse(reversed);
        
        // 按顺序执行 preHandle
        for (Interceptor interceptor : interceptors) {
            if (!interceptor.preHandle(handler, null, null)) {
                // 返回 false，触发 afterCompletion 并中断
                triggerAfterCompletion(handler, reversed);
                return false;
            }
        }
        
        try {
            // 执行目标处理逻辑
            System.out.println("执行 Handler: " + handler);
            
            // 逆向执行 postHandle
            for (Interceptor interceptor : reversed) {
                interceptor.postHandle(handler, null, null, null);
            }
        } catch (Exception e) {
            throw e;
        } finally {
            // 始终执行 afterCompletion
            triggerAfterCompletion(handler, reversed);
        }
        
        return true;
    }
    
    private void triggerAfterCompletion(Object handler, List<Interceptor> interceptors) {
        for (Interceptor interceptor : interceptors) {
            try {
                interceptor.afterCompletion(handler, null, null, null);
            } catch (Exception e) {
                // 记录日志，但不中断
                e.printStackTrace();
            }
        }
    }
}

// 使用示例
class Main {
    public static void main(String[] args) {
        InterceptorChain chain = new InterceptorChain();
        
        // 添加拦截器
        chain.addInterceptor(new AuthInterceptor());
        chain.addInterceptor(new LoggingInterceptor());
        chain.addInterceptor(new TransactionInterceptor());
        
        // 执行
        chain.execute(new UserHandler());
    }
}

// 示例拦截器
class AuthInterceptor implements Interceptor {
    @Override
    public boolean preHandle(Object handler, Object request, Object response) {
        System.out.println("AuthInterceptor.preHandle");
        return true;  // 返回 false 可中断执行
    }
    
    @Override
    public void postHandle(Object handler, Object request, Object response, Object model) {
        System.out.println("AuthInterceptor.postHandle");
    }
    
    @Override
    public void afterCompletion(Object handler, Object request, Object response, Exception ex) {
        System.out.println("AuthInterceptor.afterCompletion");
    }
}

// 模拟 Handler
class UserHandler {}
```

**追问方向**：
- preHandle 返回 false 会怎样？（中断执行，调用 afterCompletion）
- 为什么 postHandle 和 afterCompletion 要逆向执行？（FILO，先执行的后处理）
- Filter 和 Interceptor 的区别？（Filter 是 Servlet 层面的，Interceptor 是框架层面的）

---

## 5. 系统设计

### 5.1 限流器实现

**难度**：中等 | **预估用时**：10-15 分钟 | **高频指数**：⭐⭐⭐⭐⭐

**题目**：实现一个滑动窗口限流器

```java
import java.util.concurrent.*;

public class SlidingWindowRateLimiter {
    
    private final int maxRequests;      // 时间窗口内的最大请求数
    private final long windowSizeMs;    // 时间窗口大小（毫秒）
    private final Queue<Long> requests; // 请求时间戳队列
    
    public SlidingWindowRateLimiter(int maxRequests, long windowSizeMs) {
        this.maxRequests = maxRequests;
        this.windowSizeMs = windowSizeMs;
        this.requests = new LinkedList<>();
    }
    
    // 尝试获取令牌
    public synchronized boolean tryAcquire() {
        long now = System.currentTimeMillis();
        long windowStart = now - windowSizeMs;
        
        // TODO:
        // 1. 移除窗口外的请求记录
        // 2. 检查当前请求数是否已达到上限
        // 3. 如果未达到，记录当前请求并返回 true
        
        return false;
    }
}
```

**参考解答**：

```java
import java.util.*;
import java.util.concurrent.*;

public class SlidingWindowRateLimiter {
    
    private final int maxRequests;
    private final long windowSizeMs;
    private final Queue<Long> requests;
    
    public SlidingWindowRateLimiter(int maxRequests, long windowSizeMs) {
        this.maxRequests = maxRequests;
        this.windowSizeMs = windowSizeMs;
        this.requests = new ConcurrentLinkedQueue<>();
    }
    
    public boolean tryAcquire() {
        long now = System.currentTimeMillis();
        long windowStart = now - windowSizeMs;
        
        // 1. 清理窗口外的请求
        while (true) {
            Long oldest = requests.peek();
            if (oldest == null || oldest > windowStart) {
                break;
            }
            requests.poll();
        }
        
        // 2. 检查是否达到上限
        if (requests.size() >= maxRequests) {
            return false;
        }
        
        // 3. 记录当前请求
        requests.offer(now);
        return true;
    }
    
    // 阻塞直到获取令牌
    public void acquire() throws InterruptedException {
        while (!tryAcquire()) {
            Thread.sleep(10);  // 简单等待，实际应该用条件变量
        }
    }
    
    // 带超时的获取
    public boolean acquire(long timeoutMs) throws InterruptedException {
        long deadline = System.currentTimeMillis() + timeoutMs;
        while (System.currentTimeMillis() < deadline) {
            if (tryAcquire()) {
                return true;
            }
            Thread.sleep(10);
        }
        return false;
    }
    
    // 测试
    public static void main(String[] args) throws InterruptedException {
        SlidingWindowRateLimiter limiter = new SlidingWindowRateLimiter(5, 1000);
        
        for (int i = 0; i < 10; i++) {
            System.out.println("Request " + i + ": " + (limiter.tryAcquire() ? "通过" : "拒绝"));
            Thread.sleep(100);
        }
    }
}
```

**追问方向**：
- 滑动窗口和固定窗口的区别？（更精确的限流）
- 令牌桶和漏桶算法的区别？（令牌桶允许突发，漏桶平滑输出）
- 分布式环境下如何实现限流？（Redis + Lua、滑动窗口计数）

---

### 5.2 分布式 ID 生成器

**难度**：难 | **预估用时**：15-20 分钟 | **高频指数**：⭐⭐⭐⭐

**题目**：设计一个分布式 ID 生成器，要求：趋势递增、不重复、高可用

```java
/**
 * 分布式 ID 格式（64 位）：
 * 
 * | sign | timestamp | workid | sequence |
 * | 1位  |  41位     | 10位   | 12位    |
 * 
 * - sign: 始终为 0，保证正数
 * - timestamp: 从某个 epoch 开始的时间戳（毫秒）
 * - workid: 机器/服务标识（最多 1024 个节点）
 * - sequence: 同一毫秒内的序列号（最多 4096）
 */
public class SnowflakeIdGenerator {
    
    // 2024-01-01 作为 epoch
    private static final long EPOCH = 1704067200000L;
    
    private final long workerId;
    
    public SnowflakeIdGenerator(long workerId) {
        this.workerId = workerId;
    }
    
    public long generate() {
        // TODO: 实现 ID 生成
        return 0;
    }
}
```

**参考解答**：

```java
public class SnowflakeIdGenerator {
    
    // 2024-01-01 作为 epoch
    private static final long EPOCH = 1704067200000L;
    
    // 各部分占用的位数
    private static final long WORKER_ID_BITS = 10L;
    private static final long SEQUENCE_BITS = 12L;
    
    // 各部分的最大值
    private static final long MAX_WORKER_ID = ~(-1L << WORKER_ID_BITS);  // 1023
    private static final long MAX_SEQUENCE = ~(-1L << SEQUENCE_BITS);   // 4095
    
    // 各部分向左移动的位数
    private static final long WORKER_ID_SHIFT = SEQUENCE_BITS;
    private static final long TIMESTAMP_LEFT_SHIFT = SEQUENCE_BITS + WORKER_ID_BITS;
    
    // 时间戳掩码
    private static final long TIMESTAMP_MASK = ~(-1L << 41);
    
    private final long workerId;
    private long sequence = 0L;
    private long lastTimestamp = -1L;
    
    public SnowflakeIdGenerator(long workerId) {
        if (workerId < 0 || workerId > MAX_WORKER_ID) {
            throw new IllegalArgumentException("workerId 超出范围: 0-" + MAX_WORKER_ID);
        }
        this.workerId = workerId;
    }
    
    public synchronized long generate() {
        long timestamp = System.currentTimeMillis();
        
        // 时钟回拨检测
        if (timestamp < lastTimestamp) {
            throw new RuntimeException("时钟回拨，不允许生成 ID");
        }
        
        // 同一毫秒内，序列号递增
        if (timestamp == lastTimestamp) {
            sequence = (sequence + 1) & MAX_SEQUENCE;
            // 序列号用完，等待下一毫秒
            if (sequence == 0) {
                timestamp = waitNextMillis(timestamp);
            }
        } else {
            sequence = 0;
        }
        
        lastTimestamp = timestamp;
        
        // 组装 ID
        return ((timestamp - EPOCH) & TIMESTAMP_MASK) << TIMESTAMP_LEFT_SHIFT
                | (workerId << WORKER_ID_SHIFT)
                | sequence;
    }
    
    private long waitNextMillis(long timestamp) {
        while (timestamp <= lastTimestamp) {
            timestamp = System.currentTimeMillis();
        }
        return timestamp;
    }
    
    // 从 ID 中解析各部分
    public static void parse(long id) {
        System.out.println("ID: " + id);
        System.out.println("Timestamp: " + ((id >> TIMESTAMP_LEFT_SHIFT) & TIMESTAMP_MASK) + EPOCH);
        System.out.println("Worker ID: " + ((id >> WORKER_ID_SHIFT) & MAX_WORKER_ID));
        System.out.println("Sequence: " + (id & MAX_SEQUENCE));
    }
    
    public static void main(String[] args) {
        SnowflakeIdGenerator generator = new SnowflakeIdGenerator(1);
        
        for (int i = 0; i < 5; i++) {
            long id = generator.generate();
            System.out.println("Generated: " + id);
            parse(id);
        }
    }
}
```

**追问方向**：
- Snowflake 的优缺点？（趋势递增、高并发，但依赖时钟）
- 时钟回拨如何处理？（抛出异常、等待、借用未来时间）
- 雪花算法变种？（百度 UidGenerator、滴滴 TinyID）

---

## 6. 数据结构

### 6.1 LRU 缓存实现

**难度**：中等 | **预估用时**：10-15 分钟 | **高频指数**：⭐⭐⭐⭐⭐

**题目**：实现一个 LRU（最近最少使用）缓存

```java
import java.util.*;

public class LRUCache<K, V> {
    
    private final int capacity;
    private final Map<K, V> cache;
    
    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.cache = new HashMap<>();  // TODO: 改为 LinkedHashMap 实现 LRU
    }
    
    public V get(K key) {
        // TODO: 获取并移动到尾部（最近使用）
        return null;
    }
    
    public void put(K key, V value) {
        // TODO: 放入并处理容量超限
    }
}
```

**参考解答**：

```java
import java.util.*;

public class LRUCache<K, V> {
    
    private final int capacity;
    private final LinkedHashMap<K, V> cache;
    
    public LRUCache(int capacity) {
        // LinkedHashMap 的 accessOrder=true 使其成为 LRU
        this.cache = new LinkedHashMap<K, V>(capacity, 0.75f, true) {
            @Override
            protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
                return size() > LRUCache.this.capacity;
            }
        };
        this.capacity = capacity;
    }
    
    public V get(K key) {
        return cache.getOrDefault(key, null);
    }
    
    public void put(K key, V value) {
        cache.put(key, value);
    }
    
    public String toString() {
        return cache.keySet().toString();
    }
    
    public static void main(String[] args) {
        LRUCache<String, Integer> cache = new LRUCache<>(3);
        
        cache.put("A", 1);
        cache.put("B", 2);
        cache.put("C", 3);
        System.out.println("初始: " + cache);  // [A, B, C]
        
        cache.get("A");  // 访问 A
        System.out.println("访问 A: " + cache);  // [B, C, A]（A 移到尾部）
        
        cache.put("D", 4);  // 添加 D，触发删除最老的 B
        System.out.println("添加 D: " + cache);  // [C, A, D]
    }
}
```

**手写实现（不使用 LinkedHashMap）**：

```java
import java.util.*;

public class LRUCache2<K, V> {
    
    private final int capacity;
    private final Map<K, Node<K, V>> cache;
    private final Node<K, V> head;  // 虚拟头节点
    private final Node<K, V> tail;  // 虚拟尾节点
    
    public LRUCache2(int capacity) {
        this.capacity = capacity;
        this.cache = new HashMap<>();
        this.head = new Node<>();
        this.tail = new Node<>();
        head.next = tail;
        tail.prev = head;
    }
    
    public V get(K key) {
        Node<K, V> node = cache.get(key);
        if (node == null) return null;
        moveToTail(node);  // 移到尾部
        return node.value;
    }
    
    public void put(K key, V value) {
        Node<K, V> node = cache.get(key);
        if (node != null) {
            node.value = value;
            moveToTail(node);
        } else {
            Node<K, V> newNode = new Node<>(key, value);
            cache.put(key, newNode);
            addToTail(newNode);
            
            if (cache.size() > capacity) {
                Node<K, V> removed = removeHead();
                cache.remove(removed.key);
            }
        }
    }
    
    private void moveToTail(Node<K, V> node) {
        removeNode(node);
        addToTail(node);
    }
    
    private void addToTail(Node<K, V> node) {
        node.prev = tail.prev;
        node.next = tail;
        tail.prev.next = node;
        tail.prev = node;
    }
    
    private Node<K, V> removeHead() {
        Node<K, V> oldHead = head.next;
        removeNode(oldHead);
        return oldHead;
    }
    
    private void removeNode(Node<K, V> node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }
    
    private static class Node<K, V> {
        K key;
        V value;
        Node<K, V> prev;
        Node<K, V> next;
        
        Node() {}
        Node(K key, V value) {
            this.key = key;
            this.value = value;
        }
    }
}
```

**追问方向**：
- LinkedHashMap 如何实现 LRU？（removeEldestEntry + accessOrder）
- LRU 和 LFU 的区别？（最近最少使用 vs 最不经常使用）
- Redis 中 LRU 和 LFU 是如何实现的？（近似 LRU、LFU 策略）

---

### 6.2 跳表实现

**难度**：难 | **预估用时**：15-20 分钟 | **高频指数**：⭐⭐⭐

**题目**：实现一个简化的跳表（Skip List）

```java
import java.util.*;

public class SkipList<K, V> {
    
    private static final int MAX_LEVEL = 16;
    
    // 跳表节点
    private static class Node<K, V> {
        K key;
        V value;
        Node<K, V>[] forwards;  // 每层的前向指针
        
        Node(K key, V value, int level) {
            this.key = key;
            this.value = value;
            this.forwards = new Node[level];
        }
    }
    
    private Node<K, V> head;
    private int level;
    private final Comparator<K> comparator;
    
    public SkipList(Comparator<K> comparator) {
        this.comparator = comparator;
        this.head = new Node<>(null, null, MAX_LEVEL);
        this.level = 0;
    }
    
    // 查找
    public V get(K key) {
        // TODO: 实现查找
        return null;
    }
    
    // 插入
    public void put(K key, V value) {
        // TODO: 实现插入（随机层高）
    }
    
    // 删除
    public void remove(K key) {
        // TODO: 实现删除
    }
}
```

**参考解答**：

```java
import java.util.*;
import java.util.concurrent.*;

public class SkipList<K, V> {
    
    private static final int MAX_LEVEL = 16;
    private static final double PROBABILITY = 0.5;
    
    private static class Node<K, V> {
        K key;
        V value;
        Node<K, V>[] forwards;
        
        Node(K key, V value, int level) {
            this.key = key;
            this.value = value;
            this.forwards = new Node[level];
        }
    }
    
    private final Node<K, V> head;
    private int level;
    private final Comparator<K> comparator;
    private final Random random;
    
    public SkipList(Comparator<K> comparator) {
        this.comparator = comparator;
        this.head = new Node<>(null, null, MAX_LEVEL);
        this.level = 0;
        this.random = new Random();
    }
    
    public void put(K key, V value) {
        Node<K, V>[] update = new Node[MAX_LEVEL];
        int[] depths = new int[MAX_LEVEL];
        
        Node<K, V> current = head;
        for (int i = level - 1; i >= 0; i--) {
            Node<K, V> next = current.forwards[i];
            while (next != null && compare(key, next.key) > 0) {
                current = next;
                next = current.forwards[i];
            }
            update[i] = current;
            depths[i] = next != null ? depths[i] + 1 : 0;
        }
        
        // 随机层高
        int newLevel = randomLevel();
        
        Node<K, V> newNode = new Node<>(key, value, newLevel);
        for (int i = 0; i < newLevel; i++) {
            newNode.forwards[i] = update[i].forwards[i];
            update[i].forwards[i] = newNode;
        }
        
        if (newLevel > level) {
            level = newLevel;
        }
    }
    
    public V get(K key) {
        Node<K, V> current = head;
        
        for (int i = level - 1; i >= 0; i--) {
            Node<K, V> next = current.forwards[i];
            while (next != null && compare(key, next.key) > 0) {
                current = next;
                next = current.forwards[i];
            }
        }
        
        current = current.forwards[0];
        if (current != null && compare(key, current.key) == 0) {
            return current.value;
        }
        return null;
    }
    
    public void remove(K key) {
        Node<K, V>[] update = new Node[MAX_LEVEL];
        Node<K, V> current = head;
        
        for (int i = level - 1; i >= 0; i--) {
            Node<K, V> next = current.forwards[i];
            while (next != null && compare(key, next.key) > 0) {
                current = next;
                next = current.forwards[i];
            }
            update[i] = current;
        }
        
        current = current.forwards[0];
        if (current != null && compare(key, current.key) == 0) {
            for (int i = 0; i < level; i++) {
                if (update[i].forwards[i] != current) break;
                update[i].forwards[i] = current.forwards[i];
            }
            
            while (level > 0 && head.forwards[level - 1] == null) {
                level--;
            }
        }
    }
    
    private int randomLevel() {
        int level = 1;
        while (level < MAX_LEVEL && random.nextDouble() < PROBABILITY) {
            level++;
        }
        return level;
    }
    
    @SuppressWarnings("unchecked")
    private int compare(K k1, K k2) {
        if (comparator != null) {
            return comparator.compare(k1, k2);
        }
        return ((Comparable<K>) k1).compareTo(k2);
    }
    
    public void print() {
        for (int i = level - 1; i >= 0; i--) {
            System.out.print("Level " + i + ": ");
            Node<K, V> current = head.forwards[i];
            while (current != null) {
                System.out.print("[" + current.key + ":" + current.value + "] ");
                current = current.forwards[i];
            }
            System.out.println();
        }
    }
    
    public static void main(String[] args) {
        SkipList<Integer, String> skipList = new SkipList<>(Comparator.naturalOrder());
        
        skipList.put(3, "C");
        skipList.put(1, "A");
        skipList.put(5, "E");
        skipList.put(2, "B");
        skipList.put(4, "D");
        
        skipList.print();
        
        System.out.println("Get 3: " + skipList.get(3));
        System.out.println("Get 6: " + skipList.get(6));
        
        skipList.remove(3);
        System.out.println("After removing 3:");
        skipList.print();
    }
}
```

**追问方向**：
- 跳表和红黑树的区别？（跳表区间查询更快，实现更简单，高并发友好）
- 跳表的插入时间复杂度？（O(log n)，随机层高）
- Redis 中哪里用到了跳表？（ZSET 有序集合）

---

## 各身份难度参考

| 题目 | 实习 | 应届 | 社招 1-3 年 |
|------|:----:|:----:|:----------:|
| 1.1 生产者消费者 | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| 1.2 单例模式 | ⭐⭐ | ⭐⭐ | ⭐⭐ |
| 1.3 线程池 | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| 1.4 ConcurrentHashMap | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| 2.1 SQL 连续登录 | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| 2.2 索引设计 | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| 2.3 事务隔离级别 | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| 3.1 工厂模式 | ⭐⭐ | ⭐⭐ | ⭐⭐ |
| 3.2 策略模式 | ⭐⭐ | ⭐⭐ | ⭐⭐ |
| 4.1 IOC 容器 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| 4.2 拦截器链 | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| 5.1 限流器 | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| 5.2 分布式 ID | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| 6.1 LRU 缓存 | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| 6.2 跳表 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
