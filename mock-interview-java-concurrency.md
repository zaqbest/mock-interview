# Java SE 并发与多线程 · 模拟面试纪要

| 项目 | 内容 |
|---|---|
| 日期 | 2026-10-27 |
| 候选人定位 | Senior Java（10+ 年经验） |
| 面试主题 | Java SE — Concurrency & Multithreading |
| 形式 | 深挖式（20+ 题），一题一题进行，每题 1–10 分 + 改进点 |
| 当前进度 | Q1–Q12 已完成（含讲解），Q13 待作答 |
| 平均分 | 3.1 / 10（Q1–Q12：6 / 0.5 / 1.5 / 2 / 0 / 1 / 7 / 0 / 4.5 / 5 / 6.5 / 3） |

> 说明：本次会话中 Q2、Q3、Q4、Q5、Q6、Q8 的部分内容是在「不清楚 / 不知道」之后由面试官补充讲解的，评分仍按原始作答记录，但知识点已在本文档中完整补齐。

---

## 目录

1. [进度总览](#1-进度总览)
2. [Q1 volatile vs synchronized](#2-q1--volatile-vs-synchronized)
3. [Q2 happens-before](#3-q2--happens-before)
4. [Q3 final 字段与安全发布](#4-q3--final-字段与安全发布)
5. [Q4 Double-Checked Locking](#5-q4--double-checked-locking)
6. [Q5 synchronized 底层实现](#6-q5--synchronized-底层实现)
7. [Q6 wait/notify 与 Condition](#7-q6--waitnotify-与-condition)
8. [Q7 ReentrantLock vs synchronized](#8-q7--reentrantlock-vs-synchronized)
9. [Q8 AQS 原理](#9-q8--aqs-原理)
10. [Q9 CAS / ABA / LongAdder](#10-q9--cas--aba--longadder)
11. [Q10 ThreadLocal 原理](#11-q10--threadlocal-原理)
12. [Q11 线程池核心参数与执行流程](#12-q11--线程池核心参数与执行流程65--10)
13. [Q12 优雅停机](#13-q12--优雅停机3--10)
14. [附录：K8s 优雅停机实操 + SmartLifecycle 详解](#14-附录k8s-优雅停机实操)
15. [Q13 CompletableFuture](#16-q13--completablefuture00--10)
16. [Q14 死锁](#16-q14--死锁15--10)
17. [Q15 伪共享与缓存行对齐](#16-q15--伪共享缓存行对齐70--10)
18. [Q16 无锁队列与 Disruptor](#16-q16--无锁队列-mpsc-与-disruptor55--10)
19. [Q17 虚拟线程（Virtual Threads）](#17-q17--虚拟线程-virtual-threads-java-2155--10)
20. [Q18 结构化并发与 ScopedValue](#18-q18--结构化并发-structured-concurrency-与-scopedvalue65--10)
21. [Q19 ForkJoinPool 与 parallelStream](#19-q19--forkjoinpool-与-parallelstream-的坑00--10)
22. [Q20 ConcurrentHashMap](#20-q20--concurrenthashmap-原理55--10)
23. [Q21 BlockingQueue 家族对比与选型](#21-q21--blockingqueue-家族对比与选型60--10)
24. [Q22 并发 Bug 排查：jstack、JFR、jcstress、Arthas](#16-q22--并发-bug-排查jstack-jfr-jcstress-arthas00--10)
25. [剩余题库计划（Q23–Q30）](#15-剩余题库计划q23q30)
26. [高频易错清单（复习卡）](#17-高频易错清单复习卡)
27. [下一步建议](#18-下一步建议)

---

## 1. 进度总览

| 题号 | 主题 | 难度 | 得分 | 状态 |
|---|---|---|---|---|
| Q1 | volatile vs synchronized | 中 | **6.0** | ✅ 已讲解 |
| Q2 | happens-before | 中 | **0.5** | ✅ 已讲解 |
| Q3 | final 字段与安全发布 | 难 | **1.5** | ✅ 已讲解 |
| Q4 | Double-Checked Locking | 难 | **2.0** | ✅ 已讲解 |
| Q5 | synchronized 底层（monitor / 锁升级 / JEP 374） | 难 | **0.0** | ✅ 已讲解 |
| Q6 | wait/notify 与 Condition | 中 | **1.0** | ✅ 已讲解 |
| Q7 | ReentrantLock vs synchronized | 中 | **7.0** | ✅ 已讲解 |
| Q8 | AQS 原理 | 难 | **0.0** | ✅ 已讲解 |
| Q9 | CAS / ABA / LongAdder | 难 | **4.5** | ✅ 已讲解 |
| Q10 | ThreadLocal 原理 | 中 | **5.0** | ✅ 已讲解 |
| Q11 | 线程池核心参数与执行流程 | 中 | **6.5** | ✅ 已讲解 |
| Q12 | 优雅停机（含 K8s 实操） | 中 | **3.0** | ✅ 已讲解 |
| Q13 | CompletableFuture（vs Future、thenApply/thenCompose/thenAccept、线程池/shutdown、异常处理） | 难 | **0.0** | ✅ 已讲解（候选人作答全为「不清楚」） |
| Q14 | 死锁（四个条件、预防/避免/检测、jstack 排查、实战规避） | 中 | **1.5** | ✅ 已讲解（候选人部分条件答对，其余「不清楚」） |
| Q15 | 伪共享、缓存行、@Contended、AtomicLong vs LongAdder | 难 | **7.0** | ✅ 已讲解（候选人 AtomicLong/LongAdder 答对，其余补全） |
| Q16 | 无锁队列、MPSC、Disruptor（ForkJoinPool vs TPE、Michael-Scott、SPSC/MPSC、RingBuffer/Sequence/WaitStrategy） | 难 | **5.5** | ✅ 已讲解（候选人不认识 ConcurrentLinkedQueue/Disruptor） |
| Q17 | 虚拟线程（Virtual Threads、Mount/Unmount、Pinning、ScopedValue、适用场景） | 难 | **5.5** | ✅ 已讲解（候选人不了解 Pinning 和 ScopedValue） |
| Q18 | 结构化并发（StructuredTaskScope、ShutdownOnSuccess/Failure、ScopedValue vs ThreadLocal） | 难 | **6.5** | ✅ 已讲解（候选人只答对线程数量/流程控制，其余「不清楚」） |
| Q19 | ForkJoinPool 与 parallelStream（工作窃取、RecursiveTask/Action、commonPool 阻塞陷阱） | 中 | **0.0** | ✅ 已讲解（候选人全为「不清楚」） |
| Q20 | ConcurrentHashMap（1.7 vs 1.8、put 流程、get/扩容/ForwardingNode、size/复合操作） | 难 | **5.5** | ✅ 已讲解（候选人部分答对，其余「不清楚」） |
| Q21 | BlockingQueue 家族对比与选型（Array/Linked/Synchronous/Priority/Delay/TransferQueue、线程池关系、阿里手册无界队列禁令） | 中 | **6.0** | ✅ 已讲解（候选人答对 DelayQueue 核心思想，其余补全） |
| Q22 | 并发 Bug 排查（jstack 状态、死锁/活锁/饥饿区分、Arthas 常用命令、生产排查流程） | 中 | **0.0** | ✅ 已讲解（候选人全为「不清楚」） |
| Q23+ | 见题库计划 | — | — | ⬜ 未开始 |

---

## 2. Q1 — volatile vs synchronized

### 2.1 原始作答（要点）

- volatile 用于线程间内存可见性，「JVM 修改后立刻 flush」；
- volatile 不支持锁；
- synchronized 保证同步、线程隔离；
- 举例：两个线程各自 `count++`。

### 2.2 评分：6 / 10

| 子项 | 得分 | 说明 |
|---|---|---|
| 基础差异 | 4/10 | 方向对，但「立刻 flush」模型错误，示例代码不成立 |
| 追问 1.1 时序与机制对比 | +1 | 丢更新时序正确；synchronized vs CAS 机制对比太浅 |
| 追问 1.2 实现与性能 | +0 | 未答 |
| 追问 1.3 数组陷阱 | +1 | 结论对，但方案写成 `AtomicInteger[]`（错误），且无开销对比 |

### 2.3 语义对比（核心表）

| 维度 | `volatile` | `synchronized` |
|---|---|---|
| 可见性 | ✅ 写→读 happens-before | ✅ unlock→lock happens-before |
| 有序性 | ✅ 禁止 volatile 读写跨边界重排（写=release，读=acquire） | ✅ 临界区不逸出锁外 |
| 原子性 | ⚠️ 仅单个变量读/写（含 long/double 不撕裂） | ✅ 整个临界区 |
| 互斥 | ❌ | ✅ |
| 阻塞 | ❌ | ✅（竞争时可能阻塞） |
| 可重入 | — | ✅ |
| 开销 | 读≈普通读；写有 StoreLoad 屏障成本 | 无竞争时轻量，竞争时膨胀为重量级 |

### 2.4 volatile 保证什么（精确表述）

- **可见性**：对同一 volatile 变量的写 happens-before 后续的读。
- **有序性**：写之前的操作不得重排到写之后；读之后的操作不得重排到读之前。
- **不保证**：复合操作原子性、多变量不变量、互斥。

```java
volatile int a = 0;
volatile boolean flag = false;

// 线程 A
a = 42;
flag = true;      // volatile 写：a=42 不能重排到这句之后

// 线程 B
if (flag) {       // volatile 读：后续读不能重排到这句之前
    assert a == 42;   // 一定成立
}
```

### 2.5 volatile 够用的三类场景

**场景 1：单写多读的状态标志**

```java
public class Worker implements Runnable {
    private volatile boolean running = true;   // 只由一处写

    public void shutdown() { running = false; }

    @Override public void run() {
        while (running) doWork();
        System.out.println("clean exit");
    }
}
```

> 去掉 volatile，C2 可能把 `while (running)` 提升成 `while (true)`，线程永不退出——典型「生产环境才复现」的 bug。

**场景 2：一次性安全发布 + 廉价脏检查（DCL 引用）**

```java
private static volatile Singleton instance;   // 禁止 new 的重排序 + 发布
```

**场景 3：版本号 / 引用发布（单变量纯赋值）**

```java
private volatile Config current;
public Config get() { return current; }
// 注意：version++ 是复合操作，必须换 AtomicLong
```

### 2.6 必须加锁 / 用原子类的情况

- read-modify-write：`i++`、`x += y`、`map.merge(...)`
- check-then-act：`if (map.get(k) == null) map.put(k, v)`（用 `computeIfAbsent`）
- 多字段不变量（`x`、`y` 必须同时更新）
- 需要阻塞等待：`wait/notify`、`Condition`
- 需要保护对象内部状态（volatile 只管引用）

### 2.7 追问 1.1：丢更新时序

```
T1: read count -> 0
T2: read count -> 0          // 两个 read 之间无 happens-before、无互斥
T1: 0 + 1 = 1
T2: 0 + 1 = 1
T1: write 1
T2: write 1                  // 覆盖 -> 最终 1，丢了一次更新
```

`synchronized` vs `AtomicInteger`：

| | `synchronized` | `AtomicInteger` |
|---|---|---|
| 策略 | 悲观：独占临界区 | 乐观：CAS 重试 |
| 机制 | monitor enter/exit，可能阻塞 | volatile value + CAS 自旋 |
| 高竞争 | 排队，吞吐下降但不会饿死 | CAS 大量失败空转（用 `LongAdder` 缓解） |
| 多变量 | ✅ | ❌ 仅单变量 |
| 一句话 | 同一时刻只允许一个线程进来 | 多个线程都能算，只有一个能提交成功 |

### 2.8 追问 1.2：为什么 volatile 写慢、读几乎不慢

- CPU 写内存先进入 **store buffer**，异步刷入 cache → 其他核暂时看不到。
- JMM 要求 volatile 写对其他线程立即可见 → 需要 **StoreLoad 屏障**。
- HotSpot x86 实现：

```asm
mov    BYTE PTR [flag], 1
lock addl $0x0,(%rsp)     ; 或 mfence
```

  `lock` 前缀会：排空 store buffer → 触发 MESI 失效（其他核缓存行置 Invalid）→ 阻止 StoreLoad 重排。
- **读为什么免费**：x86 是 TSO 强内存模型，LoadLoad / LoadStore 天然有序，只有 StoreLoad 需要屏障；volatile 读需要的 acquire 语义在 x86 上天然满足，只需编译器屏障。
- ARM/POWER 是弱内存模型，volatile 读也要 `ldar` / `lwsync`，读就不免费了。
- 风险：多线程频繁写同一 volatile 变量 → 缓存行 ping-pong，甚至**伪共享（false sharing）**。

一句话：**volatile 读便宜是因为 x86 的 load 本来就够强；写贵是因为要排空 store buffer。**

### 2.9 追问 1.3：`volatile int[]` 的陷阱

```java
volatile int[] data = new int[10];
// A: data[0] = 1;
// B: int x = data[0];
```

- **非线程安全**：volatile 只保证 `data` 引用可见，**不保证元素可见/原子**。
- 正确方案：

| 方案 | 写法 | 机制 | 开销 | 适用 |
|---|---|---|---|---|
| `AtomicIntegerArray` | `arr.incrementAndGet(i)` | 元素级 volatile + CAS | 低，无锁 | ✅ 首选 |
| `AtomicReferenceArray` | 同上（引用） | 同上 | 低 | 对象数组 |
| `VarHandle`（Java 9+） | `VH.setVolatile(arr, i, v)` | 元素级 volatile 语义 | 最低 | 只要可见性时 |
| `synchronized` / `ReentrantLock` | 包住整段 | 互斥 | 竞争时高 | 多元素需一起原子 |
| 整体替换（COW） | `volatile int[] d; d = newArr;` | 不可变 + volatile 发布 | 写 O(n) | 读多写极少 |

> 易错点：`AtomicInteger[]` 是**错的**（数组引用不提供元素原子性），正确类型是 `AtomicIntegerArray`。
> 即使每个元素都 volatile，`arr[0]++` 仍是复合操作。

---

## 3. Q2 — happens-before

### 3.1 评分：0.5 / 10

承认不清楚（好过瞎编），但该题是 senior 必答。

### 3.2 定义

> **happens-before（HB）是 JMM 定义的偏序关系：若 A HB B，则 A 的结果对 B 可见，且 A 在逻辑上先于 B。它不是物理时间顺序，而是「不允许看到违规重排序结果」的契约。**

### 3.3 JLS 17.4.5 的规则（至少背 5 条）

| # | 规则 | 说明 |
|---|---|---|
| 1 | 程序顺序 | 同线程内前操作 HB 后操作 |
| 2 | 监视器锁 | 同一把锁的 unlock HB 后续 lock |
| 3 | volatile | 同一字段的写 HB 后续读 |
| 4 | 线程启动 | `Thread.start()` HB 子线程所有操作 |
| 5 | 线程终止 | 线程内所有操作 HB 其他线程 `join()` 返回 |
| 6 | 中断 | `interrupt()` HB 被中断线程检测到 |
| 7 | 终结器 | 构造完成 HB `finalize()` 开始 |
| 8 | 传递性 | A HB B，B HB C ⇒ A HB C |
| 附加 | final 字段 | 构造器内 final 写 HB 引用发布（JLS 17.5） |

### 3.4 推导示例

```java
int x = 0;
volatile boolean flag = false;

// A: x = 42;  (1)
//    flag = true; (2)
// B: if (flag) { (3)
//      println(x); (4)  -> 一定 42
```

```
(1) HB (2)   程序顺序
(2) HB (3)   volatile
(3) HB (4)   程序顺序
=> (1) HB (4) 传递性 => x=42 可见
```

若 `flag` 非 volatile：无 HB 边 → data race → 可能看到 `flag==true` 但 `x==0`。

### 3.5 为什么不能凭直觉

1. **编译器/JIT 重排**：as-if-serial 只保证单线程语义。
2. **CPU 乱序 + store buffer 延迟**：写滞留，其他核看不到。
3. **JMM 允许反直觉结果**：经典 4×4 测试，`r1==0 && r2==0` 合法。

### 3.6 正确的思维方式

> 不要问「谁先执行」，要问「有没有 HB 边」。

- 有 HB 边 → 有序、可见、安全。
- 无 HB 边且至少一个写 → **data race**，JMM 不保证任何结果。
- 无 data race（正确同步）→ JMM 保证顺序一致性，才可以凭直觉推理。

### 3.7 「感觉从没用过」的回应（实战清单）

HB 不是没用，而是被库封装了：

| 场景 | HB 来源 |
|---|---|
| `Thread.start()` / `join()` | 规则 4 / 5 |
| `ExecutorService.submit()` → `Future.get()` | FutureTask 内部 volatile + LockSupport |
| `BlockingQueue.put() → take()` | 内部锁 unlock → lock |
| `CompletableFuture` 链式回调 | 完成 → 下一环 |
| `ConcurrentHashMap.put → get` | Node 的 volatile 读 + tabAt |
| `CountDownLatch.countDown → await` | AQS state |
| `synchronized` / `ReentrantLock` | monitor / AQS |

真正需要亲手建立 HB 的场景：

1. **优雅关闭** flag（volatile）
2. **配置热更新**（volatile 引用替换）
3. **DCL 单例**（volatile）
4. **懒加载缓存**（DCL）
5. **Code Review 判定**：`while (!cancelled)` 而 `cancelled` 非 volatile → 拦下

### 3.8 HB 的实战价值

| 问题 | 用 HB 回答 |
|---|---|
| 这个字段要不要 volatile？ | 有没有跨线程读写 + 有没有 HB 边 |
| 能不能不加锁？ | 是否单变量纯赋值 |
| flag 为什么看不到？ | 缺 HB 边 |
| 用了 ConcurrentHashMap 还要同步吗？ | 单次操作不用；复合操作要 |
| 对象发布安全吗？ | 是否通过 volatile/锁/final/并发容器发布 |
| long 会撕裂吗？ | 非 volatile 的 64 位写可能撕裂（32 位 JVM） |
| 为什么只在生产复现？ | data race + JIT/CPU 相关，不可复现 |

### 3.9 验证工具

- `-XX:+PrintCompilation` 观察 JIT hoisting。
- **jcstress**（OpenJDK 官方并发测试工具）压力测试暴露缺 HB 的 bug。
- JOL 观察对象头。

---

## 4. Q3 — final 字段与安全发布

### 4.1 原始作答

「final 字段只能初始化一次，之后不允许修改。」

### 4.2 评分：1.5 / 10

答的是语言层语义，未触及 JMM 内存语义。

### 4.3 核心结论

> **final 字段的写，在构造器结束时被 freeze，且 freeze happens-before 该对象引用的发布。因此线程只要拿到对象引用，就一定能看到 final 字段的初始化值——即使发布过程没有 volatile 或锁。**

### 4.4 JMM 两条额外规则（JLS 17.5）

1. 构造器中对 final 字段的赋值 HB 该对象引用的发布。
2. 传递性：读到 final 字段 ⇒ 看到它指向对象在构造器内完成的状态。

### 4.5 final vs 非 final

```java
class Holder {
    final int x;   // 保证看到 1
    int y;         // 可能看到 0
    Holder() { x = 1; y = 2; }
}

static Holder holder;   // 无 volatile 发布
```

| | final `x` | 非 final `y` |
|---|---|---|
| 无同步发布后可见初始化值 | ✅ | ❌ 可能是默认值 |
| 需要 volatile/锁 | 不需要 | 需要 |
| 原理 | freeze + HB | 无 HB 边 → data race |
| 重排 | 禁止逸出构造器 | 允许 |

这就是 **部分构造对象逸出（partially constructed object escape）**。

### 4.6 freeze 到底是什么意思

**freeze 是 JMM 规范里的逻辑动作（JLS 17.5），不是 CPU 指令。**

| 层级 | 内容 | 可见性 |
|---|---|---|
| 规范层 | read/write/lock/unlock/**freeze** + HB | 文档 |
| JVM 实现层 | 编译器屏障 + StoreStore / LoadLoad 屏障 | 汇编可见 |
| 硬件层 | store buffer / MESI / mfence、ldar、dmb | CPU 手册 |

```
构造器内: write(final字段...)
              ↓
          [ freeze ]        ← 逻辑顺序点（构造器正常结束时）
              ↓
发布:      write(instance = ref)
              ↓
其他线程:  read(instance) → read(instance.field)  ✅ 必见
```

落地方式：

| 架构 | 构造器结束 | 读 final |
|---|---|---|
| x86 (TSO) | 通常无需额外指令（StoreStore 天然有序），只加编译器屏障 | 无需指令 |
| ARM64 | `dmb ishst` | `ldar` / `dmb ishld` |
| POWER | `lwsync` | `lwsync` |

> x86 上 HotSpot 的 `membar_storestore()` 展开为空操作——不是没实现，是硬件已满足。

### 4.7 三个坑

1. **`this` 在构造器内逸出** → freeze 失效：
   ```java
   Bad() { instance = this; x = 42; }   // ❌ 可能看到 x==0
   ```
2. **final 只保护直接字段与经 final 引用可达的对象**；数组元素不被保护。
3. **反射可改 final 字段**（Java 12+ 需 `--add-opens`，record/String 会抛异常）。

### 4.8 String 的不可变性分析

```java
public final class String {      // ① final 类
    private final byte[] value;  // ② final 字段：安全发布 + 不可变
    private int hash;            // ③ 非 final，缓存 hash（良性数据竞争）
}
```

- final 类：防止子类破坏不变量
- final 字段：任意线程拿到引用即见完整内容，**无需同步**
- 无修改方法

`hash` 的良性数据竞争（结果幂等，允许重复计算）：

```java
int h = hash;
if (h == 0) { h = computeHash(); hash = h; }
return h;
```

> 只适用于「读多写少 + 结果幂等」，不要照搬到有副作用的场景。

### 4.9 实践优先级

> **安全发布优先级：不可变对象（final 字段） > volatile/锁/并发容器发布 > 无同步发布（错）**

`String`、`Integer`、`BigDecimal`、`LocalDate` 能随便跨线程传，靠的就是 final。

---

## 5. Q4 — Double-Checked Locking

### 5.1 原始作答

「我感觉安全呢。」

### 5.2 评分：2 / 10

DCL 是必考陷阱题，标准答案：**不安全**。

### 5.3 题目

```java
public class Singleton {
    private static Singleton instance;
    public static Singleton getInstance() {
        if (instance == null) {                    // ← 问题在这
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

### 5.4 确切原因

`new` 不是原子操作：

```
1. memory = allocate()     // 分配，字段全为默认值
2. ctor(memory)            // 初始化
3. instance = memory       // 发布引用
```

无 HB 约束时 2、3 可重排：

```
1. allocate
3. instance = memory       // 先发布，字段还是默认值
2. ctor(memory)            // 后初始化
```

线程 B：`if (instance == null)` 读到非 null → 直接返回 → **拿到半成品对象**，且**根本没进锁**。

### 5.5 为什么 synchronized 救不了

锁保护的是临界区；线程 B 的第一次读**在锁外**，没有 HB 边。锁只能互斥，不能阻止锁外读者看到重排后的中间状态。

### 5.6 修复方案

**方案 1：volatile（标准答案）**

```java
private static volatile Singleton instance;

public static Singleton getInstance() {
    Singleton local = instance;              // 只读一次 volatile
    if (local == null) {
        synchronized (Singleton.class) {
            local = instance;
            if (local == null) {
                local = new Singleton();
                instance = local;
            }
        }
    }
    return local;
}
```

volatile 作用：禁止 2/3 重排（release）+ 保证可见性（acquire）。

**方案 2：静态内部类 Holder（生产首选）**

```java
public class Singleton {
    private Singleton() {}
    private static class Holder {
        static final Singleton INSTANCE = new Singleton();
    }
    public static Singleton getInstance() { return Holder.INSTANCE; }
}
```

- 靠 JVM 类初始化锁（`<clinit>` 保证只执行一次）
- 懒加载 + 零同步开销
- 缺点：防不住反射/反序列化

**方案 3：枚举（最防破坏）**

```java
public enum Singleton { INSTANCE; }
```

- 防反射、防反序列化
- 不能懒加载、不能继承
- 《Effective Java》推荐

### 5.7 高级加分点

1. **若所有字段都是 final**，无 volatile 的 DCL 在 Java 5+ 语义下也安全（freeze 兜底），但极其脆弱——加一个可变字段就静默破功。
2. **x86 上难以复现**：TSO 保证 StoreStore 有序，硬件不重排；但 JIT 仍可重排，ARM 风险更高。「我机器上是好的」不是正确性证明。
3. **正确性判据**：对 `instance` 的所有访问是否都在 HB 保护下？锁外那次读没有 → 不安全。

> 一句话：DCL 的 bug 不在锁里，在锁外那次「乐观读」；volatile 的价值是补上可见性 + 禁止重排。

---

## 6. Q5 — synchronized 底层实现

### 6.1 评分：0 / 10（未作答，属知识补课）

### 6.2 第一层：字节码

```java
public void inc() { synchronized (this) { count++; } }
```

```
monitorenter
  getfield count
  iconst_1
  iadd
  putfield count
monitorexit           // 正常路径
monitorexit           // 异常路径（编译器生成隐式 try/finally）
```

- `synchronized` 方法不加字节码，用访问标志 `ACC_SYNCHRONIZED`。
- 字节码里**看不到锁升级**，全部发生在 JVM runtime。

### 6.3 第二层：对象头 Mark Word（64 bit 随状态变形）

| 状态 | 末 2 位 | 内容 |
|---|---|---|
| 无锁 | `01` | hashCode(31) + age(4) + biased_lock(0) + 01 |
| 偏向锁 | `01`（biased=1） | ThreadID(54) + epoch(2) + age(4) + 1 + 01 |
| 轻量级锁 | `00` | 指向栈上 Lock Record 的指针(62) + 00 |
| 重量级锁 | `10` | 指向 ObjectMonitor 的指针(62) + 10 |
| GC 标记 | `11` | 空 |

> 用 JOL 可直接打印：`ClassLayout.parseInstance(obj).toPrintable()`

**追问：不同锁状态下 Mark Word 内容会变吗？偏向锁时 hash code 还在吗？**

Mark Word 是一个 64 bit 的**联合体（union）**，同一时刻只按一种布局解释。所以是的，每种状态布局完全不同；偏向锁状态下 31 bit 的 hash 位被 ThreadID 占用。

```text
无锁（Normal）
┌──────────┬──────────────┬────┬─────┬──────┬─────┐
│ unused   │ identity_hash│cms │ age │biased│ lock│
│ 25 bit   │   31 bit     │1bit│4 bit│ 1bit │2 bit│
└──────────┴──────────────┴────┴─────┴──────┴─────┘
                                                01

偏向锁（Biased）
┌────────────────────┬───────┬──────┬─────┬──────┬─────┐
│   JavaThread*      │ epoch │unused│ age │biased│ lock│
│      54 bit        │ 2 bit │ 1bit │4 bit│ 1bit │2 bit│
└────────────────────┴───────┴──────┴─────┴──────┴─────┘
                                            01（biased=1）

轻量级锁：ptr_to_lock_record(62) + 00
重量级锁：ptr_to_ObjectMonitor(62) + 10
GC 标记  ：转发指针/标记信息(62) + 11
```

**hash code 去哪了？**

1. **未计算时本来就不存在**：`Object.hashCode()` 是懒计算的，对象刚创建时那 31 bit 是 unused，首次调用才用 CAS 写入。
2. **已计算 + 偏向锁：JVM 直接禁止偏向**。HotSpot 在尝试偏向时检查 `mark->has_hash()`，已有 identity hash 的对象不再进入偏向锁，直接走轻量级锁。所以不是「偏向锁把 hash 弄丢了」，而是**「有 hash 的对象不允许被偏向」**。
3. **轻量级 / 重量级锁：hash 被搬走，不是丢失**。轻量级锁把旧 Mark Word（displaced mark word，含 hash）复制到栈上 Lock Record；重量级锁存进 `ObjectMonitor._header`。所以 `System.identityHashCode(obj)` 在锁状态变化后**始终返回同一个值**。

**为什么这么设计**：64 bit 要同时放 hash、GC 年龄、锁状态、线程 ID，只能复用同一块内存；代价是状态之间互斥，这也是 JEP 374 移除偏向锁的复杂度来源之一。

**常见误解对照**

| 说法 | 正确理解 |
|---|---|
| 偏向锁会覆盖 hashCode | 有 hash 的对象不会被偏向 |
| hashCode 存在对象头里 | 只有默认 identity hash 才存；重写 hashCode() 的类不存 |
| 解锁后 hash 会变 | 不会，displaced mark word 会还原 |
| 所有 JVM 都是 31 bit | 32 位 JVM 是 25 bit |

### 6.4 第三层：ObjectMonitor（HotSpot C++）

```
ObjectMonitor {
    _owner;        // 持有锁的线程
    _recursions;   // 重入计数
    _cxq;          // 竞争队列
    _EntryList;    // 阻塞等待锁
    _WaitSet;      // wait() 的线程
}
```

- 竞争失败 → `_EntryList` → park（Linux futex）
- `wait()` → `_WaitSet` + 释放锁
- `notify()` → 移到 `_EntryList`，**不释放锁**

这就是 `wait/notify` 必须在 synchronized 内的原因：操作的是 monitor。

### 6.5 锁升级路径

```
无锁
 │ 第一次 synchronized（单线程）
 ▼
偏向锁 ── 一次 CAS，之后零开销；撤销需 safepoint（贵）
 │ 第二个线程来抢
 ▼
轻量级锁 ── 栈上 Lock Record + CAS + 自适应自旋
 │ 自旋失败 / 竞争激烈
 ▼
重量级锁 ── OS 互斥量，park/unpark，上下文切换
```

**锁只能升级，不能降级。**

| 状态 | 加锁方式 | 适用 | 代价 |
|---|---|---|---|
| 偏向锁 | 一次 CAS + 线程 ID 比较 | 单线程反复进 | 撤销需 safepoint |
| 轻量级锁 | CAS + 自旋 | 交替执行、轻度竞争 | 自旋耗 CPU |
| 重量级锁 | park/unpark | 真竞争 | 内核态切换 |

### 6.6 JIT 的反向优化

- **锁消除**：逃逸分析发现锁对象未逃逸 → 删除 monitorenter/exit（如 `StringBuffer` 局部变量）。
- **锁粗化**：循环内反复加解锁 → 扩大到循环外。

> 实践：不要手写复杂锁优化，优先缩小锁粒度、减少共享。

### 6.7 JEP 374：为什么移除偏向锁（JDK 15 默认禁用，后续移除）

| 原因 | 说明 |
|---|---|
| 维护成本高 | 与 safepoint / GC / deopt / 线程生命周期强耦合，bug 高发 |
| 撤销代价大 | 撤销需 safepoint → STW 停顿尖峰，比省下的 CAS 更贵 |
| 收益变小 | 现代应用多用 `java.util.concurrent`；`Hashtable`/`Vector` 式场景减少 |
| 硬件变快 | 一次 CAS 仅十几 ns |
| 基准无显著收益 | 差异在噪声范围内 |
| 简化代码 | 为后续演进铺路 |

**现在实际路径：无锁 → 轻量级锁 → 重量级锁。**

面试话术：
> 偏向锁在 JDK 15 通过 JEP 374 被禁用并最终移除，因为它与 safepoint 耦合导致撤销停顿，维护成本高而现代收益小。

### 6.8 2026 关键实践：synchronized 与虚拟线程

> **`synchronized` 会 pin 住虚拟线程的载体线程（carrier thread）**，阻塞时无法卸载；`ReentrantLock` 不会。

```java
// 虚拟线程下不推荐
synchronized (lock) { socket.read(); }   // 阻塞 IO → pin

// 推荐
lock.lock();
try { socket.read(); } finally { lock.unlock(); }
```

建议：
- 临界区短、纯 CPU → `synchronized` 没问题
- 临界区含阻塞操作（IO / sleep / await），尤其虚拟线程 → `ReentrantLock`

---

## 7. Q6 — wait/notify 与 Condition（1 / 10）

### 7.1 原始作答（要点）

- 第 1 问：不知道。
- 第 2 问：不清楚。
- 第 3 问：更加灵活，支持 lock/unlock。

### 7.2 评分：1 / 10

第 3 问只答了 API 层面的表面差异，没答到设计动机；第 1、2 问是本题核心，都未作答。

### 7.3 为什么 wait/notify 必须在 synchronized 里

**直接原因**：`wait` / `notify` 是 `Object` 的方法，操作的是**该对象的 monitor**。JVM 会检查当前线程是否持有该 monitor，不持有就抛：

```
IllegalMonitorStateException: current thread is not owner
```

**真正原因（正确性）**：它必须让两组动作分别成为原子操作：

```
消费者：检查条件  +  进入等待     ← 必须原子
生产者：修改条件  +  发出通知     ← 必须原子
```

`wait()` 的语义正是为此设计：

```
wait() = 原子地 { 释放锁; 挂起线程; 被唤醒后重新竞争锁 }
```

**释放锁和挂起是原子的**，中间没有窗口让 `notify` 溜走。这就是必须在锁内的根本原因。

`notify` 也必须在锁内：它操作 `_WaitSet`，且必须和「修改条件变量」在同一临界区，否则会出现下面的丢失唤醒。

### 7.4 丢失唤醒（lost wakeup）

**错误代码**：

```java
boolean ready = false;

// 消费者
while (!ready) {          // ① 检查：false
    // ← CPU 切走，生产者执行
    Thread.sleep(...);    // 模拟「准备进入等待」
}
consume();

// 生产者
ready = true;             // ② 改条件
notify();                 // ③ 通知 —— 消费者还没 wait，通知被丢弃
// → 消费者永远等下去
```

**时序**：

```
消费者: 读 ready == false
生产者: ready = true; notify();   ← wait 集合是空的，通知消失
消费者: 进入 wait()               ← 永远醒不过来
```

**正确写法**：

```java
// 消费者
synchronized (lock) {
    while (!ready) {        // ① 检查
        lock.wait();        // ② 原子：释放锁 + 挂起
    }
}
consume();

// 生产者
synchronized (lock) {
    ready = true;           // 改条件
    lock.notifyAll();       // 通知
}
```

> 一句话：锁把「条件判断」和「进入等待」绑定成原子操作，消除检查与等待之间的窗口。

### 7.5 虚假唤醒（spurious wakeup）

线程**没有收到 `notify` 也可能自己醒来**，这是 POSIX/JVM 允许的行为。

**必须用 `while` 而不是 `if`**：

```java
// ❌ 错：虚假唤醒后直接往下走
synchronized (lock) {
    if (!ready) lock.wait();
    consume();          // 条件可能仍不满足！
}

// ✅ 对：醒来后重新检查
synchronized (lock) {
    while (!ready) lock.wait();
    consume();
}
```

`while` 同时解决：虚假唤醒、`notifyAll` 唤醒到「不是自己等的条件」、丢失唤醒的边界情况。

**JDK 官方模板**：

```java
synchronized (monitor) {
    while (!condition) {
        monitor.wait();
    }
    // 条件满足，执行动作
}
```

### 7.6 为什么推荐 ReentrantLock + Condition

**wait/notify 版本（有界缓冲区）**：

```java
final Object lock = new Object();
final Queue<Integer> q = new ArrayDeque<>();
final int CAP = 10;

void put(int v) throws InterruptedException {
    synchronized (lock) {
        while (q.size() == CAP) lock.wait();
        q.add(v);
        lock.notifyAll();        // 唤醒所有（包括同类生产者，浪费）
    }
}

int take() throws InterruptedException {
    synchronized (lock) {
        while (q.isEmpty()) lock.wait();
        int v = q.poll();
        lock.notifyAll();        // 同上
        return v;
    }
}
```

**Condition 版本**：

```java
final ReentrantLock lock = new ReentrantLock();
final Condition notFull  = lock.newCondition();   // 两个独立等待集
final Condition notEmpty = lock.newCondition();
final Queue<Integer> q = new ArrayDeque<>();
final int CAP = 10;

void put(int v) throws InterruptedException {
    lock.lock();
    try {
        while (q.size() == CAP) notFull.await();
        q.add(v);
        notEmpty.signal();       // 只唤醒消费者，精准
    } finally {
        lock.unlock();
    }
}

int take() throws InterruptedException {
    lock.lock();
    try {
        while (q.isEmpty()) notEmpty.await();
        int v = q.poll();
        notFull.signal();        // 只唤醒生产者
        return v;
    } finally {
        lock.unlock();
    }
}
```

**核心差异**：

| 能力 | `wait/notify` | `Condition` |
|---|---|---|
| 多个等待集 | ❌ 一个对象只有一个 `_WaitSet`，只能 `notifyAll` | ✅ 一把锁可建多个 Condition，精准唤醒 |
| 唤醒精度 | `notify()` 随机挑一个，可能挑错 | ✅ `signal()` 指定等待集里的一个 |
| 超时 | 只有 `wait(long)` | ✅ `awaitNanos`、`await(time, unit)` |
| 可中断性 | 只有可中断 | ✅ 可选 `awaitUninterruptibly()` |
| 公平性 | ❌ 无 | ✅ `new ReentrantLock(true)` |
| 尝试获取 | ❌ 无 | ✅ `tryLock()`、`tryLock(timeout)` |
| 锁语义 | 隐式、块结构、自动释放 | 显式，必须在 `finally` 里 `unlock()` |
| 诊断 | 无 | ✅ `getHoldCount()`、`getWaitingThreads()` |

**面试话术**：
> `Condition` 把「一个隐式等待集」拆成「多个显式等待集」，因此可以在生产者-消费者这类多角色场景里**只唤醒需要的那一类线程**，避免 `notifyAll` 的惊群；同时补齐了超时、可中断、公平性、可诊断性。

**选型建议**：
- 单条件、短临界区 → `synchronized` + `wait/notify`
- 多条件、需要超时/公平/可中断/诊断 → `ReentrantLock` + `Condition`
- 生产者-消费者 → 直接用 `BlockingQueue`

### 7.7 高频坑

1. `wait` 的对象和 `synchronized` 的对象必须是同一个，否则 `IllegalMonitorStateException`。
2. `if` 写成 `while`（虚假唤醒 + 惊群）。
3. 忘 `notifyAll` 用 `notify` → 唤醒错线程，程序挂死。
4. 在 `wait` 前修改条件但没在锁内 → 丢失唤醒。
5. 捕获 `InterruptedException` 后吞掉 → 破坏取消语义。
6. `await()` 必须在 `lock()` 之后。

---

## 8. Q7 — ReentrantLock vs synchronized（7 / 10）

### 8.1 原始作答（要点）

- `ReentrantLock` 多了：多个条件、公平锁、可中断、等待时间可控、可尝试获取。
- 多个条件适用于复杂唤醒条件；公平锁实现线程公平竞争；等待时间可控适用于超时；尝试获取可避免等待。
- 可重入指同一个线程可多次获取同一把锁，内部通过 AQS `state` 计数器实现；`unlock` 会重置 AQS 的 `state`。
- 公平锁就是所有线程公平竞争同一把锁；默认非公平，效率更好；非公平会导致饥饿，可能一直无法获取锁。

### 8.2 评分：7 / 10

三问都答到了核心，细节有不准之处。

### 8.3 能力对比（补全）

| 能力 | API | 典型场景 |
|---|---|---|
| 多个等待条件 | `newCondition()` | 生产者-消费者（`notFull` / `notEmpty`） |
| 公平锁 | `new ReentrantLock(true)` | 任务队列、防个别线程长期抢不到 |
| 可中断获取 | `lockInterruptibly()` | 可取消的长等待、优雅停机 |
| 超时获取 | `tryLock(3, SECONDS)` | 避免无限阻塞、防死锁兜底 |
| 尝试获取 | `tryLock()` | 快速失败、降级逻辑 |
| 可诊断 | `getHoldCount()`、`getQueueLength()` | 线上排查锁竞争 |

> 补充：`ReentrantLock` 不解决「忘记 unlock」；`synchronized` 由 JVM 保证异常时自动释放，语法安全性更强。

### 8.4 修正：unlock 不是「重置」state，而是递减

```java
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;      // state--，不是直接置 0
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {                       // ← 只有减到 0 才真正释放
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```

**忘记 `unlock()` 的后果链**：

1. `state` 永远 ≥ 1，`exclusiveOwnerThread` 始终是自己；
2. 其他线程全部阻塞在 AQS 队列，永久挂起；
3. 同一个线程再次 `lock()` 仍能成功（重入掩盖问题）；
4. `state` 达到 `Integer.MAX_VALUE` 时抛 `Error: Maximum lock count exceeded`；
5. 排查：`jstack` 看到 `WAITING (parking)`。

**标准写法**：

```java
lock.lock();
try {
    // 临界区
} finally {
    lock.unlock();
}
```

### 8.5 修正：非公平锁 ≠ 完全无序

| | 公平锁 | 非公平锁（默认） |
|---|---|---|
| 获取规则 | `hasQueuedPredecessors()` 为 false 才尝试 | 直接 CAS 抢，失败才排队（barging） |
| 吞吐 | 低（频繁唤醒 + 上下文切换） | 高（可插队） |
| 延迟 | 尾部可预期 | 平均低，尾部可能很长 |
| 饥饿 | 不会 | 理论上可能 |

要点：

1. 非公平锁只是在「队列非空时新线程仍先抢一次」，抢不到仍要排队。
2. 公平锁的「公平」仅指**锁获取顺序 FIFO**，不是线程调度公平。
3. **`tryLock()` 无参版本不受公平性约束**，即使公平锁也会直接抢。
4. 公平锁代价：每次唤醒都要上下文切换，高竞争下吞吐可能下降一个数量级。

---

## 9. Q8 — AQS 原理（0 / 10，知识补课）

### 9.1 什么是 AQS

**AQS = AbstractQueuedSynchronizer**，`ReentrantLock`、`CountDownLatch`、`Semaphore`、`ReentrantReadWriteLock` 的共同基础。

核心维护两样东西：

1. `state`（int）—— 同步状态（锁计数、许可数、剩余计数等）。
2. 一个 **FIFO 队列**（`Node` 双向链表）—— 存放获取失败、被阻塞的线程。

线程流程：先 `tryAcquire` 尝试 CAS 抢 `state`；抢不到就入队 `park`（挂起，不占 CPU）；别人释放锁后唤醒队首，重新竞争。

### 9.2 `state` 在不同同步器里的含义

| 同步器 | `state` 含义 | 举例 |
|---|---|---|
| `ReentrantLock`（独占） | 当前线程持有锁的**重入次数** | `state == 0` 未锁定；`1` 线程 A 持有；A 重入后 `2` |
| `Semaphore`（共享） | **可用许可的剩余数量** | `Semaphore(5)` 初始 `5`；`acquire` → `--`，`release` → `++` |
| `CountDownLatch`（共享） | **剩余的计数值** | `CountDownLatch(3)` 初始 `3`；`countDown()` → `--`；`state == 0` 时 `await()` 放行 |

`state` 的修改必须通过 **CAS**（或 `volatile` 语义），避免数据竞争。

### 9.3 等待队列与 Node 状态

```java
Node {
    volatile int waitStatus;   // 节点状态
    volatile Node prev;        // 前驱
    volatile Node next;        // 后继
    volatile Thread thread;    // 代表的线程
    Node nextWaiter;           // Condition 队列用
}
```

| `waitStatus` | 值 | 含义 |
|---|---|---|
| `SIGNAL` | -1 | 后继线程需要被唤醒，当前节点释放锁后会 `unpark` 下一个 |
| `CANCELLED` | 1 | 线程已取消等待（中断/超时），节点将被移出队列 |
| `CONDITION` | -2 | 节点在 `Condition` 上等待 |
| `PROPAGATE` | -3 | 共享模式下，信号需要传播给后继 |
| `0` | 0 | 初始态 |

### 9.4 `acquire` 完整流程（独占模式）

```
调用 lock() / acquire()
   ↓
1. tryAcquire(arg)             // 尝试 CAS 抢 state
   成功 → 获得锁，返回
   失败 →
        ↓
2. 创建 Node，加入队列尾部（enq）
   ↓
3. 判断前驱是 head 且可 park
   ↓
4. LockSupport.park(this)      // 线程挂起，不占 CPU
   ↓
5. 被 unpark 唤醒             // 由释放锁的线程调用 unparkSuccessor(head)
   ↓
6. 回到 acquireQueued 循环，重新 tryAcquire
   成功 → 设为 head，出队，获得锁
   失败 → 继续 park
```

**release 流程**：`tryRelease` 成功且 `state == 0` 时，`unparkSuccessor(head)` 只唤醒队首的后继，不唤醒全体。

### 9.5 独占模式 vs 共享模式

| 特征 | 独占模式 | 共享模式 |
|---|---|---|
| 含义 | 同一时刻只有一个线程持有 | 多个线程可同时持有（按许可数） |
| 代表 | `ReentrantLock`、写锁 | `Semaphore`、`CountDownLatch`、读锁 |
| state 含义 | 重入计数 / 是否被持有 | 可用资源数 / 剩余计数 |
| 唤醒策略 | 唤醒队首 1 个 | 唤醒所有满足条件的等待者 |

### 9.6 为什么 CountDownLatch 用共享模式

`state` 表示剩余计数，多个线程并发 `countDown()` 共同减少 `state`；一旦 `state == 0`，**所有** `await()` 的线程都需要被释放。这正是共享模式「唤醒所有等待者」的特性；独占模式一次只能放行一个，无法实现。

---

## 10. Q9 — CAS / ABA / LongAdder（4.5 / 10）

### 10.1 原始作答（要点）

- CAS = compare and swap，参数有期望值、新值，通过自旋尝试修改。
- ABA：T1 的 oldVal 是 A，T2 改成 B 再改回 A，T1 认为没改过会正常执行；`AtomicStampedReference` 增加时间戳，区分两个不同时刻的 A。
- `LongAdder` 不认识。

### 10.2 评分：4.5 / 10

第 1、2 问方向对，参数顺序有小错；第 3 问未答。

### 10.3 CAS 三个参数（顺序）

| 参数 | 含义 |
|---|---|
| V（内存地址） | 要修改变量的地址 |
| O（期望值 / 旧值） | 你认为内存中当前应该是多少 |
| N（新值） | 匹配成功后要改成多少 |

```java
boolean compareAndSwap(V address, O expect, N update) {
    if (*address == expect) { *address = update; return true; }
    return false;
}
```

**原子性**：比较与交换由 CPU 硬件保证不可分割（x86 上 `cmpxchg` + `lock` 前缀）。

### 10.4 ABA 具体场景（无锁栈 pop）

```
T1: 读 top = A，读 A.next = B          // T1 被挂起
T2: pop() → 弹出 A，top = B
T2: pop() → 弹出 B，top = C
T2: push(A) → top 又变回 A，且 A.next 已变成 C
T1: 醒来 CAS(top, expect=A, update=B)  // 内存 top 还是 A，CAS 成功！
                                       // 但 B 已被弹出/回收 → 数据损坏
```

CAS 只知道「现在还是 A」，不知道中间经历过 A→B→A，**值一样 ≠ 状态一样**。

### 10.5 AtomicStampedReference 如何解决

内部维护 `Pair<V>{reference, stamp}`，CAS 需要 4 个参数：

```java
compareAndSet(expectedReference, newReference, expectedStamp, newStamp)
```

T1 读到 `(A, stamp=0)`；T2 连续操作后 stamp 变为 3。T1 回来执行 `compareAndSet(A, B, 0, 1)` —— 引用匹配但 stamp 不匹配（0 vs 3），CAS **失败**，T1 知道中间发生了变化。

> 变体：`AtomicMarkableReference` 只用 boolean 标记，只关心「是否被改过」，不记录次数。

### 10.6 LongAdder：分段 Cell，依然用 CAS

`LongAdder` **没有消除 CAS**，而是把「所有人抢同一个 value」变成「大家抢不同的 Cell」：

- 维护 `base` + `Cell[]`，每个 `Cell.value` 独立 CAS。
- 线程按哈希映射到某个 Cell，对它的 `value` 执行 CAS。
- 低并发先 CAS `base`；竞争激烈时初始化/扩容 `Cell` 数组（长度保持 2 的幂）。
- 数组上限约等于 **CPU 核心数量级**（如 8 核通常几个到十几个 Cell），不会无限膨胀。
- `sum()` = `base` + 遍历所有 Cell 累加（写快、读慢）。

| | `AtomicLong` | `LongAdder` |
|---|---|---|
| 变量数量 | 1 个 `value` | `base` + 多个 `Cell` |
| CAS 目标 | 所有人抢同一个 `value` | 分散到不同 Cell 各自 CAS |
| 高并发 | 慢（自旋 + 缓存行 ping-pong） | 快（竞争被分摊） |
| 读取性能 | 极快（直接读） | 较慢（遍历求和） |
| 内存占用 | 极小 | 较大 |

**选型**：写多读少 + 高并发（QPS 统计）→ `LongAdder`；读多写少或需精确实时值 → `AtomicLong`。

---

## 11. Q10 — ThreadLocal 原理（5 / 10）

### 11.1 原始作答（要点）

- 核心数据结构是 `ThreadLocalMap`，key 是 `threadId`；用 `weakreference` 防止占用过多内存。
- 会导致内存泄漏；例如 `set` 后没执行 `remove` / `clear`；防止方法：用 `finally` 释放。
- 虚拟线程不清楚有啥问题；`ScopedValue` 是解决 `ThreadLocal` 忘记释放的问题。

### 11.2 评分：5 / 10

第 1、2 问方向对，但有关键事实错误；第 3 问基本未答对。

### 11.3 修正：key 是 ThreadLocal 实例，不是 threadId

- 每个 `Thread` 对象内部**自己持有一个 `ThreadLocalMap` 实例**，不是所有线程共享一个 Map。
- Map 的 key 是 `WeakReference<ThreadLocal>`（`ThreadLocal` 实例），value 是存入的对象（强引用）。
- **一个线程可以有多个 ThreadLocal 变量**，靠 Map 用不同 ThreadLocal 实例作 key 来区分，这就是需要 Map 的原因。

```
线程 T1 独占的 ThreadLocalMap：
┌─────────────┬──────────┐
│ key: TL#1   │ val:"A"  │   ← userName
│ key: TL#2   │ val:999  │   ← reqId
└─────────────┴──────────┘

线程 T2 独占的 ThreadLocalMap：
┌─────────────┬──────────┐
│ key: TL#1   │ val:"B"  │   ← 同名变量，不同副本
└─────────────┴──────────┘
```

### 11.4 为什么 key 用弱引用

防止 `ThreadLocal` 对象本身被 `Thread` 强引用而无法 GC。

```java
public void foo() {
    ThreadLocal<String> tl = new ThreadLocal<>();  // 局部变量
    tl.set("hello");
    tl = null;                                     // 不再指向 ThreadLocal
}
```

- **若 key 是强引用**：`Thread → ThreadLocalMap → Entry → ThreadLocal` 全是强引用链，只要线程活着，ThreadLocal 永远无法回收 → 每创建一次就泄漏一个对象。
- **弱引用（实际实现）**：Map 只弱引用 ThreadLocal，外部没有强引用时 GC 可回收，Entry 变成 `key = null`。

> JDK 7+ 在 key 为 null 时会尝试清理该 Entry，但这只是兜底，不应依赖。

### 11.5 内存泄漏的本质与防护

- key 被回收后变 `null`，但 **value 是强引用，仍被 Entry 持有**。
- 只要线程还活着（线程池线程长期复用），这个 value 就永远不会被回收 → 泄漏。

**防护**：`set()` 后必须在 `finally` 里 `remove()`。

```java
try {
    tl.set(value);
    // 业务逻辑
} finally {
    tl.remove();
}
```

### 11.6 虚拟线程下的问题与 ScopedValue

**平台线程 = OS 内核线程**；虚拟线程是 JVM 层的轻量线程，**挂载（mount）到平台线程上执行**，阻塞时卸载（unmount），平台线程去跑别的虚拟线程。

**问题**：`ThreadLocal` 挂在 `Thread` 对象上。虚拟线程 `set()` 后若没 `remove()`，值会残留在**平台线程的 ThreadLocalMap** 里；下一个虚拟线程复用同一平台线程时可能读到脏数据 → 跨虚拟线程数据泄露。

**ScopedValue（Java 21 预览）不是为了补救「忘记释放」，而是虚拟线程场景下 ThreadLocal 的替代方案**：

| | ThreadLocal | ScopedValue |
|---|---|---|
| 可变性 | 可随时 set/get | 作用域内不可变 |
| 生命周期 | 手动 remove | 作用域结束**自动解绑** |
| 存储 | 每个 Thread 一个 Map | 绑定在调用栈作用域，无 Map |
| 继承 | `InheritableThreadLocal` | 子线程/结构化并发天然继承 |
| 适用 | 线程池、长生命周期 | 虚拟线程、百万级并发 |

```java
static final ScopedValue<User> CURRENT_USER = ScopedValue.newInstance();

ScopedValue.where(CURRENT_USER, user).run(() -> {
    handleRequest();     // 作用域内可读 CURRENT_USER.get()
});                      // 离开作用域自动失效
```

---

## 12. Q11 — 线程池核心参数与执行流程（6.5 / 10）

### 12.1 原始作答（要点）

- 参数：coreSize、maxSize、queue、policy。
- coreSize 不足时启动核心线程，否则入队；队列满了启动线程直到 maxSize；到达 maxSize 且队列满则触发策略。
- 拒绝策略：丢弃、调用者执行。
- 容量：CPU 密集型设为 CPU 个数；IO 密集型可更多，太大调度效率低，太小发挥不了作用。

### 12.2 评分：6.5 / 10

主干流程正确（本题最重要部分），扣分在参数不全、拒绝策略不完整、容量公式缺推导。

### 12.3 七个参数（补全）

| 参数 | 作用 | 易错点 |
|---|---|---|
| `corePoolSize` | 核心线程数，常驻 | **默认懒创建**；需 `allowCoreThreadTimeOut(true)` 才会计时回收 |
| `maximumPoolSize` | 最大线程数 | **只有队列满了才可能突破 core**；队列无界时该参数失效 |
| `keepAliveTime` | 超出 core 的空闲线程存活时间 | 默认**只对非核心线程生效** |
| `unit` | 时间单位 | `TimeUnit.SECONDS` |
| `workQueue` | 任务队列 | 队列类型直接决定线程池行为 |
| `threadFactory` | 线程工厂 | **生产必须自定义**，否则 jstack 全是 `pool-1-thread-1` |
| `handler` | 拒绝策略 | 默认 `AbortPolicy` |

**生产写法**：

```java
ThreadPoolExecutor pool = new ThreadPoolExecutor(
    8, 16, 60L, TimeUnit.SECONDS,
    new ArrayBlockingQueue<>(200),                 // 有界队列
    new ThreadFactory() {
        private final AtomicInteger idx = new AtomicInteger(1);
        @Override public Thread newThread(Runnable r) {
            Thread t = new Thread(r, "order-worker-" + idx.getAndIncrement());
            t.setUncaughtExceptionHandler((th, e) ->
                log.error("worker {} died", th.getName(), e));
            return t;
        }
    },
    new ThreadPoolExecutor.CallerRunsPolicy()
);
```

### 12.4 执行流程（四步 + 二次检查）

```java
public void execute(Runnable command) {
    int c = ctl.get();
    // ① 工作线程数 < corePoolSize → 直接新建核心线程
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true)) return;
        c = ctl.get();
    }
    // ② RUNNING 且入队成功
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        // ②' 入队后二次检查：池已关闭 → 移除并拒绝
        if (!isRunning(recheck) && remove(command))
            reject(command);
        // ②'' core=0 场景：没有工作线程 → 补一个
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    // ③ 队列满 → 扩容到 max
    else if (!addWorker(command, false))
        // ④ 扩容失败 → 拒绝
        reject(command);
}
```

**两个易漏点**：

1. **入队后必须二次检查**：入队成功但线程池刚好 `shutdown`，任务会永远躺在队列里 → 必须 `remove` + 拒绝。
2. **core=0 边界**：任务入队却无工作线程，需 `addWorker(null, false)` 兜底。

**队列类型决定行为**：

| 队列 | 行为 |
|---|---|
| `SynchronousQueue` | 不存储，直接交给线程；配合大 max 即 `newCachedThreadPool` |
| `ArrayBlockingQueue(n)` | **有界**，推荐 |
| `LinkedBlockingQueue()` 无参 | **无界**（`Integer.MAX_VALUE`）→ **maxPoolSize 永远用不上**，可能 OOM |
| `PriorityBlockingQueue` | 优先级任务，无界 |
| `DelayQueue` | 延时任务 |

> **经典坑**：`Executors.newFixedThreadPool(n)` 用无界 `LinkedBlockingQueue`；`newCachedThreadPool` 的 max 是 `Integer.MAX_VALUE`。阿里规范禁止用 `Executors` 工厂方法，要求手动构造。

### 12.5 四种拒绝策略

| 策略 | 行为 | 适用 |
|---|---|---|
| `AbortPolicy`（默认） | 抛 `RejectedExecutionException` | 需要感知失败、快速失败 |
| `CallerRunsPolicy` | **调用者线程自己执行** | 天然背压（会阻塞提交线程） |
| `DiscardPolicy` | **静默丢弃** | 允许丢任务（如埋点） |
| `DiscardOldestPolicy` | 丢弃队列最老的，再重试 | 只关心最新数据 |

> 细节：`CallerRunsPolicy` 在池已 `shutdown` 时**不执行任务**（源码判断 `!e.isShutdown()`），直接丢弃。

### 12.6 容量设置

| 类型 | 公式 | 例（8 核） |
|---|---|---|
| CPU 密集型 | `N + 1` | 9 |
| IO 密集型 | `N × (1 + W/C)` | 等待占 90% → 80 |

更工程化（Little's Law）：

```
所需线程数 = 目标 QPS × 单请求平均耗时(s)
```

**不能太大**：上下文切换、每线程栈 1MB、缓存局部性差、下游被打垮。
**不能太小**：CPU 空闲、IO 等待无人补位、队列堆积 → 延迟飙升 → 拒绝。

> 2026 视角：IO 密集型优先用虚拟线程（`newVirtualThreadPerTaskExecutor()`），不再靠调大线程数掩盖阻塞；CPU 密集型仍用固定大小平台线程池。

---

## 13. Q12 — 优雅停机（3 / 10）

### 13.1 原始作答（要点）

- `shutdown` 等待正在执行的任务完成后才真正关闭；`shutdownNow` 立即关闭，正在执行的任务丢弃。
- 优雅停机实现：不清楚。
- `shutdownNow` 返回未执行的任务。
- Spring Boot `server.shutdown=graceful` 与线程池的关系：不清楚。

### 13.2 评分：3 / 10

第 1 问有一处关键错误（`shutdownNow` 不是丢弃而是中断），其余基本空白。

### 13.3 shutdown vs shutdownNow（修正）

| | `shutdown()` | `shutdownNow()` |
|---|---|---|
| 接收新任务 | ❌ 拒绝（走拒绝策略） | ❌ 拒绝 |
| 队列中未执行任务 | ✅ 继续执行完 | ❌ 取出并返回，不执行 |
| 正在执行任务 | ✅ 等它执行完 | ⚠️ 调用 `Thread.interrupt()` 打断 |
| 是否阻塞等待 | ❌ 立即返回 | ❌ 立即返回 |
| 池状态 | `SHUTDOWN` | `STOP` |
| 返回值 | 无 | `List<Runnable>` |

**关键**：`interrupt()` 不保证线程停下。

```java
// 能被中断
try { Thread.sleep(10000); } catch (InterruptedException e) { ... }

// 不能被中断：CPU 忙循环不检查中断标志
while (true) { compute(); }
```

所以任务必须自己响应中断：

```java
while (!Thread.currentThread().isInterrupted() && hasWork()) {
    doWork();
}
```

> `shutdown()` 只是**发起**关闭，不等待；等待必须靠 `awaitTermination`。

### 13.4 优雅停机标准模板

```java
public static void shutdown(ExecutorService pool, long timeout, TimeUnit unit) {
    pool.shutdown();                       // ① 拒绝新任务，队列继续执行
    try {
        if (!pool.awaitTermination(timeout, unit)) {
            List<Runnable> dropped = pool.shutdownNow();   // ② 超时强关
            log.warn("force shutdown, {} tasks dropped", dropped.size());
            if (!pool.awaitTermination(5, TimeUnit.SECONDS)) {
                log.error("pool still not terminated, threads may be stuck");
            }
        }
    } catch (InterruptedException e) {
        pool.shutdownNow();                // ③ 自己被中断
        Thread.currentThread().interrupt(); // 恢复中断标志
    }
}
```

| 步骤 | 作用 | 漏了会怎样 |
|---|---|---|
| `shutdown()` | 停止接收新任务 | 关闭期间还在接任务，永远关不掉 |
| `awaitTermination(timeout)` | 有界等待 | 无限阻塞，或没等完就退出 |
| `shutdownNow()` | 超时兜底 | 卡死任务永远不结束 |
| `interrupt()` 恢复 | 保留中断语义 | 上层丢失中断信号 |

**替代方案：毒丸（Poison Pill）**——适合自研 worker 循环，让线程在任务边界干净退出：

```java
private static final Runnable POISON = () -> { };

public void stop() throws InterruptedException {
    queue.put(POISON);   // 每个消费者一颗
    pool.shutdown();
}
```

### 13.5 shutdownNow 返回值为什么不完整

返回的是「已从队列取出、但从未开始执行」的任务。**不包含**：

| 任务状态 | 是否在列表 | 原因 |
|---|---|---|
| 队列中排队、未开始 | ✅ | `drainTo` 取出 |
| 已被 worker 取走但还没 `run()` | ❌ | 已不在队列 |
| 正在执行 | ❌ | 只能中断，无法「返回」 |
| `shutdown` 后被拒绝 | ❌ | 根本没进队列 |
| 自定义队列不支持 `drainTo` | ❌ 可能不在 | 依赖实现 |

所以只能当作**补偿线索**，不能当完整审计账本：

```java
for (Runnable r : dropped) {
    log.warn("task not executed: {}", r);
    // 可转存到持久化队列，重启后重放
}
```

> 若 `Runnable` 是 `FutureTask`，需 `cancel`，否则调用方永远等不到结果。

### 13.6 Spring Boot graceful 管不到线程池

| 层面 | 管什么 | 谁负责 |
|---|---|---|
| Web 容器 | 停止接新请求，等在途 HTTP 完成 | `server.shutdown=graceful` |
| 应用线程池 | 等/中断业务异步任务 | 自己（`@PreDestroy` / `SmartLifecycle`） |
| MQ 消费者 | 停止拉取、处理完在途消息 | 自己 |
| 定时任务 | 停止调度、等当前轮次结束 | 自己 |

**正确做法**：用 `SmartLifecycle` 的 `getPhase()` 控制顺序（**phase 越大，启动越晚、关闭越早**）。

| 组件 | phase |
|---|---|
| MQ 消费者 | 2048（最先停） |
| 业务线程池 / 调度 | 1024 |
| Tomcat | 0 |
| 数据源 / 客户端 | -1024（最后关） |

```java
@Component
public class PoolLifecycle implements SmartLifecycle {
    private volatile boolean running;

    @Override public void start() { running = true; }

    @Override public void stop() {
        GracefulShutdown.shutdown(bizPool, 25, TimeUnit.SECONDS);
        running = false;
    }

    @Override public boolean isRunning() { return running; }
    @Override public int getPhase() { return 1024; }
}
```

**Spring 配置**：

```yaml
server:
  shutdown: graceful
spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s
  task:
    execution:
      shutdown:
        await-termination: true
        await-termination-period: 25s
```

`ThreadPoolTaskExecutor` 必须开：

```java
e.setWaitForTasksToCompleteOnShutdown(true);   // 等价 shutdown() 而非 shutdownNow()
e.setAwaitTerminationSeconds(25);
```

---

## 14. 附录：K8s 优雅停机实操

### 14.1 Pod 删除时序（竞态根源）

```
kubectl delete / rollout / HPA 缩容
        │
        ▼
① API Server 标记 Terminating，写入 deletionTimestamp
        │
        ├────────► ② 异步：从 Endpoints / EndpointSlice 摘除
        │            （kube-proxy / Ingress / Mesh 逐步生效，几百 ms ~ 几秒）
        ▼
③ kubelet 执行 preStop hook
        │
        ▼
④ preStop 结束 → 向容器 PID 1 发 SIGTERM
        │
        ▼
⑤ JVM Shutdown Hook → Spring Context 关闭 → 线程池 shutdown
        │
        ▼
⑥ 应用退出（exit 0）
        │
        └─ 超过 terminationGracePeriodSeconds → ⑦ SIGKILL（exit 137）
```

**② 和 ④ 并行，不保证「流量已摘干净」才发 SIGTERM** → 这就是 502/503 尖刺的来源。**preStop 的作用是等流量摘除，而不是优雅关闭本身。**

### 14.2 第一件：让 SIGTERM 真正到达 JVM

```dockerfile
# ❌ sh -c 会吞掉 SIGTERM
CMD java -jar app.jar

# ❌ 脚本没 exec
CMD ["./start.sh"]

# ✅ exec 形式，PID 1 就是 java
ENTRYPOINT ["java", "-jar", "/app.jar"]

# ✅ 脚本内 exec
#!/bin/sh
exec java -jar /app.jar
```

排查：

```bash
kubectl exec -it <pod> -- ps -o pid,ppid,comm -p 1
# PID 1 必须是 java
```

需要 shell 逻辑时用 `tini`：

```dockerfile
ENTRYPOINT ["/sbin/tini", "--", "/app/start.sh"]
```

**Shutdown Hook 三个陷阱**：

| 陷阱 | 说明 |
|---|---|
| 所有 Hook 并发执行，顺序不保证 | 跨 Hook 不能假设先后，用 `SmartLifecycle` |
| Hook 里再 `System.exit()` | 死锁 |
| Hook 里向同一个池提交任务 | 池已 shutdown → `RejectedExecutionException` |

### 14.3 第二件：先摘流量

**方案 A：preStop sleep（最通用）**

```yaml
lifecycle:
  preStop:
    exec:
      command: ["sh", "-c", "sleep 8"]
```

- 延迟 SIGTERM 8 秒，给 Endpoints / kube-proxy / Ingress / Mesh 留摘除时间；
- 应用此时仍能正常响应 → 滚动发布用户无感；
- **grace period 包含 preStop 时间**。

**方案 B：preStop 先翻 readiness，再等在途归零**

```yaml
lifecycle:
  preStop:
    exec:
      command:
        - sh
        - -c
        - |
          curl -s -X POST localhost:8080/actuator/readiness/refuse || true
          sleep 8
          while [ "$(curl -s localhost:8080/actuator/inflight)" != "0" ]; do sleep 1; done
```

**必须区分 readiness / liveness**：

| 探针 | 失败后果 | 停机期间 |
|---|---|---|
| readiness | 从 Endpoints 摘除（不重启） | 期望失败 → 摘流量 |
| liveness | **重启容器** | 绝对不能失败 |

**常见事故**：liveness 也查 `/health`（含数据库），停机时连接池先关 → 探针失败 → 容器被重启 → 优雅停机被打断。liveness 只查进程是否卡死。

### 14.4 第三件：拒绝策略与关闭竞态

关停期间仍有请求（摘流有延迟），池已 `shutdown()` → 走拒绝策略。

| 策略 | 关停期表现 | 建议 |
|---|---|---|
| `AbortPolicy` | 抛异常 → 上层 503 | ✅ 推荐，客户端可重试 |
| `CallerRunsPolicy` | 调用线程执行 | ⚠️ 拖长关闭，但可接受 |
| `DiscardPolicy` | **静默丢弃** | ❌ 最危险 |
| `DiscardOldestPolicy` | 丢最老任务 | ❌ 不可控 |

```java
new RejectedExecutionHandler() {
    @Override public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        if (e.isShutdown()) {
            metrics.counter("pool.rejected.shutdown").increment();
            throw new RejectedExecutionException("pool shutting down");
        }
        metrics.counter("pool.rejected.saturated").increment();
        throw new RejectedExecutionException("pool saturated");
    }
}
```

配合 HTTP 返回 **503 + `Retry-After`**。

### 14.5 第四件：关闭顺序（上游先于下游）

```
① 摘流量（readiness / preStop）
② 停止接新请求（Tomcat connector pause）
③ 等在途 HTTP 请求完成
④ 停止 MQ 拉取
⑤ 关闭业务线程池（awaitTermination）
⑥ 关闭调度线程池
⑦ 关闭数据源 / Redis / HTTP 客户端
⑧ JVM 退出
```

反例后果：先关数据源 → 线程池任务拿不到连接全报错；先关线程池 → Tomcat 等异步结果超时；MQ 不停 → 处理一半提交 offset 失败 → 重复消费。

### 14.6 完整 Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  template:
    spec:
      terminationGracePeriodSeconds: 60     # > preStop + 应用关闭 + 缓冲
      containers:
        - name: app
          image: registry/order-service:1.2.3
          ports:
            - containerPort: 8080
          env:
            - name: JAVA_TOOL_OPTIONS
              value: "-XX:+ExitOnOutOfMemoryError"
          lifecycle:
            preStop:
              exec:
                command: ["sh", "-c", "sleep 8"]
          readinessProbe:
            httpGet: { path: /actuator/health/readiness, port: 8080 }
            periodSeconds: 2
            failureThreshold: 2
          livenessProbe:
            httpGet: { path: /actuator/health/liveness, port: 8080 }
            periodSeconds: 10
            failureThreshold: 3
          startupProbe:
            httpGet: { path: /actuator/health/liveness, port: 8080 }
            failureThreshold: 30
            periodSeconds: 2
```

**grace 计算**：

```
terminationGracePeriodSeconds
  = preStop(8s)
  + 应用优雅关闭（Tomcat 在途最多 30s）
  + 线程池 awaitTermination(25s，与 Tomcat 重叠，取大者)
  + 缓冲(10s)
  ≈ 60s
```

超时即 SIGKILL，退出码 **137**。

**K8s 补充**：

```yaml
lifecycle:
  preStop:
    exec:
      command: ["sh", "-c", "sleep 5"]   # 等 Service/Endpoint 摘除生效
terminationGracePeriodSeconds: 45
```

### 14.7 验收方法

```bash
# 持续压流量
hey -z 120s -c 50 http://order-service/api/ping

# 另一个终端触发滚动
kubectl rollout restart deployment/order-service
```

| 观察项 | 期望 |
|---|---|
| 5xx 数量 | 0（或极少数可重试 503） |
| 延迟 P99 | 无尖刺 |
| Pod 退出码 | 全 0，无 137 |
| 发布耗时 | ≈ preStop + 关闭耗时，不卡满 30s |

```bash
# 检查退出码
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.containerStatuses[0].lastState.terminated.exitCode}{"\n"}{end}'
```

```java
// 确认 SIGTERM 到达
Runtime.getRuntime().addShutdownHook(new Thread(() ->
    log.info("SIGTERM received at {}", Instant.now())));
```

### 14.8 线上五类故障对照

| 现象 | 根因 | 修复 |
|---|---|---|
| 发布时 502/503 尖刺 | 没 preStop，SIGTERM 早于摘流量 | 加 `preStop sleep 8` |
| 每次发布固定卡 30s | `sh` 没 `exec`，信号没到 JVM | `ENTRYPOINT` 改 exec 形式 |
| exit code 137 | grace 太短 / 任务不响应中断 | 调大 grace、响应中断 |
| 停机时重复消费 MQ | 消费者没先停 | MQ phase 调到最大 |
| 大量 `RejectedExecutionException` | 摘流量没完成就 shutdown | preStop 等待 + 503 重试 |

### 14.9 SmartLifecycle 阶段控制详解

#### 14.9.1 两个接口

```java
public interface Lifecycle {
    void start();
    void stop();
    boolean isRunning();
}

public interface Phased {
    int getPhase();
}

public interface SmartLifecycle extends Lifecycle, Phased {
    int DEFAULT_PHASE = Integer.MAX_VALUE;
    default boolean isAutoStartup() { return true; }
    default void stop(Runnable callback) { stop(); callback.run(); }
}
```

| 能力 | `Lifecycle` | `SmartLifecycle` |
|---|---|---|
| 指定阶段 | ❌（固定 phase = 0） | ✅ `getPhase()` |
| 自动启动 | ✅ | ✅（`isAutoStartup()` 可关） |
| 异步/超时关闭 | ❌ | ✅ `stop(Runnable callback)` |

> `Phased.DEFAULT_PHASE` 是 `0`，而 `SmartLifecycle.DEFAULT_PHASE` 是 `Integer.MAX_VALUE`。不实现 `getPhase()` 的 SmartLifecycle 会**最后启动、最先停止**——这是常见坑。

#### 14.9.2 phase 的唯一规则

> **phase 越小 → 启动越早、停止越晚；phase 越大 → 启动越晚、停止越早。停止顺序是启动顺序的严格逆序。**

```
启动（升序）                        停止（降序）
phase -1000  dataSource            phase 3000  mqConsumer
phase     0  tomcat                phase 2000  bizPool / scheduler
phase  2000  bizPool               phase    0  tomcat
phase  3000  mqConsumer            phase -1000  dataSource
```

记忆口诀：**越靠近用户/上游的，越晚启动、越早停止。**

- 启动：DB → 连接池 → 业务池 → MQ → Web
- 停止：先断入口（MQ、Web）→ 停业务 → 关底层

#### 14.9.3 执行引擎 DefaultLifecycleProcessor

**启动（onRefresh）**：

```
ContextRefreshedEvent
   ↓
收集所有 Lifecycle Bean，按 phase 分组
   ↓
for phase in 升序:
    for bean in 该组:
        if (bean.isAutoStartup() && !bean.isRunning()) bean.start()
    等该组全部完成后，再进入下一个 phase   ← 组间严格串行
```

**停止（onClose）**：

```
ContextClosedEvent
   ↓
for phase in 降序:
    该组并行 stop（调用 stop(Runnable callback)）
    等待所有 callback，或超时（spring.lifecycle.timeout-per-shutdown-phase）
```

三个事实：

1. **同一 phase 内并行，顺序不保证**；要顺序必须用不同 phase。
2. **停止是等 callback**，不是等 `stop()` 返回，所以支持异步关闭。
3. **超时只记日志、不强杀**，之后继续下一 phase。

#### 14.9.4 phase 数值选择

Spring 自身：Web 容器（Tomcat/Netty）使用接近 `Integer.MAX_VALUE` 的 phase，所以**最后启动、最先停止**（先停止接流量）。具体常量随版本变化，不要硬编码卡位。

推荐分层区间：

| 层 | phase | 组件 |
|---|---|---|
| 入口层 | `3000` | MQ 消费者、Kafka listener、WebSocket 会话 |
| 业务层 | `2000` | 业务线程池、`@Async` 执行器、调度器 |
| 缓存层 | `1000` | 本地缓存刷新、分布式锁释放 |
| 基础层 | `-1000` | 数据源、Redis 客户端、HTTP 客户端、注册中心 |

留 1000 间隔，便于以后插入新组件。

#### 14.9.5 完整示例

```java
// ① MQ 消费者（最先停）
@Component
public class MqConsumerLifecycle implements SmartLifecycle {
    private final MessageListenerContainer container;
    private volatile boolean running;

    @Override public void start() {
        container.start();
        running = true;
        log.info("[lifecycle] MQ consumer STARTED");
    }

    @Override public void stop() {
        container.stop();            // 停止拉取，等在途消息处理完
        running = false;
        log.info("[lifecycle] MQ consumer STOPPED");
    }

    @Override public boolean isRunning() { return running; }
    @Override public int getPhase() { return 3000; }     // 最大 → 最先停
}
```

```java
// ② 业务线程池（异步关闭 + callback）
@Component
public class BizPoolLifecycle implements SmartLifecycle {
    private final ThreadPoolExecutor pool;
    private volatile boolean running;

    @Override public void start() {
        running = true;
        log.info("[lifecycle] biz pool STARTED, core={}", pool.getCorePoolSize());
    }

    @Override
    public void stop(Runnable callback) {
        new Thread(() -> {
            try {
                GracefulShutdown.shutdown(pool, 25, TimeUnit.SECONDS);
                log.info("[lifecycle] biz pool STOPPED");
            } catch (Throwable e) {
                log.error("[lifecycle] biz pool stop failed", e);
            } finally {
                running = false;
                callback.run();      // ⚠️ 必须调用，否则容器等到超时
            }
        }, "biz-pool-shutdown").start();
    }

    @Override public void stop() { stop(() -> {}); }
    @Override public boolean isRunning() { return running; }
    @Override public int getPhase() { return 2000; }
}
```

```java
// ③ 调度器（同层，可与业务池并行停）
@Component
public class SchedulerLifecycle implements SmartLifecycle {
    private final ScheduledExecutorService scheduler;
    private volatile boolean running;

    @Override public void start() {
        running = true;
        log.info("[lifecycle] scheduler STARTED");
    }

    @Override
    public void stop() {
        scheduler.shutdown();
        try {
            if (!scheduler.awaitTermination(10, TimeUnit.SECONDS)) {
                scheduler.shutdownNow();
            }
        } catch (InterruptedException e) {
            scheduler.shutdownNow();
            Thread.currentThread().interrupt();
        }
        running = false;
        log.info("[lifecycle] scheduler STOPPED");
    }

    @Override public boolean isRunning() { return running; }
    @Override public int getPhase() { return 2000; }
}
```

```java
// ④ 数据源（最后停）
@Component
public class DataSourceLifecycle implements SmartLifecycle {
    private final HikariDataSource ds;
    private volatile boolean running;

    @Override public void start() {
        running = true;
        log.info("[lifecycle] datasource STARTED");
    }

    @Override public void stop() {
        ds.close();
        running = false;
        log.info("[lifecycle] datasource STOPPED");
    }

    @Override public boolean isRunning() { return running; }
    @Override public int getPhase() { return -1000; }    // 最小 → 最后停
}
```

**期望日志**：

```
启动：datasource → scheduler/biz pool → MQ consumer → Tomcat
停止：Tomcat → MQ consumer → biz pool/scheduler → datasource
```

若 `biz pool STOPPED` 出现在 `datasource STOPPED` 之后，说明 phase 配反了，任务会拿不到连接。

#### 14.9.6 与 @PreDestroy / DisposableBean / ContextClosedEvent 的关系

```
context.close()
   │
   ├─① 发布 ContextClosedEvent
   │     → @EventListener(ContextClosedEvent.class)，受 @Order 控制
   │     → 此时线程池还活着
   │
   ├─② lifecycleProcessor.onClose()
   │     → SmartLifecycle.stop()，按 phase 降序  ← 正确的关闭位置
   │
   ├─③ destroyBeans()
   │     → @PreDestroy / DisposableBean / destroy-method
   │     → 按依赖逆序，跨 Bean 顺序不可控
   │
   └─④ 关闭 BeanFactory
```

| 方式 | 时机 | 能否控顺序 | 用途 |
|---|---|---|---|
| `ContextClosedEvent` 监听 | 最早 | 仅 `@Order`，粒度粗 | 打点、告警、通知外部系统 |
| `SmartLifecycle` | 中 | ✅ 按 phase 精确控制 | 线程池/MQ/数据源关闭顺序 |
| `@PreDestroy` / `DisposableBean` | 最后 | 仅依赖关系 | 单 Bean 资源释放 |

> 结论：**要顺序用 `SmartLifecycle`；单 Bean 清理用 `@PreDestroy` 即可。**

#### 14.9.7 六个常见坑

| # | 坑 | 后果 | 解法 |
|---|---|---|---|
| 1 | 重写 `stop(Runnable)` 忘调 `callback.run()` | 容器等到超时，日志报 `Failed to shut down ... within timeout` | `finally { callback.run(); }` |
| 2 | `isRunning()` 恒为 false | **stop 根本不会被调用** | 用 `volatile boolean` 维护 |
| 3 | 不写 `getPhase()` | 默认 `Integer.MAX_VALUE` → 最后启动、最先停止 | 显式声明 |
| 4 | 同 phase 内想控制顺序 | 并行执行，顺序不保证 | 用不同 phase，间隔 ≥ 100 |
| 5 | `stop()` 里向已关闭的池提交任务 | `RejectedExecutionException` | 停止逻辑不依赖被停的池 |
| 6 | `stop(Runnable)` 里同步长阻塞 | 拖住整个关闭流程 | 异步执行 + callback |

#### 14.9.8 超时配置与行为

```yaml
spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s   # 默认 30s，每个 phase 独立计时
```

超时后：

1. 记 `ERROR`：`Failed to shut down 1 bean with phase value ... within timeout of 30000ms`；
2. **不中断线程、不 kill**；
3. 继续下一个更小 phase；
4. 最终可能被 K8s `terminationGracePeriodSeconds` `SIGKILL`。

约束关系：**phase 内总耗时 ≤ timeout ≤ grace period − preStop**。

#### 14.9.9 验证顺序真的生效

1. 开 DEBUG 日志：

```yaml
logging:
  level:
    org.springframework.context.support.DefaultLifecycleProcessor: DEBUG
```

会输出 `Starting beans in phase 2000` / `Stopping beans in phase 3000`。

2. 在 start/stop 里记录耗时：

```java
long begin = System.nanoTime();
// ...
log.info("[lifecycle] {} stopped in {} ms", name,
         (System.nanoTime() - begin) / 1_000_000);
```

3. 压测 + 滚动重启，确认顺序固定、5xx 为 0。

#### 14.9.10 什么时候不需要

| 场景 | 建议 |
|---|---|
| 只有一个线程池 | `@PreDestroy` + `shutdown/awaitTermination` 足够 |
| 需要跨组件顺序（MQ 先、DB 后） | ✅ `SmartLifecycle` |
| 需要异步/并行关闭 | ✅ `SmartLifecycle.stop(Runnable)` |
| 只想关闭时打日志告警 | `ContextClosedEvent` 监听 |

---

## 15. 剩余题库计划（Q23–Q30）

| 题号 | 主题 | 难度 |
|---|---|---|
| Q23 | `CompletableFuture` 高级：allOf/anyOf、异常传播链、超时控制（`orTimeout`/`completeOnTimeout`） | 难 |
| Q24 | `ThreadLocal` 内存泄漏深度分析：ThreadLocalMap 结构、Entry 弱引用、remove 时机、虚拟线程下残留问题 | 中 |
| Q25 | `ScheduledThreadPoolExecutor`：延时队列、周期任务累积延迟、固定延迟 vs 固定频率、线程泄漏 | 中 |
| Q26 | `CountDownLatch`、`CyclicBarrier`、`Semaphore`、`Exchanger` 原理与选型 | 中 |
| Q27 | `CopyOnWriteArrayList` / `CopyOnWriteArraySet`：读写分离、最终一致性、适用与不适用 | 中 |
| Q28 | `StampedReader`/`StampedWriter`（乐观读锁、模式升级、不可重入陷阱） | 难 |
| Q29 | `Phaser`：分阶段同步、动态注册/取消、树状调度器 | 难 |
| Q30 | 并发压测方法论：wrk/hey/JMeter、TPS/QPS/P99/P999、Little's Law、压测前后对比 | 中 |
| Q14 | 死锁：四个条件、检测（jstack/jcmd）、避免策略 | 中 |
| Q15 | 伪共享与 `@Contended`；缓存行对齐 | 难 |
| Q16 | 无锁队列 / Disruptor / MPSC 设计 | 难 |
| Q17 | 虚拟线程（Java 21）：原理、pinning、适用与不适用 | 难 |
| Q18 | Structured Concurrency / `ScopedValue`（预览特性） | 难 |
| Q19 | `ForkJoinPool` 与 `parallelStream` 的坑（公共池、阻塞任务） | 中 |
| Q20 | `ConcurrentHashMap` 1.8 原理：CAS + synchronized、扩容、size | 难 |
| Q21 | `BlockingQueue` 家族对比与选型 | 中 |
| Q22 | 并发 bug 排查：jstack、JFR、jcstress、Arthas | 中 |

---

## 17. 高频易错清单（复习卡）

**必须背下来的 20 句**

1. `volatile` = 可见性 + 有序性（acquire/release），**无原子性、无互斥**。
2. `count++` 是 read-modify-write，`volatile` 救不了 → `synchronized` 或 CAS。
3. volatile 写贵的本质是 **StoreLoad 屏障排空 store buffer**；x86 读天然 acquire 所以免费。
4. **不要问「谁先执行」，要问「有没有 HB 边」。**
5. 无 HB 边 + 至少一个写 = **data race**，结果不可预测。
6. HB 的主要来源：锁、volatile、`start/join`、`submit/get`、`put/take`、`final`。
7. final 字段靠 **freeze** 保证安全发布，freeze 是**逻辑动作**，不是指令。
8. `this` 在构造器内逸出 → final 保证失效。
9. DCL 的 bug 在**锁外那次乐观读**，修复靠 `volatile`；生产首选 Holder 或枚举。
10. `volatile int[]` 只保引用，不保元素；元素用 `AtomicIntegerArray` / `VarHandle`。
11. `wait()` 必须在锁内，因为它要**原子地「释放锁 + 挂起」**，否则丢失唤醒；等待永远用 `while` 防虚假唤醒。
12. `Condition` 的价值是**多个等待集 → 精准唤醒**，不是「支持 lock/unlock」。
13. `ReentrantLock.unlock()` 是 **state 递减，减到 0 才释放**；必须写在 `finally`。
14. AQS：`state` + FIFO 队列；`ReentrantLock`=重入次数、`Semaphore`=许可数、`CountDownLatch`=剩余计数。
15. `LongAdder` 没有消灭 CAS，而是**把竞争分摊到多个 Cell**，`sum()` 时再汇总。
16. `ThreadLocalMap` 的 key 是 **ThreadLocal 实例的弱引用**，value 是强引用 → 不 `remove()` 就泄漏。
17. 虚拟线程下 `ThreadLocal` 会残留在**平台线程**（OS 线程）上，造成跨虚拟线程脏数据。
18. 线程池顺序：**core → 队列 → max → 拒绝**；队列无界则 `maxPoolSize` 形同虚设。
19. `shutdown()` 干完手上的活；`shutdownNow()` 退回队列 + **中断**在跑的，中断不等于停止。
20. K8s 优雅停机 = **preStop 等摘流量 → SIGTERM → shutdown → awaitTermination → 超时强关**，grace 必须覆盖全程。

**常见错误说法（会被扣分）**

| ❌ 错误 | ✅ 正确 |
|---|---|
| volatile 修改后立刻 flush 到主存 | 建立 happens-before，禁止重排 |
| volatile 能保证 `i++` 安全 | 不能，复合操作 |
| `AtomicInteger[]` 元素原子 | 应用 `AtomicIntegerArray` |
| DCL 加了 synchronized 就安全 | 锁外读没有 HB，仍不安全 |
| final 只是不能修改 | 还保证安全发布（freeze） |
| 偏向锁是当前默认优化 | JDK 15 起已禁用/移除 |
| 用 ConcurrentHashMap 就完全线程安全 | 单次操作安全，复合操作仍需同步 |
| `unlock()` 会把 state 重置为 0 | 递减，减到 0 才释放；重入时不会归零 |
| `Condition` 只是「能 lock/unlock」 | 多个等待集 + 精准唤醒 + 超时/公平/诊断 |
| `ThreadLocalMap` 的 key 是 threadId | key 是 ThreadLocal 实例（弱引用） |
| 弱引用能自动解决 ThreadLocal 泄漏 | 只回收 key，value 仍强引用，必须 `remove()` |
| 偏向锁用 hash 位存线程 ID 是 bug | 有 hash 的对象不允许被偏向；锁态间 hash 存在 Lock Record / Monitor 里 |
| `LongAdder` 不用 CAS | 每个 Cell 内部仍是 CAS，只是分散竞争 |
| `shutdownNow` 会丢弃正在执行的任务 | 是**中断**它们；不响应中断的线程仍会继续跑 |
| `shutdown()` 会阻塞等线程结束 | 它立即返回，等待要靠 `awaitTermination` |
| 配了 `server.shutdown=graceful` 就万事大吉 | 只覆盖 HTTP 在途请求，线程池/MQ/调度要自己管 |
| K8s 发 SIGTERM 时流量已摘干净 | Endpoints 摘除是异步的，必须用 preStop 等待 |

---

## 16. Q22 — 并发 Bug 排查：jstack、JFR、jcstress、Arthas（讲解版）

### 16.1 常用并发排查工具概览

| 工具 | 核心定位 | 擅长解决什么问题 |
|---|---|---|
| `jstack` / `jcmd Thread.print` | 生成 Java 线程转储（Thread Dump），显示所有线程的堆栈、状态、锁等待关系 | 死锁检测、线程池打满、线程长期阻塞、CPU 高但不知道在忙什么（配合 `top -H -p pid`） |
| `jcmd` | JDK 自带诊断工具，比 `jstack` 更全面 | 导出堆外内存（`GC.heap_info`）、查看 JVM 参数（`VM.flags`）、执行线程转储、`GC.run` 等 |
| Arthas (Alibaba) | 线上 Java 诊断神器，无需重启应用即可 attach | 实时看线程状态（`thread`）、看方法调用耗时（`trace`）、看哪个线程占 CPU 高（`thread -b`）、反编译确认代码（`jad`）、监控热点方法（`watch`） |
| JFR (Java Flight Recorder) | JDK 自带的飞行记录仪，低开销持续采集运行数据 | 事后分析「为什么在某个时间点变慢了」、GC 停顿、锁竞争、线程阻塞事件、代码热点 |
| jcstress | OpenJDK 官方并发压力测试工具 | 验证并发原语是否正确（如自写无锁代码、Volatile 是否真的能防止重排序），属于「事前验证」 |
| JOL (Java Object Layout) | 查看 Java 对象在内存中的真实布局 | 排查伪共享（False Sharing）、对象头、对齐填充导致的性能问题 |

### 16.2 `jstack` 线程状态详解

| 状态 | 含义 | 对应场景 |
|---|---|---|
| `RUNNABLE` | 线程在 JVM 层面是可运行的（可能正在 CPU 上执行，也可能正在等 IO） | CPU 占用高的线程通常是这个状态。看堆栈定位是正在做计算，还是卡在网络/磁盘 IO 上。 |
| `BLOCKED` | 线程正在**等待进入 `synchronized` 同步块**，被另一个持有锁的线程挡住了 | 怀疑锁竞争或死锁时重点看。 |
| `WAITING` | 线程在无限等待某个条件，不会自动醒来 | 调用了 `Object.wait()`（无参数）、`Thread.join()`（无超时）、`LockSupport.park()`。典型场景：线程池里的空闲线程在等任务。 |
| `TIMED_WAITING` | 带超时的等待，超时后会自动醒来 | `Thread.sleep()`、`Object.wait(timeout)`、`LockSupport.parkNanos()`、`ArrayBlockingQueue.poll(timeout)` 等。 |

**重点关注**：
- **怀疑死锁**：搜索 `Found one Java-level deadlock:`；或者找一堆 `BLOCKED` 线程，它们的堆栈里互相 `locked <0xXXXX>` 和 `waiting to lock <0xYYYY>`，形成环。
- **线程池打满**：大量线程名是 `pool-X-thread-Y`，且状态大多是 `RUNNABLE`（正在干活）或 `BLOCKED`（在等锁）。
- **线程泄漏**：线程数一直上涨，大量 `TIMED_WAITING` 在 `Object.wait()` 上，但业务上已经没人用这些线程了。

> `WAITING` 和 `TIMED_WAITING` 本身不是问题（线程池空闲线程就是这样），问题在于**大量业务线程卡在这些状态上不动**。

### 16.3 死锁 vs 活锁 vs 饥饿：怎么区分？

| 现象 | 本质 | `jstack` 里长什么样 | 是否会自动好转 |
|---|---|---|---|
| **死锁（Deadlock）** | 两个或多个线程互相持有对方需要的锁，永远等下去 | 多个线程状态为 `BLOCKED`，且堆栈里形成**环形锁等待**。`jstack` 有时会直接打印 `Found one Java-level deadlock:`。 | 不会，必须人为干预。 |
| **活锁（Livelock）** | 线程不断重试但始终无法成功，CPU 被浪费在无效循环里 | 线程状态通常是 `RUNNABLE`，但堆栈里不断出现 `compareAndSwap` 失败重试、`while(true)` 自旋、重复抛异常重试等。 | 不会，除非竞争条件消失。 |
| **线程饥饿（Starvation）** | 某个线程长期拿不到 CPU 时间片或锁，被其他线程一直抢走 | 某个特定业务线程长期处于 `BLOCKED` 或 `TIMED_WAITING`，而其他同类线程正常运行。 | 可能会好转，也可能永远得不到机会。 |

**快速口诀**：`BLOCKED` + 互相等锁 → **死锁**；`RUNNABLE` + 空转重试 → **活锁**；某个线程一直 `BLOCKED` 而其他都在跑 → **饥饿**。

### 16.4 Arthas 排查示例

场景：线上接口 QPS 骤降、CPU 飙高。

| 步骤 | 命令 | 作用 |
|---|---|---|
| 1 | `thread -b` | 找到当前阻塞其他线程最多的线程（持有锁不释放的「罪魁祸首」），以及 CPU 占用最高的线程。 |
| 2 | `thread --states` | 统计有多少 `RUNNABLE`、`BLOCKED`、`WAITING`、`TIMED_WAITING`。`BLOCKED` 特别多说明锁竞争严重。 |
| 3 | `thread <线程ID>` | 把那个 CPU 高或阻塞的线程完整堆栈打出来，看卡在哪个类、哪个方法、哪一行。 |
| 4 | `trace com.example.OrderService placeOrder` | 看到方法内部调用了哪些子方法、每个方法耗时多久、哪一步在慢。 |
| 5 | `watch com.example.OrderService placeOrder "{params, returnObj, #cost}"` | 实时看到入参、返回值、执行耗时，判断是不是某些特定参数导致的慢查询或死循环。 |

**常用命令速记**：`thread -b`（扣 CPU/找阻塞者）→ `thread --states`（看整体状态分布）→ `thread <id>`（看具体堆栈）→ `trace`（看方法链路）→ `watch`（看入参返回值）。

### 16.5 生产排查标准流程

1. **先看现象**：CPU 高？还是响应变慢？还是接口超时报错？
2. **抓线程转储**：`jstack -l pid > dump.txt`，或直接用 Arthas `thread`。
3. **定位异常线程**：找 `BLOCKED` 线程（锁竞争/死锁），找 `RUNNABLE` 且堆栈在忙计算（CPU 高），找大量 `TIMED_WAITING`（可能在等下游超时）。
4. **验证死锁**：`jstack` 有没有打印 `Found one Java-level deadlock`；有的话直接看死锁涉及的线程和锁对象地址。
5. **用 Arthas 实时确认**：`thread -b` 看谁在挡路，`trace` 看哪个方法慢。
6. **如果需要复现/验证并发原语是否有问题**：用 `jcstress` 写压力测试，跑在本地或 CI 里验证。
7. **如果需要事后分析某个时间段发生了什么**：开启 `JFR` 录制，事后用 JMC 或 `jfr` 命令分析锁事件、线程阻塞事件、GC 事件。
8. **出结果并修复**：不是瞎猜，是有堆栈证据的。

### 16.6 速记版

| 问题 | 一句话回答 |
|---|---|
| **常用工具** | `jstack` 看线程堆栈，`Arthas` 线上实时诊断，`JFR` 事后飞行记录分析，`jcstress` 事前并发验证，`JOL` 看对象内存布局。 |
| **`jstack` 状态** | `RUNNABLE`（正在跑/等 IO）、`BLOCKED`（等 `synchronized` 锁）、`WAITING`（无限等待）、`TIMED_WAITING`（带超时等待）。 |
| **死锁 vs 活锁 vs 饥饿** | 死锁：`BLOCKED` 互相等锁，不会好；活锁：`RUNNABLE` 空转重试，CPU 被浪费；饥饿：某线程长期拿不到资源。 |
| **Arthas 常用命令** | `thread -b`（找阻塞者/高 CPU）、`thread --states`（看状态分布）、`thread <id>`（看堆栈）、`trace`（看方法链路耗时）、`watch`（看入参返回值）。 |

**核心加分点**：排查并发 Bug 最忌讳「猜」。有了 `jstack` 或 Arthas 的线程堆栈，你就有证据说「线程卡在第几行、在等哪把锁、在等哪个下游接口」，修复起来才有把握。

---

## 17. 下一步建议

**中期（两周内）**
1. 补齐三大支柱：**JMM（HB/freeze/安全发布）→ 同步原语（monitor/AQS/CAS）→ 工具（线程池/并发容器/CompletableFuture）**。
2. 用 **jcstress** 亲手跑 3 个实验：DCL、`x=1;flag=true`、final 字段发布。
3. 用 **JOL** 打印对象头，观察 Mark Word 状态变化。
4. 用 **jstack** 制造并分析一次死锁，一次线程池耗尽。

**长期（面试表达）**
- 每个知识点按「**是什么 → 为什么 → 怎么用 → 踩过什么坑**」四段式组织。
- 准备 2 个真实的生产案例（优雅关闭 flag、配置热更新 / DCL、线程池耗尽）。
- 不要用「感觉安全」这类表述；答案要有**判据**（HB 边、原子性、可见性、互斥）。

---

*文档由模拟面试会话整理生成 · 2026-10-27*
