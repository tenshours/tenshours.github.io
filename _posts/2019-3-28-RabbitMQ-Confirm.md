---
layout: post

title: "RabbitMQ message confirm"

date: 2019-3-28

tags: [RabbitMQ]

comments: true
---

### Message confirm/ack异常

消息生产着使用confirm/transaction确定消息投递成功。

Q1: 但是如果server返回confirm信息时，网络中断，消息生产者并没收到确认信息，会认为消息没有没有投递成功，生产者会再次发送消息，这个时候怎么解决。

消息消费用使用ACK确认消息消费成功，然后RabbitMQserver移除消息。

Q2: 如果返回ACK信息的时候，网络发生异常，server会认为消费方没有接受到消息，该消息不会被移除，还会发送到消费方。

两个问题，目前都找到解决办法。就只能消息重新发送，和消息重复消费。如果消息不是幂等的，那么就要用数据来存储已经消费过的消息，来防止重复消费。
