###1、基础
---
设计要素
• Version
• RequstID
• Auth & Signature
• Rate Limit
• Docs
• Error Code & Message(除了成功，失败，还有超时)

###2、MSA
---
1. API Gateway

1. mq或kafara

1. 服务发现/注册

1. 异步日志（flume）

1. 降级和限流

1. 实时监控（spark）

1. redis

1. elasticsearch

1. cdn/Varnish/ngnix

1. 弹性扩容/缩容

1. 服务治理

1. 分布式事务

1. 持续集成

1. 自动化测试

1. 部署（docker）

###3、拆分
---
1. cms咨询

2. 图片处理

3. 推送

4. 定时任务

5. 邮件发送

###4、实践
1. 没有共享的数据库
2. 处理更新的一致性
3. 微服务安全
4. 微服务组合
5. 避免过度依赖





参考：

[正在考虑微服务架构的松耦合？小心这些陷阱！](http://dockone.io/article/1462)