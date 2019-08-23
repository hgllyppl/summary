# 操作系统

# 协议
## tcp/ip
0. OSI七层模型及其功能
    - 物理层，比特流与电信号之间的转换
    - 链路层，比特流与数据桢之间的转换以及分段转发
    - 网络层，地址管理与路由选择，
    - 传输层，管理两个节点之间的数据可靠传输
    - 会话层，通信管理，负责建立和断开通信连接
    - 表示层，固有数据格式和网络标准数据格式的转换
    - 应用层，针对特定应用的协议
0. 什么是tcp/ip，与OSI区别
    - 链路层(ppp)
    - 网络层(ip icmp igmp ipsec)
    - 传输层(tcp udp)
    - 应用层(http)
0. 解释桢、包、段、消息
0. 以http协议为例，讲述数据在每一层的流转过程
0. arp归属哪一层
0. 定义tcp、udp
0. tcp/ip如何识别一个连接
0. 解释3次握手与4次挥手，为什么是3次握手，为什么是4次挥手，为什么等待"2MSL"，为什么必须等待2MSL
    - 3次握手
        -> SYN_SENT      SYN=1, seq=i
        <- SYN_RECV      SYN=1, ACK=1, ack=i+1, seq=j
        -> ESTABLISHIED  SYN=1, ACK=1, ack=j+1
    - 4次挥手
        模式1
        -> FIN_WAT_1   FIN=N
        <- CLOSE_WAIT  ack=N+1
        <- LAST_ACK    FIN=M
        -- FIN_WAT_2
        -> TIME_WAIT   ACK=1, ack=M+1 
        -- CLOSED
        模式2
        -><- FIN_WAT_1   FIN=N, FIN=M
        -><- CLOSING  ACK=1, ack=N+1, ack=M+1
        ---- TIME_WAIT
    - 为什么3次握手、4次挥手
        需要确认客户端
        双端都需要关闭
    - 为什么等待2MSL
        发出ACK是一个MSL，如果对端没有收到ACK重发FIN是一个MSL
    - 为什么必须等待2MSL
        让报文有足够的时间送达对端
        不让新连接读到旧数据
0. tcp如何实现可靠传输、超时重传
0. 解释滑动窗口与流量控制
0. 解释慢启动与拥塞控制，快重传与快恢复
0. 解释MTU、MSS，MSL、TTL、RTT、RTO
0. 解释IP地址组成与分类
    - IP地址由网络标识+主机标识构成
    - A类，首位已0开头，前8位为网络标识，0.0.0.0-127.0.0.0
    - B类，前2位已10开头，前16位为网络标识，128.0.0.0-191.255.0.0
    - C类，前3位已110开头，前24位为网络标识，192.0.0.0-223.255.255.0
    - D类，前4位已1110开头，前32位为网络标识，224.0.0.0-239.255.255.255
0. 解释单播、广播、多播
0. 解释子网掩码
    - 网络标识为1，主机标识为0，网络标识位数自己规划
    * 172.20.100.52/26
    * 172.20.100.0-63/26
0. 解释如何路由

## http
0. 什么是http
0. 什么是uri，uri与url的区别，urn是与它们是什么关系
0. http报文格式
0. 常用请求方法
0. 常用状态码
0. http常用header
0. 如何保持连接
0. 为什么设置TCP_NODELAY
0. TIME_WAIT为什么会影响性能测试
0. 客户端如何保持会话
0. 基本认证与摘要认证

# 算法
0. 树、二叉树、二叉查找树、平衡二叉查找树
0. B树/B+树/B*树、红黑树
0. 跳跃表

# 数据库
0. 数据库的核心组件
    - 查询[解析/重写/优化/执行]
    - 数据[事务/数据访问/缓存]
    - 核心[内存/进程/网络/文件/安全/客户端]
    - 工具[备份/服务/监控]
0. SQL的执行过程
    - 解析 -> 重写 -> 优化 -> 执行
0. 事务属性及隔离级别，MySQL默认隔离级别
    - 原子性，事务中的操作要么全部执行，要么全部不执行
    - 一致性，事务执行前后，数据完整性没有被破坏
    - 隔离性，一个事务操作的结果在何时以何种方式对其他并发的事务操作可见
    - 持久性，事务结束后，对数据的修改是永久的
    * 脏读，读取到未提交事务修改的数据
    * 不可重发读，读取到已提交事务修改的数据
    * 幻读，读取到新插入的数据
    * 串行读，每次读都需要加表级共享锁，读写互斥
0. 悲观锁、乐观锁、排它锁、共享锁、行锁、表锁、2PL
0. 乐观锁的业务场景及实现方式
0. 如何保证ACID
0. 主从复制原理
    - 原理，从库复制主库binlog并串行回放
        * 主库宕机，数据丢失
        * 从库串行回放，数据延时
    - 半同步复制，强行将新的binlog同步给从库
    - 并行复制，从库并行回放binlog
0. MySQL记录binlog的方式有哪几种模式，每种模式的优缺点是什么？
    - row，记录每行数据被修改成什么样子，易复制但日志多
    - statement，记录sql语句，不易复制日志少
    - mixed，

# java
## 基础
### collection
0. 集合架构
0. Arraylist与LinkedList区别
0. List与Map区别
0. HashMap的数据结构，HashMap的长度为什么是2的幂
0. TreeMap的数据结构
0. ConcurrentHashMap的数据结构，并发读写时如何保证线程安全，怎么扩容以及如何保证线程安全
0. comparable与comparator的区别

### concurrency
0. 解释进程与线程，它们的联系与区别
    - 进程是程序关于某数据集合上的一次运行活动，是操作系统进行资源分配和调度的基本单位。
    - 线程是进程的实际运作单位，也是操作系统进行运算调度的最小单位。
0. 如何创建一个线程
0. 为什么要给线程设置名称和异常处理器
0. 线程的生命周期
0. 为什么创建线程池，如何合理配置线程池的大小
0. 线程池的核心参数及工作原理
0. 解释并发与并行
0. 什么是上下文切换
0. 什么是现线程安全，如何实现线程安全
    - 如果程序被多线程访问后仍能保持正确性，那么程序就是线程安全的
0. 什么是原子性、可见性、有序性
0. volatile实现原理
0. synchronized实现原理及优化
    - monitorEnter/monitorExit/ACC_SYNCHRONIZED
    - 偏向锁、轻量级锁、自旋锁、重量级锁(mutex)
    - 锁消除、锁粗化
0. synchronized与volatile区别
0. synchronized与ReentrantLock区别
0. Threadlocal原理，为什么会导致内存泄露
    static class Entry extends WeakReference<ThreadLocal<?>> {
        /** The value associated with this ThreadLocal. */
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }
0. 简述AQS工作原理以及它对资源的共享方式
0. 解释jmm
    - 对现代硬件内存模型的抽象，分为线程本地内存和主内存两个部分
    - 解决线程间通信和同步两个问题

### reflect
0. 反射的用途及实现
    method.invoke -> MethodAccessor.invoke

### proxy
0. 代理基本原理
0. jdk代理原理
0. cglib代理原理

### io
0. 什么是流，按照传输单位、方向、功能，分别分成哪两种流
    - 节点流、处理流
0. 如何将控制台输出定向到一个文件
0. 读写数据用哪两个流
0. 什么是序列化与反序列化，序列化对象需要做什么(实现什么接口)，不想序列化某些字段怎么做，serialVersionUID有什么用
0. 怎么关闭多层包装的流
0. 什么是nio，io与nio区别
    - io，面向流的阻塞io模型
    - nio，面向缓冲区的非阻塞io模型
0. 解释5种io模型
    - 阻塞io
    - 非阻塞io
    - 多路复用io
    - 信号驱动io
    - 异步io
0. 解释同步、异步、阻塞、非阻塞
    - 涉及概念
        - 线程、锁、io、用户空间、内核空间
    - 定义
        - 同步，发起一个调用，在得到结果后返回
        - 异步，发起一个调用，不得到结果就返回，结果由被调用者通知回来
        - 阻塞，发起一个调用，该调用会阻塞线程直至得到结果
        - 非阻塞，发起一个调用，该调用不会阻塞线程而会立刻返回
        * 同步异步区别，该调用是否由调用者发起并完成
        * 阻塞非阻塞区别，该调用是否会阻塞线程
    - 从调用者角度出发，在限定调用的场景下作一个相对合理的解释
        - 普通调用
            - 没有同步异步、阻塞非阻塞之分，均为同步非阻塞调用
        - 锁调用
            - 没有同步异步之分，均为同步调用
            - 因为锁的特性，有阻塞与非阻塞之分
                * lock.lock()
                * lock.tryLock()
        - io调用(磁盘/网络等等)
            - 因为io的特性，有同步异步、阻塞非阻塞之分
            - 同步，由用户空间线程发起调用并等待操作完成
                * 阻塞
                    socket.getInputStream().read()
                * 非阻塞[此场景没有实际使用意义，因为它会消耗CPU]
                    socket.getChannel().configureBlocking(false);
                    while ((len = socket.getChannel().read(byteBuffer)) != -1) {
                        // do something
                    }
            - 异步，由用户空间线程发起调用且不等待操作完成，之后由内核空间完成操作并通知用户空间线程
                * win，iocp
                * linux，没有异步io实现
                * java
                    - nio，此模式并非异步io，因为io操作是由用户空间线程完成
                    - aio，此模式也非异步io，因为它是由java平台开启线程池模拟内核空间完成io操作后通知用户空间线程
            - 同步异步区别
                * 同步是由用户空间发起调用并完成操作，异步是由用户空间发起调用内核空间完成操作
0. 解释2种高性能io模型
    - reactor
    - proactor

## 框架
### slf4j
0. slf4j架构

### mybatis

### spring
0. 解释ioc与di
0. bean的作用域和生命周期
    - 作用域[singleton/prototype/request/session/global-session]
    - 生命周期
        - resolveBeforeInstantiation
            - postProcessBeforeInstantiation
        - createBeanInstance
        - populateBean
            - postProcessAfterInstantiation
            - findPropertyValues
            - postProcessPropertyValues
            - applyPropertyValues
        - initializeBean
            - invokeAwareMethods
            - postProcessBeforeInitialization
            - invokeInitMethods
            - postProcessAfterInitialization
        - destroyBean
0. aop原理及应用场景
    - 创建代理，ProxyFactory -> AopProxyFactory -> AopProxy
    - 调用，代理 -> 创建拦截器链条(AdvisorChainFactory)
                        ┌──────────┐
                        │          │
                        ▼          │
                -> 执行拦截器链 -> 拦截器 -> 目标方法
0. spring事务原理及传播行为
    - 传播行为
        PROPAGATION_REQUIRED
        PROPAGATION_REQUIRES_NEW
        PROPAGATION_NESTED
        PROPAGATION_SUPPORTS
        PROPAGATION_MANDATORY
        PROPAGATION_NOT_SUPPORTED
        PROPAGATION_NEVER
    - 原理
        - 事务管理器通过事务切面来打开/提交/回滚事务
        - PlatformTransactionManager/TransactionInterceptor
            - getTransaction会将connection绑定到TL
0. mvc原理
    - getHandler,                      HandlerMapping通过URL查找handler
    - getHandlerAdapter,               通过handler查找匹配的HandlerAdapter
    - mappedHandler.applyPreHandle,    调用前置拦截
    - ha.handle,                       HandlerAdapter调用handler
    - mappedHandler.applyPostHandle,   调用后置拦截
    - processDispatchResult,           渲染视图

### springboot
0. springboot比spring做了哪些改进
    - starter、autoconfig
    - fatjar、embed-tomcat
0. 简述springboot启动过程

## 容器
### tomcat

## jvm
0. 程序计数器的作用
    - 记录下一条指令地址
0. 运行时数据区划分
    - 类加载器
    - 代码缓存 永久代 堆 栈 本地方法栈 程序计数器
        - 栈
            - 栈桢[本地变量表/操作数/返回值/class引用]
        - 堆
            - 新生代[eden/s0/s1]
            - 老年代
        - 永久代
            - 字符串池
            - 方法区[运行时常量池]
    - VM执行引擎
        - 即时编译器
        - 垃圾收集器
    - 本地库接口
        - 本地方法库
0. gc算法及收集器
    - 复制[复制存活对象]
        - 使用eden+s0，复制存活对象至s1，使用eden+s1，复制存活对象至s0，如此循环
        - serial/parallel scavenge/parnew/
    - 标记整理
        - serial old/parallel old
    - 标记清除
        - cms
0. young gc、major gc、full gc的区别
    partial gc
        young gc  [收集新生代]
        old gc    [收集老年代，只有CMS的concurrent collection是这个模式]
        mixed gc  [收集新生代和部分老年代，只有G1有这个模式]
    full gc       [收集新生代、老年代、永久代]
    major gc      [通常跟full gc等价，但一定要问清楚是仅指old gc还是full gc]
0. gc触发条件
    新生代/老年代/永久代耗尽则触发gc
    full gc直接触发条件
        新生代晋升时，老年代内存不足
        分配大对象时，老年代内存不足
        老年代耗尽
        System.gc()
        heap dump
    serial/parallel组合
        young gc [eden区耗尽]
        full gc
            - 当准备触发young gc时，如果之前的统计数据说明young gc的平均晋升大小 > old gen的剩余空间，则不会触发young gc转而触发full gc
            - parallel gc会默认触发一次young gc以减轻full gc的负担，两次gc之间能让应用稍微运行一下
    parnew/cms组合
        young gc [eden区耗尽]
        old gc   [老年代使用率超过阀值]
        full gc  [cms gc失败]
0. 类加载机制
    - 装载[查找class并将其转为class对象]
    - 链接
        - 验证[文件格式/元数据/字节码/符号引用]
        - 准备[为静态变量分配内存并初始化为默认值]
        - 解析[将符号引用转换为直接引用]
    - 初始化[初始化静态变量和代码块]
0. 什么是双亲委派模型，双亲委派模型的好处，什么情况下需要破坏双亲委派模型
    - bootstrap/ext/app
    - 安全
    - SPI & ThreadContextClassLoader

# 设计
## 设计模式

# 分布式
## redis
0. 有哪些数据结构
    string、list、set、hash、sortedset、hyperlog、geo
0. 如何查找一批key
    - keys，阻塞服务器
    - scan，慢+重复概率
0. 为什么用pipeline
    - 批量提交命令
0. 用做队列的几种方式
    - list，lpursh、rpop、brpop
    - pub/sub
0. 有哪些淘汰策略，默认策略
    - volatile-lru/ttl/random
    - always-lru/random
    - no-enviction
    * volatile-random + 被动触发(客户端操作key时，如果过期则删除)
0. 事务特性，原子性是否同SQL一样，为什么不一样
    - 原子性+隔离性
    - 有所不同，不支持回滚
        - redis只会因编程错误而失败，回滚并不能解决编程错误
        - 因为不需要回滚，redis内部可以保持简洁快速
0. 如何持久化
    - RDB+AOF(for+cow)
0. 主从同步机制
    - bgsave+增量回放
0. 哪些hash算法
    - hash，扩容时会导致重新hash带来雪崩
    - 一致性hash，节点过少时会导致key偏移到某个节点上去
        * 将hash值空间组成一个圆环，对节点hash并将其落在hash环上
        * 对key进行hash，按顺时针将其落在第一个节点上
    - crc16(key) % 16384，固定槽位方便扩容
0. 如何实现高可用
    - 集群+主从+哨兵+gossip
0. 为什么快，如果采用多线程会有什么问题
    - 单线程、读写内存、epoll
    - 线程切换、竞争、死锁
0. 如何解决雪崩问题
    - 缓存集中过期，过期时间加随机值
    - redis宕机
        * 事发前，高可用架构
        * 事发中，限流+(迅速恢复redis并增加redis节点)
        * 事发后，总结
0. 如何解决穿透问题
    - 拦截不合法请求(布隆过滤器)
    - 设置短期空缓存
0. 如何解决缓存与数据库的一致性(DTP)
    - 允许弱一致
    - 不允许弱一致
        - XA(redis不支持XA，无法实现)
        - Cache Aside
            - 先更新数据库、后删缓存
            - 避免不一致
                将更新和删除操作放在一个事务里
                将读取和写入操作放在一个事务里
                    - 读加共享锁，读锁和写锁必须互斥
        - Read/Write through(缓存代理)
        - Write behind caching(写缓存异步刷库)
        - 内存队列(读/写请求串行化)

## 分布式锁
0. 分布式锁必须保证的特性
    - 互斥
    - 无死锁
    - 容错
    * 阻塞与非阻塞
    * 可重入
0. 如何使用redis实现分布式锁
    - 获取锁
        SET key random_val NX PX 30000
    - 释放锁
        if redis.call("get", KEYS[1]) == ARGV[1] then
            return redis.call("del", KEYS[1])
        else
            return 0
        end
    - 多数派(奇数个独立的redis节点)
    - 安全性
        MIN_VALIDITY = TTL - (T2 - T1) - CLOCK_DRIFT
    - 崩溃恢复
        fsync=always or 延迟重启
0. 使用分布式锁需要注意什么
    - 计算“有效时间” & 在有效时间内完成“业务逻辑”，若不能完成则撤销之前的“操作”
    - “业务逻辑”应该尽可能的精简，不要掺杂不必要的代码

## mq
0. mq的优缺点
    - 优点，解耦、异步、削峰
    - 缺点，可用性降低、复杂度提高、分布式事务
0. 介绍几种mq及其优缺点
    - ActiveMQ、RocketMQ、RabbitMQ、Kafka
    - 社区活跃度、吞吐量、延时性、可用性、可靠性、功能完备性
0. 如何保证消息队列的高可用
    - RabbitMQ
        - 单机
        - 普通集群(仅同步queue-meta)
        - 镜像集群
    - kafka
        - partion + replica
0. 如何保证消息的可靠性
    - RabbitMQ
        - 生产者
            - 事务机制
            - confirm机制
            - 异常时写日志
        - 队列
            - 持久化
        - 消费者
            - 手动ack
    - kafka
        - 生产者
            - (acks=all, retries=N)
            - 异常时写日志
        - 队列
            - replication.factor > 1，每个partition至少有2个副本
            - min.insync.replicas > 1，每个leader至少和2个follower保持联系
        - 消费者
            - 手动ack
0. 如何保证消费的幂等性
    - 唯一索引保证不重复插入
    - 唯一ID保证不重复消费
0. 如何保证消息的顺序性
    - 一个queue一个consumer
0. 如何解决消息过期丢失的问题
    - 跑程序恢复丢失的消息
0. 如何解决有几百万消息持续积压几小时的问题
    - 恢复正常消费 + 扩容消费端
0. 如何解决队列写满的问题
    - 恢复正常消费 + 扩容消费端
    - 清空队列 + 恢复正常消费+ 跑程序恢复丢失的消息

## springcloud
0. 使用什么协议及如何序列化
    - http
    - 序列化方式跟content-type相关，一般是form或json
0. 负载均衡原理及默认策略
    - 提供了随机、轮询、权重等几种策略，默认轮询
0. 服务是如何注册与发现其他服务的
    - 客户端
        - register ，生成 InstanceInfo ，并将其发送给 eureka-server
            - InstanceInfo，instanceId/appName/hostName/port/leaseInfo
                - leaseInfo，registrationTimestamp/lastRenewalTimestamp/evictionTimestamp
        - renew ，定时向 eureka-server 发送 InstanceInfo ，默认90s
        - fetchRegistry ，定时从 eureka-server 拉取服务信息， 默认30s
    - 服务端
        - 数据结构
            - l1, Map<key, Value>
                - key, entityName/entityType/requestType/requestVersion
                - value, payLoad/gzipped
                * 删除/加载，TimerTask定时读L2到L1
            - l2, LoadingCache<key, Value>
                * 删除，客户端register|renew|cancel/驱逐任务/guava缓存到期
                * 加载，客户端拉取服务，L1没有且L2也没有时(如果L1没有开启则直接读L2)，从registry加载数据到L2
            - registry, Map<spring.application.name, Map<instanceId, Lease<InstanceInfo>>>
        - 服务处理
            - register
                * 保存实例信息到注册表
                * 添加事件到更新队列以供客户端增量同步
                * 清空L2
                * 更新阀值供剔除服务使用
                * 同步实例信息至对等节点
            - renew
                * 更新实例的最近续约时间(lastUpdateTimestamp)
                * 同步实例信息至对等节点
            - cancel
                * 从注册表删除实例信息
                * 添加事件到更新队列以供客户端增量同步
                * 清空L2
                * 更新阀值供剔除服务使用
                * 同步实例信息至对等节点
            - 剔除服务(略)
            - 同步服务(略)
0. 简述熔断机制及其实现原理
    - 命令 + 熔断器 + 资源隔离
        命令，request -> 熔断器 -> 线程池 -> 服务调用，如果熔断器开启 or 线程池拒绝，将直接返回
        熔断器
            * 健康状况 = 健康数/请求数
            * 关闭，当 健康状况 >= 阀值
            * 打开，当 健康状况 < 阀值
            * 半开，当经过开启时间窗口后，熔断器只允许一个请求通过，request succ ? 关闭 : 打开
        资源隔离，每个服务配独立线程池
0. 简述服务链路追踪的原理
   Client Tracer                                                  Server Tracer     
┌───────────────────────┐                                       ┌───────────────────────┐
│                       │                                       │                       │
│   TraceContext        │          Http Request Headers         │   TraceContext        │
│ ┌───────────────────┐ │         ┌───────────────────┐         │ ┌───────────────────┐ │
│ │ TraceId           │ │         │ X-B3-TraceId      │         │ │ TraceId           │ │
│ │                   │ │         │                   │         │ │                   │ │
│ │ ParentSpanId      │ │ Inject  │ X-B3-ParentSpanId │ Extract │ │ ParentSpanId      │ │
│ │                   ├─┼────────>│                   ├─────────┼>│                   │ │
│ │ SpanId            │ │         │ X-B3-SpanId       │         │ │ SpanId            │ │
│ │                   │ │         │                   │         │ │                   │ │
│ │ Sampling decision │ │         │ X-B3-Sampled      │         │ │ Sampling decision │ │
│ └───────────────────┘ │         └───────────────────┘         │ └───────────────────┘ │
│                       │                                       │                       │
└───────────────────────┘                                       └───────────────────────┘

## 分布式事务
0. 什么是分布式事务
    - 分布式事务是指事务的参与者、协调者、事务管理器、资源管理器分别位于分布式系统的不同节点之上
0. 什么是flp、cap、base
0. 什么是2pc、3pc、paxos
0. 常见的解决方案并比较其优缺点