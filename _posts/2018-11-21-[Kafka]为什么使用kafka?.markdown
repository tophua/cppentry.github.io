---
layout:     post
title:      [Kafka]为什么使用kafka?
---
<div id="article_content" class="article_content clearfix csdn-tracking-statistics" data-pid="blog" data-mod="popu_307" data-dsm="post">
								<div class="article-copyright">
					版权声明：本文为博主原创文章，未经博主允许不得转载。					https://blog.csdn.net/SJF0115/article/details/78480433				</div>
								            <link rel="stylesheet" href="https://csdnimg.cn/release/phoenix/template/css/ck_htmledit_views-f76675cdea.css">
						<div class="htmledit_views" id="content_views">
                
<p style="color:rgb(51,51,51);font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-size:14px;">
在介绍为什么使用kafka之前，我们有必要来了解一下什么是kafka？</p>
<h3 id="1-kafka" style="font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-weight:500;line-height:1.1;color:rgb(51,51,51);font-size:1.5em;">
1. 什么是kafka？<a href="http://gitlab.corp.qunar.com/jifeng.si/learningnotes/blob/master/IT/%E5%A4%A7%E6%95%B0%E6%8D%AE/Kafka/%5BKafka%5D%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%BF%E7%94%A8kafka%3F.md#1-kafka" rel="nofollow" style="color:rgb(68,110,155);display:inline-block;width:16px;"></a></h3>
<p style="color:rgb(51,51,51);font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-size:14px;">
Kafka是由LinkedIn开发的一个分布式的消息系统，使用Scala编写，它以可水平扩展和高吞吐率而被广泛使用。目前越来越多的开源分布式处理系统如Storm，Spark，Flink都支持与Kafka集成。现在我们的数据实时处理平台也使用到了kafka。现在它已被多家不同类型的公司作为多种类型的数据管道和消息系统使用。</p>
<h3 id="2" style="font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-weight:500;line-height:1.1;color:rgb(51,51,51);font-size:1.5em;">
2. 为什么使用消息系统？<a href="http://gitlab.corp.qunar.com/jifeng.si/learningnotes/blob/master/IT/%E5%A4%A7%E6%95%B0%E6%8D%AE/Kafka/%5BKafka%5D%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%BF%E7%94%A8kafka%3F.md#2" rel="nofollow" style="background:transparent;color:rgb(68,110,155);"></a></h3>
<p style="color:rgb(51,51,51);font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-size:14px;">
上面我们提到kafka是一个分布式的消息系统。那为什么要在我们的数据处理平台中使用这样的一个消息系统呢？消息系统能给我们带来什么样的好处呢？</p>
<p style="color:rgb(51,51,51);font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-size:14px;">
(1) 解耦</p>
<p style="color:rgb(51,51,51);font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-size:14px;">
在项目启动之初来预测将来项目会碰到什么需求，是极其困难的。消息系统在处理过程中间插入了一个隐含的、基于数据的接口层，两边的处理过程都要实现这一接口。这允许你独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束。</p>
<p style="color:rgb(51,51,51);font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-size:14px;">
(2) 冗余</p>
<p style="color:rgb(51,51,51);font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-size:14px;">
有些情况下，处理数据的过程会失败。除非数据被持久化，否则将造成丢失。消息队列把数据进行持久化直到它们已经被完全处理，通过这一方式规避了数据丢失风险。许多消息队列所采用的"插入-获取-删除"范式中，在把一个消息从队列中删除之前，需要你的处理系统明确的指出该消息已经被处理完毕，从而确保你的数据被安全的保存直到你使用完毕。</p>
<p style="color:rgb(51,51,51);font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-size:14px;">
(3) 扩展性</p>
<p style="color:rgb(51,51,51);font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-size:14px;">
因为消息队列解耦了你的处理过程，所以增大消息入队和处理的频率是很容易的，只要另外增加处理过程即可。不需要改变代码、不需要调节参数。扩展就像调大电力按钮一样简单。</p>
<p style="color:rgb(51,51,51);font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-size:14px;">
(4) 灵活性 &amp; 峰值处理能力</p>
<p style="color:rgb(51,51,51);font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-size:14px;">
在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见；如果为以能处理这类峰值访问为标准来投入资源随时待命无疑是巨大的浪费。使用消息队列能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃。</p>
<p style="color:rgb(51,51,51);font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-size:14px;">
(5) 顺序保证</p>
<p style="color:rgb(51,51,51);font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-size:14px;">
在大多使用场景下，数据处理的顺序都很重要。大部分消息队列本来就是排序的，并且能保证数据会按照特定的顺序来处理。Kafka保证一个Partition内的消息的有序性。</p>
<p style="color:rgb(51,51,51);font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-size:14px;">
(6) 缓冲</p>
<p style="color:rgb(51,51,51);font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-size:14px;">
在任何重要的系统中，都会有需要不同的处理时间的元素。例如，加载一张图片比应用过滤器花费更少的时间。消息队列通过一个缓冲层来帮助任务最高效率的执行———写入队列的处理会尽可能的快速。该缓冲有助于控制和优化数据流经过系统的速度。</p>
<h3 id="3-kafka" style="font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-weight:500;line-height:1.1;color:rgb(51,51,51);font-size:1.5em;">
3. 为什么是kafka？<a href="http://gitlab.corp.qunar.com/jifeng.si/learningnotes/blob/master/IT/%E5%A4%A7%E6%95%B0%E6%8D%AE/Kafka/%5BKafka%5D%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%BF%E7%94%A8kafka%3F.md#3-kafka" rel="nofollow" style="background:transparent;color:rgb(68,110,155);"></a></h3>
<p style="color:rgb(51,51,51);font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-size:14px;">
上面我们知道我们有必要在数据处理系统中使用一个消息系统，但是我们为什么一定要选kafka呢？现在的消息系统可不只有kafka，俗话说得好，货比三家，我们看一下kafka与其他消息系统的区别。</p>
<p style="color:rgb(51,51,51);font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-size:14px;">
LinkedIn团队做了个实验研究，对比Kafka与Apache ActiveMQ V5.4和RabbitMQ V2.4的性能。LinkedIn在两台Linux机器上运行他们的实验，每台机器的配置为8核2GHz、16GB内存，6个磁盘使用RAID10。两台机器通过1GB网络连接。一台机器作为代理，另一台作为生产者或者消费者。</p>
<h4 id="3-1" style="font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-weight:500;line-height:1.1;color:rgb(51,51,51);font-size:1.2em;">
3.1 生产者测试<a href="http://gitlab.corp.qunar.com/jifeng.si/learningnotes/blob/master/IT/%E5%A4%A7%E6%95%B0%E6%8D%AE/Kafka/%5BKafka%5D%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%BF%E7%94%A8kafka%3F.md#3-1" rel="nofollow" style="background:transparent;color:rgb(68,110,155);"></a></h4>
<p style="color:rgb(51,51,51);font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-size:14px;">
对每个系统，运行一个生产者，总共发布1000万条消息，每条消息200字节。Kafka生产者以1和50批量方式发送消息。ActiveMQ和RabbitMQ似乎没有简单的办法来批量发送消息，LinkedIn假定它的批量值为1。结果如下图所示：</p>
<p style="color:rgb(51,51,51);font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-size:14px;">
<img src="https://res.infoq.com/articles/apache-kafka/zh/resources/0609015.png" alt="" style="border:0px;vertical-align:middle;"></p>
<p style="color:rgb(51,51,51);font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-size:14px;">
Kafka性能要好很多的主要原因包括：</p>
<p style="color:rgb(51,51,51);font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-size:14px;">
(1) Kafka不等待代理的确认，以代理能处理的最快速度发送消息。</p>
<p style="color:rgb(51,51,51);font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-size:14px;">
(2)Kafka有更高效的存储格式。平均而言，Kafka每条消息有9字节的开销，而ActiveMQ有144字节。其原因是JMS所需的沉重消息头，以及维护各种索引结构的开销。LinkedIn注意到ActiveMQ一个最忙的线程大部分时间都在存取B-Tree以维护消息元数据和状态。</p>
<h4 id="3-2" style="font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-weight:500;line-height:1.1;color:rgb(51,51,51);font-size:1.2em;">
3.2 消费者测试<a href="http://gitlab.corp.qunar.com/jifeng.si/learningnotes/blob/master/IT/%E5%A4%A7%E6%95%B0%E6%8D%AE/Kafka/%5BKafka%5D%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%BF%E7%94%A8kafka%3F.md#3-2" rel="nofollow" style="background:transparent;color:rgb(68,110,155);"></a></h4>
<p style="color:rgb(51,51,51);font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-size:14px;">
为了做消费者测试，LinkedIn使用一个消费者获取总共1000万条消息。LinkedIn让所有系统每次拉请求都预获取大约相同数量的数据，最多1000条消息或者200KB。对ActiveMQ和RabbitMQ，LinkedIn设置消费者确认模型为自动。结果如下图所示:</p>
<p style="color:rgb(51,51,51);font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-size:14px;">
<img src="https://res.infoq.com/articles/apache-kafka/zh/resources/0609016.png" alt="" style="border:0px;vertical-align:middle;"></p>
<p style="color:rgb(51,51,51);font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-size:14px;">
Kafka性能要好很多的主要原因包括：</p>
<p style="color:rgb(51,51,51);font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-size:14px;">
(1) Kafka有更高效的存储格式；在Kafka中，从代理传输到消费者的字节更少。</p>
<p style="color:rgb(51,51,51);font-family:'Helvetica Neue', Helvetica, Arial, sans-serif;font-size:14px;">
(2) ActiveMQ和RabbitMQ两个容器中的代理必须维护每个消息的传输状态。LinkedIn团队注意到其中一个ActiveMQ线程在测试过程中，一直在将KahaDB页写入磁盘。与此相反，Kafka代理没有磁盘写入动作。最后，Kafka通过使用sendfile API降低了传输开销。</p>
            </div>
                </div>