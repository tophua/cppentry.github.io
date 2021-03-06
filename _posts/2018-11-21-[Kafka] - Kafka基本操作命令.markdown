---
layout:     post
title:      [Kafka] - Kafka基本操作命令
---
<div id="article_content" class="article_content clearfix csdn-tracking-statistics" data-pid="blog" data-mod="popu_307" data-dsm="post">
								            <link rel="stylesheet" href="https://csdnimg.cn/release/phoenix/template/css/ck_htmledit_views-f76675cdea.css">
						<div class="htmledit_views" id="content_views">
                
<h1 class="postTitle" style="line-height:1.5;clear:both;font-size:14px;font-family:Verdana, Arial, Helvetica, sans-serif;background-color:rgb(254,254,242);">
<a id="cb_post_title_url" class="postTitle2" href="http://www.cnblogs.com/liuming1992/p/6423458.html" rel="nofollow" style="color:rgb(7,93,179);">[Kafka] - Kafka基本操作命令</a></h1>
<div class="clear" style="clear:both;font-family:Verdana, Arial, Helvetica, sans-serif;background-color:rgb(254,254,242);">
</div>
<div class="postBody" style="line-height:1.5;font-size:13px;font-family:Verdana, Arial, Helvetica, sans-serif;background-color:rgb(254,254,242);">
<div id="cnblogs_post_body">
<p style="line-height:1.5;">
Kafka支持的基本命令位于${KAFKA_HOME}/bin文件夹中，主要是kafka-topics.sh命令；Kafka命令参考页面: <a href="http://kafka.apache.org/082/documentation.html#quickstart" rel="nofollow" style="color:rgb(7,93,179);">kafka-0.8.x-帮助文档</a></p>
<p style="line-height:1.5;">
<img src="http://images2015.cnblogs.com/blog/708990/201702/708990-20170221113215991-2001140852.png" alt="" style="border:0px;"></p>
<p style="line-height:1.5;">
 </p>
<p style="line-height:1.5;">
<span style="line-height:1.5;font-size:18pt;"><span> -1. 查看帮助信息</span></span></p>
<p style="line-height:1.5;">
bin/kafka-topics.sh --help</p>
<p style="line-height:1.5;">
<img src="http://images2015.cnblogs.com/blog/708990/201702/708990-20170221113453460-1048090563.png" alt="" style="border:0px;"></p>
<p style="line-height:1.5;">
 </p>
<p style="line-height:1.5;">
<span><span style="line-height:1.5;font-size:18pt;">-2. 创建Topic</span></span></p>
<p style="line-height:1.5;">
bin/kafka-topics.sh --create --topic test0 --zookeeper 192.168.187.146:2181 --config max.message.bytes=12800000 --config flush.messages=1 --partitions 5 --replication-factor 1</p>
<p style="line-height:1.5;">
--create： 指定创建topic动作</p>
<p style="line-height:1.5;">
--topic：指定新建topic的名称</p>
<p style="line-height:1.5;">
--zookeeper： 指定kafka连接zk的连接url，该值和server.properties文件中的配置项{zookeeper.connect}一样</p>
<p style="line-height:1.5;">
--config：指定当前topic上有效的参数值，参数列表参考文档为: <a href="http://kafka.apache.org/082/documentation.html#brokerconfigs" rel="nofollow" style="color:rgb(7,93,179);">Topic-level configuration</a></p>
<p style="line-height:1.5;">
--partitions：指定当前创建的kafka分区数量，默认为1个</p>
<p style="line-height:1.5;">
--replication-factor：指定每个分区的复制因子个数，默认1个</p>
<p style="line-height:1.5;">
<img src="http://images2015.cnblogs.com/blog/708990/201702/708990-20170221113702195-2072284331.png" alt="" style="border:0px;"></p>
<p style="line-height:1.5;">
 </p>
<p style="line-height:1.5;">
<span style="line-height:1.5;font-size:18pt;"><span>-3. 查看当前Kafka集群中Topic的情况</span></span></p>
<p style="line-height:1.5;">
bin/kafka-topics.sh --list --zookeeper 192.168.187.146:2181</p>
<p style="line-height:1.5;">
<img src="http://images2015.cnblogs.com/blog/708990/201702/708990-20170221114428929-1189630123.png" alt="" style="border:0px;"></p>
<p style="line-height:1.5;">
 </p>
<p style="line-height:1.5;">
<span style="line-height:1.5;font-size:18pt;"><span>-4. 查看对应topic的描述信息</span></span></p>
<p style="line-height:1.5;">
bin/kafka-topics.sh --describe --zookeeper 192.168.187.146:2181  --topic test0</p>
<p style="line-height:1.5;">
--describe： 指定是展示详细信息命令</p>
<p style="line-height:1.5;">
--zookeeper： 指定kafka连接zk的连接url，该值和server.properties文件中的配置项{zookeeper.connect}一样</p>
<p style="line-height:1.5;">
--topic：指定需要展示数据的topic名称</p>
<p style="line-height:1.5;">
<img src="http://images2015.cnblogs.com/blog/708990/201702/708990-20170221114952445-1914574102.png" alt="" style="border:0px;"></p>
<p style="line-height:1.5;">
 </p>
<p style="line-height:1.5;">
<span style="line-height:1.5;font-size:18pt;"><span>-5. Topic信息修改</span></span></p>
<p style="line-height:1.5;">
bin/kafka-topics.sh --zookeeper 192.168.187.146:2181 --alter --topic test0 --config max.message.bytes=128000<br>
bin/kafka-topics.sh --zookeeper 192.168.187.146:2181 --alter --topic test0 --delete-config max.message.bytes<br>
bin/kafka-topics.sh --zookeeper 192.168.187.146:2181 --alter --topic test0 --partitions 10 <br>
bin/kafka-topics.sh --zookeeper 192.168.187.146:2181 --alter --topic test0 --partitions 3 ## Kafka分区数量只允许增加，不允许减少</p>
<p style="line-height:1.5;">
<img src="http://images2015.cnblogs.com/blog/708990/201702/708990-20170221115303007-41563006.png" alt="" style="border:0px;"></p>
<p style="line-height:1.5;">
 </p>
<p style="line-height:1.5;">
<span style="line-height:1.5;font-size:18pt;"><span>-6. Topic删除</span></span></p>
<p style="line-height:1.5;">
默认情况下Kafka的Topic是没法直接删除的，需要进行相关参数配置</p>
<p style="line-height:1.5;">
bin/kafka-topics.sh --delete --topic test0 --zookeeper 192.168.187.146:2181</p>
<p style="line-height:1.5;">
Note: This will have no impact if delete.topic.enable is not set to true.## 默认情况下，删除是标记删除，没有实际删除这个Topic；如果运行删除Topic，两种方式：<br>
方式一：通过delete命令删除后，手动将本地磁盘以及zk上的相关topic的信息删除即可<br>
方式二：配置server.properties文件，给定参数delete.topic.enable=true，重启kafka服务，此时执行delete命令表示允许进行Topic的删除</p>
<p style="line-height:1.5;">
 </p>
</div>
</div>
            </div>
                </div>