---
title: Celery+RabbitMQ异步任务队列
date: 2019-09-06 18:25:21
categories:
 - python
tags:
 - celery
 - rabbitmq
---
后端开发过程中，有一些比较耗时的任务（大批量数据上传解析）或者耗时无法预估的任务（与第三方交互），或者需要快速响应的场景，需要将这部分放到后台执行，这时就需要使用消息队列进行处理了。
<escape><!-- more --></escape>


celery的详细教程参见 [Celery - 分布式任务队列](http://docs.jinkan.org/docs/celery/index.html#)

### Celery
#### 消息队列简述
Celery是一个消息队列框架，其基本工作流程如下，此流程就是消息队列的实现的流程
![Image1.png](RabbitMQ+Celery异步任务队列/Image1.png)
可以看到一个消息队列，需要有三部分组成： **生产者**，**中间人**，**消费者**
**生产者**：图中的*user application*，产生任务数据
**中间人**：图中的*Broker*，保存生产者产生的任务数据直到被消费者取出
**消费者**：图中的*celer worker X*，从中间人处取任务数据并执行

除此之外，Celery还有一个**Result Storege**，我们知道，消息队列中消费者执行完任务后的返回值如果没有特殊处理是无处安放的，因此Celery提供了一个**Result Storege**用于存储任务的执行返回值
#### 安装
celery是python的一个第三方库，所有直接可以使用pip安装
> pip install celery

#### 使用
下面是个简单的的celery示例代码
###### 消费者
需要指定中间人和定义消费者的具体行为
```python
from celery import Celery

app = Celery(main='tasks', broken='amqp://guest@localhost//')

@app.task()
def celery_task1(*args, **kwargs):
    # TODO: do something
    return 
```
###### 生产者
实际中需要放到消息队列的低分，需要指定消费者
```python
# do something
celery_task1.delay(*args, **kwargs)
# do something
```
###### 中间人
作为一个服务，接受生产者的数据，并提供给消费者
> celery -A ***tasks*** worker -l info  

这里网上的教程中很多都说 **-A** 参数后面让我们用模块名，其实是不对的。
**-A** 参数后面是我们Celery实例所在的位置，且跟实例中 **main** 参数也没有任何关系。

例如，如果我们celery的实例的位置是在 tasks.task1.task1_1 中那么我们的命令将是
>  celery -A tasks.task1.task1_1 worker -l info

当然如果我们在 tasks 中 import tasks.task1.task1_1 后使用上面的也是可以的（等于说有了两种启动中间人的方法）

一般来说一个app实例起一个中间人，当然你也可以将所有app实例聚集在一起（import 或者 代码放在一起的方式），然后启动一个中间人来处理所有任务

#### 环境问题
windows环境若无法正常使用，请切换celery版本至3.4.0，或使用子系统

### RabbitMQ
#### 选用 RabbitMQ 而不是 Redis 的原因
使用 *celery + redis* 作为消息队列有一个坑，redis 作为 celery 的 broker 时，如果一个任务太耗时，任务完成时间超过了 *Redis as Broker* 的时间（Redis默认为一小时）则任务会被再次分配到worker。从而会被重复执行。所以这边使用 RabbitMQ 作为消息队列了。
#### 安装
##### windows环境
###### 安装erlang
RabbitMQ服务端代码是使用并发式语言Erlang编写的，安装Rabbit MQ的前提是安装Erlang  
链接：[http://www.erlang.org/downloads](http://www.erlang.org/downloads)  
###### 安装RabbitMQ  
下载地址：[http://www.rabbitmq.com/download.html](http://www.rabbitmq.com/download.html)  
###### 配置环境变量
erlang 是 安装目录/bin/  
rabbitmq 是 安装目录/sbin/  
###### 启动RabbitMQ服务  
默认情况下会自动启动的，如果没有自动启动可以通过下面这个命令启动  
> rabbitmq-service.bat start  

停止服务
> rabbitmq-service.bat stop  

###### RabbitMQ管理后台
使用 RabbitMQ 管理后台需要先启用  
> rabbitmq-plugins.bat enable rabbitmq_management  

然后访问 http://localhost:15672/ 即可进入管理后台   
##### Linux环境   
###### 安装erlang  
> sudo apt-get install erlang-nox  

###### 安装RabbitMQ  
> sudo apt-get install rabbitmq-server  
  
###### 启动RabbitMQ服务  
> sudo rabbitmq-server start  

 停止服务  
> sudo rabbitmq-server stop  


###### RabbitMQ管理后台  
> sudo rabbitmq-plugins enable rabbitmq_management  



#### 参考资料  
1. [各类消息队列 rabbitMQ/activeMQ/zeroMQ/Kafka/Redis 的比较](https://www.cnblogs.com/valor-xh/p/6348009.html)  