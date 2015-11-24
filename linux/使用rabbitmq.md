# RabbitMQ
## 简介
[RabbitMQ官网](http://www.rabbitmq.com/)

## 安装
### MAC OS X
``` shell
brew install rabbitmq
echo 'PATH=$PATH:/usr/local/sbin' >> ~/.bash_profile 
```

### Centos 7
``` shell
yum install erlang
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/rabbitmq_v3_5_6/rabbitmq-server-3.5.6-1.noarch.rpm
yum install rabbitmq-server-3.5.6-1.noarch.rpm
```

## 消息队列模式
### rabbitMQ的消息发送流程
Producter -> MQ Server(queue) - > Customer
![](http://www.rabbitmq.com/img/tutorials/python-two.png)

### 消息确认机制（默认打开） 
Message acknowledgment 不启用情况下rabbitMQ当customer接收message后回销毁队列中的message
但是这种情况下当customer正在处理消息时死掉，会导致消息丢失，解决这种情况
rebbit使用acknowledgment，当customer成功处理一个消息会告知server删除这个消息。

### 持久化消息机制
Message durability 消息确认机制保证了client失效的情况下，消息不会丢失
二持久化消息机制保证了server端实效的情况下消息不会丢失

两步：

1、创建对列的时候申明为durable
``` python
channel.queue_declare(queue='hello', durable=True)
```

2、发布消息的时候delivery_mode = 2 
``` python
channel.basic_publish(exchange='',
                      routing_key="task_queue",
                      body=message,
                      properties=pika.BasicProperties(
                         delivery_mode = 2, # make message persistent
                      ))
```

### 均衡的发布消息
设置 basic.qos prefetch_count＝1后，rabbit只会在一个customer成功处理
完一个message后在发送给它新的message.
``` python
channel.basic_qos(prefetch_count=1)
```

## 发布／订阅模式
### 交换器（exchanges）
![](http://www.rabbitmq.com/img/tutorials/exchanges.png)

> P :Product , X : Exchanges , 红色： 队列

### 临时队列(Temporary queues)
－ 系统会自动分配随机的队列名称

－ 在关闭连接时会自动销毁

``` python
result = channel.queue_declare(exclusive=True)
```

### 绑定（Bindings）
![](http://www.rabbitmq.com/img/tutorials/bindings.png)
``` python
channel.queue_bind(exchange='logs',
                   queue=result.method.queue)
```
## 路由
为每个customer 定制消息队列
### 绑定（Bindings）
``` python
channel.queue_bind(exchange=exchange_name,
                   queue=queue_name,
                   routing_key='black')
```

![](http://www.rabbitmq.com/img/tutorials/direct-exchange.png)

## Topics
提供更加灵活的路由转发机制

－ ＊ 占位确切的单词.

－ ＃ 占位0个或多个单词

## Remote procedure call (RPC)
![](http://www.rabbitmq.com/img/tutorials/python-six.png)



