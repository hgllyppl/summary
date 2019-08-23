# 本篇说明
本篇介绍mq，<span style="color:red">红字</span>为重要说明，<span style="color:green">绿字</span>高亮，<span style="color:orange">橙字</span>为不确定说明。<br>

# MQ-FAQ
0. mq的优缺点
    - 优点，解耦、异步、削峰
    - 缺点，系统可用性降低、复杂度升高，还可能带来分布式事务
0. 如何保证消费的幂等性
    - 唯一索引保证不重复插入
    - 唯一ID保证不重复消费
0. 如何保证消息的顺序性
    <br>一个queue一个consumer
0. 如何解决消息过期丢失的问题
    <br>如果可以，跑程序恢复丢失的消息
0. 如何解决消息持续积压或队列写满的问题
    - 恢复正常消费 + 扩容消费端
    - 清空队列 + 恢复正常消费 + 跑程序恢复丢失的消息

# rabbitMQ
## 简介
## 集群
- 单机
- 普通集群(仅同步队列信息，不同步队列数据)
- 镜像集群(同步队列信息和数据)

## 可靠消息
- 生产者
    - 事务消息
    - confirm消息
    - 异常时写日志
- 队列
    - 持久化
- 消费者
    - 手动ack

## 客户端
### amqp-client
- RecoveryAwareAMQConnection(MainLoop/ConsumerWorkService/HeartbeatSender)

### spring-amqp
- RabbitAutoConfiguration/RabbitListener/RabbitHandler

# 参考引用
0. [面试：分布式之消息队列要点复习 - 从小工到大家 - SegmentFault 思否](https://segmentfault.com/a/1190000015301449)
0. [如何保证消息队列的高可用](https://github.com/doocs/advanced-java/blob/master/docs/high-concurrency/how-to-ensure-high-availability-of-message-queues.md)
0. [如何保证消息不被重复消费](https://github.com/doocs/advanced-java/blob/master/docs/high-concurrency/how-to-ensure-that-messages-are-not-repeatedly-consumed.md)
0. [如何保证消息的顺序性](https://github.com/doocs/advanced-java/blob/master/docs/high-concurrency/how-to-ensure-the-order-of-messages.md)
0. [如何保证消息的可靠性传输](https://github.com/doocs/advanced-java/blob/master/docs/high-concurrency/how-to-ensure-the-reliable-transmission-of-messages.md)
0. [如何解决消息队列的延时以及过期失效问题](https://github.com/doocs/advanced-java/blob/master/docs/high-concurrency/mq-time-delay-and-expired-failure.md)