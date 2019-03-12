---
title: activemq入门案例
date: 2019-03-12 19:44:03
tags: 消息队列
---

### 安装启动

> 在linux环境下下载好了activemq的压缩包后使用`tar -zxvf` 进行解压即可
>
> **启动/停止命令：**`./activemq start/stop`
>
> 启动后可以访问http://localhost:8161/admin/查看activemq监控页面

### JMS的消息模型

#### 点对点模型（基于队列）

> 消息的生产者和消费者没有时间上的相关性。即：生产者生产了消息但消费者不在线，等到消费者上线以后依旧可以被消费。例如：微信发消息给一个不在线的人，等到他上线后就可看到之前发送的消息
>
> 生产者把消息发送到队列（Queue）中，可以有多个发送者，但只能被一个消费者消费，，一个消息只能被消费一次
>
> 消费者无需订阅，当消费者未消费消息时就会处于阻塞状态

#### 发布者/订阅者模型（基于主题）

> 生产者和消费者之间有时间上的相关性，订阅一个主题的消费者只能消费自它订阅之后发布的消息。即：生产者生产了消息，但是此时消费者未上线，则消费者上线时无法消费生产者之前生产的消息。类似新加了微信群，但加群前的消息是无法被看到的
>
> 生产者将消息发送到主题（Topic）上
>
> 消费者必须先订阅，JMS规范允许提供客户端创建`持久订阅`。即：即使网络断开，消息服务器也记住所有持久化订阅者。如果有新消息，也会知道必定有人来消费

### 基于点对点模型

#### 生产者

~~~java
public class QueueProducer {

    public static void main(String[] args) throws JMSException {
        Connection conn = null;
        Session session = null;
        MessageProducer producer = null;

        try {
            String brokerURL = "tcp://localhost:61616";
            ConnectionFactory connectionFactory = 
                new ActiveMQConnectionFactory(brokerURL);
            conn= connectionFactory.createConnection();
            conn.start();
            session = conn.createSession(false, Session.AUTO_ACKNOWLEDGE);
            Queue queue = session.createQueue("queue test");
            producer = session.createProducer(queue);
            TextMessage msg = session.createTextMessage("Hello World");
            producer.send(msg);
        } catch (JMSException e) {
            e.printStackTrace();
        } finally {
            producer.close();
            session.close();
            conn.close();
        }
    }
}

~~~

#### 消费者

~~~java
public class QueueConsumer {

    public static void main(String[] args) throws Exception {
        String brokerURL = "tcp://localhost:61616";
        ConnectionFactory connFactory = new ActiveMQConnectionFactory(brokerURL);
        Connection conn = connFactory.createConnection();
        conn.start();
        Session session = conn.createSession(false, Session.AUTO_ACKNOWLEDGE);
        Queue queue = session.createQueue("queue test");
        MessageConsumer consumer = session.createConsumer(queue);
        consumer.setMessageListener(message -> {
            if (message instanceof TextMessage) {
                TextMessage msg = (TextMessage) message;
                String text = null;
                try {
                    text = msg.getText();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
                System.out.println(text);
            }
        });

        System.in.read();
        consumer.close();
        session.close();
        conn.close();
    }
}
~~~

### 基于发布者/订阅者模型

#### 生产者

~~~java
public class TopicProducer {

    public static void main(String[] args) throws JMSException {
        Connection conn = null;
        Session session = null;
        MessageProducer producer = null;

        try {
            String brokerURL = "tcp://localhost:61616";
            ConnectionFactory connFactory = new ActiveMQConnectionFactory(brokerURL);
            conn = connFactory.createConnection();
            conn.start();
            session = conn.createSession(false, Session.AUTO_ACKNOWLEDGE);
            Topic topic = session.createTopic("topic test");
            producer = session.createProducer(topic);
            TextMessage msg = session.createTextMessage("Hello Topic Test");
            producer.send(msg);
        } catch (JMSException e) {
            e.printStackTrace();
        } finally {
            conn.close();
            session.close();
            producer.close();
        }
    }
}

~~~



#### 消费者

~~~java
public class TopicConsumer {

    public static void main(String[] args) throws JMSException, IOException {
        String brokerURL = "tcp://localhost:61616";
        ConnectionFactory connFactory = new ActiveMQConnectionFactory(brokerURL);
        Connection conn = connFactory.createConnection();
        conn.start();
        Session session = conn.createSession(false, Session.AUTO_ACKNOWLEDGE);
        Topic topic = session.createTopic("topic test");
        MessageConsumer consumer = session.createConsumer(topic);
        consumer.setMessageListener(message -> {
            if (message instanceof TextMessage) {
                TextMessage tm = (TextMessage) message;
                try {
                    String text = tm.getText();
                    System.out.println(text);
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        });

        System.in.read();
        conn.close();
        session.close();
        consumer.close();
    }
}

~~~

### 总结

> **生产者：**
>
> 1. 创建ConnectionFactory，用于连接broker
>
> 2. 通过工厂创建Connection
>
> 3. 连接启动
>
> 4. 通过连接获取Session会话
>
> 5. 通过Session创建destination，两种目的地Queue、Topic
>
> 6. 通过Session创建MessageProducer
>
> 7. 通过Session创建Message
>
> 8. 通过Producer发送消息



> **消费者：**
>
> 1. 创建ConnectionFactory，用于连接broker
>
> 2. 通过工厂创建Connection
>
> 3. 连接启动
>
> 4. 通过连接获取Session会话
>
> 5. 通过Session创建MessageConsumer
>
> 6. 通过MessageConsumser创建接收信息（receive（同步）或监听器（异步））
>
> 7. 处理信息
>
> 8. 关闭资源

