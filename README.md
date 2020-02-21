# rabbitmq 整合springboot

## 1.springboot整合配置详解

1.publisher-confirms,实现一个监听器用于监听Broker端给我们返回的确认请求:RabbitTemplate.ConfirmCallback(接口需要实现)

2.publisher-returns,宝成消息对broker端是可达的,如果出现路右键不可达的情况,则使用监听器对不可达的消息进行后续的处理,保证消息的路由成功:RabbitTemplate.ReturnCallback

