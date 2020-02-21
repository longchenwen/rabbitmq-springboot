# rabbitmq 整合springboot

## 1.springboot整合-生产端-配置详解

1.publisher-confirms,实现一个监听器用于监听Broker端给我们返回的确认请求:RabbitTemplate.ConfirmCallback(接口需要实现)

2.publisher-returns,完成消息对broker端是可达的,如果出现路由键不可达的情况,则使用监听器对不可达的消息进行后续的处理,保证消息的路由成功:RabbitTemplate.ReturnCallback
```
spring.rabbitmq.addresses=192.168.3.94:5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
spring.rabbitmq.virtual-host=/
spring.rabbitmq.connection-timeout=15000

#确认接收消息ack
spring.rabbitmq.publisher-confirms=true
spring.rabbitmq.publisher-returns=true
#mandatory为true的时候才能接收到返回的消息
spring.rabbitmq.template.mandatory=true
#这配置是否是确认配置
#spring.rabbitmq.publisher-confirm-type=

```
## 2.springboot整合-消费端-配置详解

1.首先配置手工确认模式,用于ACK的手工处理,这样我们可以保证消息的可靠性送达,或者再消费端消费失败的时候可以做到重回队列,喝酒业务记录日志等处理

2.可以设置消费端的监听个数和最大个数,用于控制消费端的并发情况

3.@RabbitListener注解使用
   
    3.1 消费端监听@RabbitMQListener注解,这个对于在实际工作中非常好用
    
    3.2 @RabbitMQListener是一个组合注解,里面可以配置注解,@QueueBinding,@Queue,@Exchange直接通过这个组合注解一次性搞定消费端交换机,队列,绑定,路由,并且配置监听功能等
