# 本篇说明
本篇介绍redis，<span style="color:red">红字</span>为重要说明，<span style="color:green">绿字</span>高亮，<span style="color:orange">橙字</span>为不确定说明。<br>

# 使用
## redis简介

## redis数据结构
- string、list、set、hash、sortedset、hyperlog、geo

## redis命令
- keys，阻塞服务器
- scan，慢+重复概率
- pipeline，批量提交命令
- lpursh、rpop、brpop

## redis事务
- 原子性 + 隔离性
- 不支持回滚
    - redis只会因编程错误而失败，回滚并不能解决编程错误
    - 因为不需要回滚，redis内部可以保持简洁快速

## redis脚本

## redis键空间通知/发布订阅

## redis持久化
- RDB + AOF(for + cow)

## redis同步
- bgsave + 增量回放

## redis集群
- 集群
    - 主从 + 哨兵

## redis复制

## redis哨兵

# 原理
## 快的秘密和多线程问题
- 单线程、epoll、读写内存
- 多线程会导致 线程切换、竞争、死锁

## redis淘汰策略
- 六种策略
    - volatile-lru，从不稳定缓存中删除最近最少使用的key
    - volatile-ttl，从不稳定缓存中删除即将过期的key
    - volatile-random，从不稳定缓存中随机删除key
    - always-lru，从缓存中删除最近最少使用的key
    - always-random，从缓存中随机删除key
    - no-enviction，无淘汰策略
- 默认策略
    - volatile-random + 被动触发(客户端操作key时，如果过期则删除)

## redis分片算法
- hash，扩容时会导致重新hash带来雪崩
- 一致性hash，节点过少时会导致key偏移到某个节点上去
    - 将hash值空间组成一个圆环，对节点hash并将其落在hash环上
    - 对key进行hash，按顺时针将其落在第一个节点上
- crc16(key) % 16384，固定槽位方便扩容

## redis通信协议

# 应用场景
## 雪崩、穿透
- 雪崩
    - 缓存集中过期，过期时间加随机值
    - redis宕机
        - 事发前，高可用架构
        - 事发中，限流 + (恢复redis + 扩容redis)
        - 事发后，总结
- 穿透
    - 拦截不合法请求(布隆过滤器)
    - 设置短期空缓存

## 缓存与数据库的双写一致性(DTP)
- 允许弱一致
- 不允许弱一致
    - XA(redis不支持XA，无法实现)
    - Cache Aside Pattern
        - 先更新数据库、后删缓存
        - 避免不一致
            - 将更新和删除操作放在一个事务里
            - 将读取和写入操作放在一个事务里
            - 读加共享锁，读锁和写锁必须互斥
    - Read/Write through(缓存代理)
    - Write behind caching(写缓存异步刷库)
    - 内存队列(读/写请求串行化)

## 分布式锁
- 分布式锁必须保证的特性
    - 互斥
    - 无死锁
    - 容错
- 如何使用redis实现分布式锁
    - 获取锁

            SET key random_val NX PX 30000
    - 释放锁

            if redis.call("get", KEYS[1]) == ARGV[1] then
                return redis.call("del", KEYS[1])
            else
                return 0
            end
    - 多数派
        - 必须是奇数个独立的redis节点
    - 安全性
        - MIN_VALIDITY = TTL - (T2 - T1) - CLOCK_DRIFT
    - 崩溃恢复
        - fsync = always
        - 延迟一个最长的锁周期后重启
- 使用分布式锁需要注意什么
    - 计算“有效时间” & 在有效时间内完成“业务逻辑”，若不能完成则撤销之前的“操作”
    - “业务逻辑”应该尽可能的精简，不要掺杂不必要的代码

# 参考引用
0. [redis实战](https://www.amazon.cn/dp/B016YLS2LM/ref=tmm_pap_swatch_0?_encoding=UTF8&qid=1561105812&sr=8-1)
0. [redis设计与实现](https://www.amazon.cn/dp/B00L4XHH0S/ref=tmm_pap_swatch_0?_encoding=UTF8&qid=1561105812&sr=8-2)
0. [面试前必须要知道的Redis面试题 - Java3y - 博客园](https://www.cnblogs.com/Java3y/p/10266306.html)
0. [如何保证缓存与数据库的双写一致性](https://github.com/doocs/advanced-java/blob/master/docs/high-concurrency/redis-consistence.md)
0. [缓存更新的套路 | | 酷 壳 - CoolShell](https://coolshell.cn/articles/17416.html)
0. [???一致性HASH算法详解 - 简书](https://www.jianshu.com/p/e8fb89bb3a61)
0. [???什么是一致性Hash算法？ - 知乎](https://zhuanlan.zhihu.com/p/34985026)
0. [???详解布隆过滤器的原理，使用场景和注意事项 - 知乎](https://zhuanlan.zhihu.com/p/43263751)
0. [再有人问你分布式锁，这篇文章扔给他 - 掘金](https://juejin.im/post/5bbb0d8df265da0abd3533a5)
0. [REDIS distlock -- Redis中国用户组（CRUG）](http://redis.cn/topics/distlock.html)
0. [《Redis官方文档》用Redis构建分布式锁 | 并发编程网 – ifeve.com](http://ifeve.com/redis-lock/)