---
layout:     post
title:      Kafka Java示例
---
<div id="article_content" class="article_content clearfix csdn-tracking-statistics" data-pid="blog" data-mod="popu_307" data-dsm="post">
								            <link rel="stylesheet" href="https://csdnimg.cn/release/phoenix/template/css/ck_htmledit_views-f76675cdea.css">
						<div class="htmledit_views" id="content_views">
                
<p>   <span> </span>由于项目原因需要升级kafka，需要重新申请kafka服务，用新的Java API来编写producer和consumer，顺便接触一下Kafka。准备工作就是安装zookeeper和kafka，由于关于安装方面的文章很多，在此就不细说了。</p>
<p><strong>一、一些问题</strong></p>
<p><span style="font-weight:bold;">1.1 kafka错误</span></p>
<div style="line-height:1.75;font-size:14px;display:inline !important;">
<span style="font-weight:bold;">错误：</span></div>
<span style="font-size:14px;text-align:justify;">WARN Error while fetching metadata with correlation id 0 : {test-topic=LEADER_NOT_AVAILABLE} (org.apache.kafka.clients.NetworkClient)</span>
<p><span style="font-size:14px;color:rgb(63,63,63);font-weight:bold;">解决方案</span><span style="font-size:14px;color:rgb(63,63,63);"><strong>：</strong></span><span style="font-size:14px;color:rgb(0,56,132);"><a href="https://github.com/wurstmeister/kafka-docker/issues/85" rel="nofollow" style="font-size:14px;">https://github.com/wurstmeister/kafka-docker/issues/85https://github.com/wurstmeister/kafka-docker/issues/85</a></span></p>
<p><span style="color:rgb(63,63,63);font-size:14px;">config/server.properties：</span></p>
<p><span style="font-size:14px;color:rgb(63,63,63);">&gt;set advertised.host.name = ip。</span></p>
<div style="line-height:1.75;"><span style="font-size:14px;"><strong>1.2 kafka命令行</strong></span></div>
<div style="line-height:1.75;"><span style="font-size:14px;"><strong>&gt;创建topic：</strong>sh kafka-topics.sh --create --zookeeper 10.95.177.100:2181,10.95.97.175:2181,10.95.176.44:2181  --replication-factor 1 --partitions 1 --topic  topic-test<strong><br></strong></span></div>
<div style="line-height:1.75;"><span style="font-size:14px;"><strong>&gt;查看topic：</strong>sh kafka-topics.sh --list --zookeeper 10.95.177.192:2181,10.95.97.175:2181,10.95.176.44:2181<strong><br>
&gt;删除topic：<br></strong>sh kafka-topics.sh --delete --topic topic-test --zookeeper 10.95.177.192:2181,10.95.97.175:2181,10.95.176.44:2181 <strong><br>
&gt;生产消息：</strong>sh kafka-console-producer.sh --broker-list 10.95.177.192:9092,10.95.97.175:9092 --topic topic-test<strong><br>
&gt;消费消息：</strong>sh kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic topic-test --from-beginning  --zookeeper 10.95.177.192:2181,10.95.97.175:2181,10.95.176.44:2181 <strong><br>
&gt;启动kafka服务：<br></strong>./kafka-server-start.sh -daemon ../config/server.properties<strong><br>
&gt;停止kafka服务：<br></strong>./kafka-server-stop<br></span></div>
<div style="line-height:1.75;"><span style="font-weight:bold;color:rgb(79,79,79);text-align:justify;"><br></span></div>
<div style="line-height:1.75;"><span style="font-weight:bold;color:rgb(79,79,79);text-align:justify;">1.3 Java API版本</span></div>
<p>一种：<span style="border:0px;font-size:15px;line-height:inherit;font-family:Arial, 'Helvetica Neue', Helvetica, sans-serif;vertical-align:baseline;color:rgb(36,39,41);text-align:left;">kafka.javaapi.*：<br></span></p>
<pre><code class="language-java">import kafka.javaapi.producer.Producer;
import kafka.producer.KeyedMessage;
import kafka.producer.ProducerConfig;</code></pre>另一种：<span style="border:0px;font-size:15px;line-height:inherit;font-family:Arial, 'Helvetica Neue', Helvetica, sans-serif;vertical-align:baseline;color:rgb(36,39,41);text-align:left;">org.apache.kafka.*:</span>
<p></p>
<p><span style="border:0px;font-size:15px;line-height:inherit;font-family:Arial, 'Helvetica Neue', Helvetica, sans-serif;vertical-align:baseline;color:rgb(36,39,41);text-align:left;"><span style="border:0px;font-size:15px;line-height:inherit;font-family:Arial, 'Helvetica Neue', Helvetica, sans-serif;vertical-align:baseline;color:rgb(36,39,41);text-align:left;"></span></span></p>
<pre><code class="language-java">import org.apache.kafka.clients.producer.KafkaProducer&lt;K,V&gt;
</code></pre>在Kafka 0.8.2之前，kafka.javaapi.producer.Producer是为唯一官方用Scala实现的Java Client。<br>
在Kafka 0.8.2之后，有新的Java Producer API，org.apache.kafka.clients.producer.KafkaProducer,完全用Java实现的。
<p></p>
<p><span style="border:0px;font-size:15px;line-height:inherit;font-family:Arial, 'Helvetica Neue', Helvetica, sans-serif;vertical-align:baseline;color:rgb(36,39,41);text-align:left;"><span style="border:0px;font-size:15px;line-height:inherit;font-family:Arial, 'Helvetica Neue', Helvetica, sans-serif;vertical-align:baseline;color:rgb(36,39,41);text-align:left;"><strong>二、Java
 Kafka示例</strong></span></span></p>
<p style="text-align:left;"><span style="font-family:Arial, 'Helvetica Neue', Helvetica, sans-serif;color:#242729;"><span style="font-size:15px;"><strong>pom.xml:</strong></span></span></p>
<p style="text-align:left;"><span style="font-family:Arial, 'Helvetica Neue', Helvetica, sans-serif;color:#242729;"><span style="font-size:15px;"></span></span></p>
<pre><code class="language-html">&lt;dependencies&gt;
    &lt;dependency&gt;
        &lt;groupId&gt;org.apache.kafka&lt;/groupId&gt;
        &lt;artifactId&gt;kafka-clients&lt;/artifactId&gt;
        &lt;version&gt;0.10.2.0&lt;/version&gt;
    &lt;/dependency&gt;
    &lt;dependency&gt;
        &lt;groupId&gt;org.apache.kafka&lt;/groupId&gt;
        &lt;artifactId&gt;kafka_2.11&lt;/artifactId&gt;
        &lt;version&gt;0.10.2.0&lt;/version&gt;
    &lt;/dependency&gt;
&lt;/dependencies&gt;</code></pre><span style="color:rgb(36,39,41);font-family:Arial, 'Helvetica Neue', Helvetica, sans-serif;font-size:15px;"><strong>2.1 老版本Producer：</strong></span>
<p></p>
<p style="text-align:left;"><span style="color:rgb(36,39,41);font-family:Arial, 'Helvetica Neue', Helvetica, sans-serif;font-size:15px;"><strong>KafkaProducerOld.java:</strong></span></p>
<p style="text-align:left;"><span style="color:rgb(36,39,41);font-family:Arial, 'Helvetica Neue', Helvetica, sans-serif;font-size:15px;"></span></p>
<pre><code class="language-java">import kafka.javaapi.producer.Producer;
import kafka.producer.KeyedMessage;
import kafka.producer.ProducerConfig;

import java.util.Properties;

/**
 * kafka.javaapi.producer.Producer
 * */
public class KafkaProducerOld {

    private Producer&lt;String, String&gt; producer;

    public final static String TOPIC = "didi-topic-test";

    private KafkaProducerOld() {
        Properties props = new Properties();
        props.put("metadata.broker.list", "10.95.177.192:9092,10.95.97.175:9092");
        props.put("serializer.class", "kafka.serializer.StringEncoder");
        ProducerConfig config = new ProducerConfig(props);
        producer = new Producer&lt;String, String&gt;(config);
    }

    public void produce() {
        int messageNo = 0;
        final int COUNT = 10;

        while(messageNo &lt; COUNT) {
            String data = String.format("hello kafka.javaapi.producer message %d", messageNo);

            KeyedMessage&lt;String, String&gt; msg = new KeyedMessage&lt;String, String&gt;(TOPIC, data);
            
            try {
                producer.send(msg);
            } catch (Exception e) {
                e.printStackTrace();
            }

            messageNo++;
        }

        producer.close();
    }

    public static void main(String[] args) {
        new KafkaProducerOld().produce();
    }

}</code></pre><strong>2.2 老版本Consumer</strong>
<p></p>
<p style="text-align:left;"><span style="color:rgb(36,39,41);font-family:Arial, 'Helvetica Neue', Helvetica, sans-serif;font-size:15px;"><strong>KafkaConsumerOld.java:</strong></span></p>
<p style="text-align:left;"><span style="color:rgb(36,39,41);font-family:Arial, 'Helvetica Neue', Helvetica, sans-serif;font-size:15px;"></span></p>
<pre><code class="language-java">import kafka.consumer.ConsumerConfig;
import kafka.consumer.ConsumerIterator;
import kafka.consumer.KafkaStream;
import kafka.javaapi.consumer.ConsumerConnector;
import kafka.serializer.StringDecoder;
import kafka.utils.VerifiableProperties;

import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Properties;

/**
* Consumer: kafka.javaapi.consumer.Consumer
* */
public class KafkaConsumerOld {

    private final ConsumerConnector consumerConnector;

    public final static String TOPIC = "didi-topic-test";

    private KafkaConsumerOld() {
        Properties props = new Properties();
        props.put("zookeeper.connect", "10.95.177.192:2181,10.95.97.175:2181,10.95.176.44:2181");
        props.put("group.id", "group-1");

        //zk连接超时
        props.put("zookeeper.session.timeout.ms", "400");
        props.put("zookeeper.sync.time.ms", "200");
        props.put("auto.commit.interval.ms", "1000");
        props.put("auto.offset.reset", "smallest");
        //序列化类
        props.put("serializer.class", "kafka.serializer.StringDecoder");

        ConsumerConfig config = new ConsumerConfig(props);

        consumerConnector = kafka.consumer.Consumer.createJavaConsumerConnector(config);
    }

    public void consume() {
        Map&lt;String, Integer&gt; topicCountMap = new HashMap&lt;String, Integer&gt;();
        topicCountMap.put(KafkaProducerNew.TOPIC, new Integer(1));

        StringDecoder keyDecoder = new StringDecoder(new VerifiableProperties());
        StringDecoder valueDecoder = new StringDecoder(new VerifiableProperties());

        Map&lt;String, List&lt;KafkaStream&lt;String, String&gt;&gt;&gt; consumerMap = consumerConnector.createMessageStreams(topicCountMap, keyDecoder, valueDecoder);

        List&lt;KafkaStream&lt;String, String&gt;&gt; streams = consumerMap.get(TOPIC);
        for (final KafkaStream stream : streams) {
            ConsumerIterator&lt;String, String&gt; it = stream.iterator();
            while (it.hasNext()) {
                System.out.println("this is kafka consumer : " + new String( it.next().message().toString()) );
            }
        }
    }

    public static void main(String[] args) {
        new KafkaConsumerOld().consume();
    }
}
</code></pre><strong>2.3 新版本Producer</strong>：
<p></p>
<p style="text-align:left;"><span style="color:rgb(36,39,41);font-family:Arial, 'Helvetica Neue', Helvetica, sans-serif;font-size:15px;"><strong>KafkaProducerNew.java:</strong></span></p>
<p style="text-align:left;"><span style="color:rgb(36,39,41);font-family:Arial, 'Helvetica Neue', Helvetica, sans-serif;font-size:15px;"></span></p>
<pre><code class="language-java">import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;

/**
* producer: org.apache.kafka.clients.producer.KafkaProducer;
 * */

public class KafkaProducerNew {

    private final KafkaProducer&lt;String, String&gt; producer;

    public final static String TOPIC = "didi-topic-test";

    private KafkaProducerNew() {
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "10.95.177.192:9092,10.95.97.175:9092");
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

        producer = new KafkaProducer&lt;String, String&gt;(props);
    }

    public void produce() {
        int messageNo = 1;
        final int COUNT = 10;

        while(messageNo &lt; COUNT) {
            String key = String.valueOf(messageNo);
            String data = String.format("hello KafkaProducer message %s", key);
            
            try {
                producer.send(new ProducerRecord&lt;String, String&gt;(TOPIC, data));
            } catch (Exception e) {
                e.printStackTrace();
            }

            messageNo++;
        }
        
        producer.close();
    }

    public static void main(String[] args) {
        new KafkaProducerNew().produce();
    }

}
</code></pre><strong>2.4 新版本Consumer：</strong>
<p></p>
<p style="text-align:left;"><span style="color:rgb(36,39,41);font-family:Arial, 'Helvetica Neue', Helvetica, sans-serif;font-size:15px;"><strong>KafkaConsumerNew.java:</strong></span></p>
<p style="text-align:left;"><span style="color:rgb(36,39,41);font-family:Arial, 'Helvetica Neue', Helvetica, sans-serif;font-size:15px;"></span></p>
<pre><code class="language-java">import org.apache.kafka.clients.consumer.*;

import java.util.Arrays;
import java.util.Properties;

/**
* consumer: org.apache.kafka.clients.consumer.Consumer
* */
public class KafkaConsumerNew {

    private Consumer&lt;String, String&gt; consumer;

    private static String group = "group-1";

    private static String TOPIC = "didi-topic-test";

    private KafkaConsumerNew() {
        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "10.95.177.192:9092,10.95.97.175:9092");
        props.put(ConsumerConfig.GROUP_ID_CONFIG, group);
        props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "latest");
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true"); // 自动commit
        props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "1000"); // 自动commit的间隔
        props.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, "30000");
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");
        consumer = new KafkaConsumer&lt;String, String&gt;(props);
    }

    private void consume() {
        consumer.subscribe(Arrays.asList(TOPIC)); // 可消费多个topic,组成一个list

        while (true) {
            ConsumerRecords&lt;String, String&gt; records = consumer.poll(1000);
            for (ConsumerRecord&lt;String, String&gt; record : records) {
                System.out.printf("offset = %d, key = %s, value = %s \n", record.offset(), record.key(), record.value());
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) {
        new KafkaConsumerNew().consume();
    }
}
</code></pre><br><p></p>
<p style="color:rgb(51,51,51);font-family:Arial;font-size:14px;line-height:26px;">
<span style="font-family:'Microsoft YaHei';font-size:18px;">Author：忆之独秀</span></p>
<p style="color:rgb(51,51,51);font-family:Arial;font-size:14px;line-height:26px;">
<span style="font-family:'Microsoft YaHei';font-size:18px;">Email：leaguenew@qq.com</span></p>
<p style="color:rgb(51,51,51);font-family:Arial;font-size:14px;line-height:26px;">
<span style="font-family:'Microsoft YaHei';font-size:18px;">注明出处：<a href="http://blog.csdn.net/lavorange/article/details/78970977" rel="nofollow">http://blog.csdn.net/lavorange/article/details/78970977</a></span></p>
<p style="color:rgb(51,51,51);font-family:Arial;font-size:14px;line-height:26px;">
<br></p>
            </div>
                </div>