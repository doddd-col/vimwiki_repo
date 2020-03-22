## RabbitMQ
RabbitAutoConfiguration
1. 自动配置了连接工厂ConnectionFactory
2. RabbitPropertis封装了所有属性
3. RabbitTemplate, 给RabbitMQ发送和接受消息
4. AmqpAdmin, RabbitMQ系统管理功能组件


AmqpAdmin
```java
@Autowired
AmqpAdmin amqpAdmin;
@Test
void createEx(){
	amqpAdmin.declareExchange(new DirectExchange("amqp_ex"));
	amqpAdmin.declareQueue(new Queue("ex_queue",true));
	amqpAdmin.declareBinding(new Binding("ex_queue", Binding.DestinationType.QUEUE,"amqp_ex","amqp",null));
}

# 还有很多API
```

配置MessageConverter
```java
@Bean
public MessageConverter messageConverter(){
	return new Jackson2JsonMessageConverter();
}
```

**通过 @RabbitListener 注解声明 Binding**
通过 @RabbitListener 的 bindings 属性声明 Binding（若 RabbitMQ 中不存在该绑定所需要的 Queue、Exchange、RouteKey 则自动创建，若存在则抛出异常）

```java
@RabbitListener(bindings = @QueueBinding(
        exchange = @Exchange(value = "topic.exchange",durable = "true",type = "topic"),
        value = @Queue(value = "consumer_queue",durable = "true"),
        key = "key.#"
))
public void processMessage1(Message message) {
    System.out.println(message);
}
```

也可以通过@RabbitListener监听
```java
@RabbitListener(queues = "doddd.news")
public void receive(Book book){
	System.out.println("获取到的信息: "+book);
}

@RabbitListener(queues = "aqx.news")
public void receiveMsg(Message message){
	System.out.println(message.getBody());
	System.out.println(message.getMessageProperties());
}
```

@RabbitListener 和 @RabbitHandler 搭配使用
- @RabbitListener 可以标注在类上面，需配合 @RabbitHandler 注解一起使用
- @RabbitListener 标注在类上面表示当有收到消息的时候，就交给 @RabbitHandler 的方法处理，具体使用哪个方法处理，根据 MessageConverter 转换后的参数类型
```java
@Component
@RabbitListener(queues = "consumer_queue")
public class Receiver {

    @RabbitHandler
    public void processMessage1(String message) {
        System.out.println(message);
    }

    @RabbitHandler
    public void processMessage2(byte[] message) {
        System.out.println(new String(message));
    }
    
}
```

  


实例
```java
@Test
void contextLoads() {
	val map = new HashMap<String, Object>();
	map.put("msg", "这是第一个消息");
	map.put("data", Arrays.asList("helloworld", 123, true));
	rabbitTemplate.convertAndSend("exchange.direct", "doddd.news", map);

}

@Test
void receive() {
	val o = rabbitTemplate.receiveAndConvert("doddd.news");
	System.out.println(o.getClass());
	System.out.println(o);

}

@Test
void sendMsg(){
	Book book = Book.builder().name("西游记").author("张三").build();
	rabbitTemplate.convertAndSend("exchange.fanout","",book);
}
```

 
