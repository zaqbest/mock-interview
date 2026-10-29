# 模拟面试：缓存专项（Java + Redis + 系统设计）

> 候选人定位：高级 Java 开发（10年+）
> 面试官角色：面试官
> 评分标准：每题满分 10 分，给分 + 点评 + 常见踩坑点与避免方案

---

## 题目清单与得分汇总

| 题号 | 题目 | 得分 | 核心考点 |
|:----:|------|:----:|---------|
| Q1 | 缓存价值与引入复杂性 | 7/10 | 性能提升来源、一致性/可用性复杂性 |
| Q2 | 缓存穿透/击穿/雪崩 | 7/10 | 三个概念区分、防御手段（Bloom Filter、延迟双删、TTL+jitter） |
| Q3 | 缓存与数据库一致性（Cache-Aside） | 8/10 | 先删缓存 vs 先更新 DB、延迟双删、版本号、Binlog/CDC |
| Q4 | Redis 高可用与集群架构 | 5/10 | 单机/主从/哨兵/Cluster 选型、Lettuce vs Jedis、命中率排查 |
| Q5 | 秒杀系统缓存层设计 | 8/10 | Lua 原子扣减、Kafka 异步回写、DB 乐观锁、逻辑过期 |
| Q6 | 缓存不一致的检测与补偿 | 6/10 | 预扣减+确认、超时补偿、幂等、死信队列 |
| Q7 | Redis 分布式锁与可靠性 | 7/10 | SETNX 非原子、SET ... NX EX、Lua 解锁、watchdog、时钟漂移、Redlock |
| Q8 | Redis 持久化与缓存恢复 | 6/10 | RDB/AOF 机制、冷启动预热、脏数据发现与修复 |
| Q9 | 集群扩容与本地缓存一致性 | 7/10 | Redis Cluster vs 客户端分片、热点 Key、Caffeine 二级缓存失效通知 |
| Q10 | Redis Pub/Sub 失效通知是否丢失 | 讨论 | Pub/Sub 无持久化、会丢消息、需 TTL 兜底 + 版本号自我纠错 |
| Q11 | 缓存穿透与大 Key | 6/10 | 空值短 TTL、Bloom Filter、--bigkeys、Hash 分桶、List 分页 |
| Q12 | 缓存预热与雪崩兜底 | 7/10 | 预热场景（冷启动/秒杀前/定期）、TTL+jitter、本地缓存、熔断降级 |
| Q13 | 缓存读写一致性工程权衡 | 6/10 | 强/最终/弱一致性、Cache-Aside 并发时序、延迟双删、Binlog/CDC、秒杀库存方案 |
| Q14 | 缓存架构选型与客户端 | 8/10 | 哨兵 vs Cluster 选型、Jedis/Lettuce/Redisson 区别、Key 设计规范 |

**综合得分：约 7.3 / 10**  
**定位：高级 Java 后端开发（中高级偏上）**

---

## Q1：为什么要在系统中引入缓存？

### 参考答案

**核心价值**：
1. **降低 IO 延迟**：Redis 在内存中，响应时间通常在亚毫秒级，远快于磁盘上的数据库（毫秒级甚至更高）。
2. **减轻数据库压力**：缓存命中后不需要建立数据库连接、不需要 SQL 解析和执行计划、不需要行锁/表锁，数据库吞吐量和连接数得到释放。
3. **降低成本**：用廉价的内存缓存承载大量读请求，可减少数据库扩容和只读实例的数量。

**引入缓存后的新复杂性**：
- **数据一致性**：缓存与数据库可能出现不一致，需要 Cache-Aside、延迟双删、Binlog 等方案兜底。
- **缓存穿透/击穿/雪崩**：无效 Key 穿透、热点 Key 过期、大面积 Key 同时过期会击穿数据库。
- **缓存失效策略**：TTL 设置过短导致命中率下降，过长导致脏数据保留时间过长。
- **序列化与客户端选择**：Java 对象序列化成本、Jedis/Lettuce/Redisson 的选型。
- **可用性**：Redis 宕机后是否打穿数据库，需要熔断和本地缓存兜底。

### 得分要点
- 提到降低 IO 延迟、减轻 DB 压力：+3
- 提到一致性、可用性等复杂性：+3
- 提到避免 SQL 解析、连接开销等额外性能来源：+2
- 提到序列化成本、客户端选项：+2

### 常见踩坑点与避免
1. **只关注“内存快于磁盘”**：忽略了避免数据库连接建立、SQL 解析、锁竞争等额外开销。
2. **缓存不设置 TTL 或 TTL 过长**：脏数据长期驻留，业务变更后用户长期读到旧值。

---

## Q2：缓存穿透、缓存击穿、缓存雪崩

### 参考答案

| 现象 | 定义 | 常见防御手段 |
|------|------|-------------|
| **缓存穿透** | 请求访问一个**不存在**的 Key，缓存和数据库都查不到，请求穿透到 DB | 1. 缓存空值/NULL 值并设短 TTL（如 30~120 秒）；2. **Bloom Filter** 在缓存前拦截不存在的 Key；3. 对非法 Key 做参数校验和限流 |
| **缓存击穿** | 热点 Key 过期瞬间，大量并发请求直接打 DB | 1. **逻辑过期/永不过期**：Key 物理不过期，value 自带逻辑过期时间，过期后仅一个请求异步重建，其他返回旧值；2. **互斥锁/Single Flight**：只有一个请求回源，其他等待或返回旧值；3. 将热点 Key 设为永不过期或极长 TTL |
| **缓存雪崩** | 大量 Key **同时过期**，或 Redis 整体不可用，所有请求打 DB | 1. **TTL + jitter（随机偏移）**：避免 Key 集体过期；2. **本地缓存（Caffeine）+ Redis 两级缓存**：Redis 挂了或过期时，本地缓存兜底；3. **熔断与限流**：Redis 不可用时直接降级或拒绝请求，保护数据库 |

### 代码示例：逻辑过期/永不过期（防击穿）

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class HotProductCache {
    private Integer stock;
    private long logicExpireTime; // 逻辑过期时间戳
}

// 获取库存
public Integer getStock(Long productId) {
    String key = "seckill:stock:" + productId;
    String json = redisTemplate.opsForValue().get(key);
    
    if (StrUtil.isBlank(json)) {
        // 物理缓存缺失，同步加载
        return loadAndCache(productId, key);
    }
    
    HotProductCache cache = JSONUtil.toBean(json, HotProductCache.class);
    
    // 未逻辑过期，直接返回旧值，无阻塞
    if (System.currentTimeMillis() < cache.getLogicExpireTime()) {
        return cache.getStock();
    }
    
    // 已逻辑过期，尝试抢重建锁，只有一个线程回源
    String lockKey = "rebuild:lock:" + productId;
    Boolean acquired = redisTemplate.opsForValue().setIfAbsent(lockKey, "1", 30, TimeUnit.SECONDS);
    
    if (Boolean.TRUE.equals(acquired)) {
        asyncRebuildCache(productId, key); // 异步重建，主线程直接返回旧值
    }
    
    return cache.getStock();
}

private void asyncRebuildCache(Long productId, String key) {
    CompletableFuture.runAsync(() -> {
        try {
            Integer stock = productService.getStockFromDb(productId);
            HotProductCache newCache = new HotProductCache(
                stock,
                System.currentTimeMillis() + 5 * 60 * 1000
            );
            redisTemplate.opsForValue().set(key, JSONUtil.toJsonStr(newCache));
        } finally {
            redisTemplate.delete("rebuild:lock:" + productId);
        }
    });
}
```

### 得分要点
- 三个概念区分准确：+4
- 穿透：空值短 TTL + Bloom Filter：+2
- 击穿：单请求回源/互斥锁、逻辑过期：+2
- 雪崩：TTL+jitter、本地缓存/熔断：+2

### 常见踩坑点与避免
1. **空值缓存导致脏数据**：业务数据后来创建了，缓存里还在存空值。避免：空值 TTL 极短，业务写入时主动删除空值 Key。
2. **互斥锁死锁或请求堆积**：使用 `tryLock` + 超时，超时后返回降级结果，避免耗尽 Java 线程池。
3. **TTL+jitter 偏移过大**：导致缓存提前过期，命中率下降。建议 jitter 在 TTL 的 5%~15%。
4. **Redis 宕机后雪崩级联**：配合本地缓存（Caffeine/Guava）两级缓存，并开启 Sentinel/Resilience4j 限流熔断。

---

## Q3：缓存与数据库一致性（Cache-Aside）

### 参考答案

**写入顺序问题**：
- **先更新数据库，再删缓存**（推荐用于读多写少）：并发下可能出现短暂脏读（缓存未删除前读到旧值），但可接受；风险在于删除缓存失败会导致长期脏数据。
- **先删缓存，再更新数据库**：并发下会出现**脏回写**——读请求在缓存被删除、数据库尚未更新的窗口期查到旧值并写回缓存，导致脏数据长期驻留。

**延迟双删（Delayed Double Delete）**：
```java
public void updateStock(Long productId, int newStock) {
    redisTemplate.delete("stock:" + productId);        // 第一次删除
    dbMapper.updateStock(productId, newStock);         // 更新数据库
    scheduler.schedule(() -> redisTemplate.delete("stock:" + productId), 1, TimeUnit.SECONDS); // 第二次删除
}
```

**强一致性工程方案**：
1. **版本号/CAS**：缓存和数据库都带 `version` 字段，更新时比对版本，仅高版本覆盖。
2. **分布式锁（Redisson）**：串行化读写，避免并发竞争。
3. **Binlog/CDC（Canal）**：订阅数据库 Binlog，异步删除/更新缓存，实现最终一致性。
4. **TTL 兜底**：即使同步失败，脏数据也会在 TTL 后自动过期。

### 得分要点
- 区分两种写入顺序：+3
- 指出先删缓存的脏回写问题：+3
- 延迟双删、TTL 兜底：+2
- 版本号/CAS、Binlog/CDC：+2

### 常见踩坑点与避免
1. **先删缓存再更新 DB 的脏回写**：避免方案是延迟双删，或读请求加分布式锁，未获锁的读请求读旧值或降级。
2. **先更新 DB 再删缓存的删除失败**：订阅 DB Binlog（Canal）异步重试删除，或消息队列保证最终一致。
3. **延时双删时间设置错误**：等待时间小于主从延迟则无效，过大则缓存空窗期过长。根据实际主从同步延迟监控设置。
4. **版本号方案未处理缓存写入竞争**：缓存中也存版本号，写入前比较版本，仅高版本覆盖（CAS 语义）。

---

## Q4：Redis 高可用与集群架构

### 参考答案

**架构选型**：
- **单机**：开发测试环境，生产环境禁用（单点故障）。
- **主从**：读写分离，适合读多写少、数据量不大的场景；但故障切换需手动或依赖哨兵。
- **哨兵（Sentinel）**：主从 + 自动故障切换，适合容量不大但需要高可用的场景。
- **集群（Cluster）**：数据分布在多个节点（16384 槽），支持线性扩容，适合数据量大、高并发的生产环境。

**Java 客户端**：
- **Jedis**：同步阻塞 IO，连接非线程安全，需配合连接池，适合简单场景。
- **Lettuce**：基于 Netty，异步非阻塞，连接线程安全，一个连接可共享，Spring Boot 默认推荐，高并发首选。
- **Redisson**：在 Lettuce/Jedis 基础上封装分布式锁、限流器、可重入锁等，适合分布式协同场景。

**命中率排查与优化**：
- `INFO stats` 查看 `keyspace_hits/misses` 计算命中率；
- `bigkeys` 分析大 Key，`slowlog` 定位慢命令；
- 优化：热点 Key 预热、本地缓存两级（Caffeine + Redis）、合理 TTL 与 jitter、Key 设计避免大 Key。

### 得分要点
- 架构选型及适用场景：+4
- 客户端区别（Jedis/Lettuce/Redisson）：+3
- 命中率排查方法：+3

### 常见踩坑点与避免
1. **只部署 Redis 单机**：生产环境单点故障。避免：至少主从+哨兵，数据量大用 Cluster。
2. **本地缓存无上限导致 OOM**：使用 Caffeine 设置 `maximumSize` + `expireAfterWrite` + 驱逐策略。
3. **Jedis 高并发线程不安全**：高并发场景首选 Lettuce。
4. **盲目扩大缓存容量提高命中率**：先分析访问日志和 BigKeys，优化 Key 模式和热点数据预热。

---

## Q5：秒杀系统缓存层设计

### 参考答案

**架构设计**：
```
用户请求 → 网关限流 → 本地缓存（Caffeine，L1） → Redis 缓存（L2，逻辑过期） → DB（乐观锁兜底）
                                          ↓
                                    Lua 脚本原子扣减（防超卖）
                                          ↓
                                    Kafka 异步回写 DB + 最终一致
```

**核心代码结构**：

```java
// 库存缓存包装类（带逻辑过期）
@Data
@AllArgsConstructor
@NoArgsConstructor
public class SeckillStockCache {
    private Integer stock;
    private long version;
    private long logicExpireTime;
}

// Lua 脚本：原子扣减
private static final String DEDUCT_STOCK_LUA =
    "local stock = tonumber(redis.call('get', KEYS[1]))\n" +
    "if stock == false then return -1 end\n" +
    "if stock <= 0 then return 0 end\n" +
    "redis.call('set', KEYS[1], tostring(stock - 1))\n" +
    "return stock - 1";

public Result deductStock(Long productId, Long userId) {
    // 1. Bloom Filter 防穿透
    if (!bloomFilter.contains(productId)) return Result.fail("商品不存在");
    
    String stockKey = "seckill:stock:" + productId;
    
    // 2. 逻辑过期检查，触发异步重建，不阻塞
    String json = redisTemplate.opsForValue().get(stockKey);
    if (StrUtil.isNotBlank(json)) {
        SeckillStockCache cache = JSONUtil.toBean(json, SeckillStockCache.class);
        if (System.currentTimeMillis() >= cache.getLogicExpireTime()) {
            tryRebuildAsync(productId, stockKey);
        }
    }
    
    // 3. Lua 原子扣减
    Long result = (Long) redisTemplate.execute(
        new DefaultRedisScript<>(DEDUCT_STOCK_LUA, Long.class),
        Collections.singletonList(stockKey)
    );
    
    if (result == null || result < 0) return Result.fail("系统错误");
    if (result == 0) return Result.fail("库存不足");
    
    // 4. Kafka 异步回写 DB
    kafkaTemplate.send("seckill.stock.deducted", productId.toString(), 
        JSONUtil.toJsonStr(new StockDeductionEvent(productId, userId)));
    
    return Result.success("扣减成功，剩余库存：" + result);
}
```

**Kafka 消费端（DB 最终一致）**：
```java
@KafkaListener(topics = "seckill.stock.deducted", groupId = "seckill-stock-group")
public void consume(String message) {
    StockDeductionEvent event = JSONUtil.toBean(message, StockDeductionEvent.class);
    // DB 乐观锁条件更新
    int updated = productMapper.deductStockOptimistic(event.getProductId(), event.getVersion());
    if (updated > 0) {
        asyncUpdateCache(event.getProductId(), event.getVersion());
    }
}
```

### 得分要点
- Lua 脚本原子扣减防超卖：+3
- Kafka 异步回写 DB：+2
- 逻辑过期防击穿，非阻塞返回旧值：+2
- DB 乐观锁兜底：+1

### 常见踩坑点与避免
1. **Lua 扣减后 DB 回写失败导致不一致**：Kafka 消费失败需重试；DB 更新失败时应回滚缓存或标记脏。
2. **缓存与 DB 双写顺序错误**：高并发下不应先更新 DB 再更新缓存，应以缓存为准快速响应，异步批量回写 DB。
3. **Kafka 消费顺序与幂等**：同一商品多个扣减事件须顺序消费，DB 更新具备幂等（如 event_id 去重）。
4. **缓存穿透（秒杀商品不存在）**：提前校验商品 ID 合法性，或对不存在商品缓存空值短 TTL。

---

## Q6：缓存与 DB 不一致的检测与补偿

### 参考答案

**检测手段**：
- **定期对账任务**：每天凌晨或每小时扫描 Redis 缓存与数据库，对比关键字段（如库存、版本号），发现不一致时告警并修复。
- **监控延迟**：监控 Kafka 消费 lag，消费端处理慢会导致缓存与 DB 长期不一致。

**修复与补偿机制**：
1. **预扣减 + 确认机制**：先在 Redis 预扣库存，扣减结果通过消息通知 DB 确认；若长时间未收到确认，触发补偿，撤销预扣（将库存加回 Redis）。
2. **幂等补偿**：补偿操作需带唯一 ID（order_id + compensate_id），数据库唯一索引去重，避免重复补偿导致库存多找回。
3. **TTL 兜底**：脏缓存设置合理 TTL，自动过期后重新回源。
4. **死信队列（DLQ）**：Kafka 消费失败进入 DLQ，监控报警 + 人工介入。

### 得分要点
- 预扣减 + 确认/补偿思路：+3
- 幂等性（唯一 ID 去重）：+2
- TTL 兜底、死信队列：+1

### 常见踩坑点与避免
1. **补偿操作不幂等**：重试补偿可能重复加回库存。避免：唯一 ID + 数据库唯一索引。
2. **Redis 回写与 DB 更新顺序反了**：补偿时先加回 Redis 再更新 DB，中间崩溃会导致 Redis 库存比 DB 多。避免：以 DB 为准，DB 回滚成功后再恢复 Redis。
3. **消息丢失导致不一致长期不被发现**：Kafka 开启 `acks=all`，消费端失败进入 DLQ，监控报警。

---

## Q7：Redis 分布式锁与可靠性

### 参考答案

**正确实现方式**：
1. **加锁**：使用一条原子命令 `SET key token NX EX milliseconds`，`token` 为唯一标识（如 UUID），`EX` 设置过期时间避免死锁。
   - ❌ 不要使用 `SETNX + EXPIRE` 两条命令，非原子，`SETNX` 成功后实例宕机会导致锁永久有效。
2. **解锁**：使用 **Lua 脚本** 原子校验并删除：
   ```lua
   if redis.call("get", KEYS[1]) == ARGV[1] then
       return redis.call("del", KEYS[1])
   else
       return 0
   end
   ```
   防止误删其他线程已续期后的锁。
3. **看门狗（Watchdog）**：业务执行时间可能超过锁过期时间，需后台线程定期续期（如 Redisson 的看门狗机制），避免锁提前失效导致并发。
4. **时钟漂移（Clock Drift）**：多个 Redis 实例时钟不一致会导致锁实际有效时间不确定，削弱分布式锁可靠性。依赖 NTP 同步时钟。

**Redlock 争议**：
- Redlock 基于多个独立 Redis 实例取多数派成功，理论上提高可靠性。
- 争议点：Martin Kleppmann 等指出，由于时钟漂移、GC 停顿、网络分区，Redlock 无法提供绝对安全的互斥保证。对一致性要求极高的场景，不应仅依赖分布式锁，应配合数据库乐观锁或 Lua 脚本兜底。

### 得分要点
- `SET key token NX EX` 原子加锁：+3
- Lua 脚本原子解锁（校验 token）：+3
- 看门狗续期、时钟漂移：+2
- Redisson  vs 手写：+2

### 常见踩坑点与避免
1. **非原子加锁导致锁丢失**：`SETNX` 成功后 `EXPIRE` 未执行即宕机。避免：使用 `SET ... NX EX` 一条命令。
2. **误删他人锁**：解锁时直接 `DEL key` 可能删掉别人续期后的锁。避免：Lua 脚本比较 `get(key) == token` 再 `del(key)`。
3. **看门狗续期失败**：GC 停顿或网络抖动导致锁过期，原线程仍在执行。避免：业务逻辑内部加版本号/CAS 校验。
4. **时钟漂移**：多实例时钟不一致导致锁失效时间不确定。避免：NTP 同步；高一致性场景不依赖分布式锁。

---

## Q8：Redis 持久化与缓存恢复

### 参考答案

**RDB（快照）**：
- 在指定时间点将内存数据写入磁盘文件（dump.rdf），恢复速度快。
- 缺点：两次快照之间的数据可能丢失（取决于 `save` 配置）。
- 生产配置：例如 `save 900 1`（900 秒内至少 1 个 key 变更则快照），可根据容忍丢失数据量调整。

**AOF（追加文件）**：
- 记录每一条写操作命令，恢复时逐条重放。
- 缺点：文件体积大，恢复慢；需开启 `auto-aof-rewrite` 定期重写。
- 生产配置：`appendfsync everysec`（每秒同步一次，性能与可靠性平衡）。

**混合使用**：生产环境建议 RDB + AOF 混合，定期快照，中间过程用 AOF 减少数据丢失窗口。

**缓存恢复期间防止数据库被压垮**：
1. **本地缓存（Caffeine）**：每个 Java 实例本地缓存热点数据，TTL 设短，Redis 恢复期间先读本地。
2. **后台预热**：Redis 恢复可读写后，由后台任务从数据库批量加载热点 Key 到 Redis，使用 Pipeline 批量写入，分批控制 QPS。
3. **熔断与限流**：Redis 恢复初期只放小比例流量（如 5%），阶梯式放量。
4. **随机延迟（jitter）**：避免所有请求在同一时刻回源。

**脏数据发现与修复**：
- **发现**：定时对账任务扫描 Redis Key 与数据库比对；监控 `keyspace_hits/misses` 异常。
- **修复**：以数据库为准，异步更新 Redis；给恢复的缓存 Key 设置较短 TTL（如 10 分钟），让脏数据自动失效。

### 得分要点
- RDB/AOF 定义与优缺点：+4
- 生产配置建议：+2
- 冷启动防雪崩手段：+2
- 脏数据发现与修复：+2

### 常见踩坑点与避免
1. **只配 RDB 默认 `save 900 1`**：15 分钟内数据变更可能丢失。避免：根据业务容忍度调整 `save` 策略，或开启 AOF `everysec`。
2. **AOF 无限增长不重写**：重启时回放日志极慢。避免：开启 `auto-aof-rewrite-percentage` 和 `auto-aof-rewrite-min-size`。
3. **冷启动所有请求直击 DB**：避免：本地缓存 + 后台预热热点数据 + 限流。
4. **脏数据无法自愈**：仅靠 TTL 脏数据会长期存在。避免：定时对账 + 异步修复 + TTL 兜底。

---

## Q9：集群扩容与本地缓存一致性

### 参考答案

**Redis Cluster vs 客户端分片**：
- **Redis Cluster**：服务端自动管理 16384 个哈希槽，节点加入/退出自动重新分片，客户端只需感知集群拓扑。适合生产环境。
- **客户端分片**：客户端根据 Key 计算节点（如取模），实现简单，但节点变动后所有 Key 映射位置改变，命中率骤降，可能导致数据重新加载甚至丢失。生产环境不推荐。

**热点 Key 问题**：
- 在 Cluster 下，所有针对同一热点商品的 Key 会落在同一个槽和同一节点，导致单节点访问压力过高，吞吐量下降。
- 解决：读请求优先走本地缓存（Caffeine），仅少量更新请求打 Redis；或将热点 Key 拆分（如 `stock:iphone16` → `stock:iphone16:bucket:0~9`，随机或轮询访问不同 bucket）。

**本地缓存（Caffeine）与 Redis 两级一致性**：
- 引入本地缓存作为 L1，Redis 作为 L2。
- 失效通知：Redis 更新后，通过 **Redis Pub/Sub** 广播失效事件，各实例订阅并立即 `invalidate` 本地 Key；或使用 **Nacos/Apollo 配置中心** 推送版本号变更，实例按需拉取。
- 兜底：本地缓存设置短 TTL（如 1~5 分钟），即使失效通知丢失，最多几分钟后自动过期并重新加载。
- 容量控制：Caffeine 设置 `maximumSize` / `maximumWeight` + `expireAfterWrite`，避免 OOM。

### 得分要点
- Cluster vs 客户端分片对比：+3
- 热点 Key 影响与解决：+2
- 本地缓存失效通知（Pub/Sub 或配置中心）：+2
- 本地缓存容量与 OOM 控制：+1

### 常见踩坑点与避免
1. **客户端分片扩缩容导致命中率归零**：避免：生产环境使用 Redis Cluster 或 Codis/Twemproxy 代理层。
2. **仅改 Key 名无法解决热点读**：避免：热点 Key 先在本地缓存，少量更新请求打 Redis。
3. **本地缓存失效通知缺失**：避免：Redis Pub/Sub 或配置中心推送版本变更。
4. **本地缓存 OOM**：避免：设置 `maximumSize` 和过期时间，监控内存占用。

---

## Q10：Redis Pub/Sub 失效通知是否会丢失？

### 参考答案

**会丢失。** Redis Pub/Sub 是“发布即投递，无缓存、无重试、无持久化”的火并模式。

**丢失场景**：
- 订阅者实例正在 Full GC 停顿；
- 网络瞬间抖动，Redis 连接断开；
- 应用重新部署，订阅者还未上线；
- Redis 主从切换导致连接中断。

**生产环境弥补方案**：
1. **TTL 兜底（必须做）**：本地缓存设置短 `expireAfterWrite`（如 5 分钟），即使失效通知丢失，最多几分钟后自动过期并重新加载。
2. **版本号自我纠错**：读取本地缓存前，对比 Redis 中的版本号；若本地版本旧于 Redis 版本，主动失效本地缓存并重新加载。（注意：不要每次读都查 Redis 版本，否则失去本地缓存意义，可结合定时校验或仅在疑似脏读时校验。）
3. **消息队列替代（高一致性要求）**：将 Pub/Sub 替换为 RocketMQ/Kafka，利用消息重试和死信队列保证“至少投递一次”，消费端做幂等处理。

### 一句话总结
> Redis Pub/Sub 适合做“尽力而为的实时通知”，不能作为缓存一致性的唯一保障。必须配合 **TTL 兜底 + 定时校验**，做到“失效通知丢了也没关系”。

---

## Q11：缓存穿透与大 Key

### 缓存穿透

**定义**：请求访问一个缓存和数据库中都不存在的 Key，请求穿透缓存直接打数据库。

**防御方案**：
1. **缓存空值短 TTL**：数据库查不到时，把 `"__NULL__"` 写 Redis，设短 TTL（30~120 秒）。
   - 缺点：脏数据风险（数据后来创建了，缓存还在存空值）；内存浪费（攻击者可构造大量无效 Key）。
2. **Bloom Filter**：在缓存层前拦截，若 Key 肯定不存在则直接返回，不查缓存不查 DB。
   - 缺点：有误报，不支持删除（标准 Bloom Filter）。

**大 Key**

**定义**：Value 过大（String 超过 100KB）或集合元素过多（Hash/List/Set/ZSet 超过 5000~10000 个元素）。

**影响**：
- Redis 单线程阻塞，P99 延迟飙升；
- 内存占用集中，易导致 OOM；
- 主从复制延迟，持久化压力大；
- Java 客户端反序列化大对象造成 Young GC / Full GC，Jedis 连接池可能耗尽。

**发现**：
- `redis-cli --bigkeys` 人工排查；
- 慢查询日志（slowlog）分析大 Key 访问；
- 定时任务用 `SCAN` + `STRLEN`/`HLEN` 自动检测并告警；
- 云厂商控制台可视化大 Key 分析。

**拆分优化**：
- String 大 Key → 拆成多个小 Key（如 `product:detail:10086:base`、`:spec`、`:desc`）；
- Hash 大 Key → 分桶（`user:behavior:10086:bucket:0~99`）；
- List 大 Key → 分页存储，或冷数据存数据库；
- 文件/大文本不存 Redis，存 OSS，Redis 只存 URL。

---

## Q12：缓存预热与缓存雪崩兜底

### 缓存预热

**定义**：在系统启动、缓存清空或大流量到来前，主动将热点数据加载到缓存，避免首批请求打穿数据库。

**实现场景**：
1. **冷启动预热**：应用启动后异步任务，从数据库查询 Top N 热点数据，用 Redis Pipeline 批量写入，设置 TTL。
2. **秒杀前预热**：秒杀开始前 N 分钟，将参与秒杀商品库存写入 Redis，TTL 略长于秒杀时长。
3. **定期预热**：每天根据访问日志统计热点 Key，批量加载并续期。

**避免预热本身成为雪崩**：
- 分批查询数据库（如每批 500 条），批次间休眠，控制 QPS；
- 使用 Pipeline 批量写入 Redis；
- 预热任务使用独立数据库连接池或读从库；
- 预热 Key 设置 TTL + jitter，避免集体过期。

### 缓存雪崩兜底

**定义**：短时间内大量 Key 集体过期，或 Redis 整体不可用，导致所有请求打数据库。

**工程化防御手段**（分层）：
1. **TTL + jitter**：避免 Key 集体过期。
2. **本地缓存（Caffeine）+ Redis 两级**：Redis 过期或不可用时，先读本地缓存。
3. **互斥锁/Single Flight**：缓存失效时只有一个请求回源，其他返回旧值或等待。
4. **逻辑过期/永不过期**：Key 物理不过期，后台异步重建，主线程无阻塞。
5. **熔断与限流**：Redis 连续失败时触发熔断，读降级或拒绝，写直接拒绝；网关层限流保护数据库。
6. **DB 层保护**：数据库连接池限流、中间件限流，防止数据库被打崩。

**Redis 全挂时的兜底（读 / 写 / 降级）**：
- **读**：本地缓存兜底；或允许部分请求直连 DB（需 DB 层限流）；非核心数据返回默认值。
- **写**：秒杀扣库存等写操作，Redis 不可用时直接返回“系统繁忙”，避免超卖。
- **恢复探测**：使用 Sentinel/Resilience4j 半开状态，定时探测 Redis 是否恢复。

---

## Q13：缓存读写一致性的工程权衡

### 一致性模型

| 模型 | 定义 | 缓存场景适用性 |
|------|------|--------------|
| **强一致性** | 写完成后，所有后续读立即读到最新值 | 银行转账、订单状态等核心链路，性能代价高 |
| **最终一致性** | 写完成后短期内可能不一致，但最终收敛 | 商品详情、秒杀库存、点赞数等大多数缓存场景 |
| **弱一致性** | 写完成后，即使过一段时间也不一定读到最新，不保证最终一致 | PV 统计等允许丢失数据的非核心场景 |

**缓存场景最常用最终一致性**，兼顾性能与实现复杂度。

### Cache-Aside 写入顺序与并发问题

| 顺序 | 并发问题 |
|------|---------|
| **先删缓存，再更新数据库** | 脏回写：读请求在缓存被删除、数据库未更新的窗口期查到旧值并写回缓存，脏数据长期驻留 |
| **先更新数据库，再删缓存** | 短暂脏读（可接受）；删除缓存失败会导致长期脏数据 |

**延迟双删**：先删缓存 → 更新数据库 → 延迟一小段时间再删一次缓存，覆盖读请求回源写回的时间窗。

### 最终一致性工程方案

1. **延迟双删 + TTL 兜底**：最简单，适合读多写少。
2. **Binlog/CDC（Canal）**：订阅数据库 Binlog，异步更新/删除缓存，解耦业务与缓存逻辑，但引入运维成本和同步延迟。
3. **本地消息表（Outbox Pattern）**：业务事务内更新数据库 + 插入消息表，独立任务发送消息到 MQ，消费端更新缓存。不依赖 Binlog，本地事务保证不丢消息。
4. **版本号/CAS**：缓存和数据库都带 version，更新时比对版本，防止脏回写和丢失更新。

### 秒杀库存扣减的一致性实践

**不要每次扣减都查数据库**。推荐方案：

```
用户请求 → Redis Lua 脚本原子扣减 → 成功发送 Kafka 事件 → Kafka 消费端更新数据库（乐观锁兜底）
```

- 缓存是快路径，数据库是最终真相；
- 缓存设置 TTL，异步回写失败时脏数据自动过期；
- Kafka 消费幂等（event_id 去重），DB 更新 `WHERE stock > 0 AND version = ?`。

---

## Q14：缓存架构选型与客户端

### 架构选型

| 模式 | 适用场景 | 优点 | 缺点 |
|------|---------|------|------|
| **单机** | 开发测试 | 简单 | 生产环境禁用，单点故障 |
| **主从** | 读多写少、数据量不大 | 读写分离 | 故障切换需手动或依赖哨兵 |
| **哨兵（Sentinel）** | 容量不大但需要高可用 | 自动故障切换 | 数据量扩展受限 |
| **集群（Cluster）** | 数据量大、高并发 | 线性扩容、自动分片 | 不支持跨槽多 Key 操作（CROSSSLOT） |

**Java 客户端选型**：
- **Jedis**：同步阻塞 IO，连接非线程安全，需连接池，适合简单场景。
- **Lettuce**：基于 Netty，异步非阻塞，连接线程安全，Spring Boot 默认推荐，高并发首选。
- **Redisson**：封装分布式锁、限流器、可重入锁，适合分布式协同场景。

**分布式锁选型**：
- 生产环境推荐 **Redisson**，成熟实现了看门狗续期、可重入锁、Redlock。
- 不要直接用 `SETNX + EXPIRE` 手写，应使用 `SET key token NX EX` + Lua 脚本解锁。

### 缓存 Key 设计规范

1. **命名空间**：`业务线:系统名:模块:实体类型:ID:版本号`
   - 示例：`seckill:platform:stock:product:10086:v1`
2. **过期时间**：根据业务特点调整，频繁变动数据设较短 TTL，较稳定数据设较长 TTL；所有 Key 必须设 TTL，避免永久占用内存。
3. **防止集体过期**：TTL 加随机偏移（jitter，如 TTL 的 5%~15%）。
4. **大 Key 控制**：String 不超过 100KB，集合类型元素不超过 5000~10000；大 Key 拆分（Hash 分桶、List 分页）。
5. **监控**：`slowlog` 慢查询、`--bigkeys` 大 Key 分析、构建 Dashboard 监控命中率、内存、连接数。

---

## 面试总结与改进建议

**优势**：
- 缓存实战经验扎实，秒杀系统设计思路清晰（Lua + Kafka + 乐观锁）。
- 理解缓存失效三大问题及基本防御。
- 知道最终一致性优于强一致性在缓存场景的适用性。

**建议补强**：
1. **Redis 集群细节**：Cluster 16384 槽、hash tag 解决 CROSSSLOT、故障转移机制。
2. **一致性并发时序**：能画出先删缓存/先更新 DB 在并发下的具体时序图，说明脏回写根因。
3. **生产运维**：RDB/AOF 混合持久化配置、`save` 策略、AOF 重写、慢查询日志分析。
4. **分布式锁边界**：Redisson 看门狗、GC 停顿对锁可靠性的影响，何时不应仅依赖分布式锁。

---

## 推荐下一步练习方向

- 分布式会话缓存设计（Spring Session + Redis）
- 缓存预热架构（冷启动 + 定期热点预热 + Pipeline 批量写入）
- 全局 ID 生成器（雪花算法 + Redis 兜底）
- 秒杀接口整体设计（网关限流 → 本地缓存 → Redis Lua → Kafka → DB）
