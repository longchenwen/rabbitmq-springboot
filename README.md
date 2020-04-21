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

#开启confirms确认机制
spring.rabbitmq.publisher-confirms=true
开启returns确认机制
spring.rabbitmq.publisher-returns=true
#设置为true后 消费者在消息没有被路由到合适队列情况下会被return监听，而不会自动删除
spring.rabbitmq.template.mandatory=true
#这配置是否是确认配置
#spring.rabbitmq.publisher-confirm-type=

```

3.生产端代码:
```
//自动注入RabbitTemplate模板类
	@Autowired
	private RabbitTemplate rabbitTemplate;  
	
	//回调函数: confirm确认
	final ConfirmCallback confirmCallback = new RabbitTemplate.ConfirmCallback() {
		@Override
		public void confirm(CorrelationData correlationData, boolean ack, String cause) {
			System.err.println("correlationData: " + correlationData);
			System.err.println("ack: " + ack);
			if(!ack){
				System.err.println("异常处理....");
			}
		}
	};
	
	//回调函数: return返回
	final ReturnCallback returnCallback = new RabbitTemplate.ReturnCallback() {
		@Override
		public void returnedMessage(org.springframework.amqp.core.Message message, int replyCode, String replyText,
				String exchange, String routingKey) {
			System.err.println("return exchange: " + exchange + ", routingKey: " 
				+ routingKey + ", replyCode: " + replyCode + ", replyText: " + replyText);
		}
	};
	
	//发送消息方法调用: 构建Message消息
	public void send(Object message, Map<String, Object> properties) throws Exception {
		MessageHeaders mhs = new MessageHeaders(properties);
		Message msg = MessageBuilder.createMessage(message, mhs);
		rabbitTemplate.setConfirmCallback(confirmCallback);
		rabbitTemplate.setReturnCallback(returnCallback);
		//id + 时间戳 全局唯一 
		CorrelationData correlationData = new CorrelationData("1234567890");
		rabbitTemplate.convertAndSend("exchange-1", "springboot.abc", msg, correlationData);
	}
	
	//发送消息方法调用: 构建自定义对象消息
	public void sendOrder(Order order) throws Exception {
		rabbitTemplate.setConfirmCallback(confirmCallback);
		rabbitTemplate.setReturnCallback(returnCallback);
		//id + 时间戳 全局唯一 
		CorrelationData correlationData = new CorrelationData("0987654321");
		rabbitTemplate.convertAndSend("exchange-2", "springboot.def", order, correlationData);
	}
	
```

## 2.springboot整合-消费端-配置详解

1.首先配置手工确认模式,用于ACK的手工处理,这样我们可以保证消息的可靠性送达,或者再消费端消费失败的时候可以做到重回队列,根据业务记录日志等处理

2.可以设置消费端的监听个数和最大个数,用于控制消费端的并发情况

3.@RabbitListener注解使用
   
    3.1 消费端监听@RabbitMQListener注解,这个对于在实际工作中非常好用
    
    3.2 @RabbitMQListener是一个组合注解,里面可以配置注解,@QueueBinding,@Queue,@Exchange直接通过这个组合注解一次性搞定消费端交换机,队列,绑定,路由,并且配置监听功能等
  
 ```
 #签收模式:手工签收
spring.rabbitmq.listener.simple.acknowledge-mode=manual
#消费端的监听个数
spring.rabbitmq.listener.simple.concurrency=5
#消费端的监听-最大-个数
spring.rabbitmq.listener.simple.max-concurrency=10

spring.rabbitmq.listener.order.queue.name=queue-2
spring.rabbitmq.listener.order.queue.durable=true
spring.rabbitmq.listener.order.exchange.name=exchange-2
spring.rabbitmq.listener.order.exchange.durable=true
spring.rabbitmq.listener.order.exchange.type=topic
spring.rabbitmq.listener.order.exchange.ignoreDeclarationExceptions=true
spring.rabbitmq.listener.order.key=springboot.*
 ```
4.消费端代码:
```
@RabbitListener(bindings = @QueueBinding(
			value = @Queue(value = "queue-1", 
			durable="true"),
			exchange = @Exchange(value = "exchange-1", 
			durable="true", 
			type= "topic", 
			ignoreDeclarationExceptions = "true"),
			key = "springboot.*"
			)
	)
	@RabbitHandler
	public void onMessage(Message message, Channel channel) throws Exception {
		System.err.println("--------------------------------------");
		System.err.println("消费端Payload: " + message.getPayload());
		Long deliveryTag = (Long)message.getHeaders().get(AmqpHeaders.DELIVERY_TAG);
		//手工ACK
		channel.basicAck(deliveryTag, false);
	}
	
	
	/**
	 * 
	 * @param order
	 * @param channel
	 * @param headers
	 * @throws Exception
	 */
	@RabbitListener(bindings = @QueueBinding(
			value = @Queue(value = "${spring.rabbitmq.listener.order.queue.name}", 
			durable="${spring.rabbitmq.listener.order.queue.durable}"),
			exchange = @Exchange(value = "${spring.rabbitmq.listener.order.exchange.name}", 
			durable="${spring.rabbitmq.listener.order.exchange.durable}", 
			type= "${spring.rabbitmq.listener.order.exchange.type}", 
			ignoreDeclarationExceptions = "${spring.rabbitmq.listener.order.exchange.ignoreDeclarationExceptions}"),
			key = "${spring.rabbitmq.listener.order.key}"
			)
	)
	@RabbitHandler
	public void onOrderMessage(@Payload com.bfxy.springboot.entity.Order order, 
			Channel channel, 
			@Headers Map<String, Object> headers) throws Exception {
		System.err.println("--------------------------------------");
		System.err.println("消费端order: " + order.getId());
		Long deliveryTag = (Long)headers.get(AmqpHeaders.DELIVERY_TAG);
		//手工ACK
		channel.basicAck(deliveryTag, false);
	}
```
