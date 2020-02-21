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

