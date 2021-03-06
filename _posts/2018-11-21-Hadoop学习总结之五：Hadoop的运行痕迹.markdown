---
layout:     post
title:      Hadoop学习总结之五：Hadoop的运行痕迹
---
<div id="article_content" class="article_content clearfix csdn-tracking-statistics" data-pid="blog" data-mod="popu_307" data-dsm="post">
								            <link rel="stylesheet" href="https://csdnimg.cn/release/phoenix/template/css/ck_htmledit_views-f76675cdea.css">
						<div class="htmledit_views" id="content_views">
                
<p>转自：<a href="http://www.cnblogs.com/forfuture1978/archive/2010/11/23/1884967.html" rel="nofollow">博客园 觉先</a></p>
<p></p>
<div class="postTitle" style="font-weight:bolder;font-size:13px;border-bottom-color:rgb(220,220,220);border-bottom-width:1px;border-bottom-style:solid;font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;background-color:rgb(245,245,245);">
<a id="cb_post_title_url" href="http://www.cnblogs.com/forfuture1978/archive/2010/11/23/1884967.html" rel="nofollow" style="color:#000080;text-decoration:none;">Hadoop学习总结之五：Hadoop的运行痕迹</a></div>
<div class="postText" style="font-size:13px;line-height:1.5;margin-left:5px;font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;">
<div id="cnblogs_post_body">
<p style="line-height:1.5;"><span style="line-height:normal;background-color:rgb(245,245,245);"><strong><a id="ctl03_TitleUrl" href="http://www.cnblogs.com/forfuture1978/archive/2010/03/14/1685351.html" rel="nofollow" style="color:#000080;text-decoration:none;">Hadoop
 学习总结之一：HDFS简介</a></strong></span></p>
<p style="line-height:1.5;"><span style="line-height:normal;background-color:rgb(245,245,245);"><strong><a href="http://www.cnblogs.com/forfuture1978/archive/2010/11/10/1874222.html" rel="nofollow" style="color:#000080;text-decoration:none;">Hadoop学习总结之二：HDFS读写过程解析</a></strong></span></p>
<p style="line-height:1.5;"><span style="line-height:normal;background-color:rgb(245,245,245);"><strong><a href="http://www.cnblogs.com/forfuture1978/archive/2010/11/14/1877086.html" rel="nofollow" style="color:#000080;text-decoration:none;">Hadoop学习总结之三：Map-Reduce入门</a></strong></span></p>
<p style="line-height:1.5;"><span style="line-height:normal;background-color:rgb(245,245,245);"><strong><a href="http://www.cnblogs.com/forfuture1978/archive/2010/11/19/1882268.html" rel="nofollow" style="color:#000080;text-decoration:none;">Hadoop学习总结之四：Map-Reduce的过程解析</a></strong></span> </p>
<p style="line-height:1.5;"> </p>
<p style="line-height:1.5;">在使用hadoop的时候，可能遇到各种各样的问题，然而由于hadoop的运行机制比较复杂，因而出现了问题的时候比较难于发现问题。</p>
<p style="line-height:1.5;">本文欲通过某种方式跟踪Hadoop的运行痕迹，方便出现问题的时候可以通过这些痕迹来解决问题。</p>
<h1 style="font-size:28px;line-height:56px;">一、环境的搭建</h1>
<p style="line-height:1.5;">为了能够跟踪这些运行的痕迹，我们需要搭建一个特殊的环境，从而可以一步步的查看上一节提到的一些关键步骤所引起的变化。</p>
<p style="line-height:1.5;">我们首先搭建一个拥有一个NameNode(namenode:192.168.1.104)，三个DataNode(datanode01:192.168.1.105, datanode02:192.168.1.106, datanode03:192.168.1.107)的Hadoop环境，其中SecondaryNameNode和NameNode运行在同一台机器上。</p>
<p style="line-height:1.5;">对于这四台机器上的Hadoop，我们需要进行如下相同的配置：</p>
<ul style="margin-left:45px;"><li style="list-style: !important;">NameNode，SeondaryNameNode，JobTracker都应该运行在namenode:192.168.1.104机器上</li><li style="list-style: !important;">DataNode，TaskTracker，以及生成的Map和Reduce的Task JVM应该运行在datanode01, datanode02, datanode03上</li><li style="list-style: !important;">数据共有三份备份</li><li style="list-style: !important;">HDFS以及Map-Reduce运行的数据放在/data/hadoop/dir/tmp文件夹下</li></ul><table cellspacing="0" cellpadding="2" width="400" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="400" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;">&lt;property&gt;</p>
<p style="font-size:13px;line-height:1.5;">  &lt;name&gt;fs.default.name&lt;/name&gt;</p>
<p style="font-size:13px;line-height:1.5;">  &lt;value&gt;hdfs://192.168.1.104:9000&lt;/value&gt;</p>
<p style="font-size:13px;line-height:1.5;">&lt;/property&gt;</p>
<p style="font-size:13px;line-height:1.5;">&lt;property&gt;</p>
<p style="font-size:13px;line-height:1.5;">  &lt;name&gt;mapred.job.tracker&lt;/name&gt;</p>
<p style="font-size:13px;line-height:1.5;">  &lt;value&gt;192.168.1.104:9001&lt;/value&gt;</p>
<p style="font-size:13px;line-height:1.5;">&lt;/property&gt;</p>
<p style="font-size:13px;line-height:1.5;">&lt;property&gt;</p>
<p style="font-size:13px;line-height:1.5;">  &lt;name&gt;dfs.replication&lt;/name&gt;</p>
<p style="font-size:13px;line-height:1.5;">  &lt;value&gt;3&lt;/value&gt;</p>
<p style="font-size:13px;line-height:1.5;">&lt;/property&gt;</p>
<p style="font-size:13px;line-height:1.5;">&lt;property&gt;</p>
<p style="font-size:13px;line-height:1.5;">  &lt;name&gt;hadoop.tmp.dir&lt;/name&gt;</p>
<p style="font-size:13px;line-height:1.5;">  &lt;value&gt;<strong>/data/hadoopdir/tmp</strong>&lt;/value&gt;</p>
<p style="font-size:13px;line-height:1.5;">  &lt;description&gt;A base for other temporary directories.&lt;/description&gt;</p>
<p style="font-size:13px;line-height:1.5;">&lt;/property&gt;</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">然而由于Map-Reduce过程相对复杂，为了能够对Map和Reduce的Task JVM进行远程的调试，从而能一步一步观察，因而对NameNode和三个DataNode有一些不同的配置：</p>
<p style="line-height:1.5;">对于NameNode:</p>
<ul style="margin-left:45px;"><li style="list-style: !important;">设置mapred.job.reuse.jvm.num.tasks为-1，使得多个运行于同一个DataNode上的Map和Reduce的Task共用同一个JVM，从而方便对此JVM进行远程调试，并且不会因为多个Task JVM监听同一个远程调试端口而发生冲突</li><li style="list-style: !important;">对于mapred.tasktracker.map.tasks.maximum和mapred.tasktracker.reduce.tasks.maximum的配置以DataNode上的为准</li><li style="list-style: !important;">设置io.sort.mb为1M(原来为100M)，是为了在Map阶段让内存中的map output尽快的spill到文件中来，从而我们可以观察map的输出</li><li style="list-style: !important;">设置mapred.child.java.opts的时候，即设置Task JVM的运行参数，添加远程调试监听端口8333</li></ul><table cellspacing="0" cellpadding="2" width="734" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="732" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;">  &lt;property&gt;<br>
    &lt;name&gt;mapred.job.reuse.jvm.num.tasks&lt;/name&gt;<br>
    &lt;value&gt;-1&lt;/value&gt;<br>
    &lt;description&gt;&lt;/description&gt;<br>
  &lt;/property&gt;<br>
  &lt;property&gt;<br>
    &lt;name&gt;mapred.tasktracker.map.tasks.maximum&lt;/name&gt;<br>
    &lt;value&gt;1&lt;/value&gt;<br>
    &lt;description&gt;&lt;/description&gt;<br>
  &lt;/property&gt;<br>
  &lt;property&gt;<br>
    &lt;name&gt;mapred.tasktracker.reduce.tasks.maximum&lt;/name&gt;<br>
    &lt;value&gt;1&lt;/value&gt;<br>
    &lt;description&gt;&lt;/description&gt;<br>
  &lt;/property&gt;<br>
  &lt;property&gt;<br>
    &lt;name&gt;io.sort.mb&lt;/name&gt;<br>
    &lt;value&gt;1&lt;/value&gt;<br>
    &lt;description&gt;&lt;/description&gt;<br>
  &lt;/property&gt;<br>
  &lt;property&gt;<br>
    &lt;name&gt;mapred.child.java.opts&lt;/name&gt;<br>
    &lt;value&gt;-Xmx200m -agentlib:jdwp=transport=dt_socket,address=8883,server=y,suspend=y&lt;/value&gt;<br>
    &lt;description&gt;&lt;/description&gt;<br>
  &lt;/property&gt;</p>
<p style="font-size:13px;line-height:1.5;">  &lt;property&gt;<br>
    &lt;name&gt;mapred.job.shuffle.input.buffer.percent&lt;/name&gt;<br>
    &lt;value&gt;0.001&lt;/value&gt;<br>
    &lt;description&gt;&lt;/description&gt;<br>
  &lt;/property&gt;</p>
<p style="font-size:13px;line-height:1.5;">&lt;property&gt;<br>
    &lt;name&gt;mapred.job.shuffle.merge.percent&lt;/name&gt;<br>
    &lt;value&gt;0.001&lt;/value&gt;<br>
    &lt;description&gt;&lt;/description&gt;<br>
    &lt;/property&gt;</p>
<p style="font-size:13px;line-height:1.5;">  &lt;property&gt;<br>
    &lt;name&gt;io.sort.factor&lt;/name&gt;<br>
    &lt;value&gt;2&lt;/value&gt;<br>
    &lt;description&gt;&lt;/description&gt;<br>
  &lt;/property&gt;</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">对于DataNode:</p>
<ul style="margin-left:45px;"><li style="list-style: !important;">对于datanode01:192.168.1.105，设置同时运行的map task的个数(mapred.tasktracker.map.tasks.maximum)为1，同时运行的reduce task的个数(mapred.tasktracker.reduce.tasks.maximum)为0</li><li style="list-style: !important;">对于datanode02:192.168.1.106，设置同时运行的map task的个数(mapred.tasktracker.map.tasks.maximum)为0，同时运行的reduce task的个数(mapred.tasktracker.reduce.tasks.maximum)为0</li><li style="list-style: !important;">对于datanode02:192.168.1.107，设置同时运行的map task的个数(mapred.tasktracker.map.tasks.maximum)为0，同时运行的reduce task的个数(mapred.tasktracker.reduce.tasks.maximum)为1</li><li style="list-style: !important;">之所以这样设置，是因为我们虽然可以控制多个Map task共用同一个JVM，然而我们不能控制Map task和Reduce Task也共用一个JVM。从而当Map task的JVM和Reduce Task的JVM同时在同一台机器上启动的时候，仍然会出现监听远程调用端口冲突的问题。</li><li style="list-style: !important;">经过上面的设置，从而datanode01专门负责运行Map Task，datanode03专门负责运行Reduce Task，而datanode02不运行任何的Task，甚至连TaskTracker也不用启动了</li><li style="list-style: !important;">对于Reduce Task设置mapred.job.shuffle.input.buffer.percent和mapred.job.shuffle.merge.percent为0.001，从而使得拷贝，合并阶段的中间结果都因为内存设置过小而写入硬盘，我们能够看到痕迹</li><li style="list-style: !important;">设置io.sort.factor为2，使得在map task输出不多的情况下，也能触发合并。</li></ul><p style="line-height:1.5;">除了对Map task和Reduce Task进行远程调试之外，我们还想对NameNode，SecondaryName，DataNode，JobTracker，TaskTracker进行远程调试，则需要修改一下bin/hadoop文件：</p>
<table cellspacing="0" cellpadding="2" width="782" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="780" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;">if [ "$COMMAND" = "<strong>namenode</strong>" ] ; then</p>
<p style="font-size:13px;line-height:1.5;">  CLASS='org.apache.hadoop.hdfs.server.namenode.NameNode'</p>
<p style="font-size:13px;line-height:1.5;">  HADOOP_OPTS="$HADOOP_OPTS $HADOOP_NAMENODE_OPTS -agentlib:jdwp=transport=dt_socket,address=<strong>8888</strong>,server=y,suspend=n"</p>
<p style="font-size:13px;line-height:1.5;">elif [ "$COMMAND" = "<strong>secondarynamenode</strong>" ] ; then</p>
<p style="font-size:13px;line-height:1.5;">  CLASS='org.apache.hadoop.hdfs.server.namenode.SecondaryNameNode'</p>
<p style="font-size:13px;line-height:1.5;">  HADOOP_OPTS="$HADOOP_OPTS $HADOOP_SECONDARYNAMENODE_OPTS -agentlib:jdwp=transport=dt_socket,address=<strong>8887</strong>,server=y,suspend=n"</p>
<p style="font-size:13px;line-height:1.5;">elif [ "$COMMAND" = "<strong>datanode</strong>" ] ; then</p>
<p style="font-size:13px;line-height:1.5;">  CLASS='org.apache.hadoop.hdfs.server.datanode.DataNode'</p>
<p style="font-size:13px;line-height:1.5;">  HADOOP_OPTS="$HADOOP_OPTS $HADOOP_DATANODE_OPTS -agentlib:jdwp=transport=dt_socket,address=<strong>8886</strong>,server=y,suspend=n"</p>
<p style="font-size:13px;line-height:1.5;">……</p>
<p style="font-size:13px;line-height:1.5;">elif [ "$COMMAND" = "<strong>jobtracker</strong>" ] ; then</p>
<p style="font-size:13px;line-height:1.5;">  CLASS=org.apache.hadoop.mapred.JobTracker</p>
<p style="font-size:13px;line-height:1.5;">  HADOOP_OPTS="$HADOOP_OPTS $HADOOP_JOBTRACKER_OPTS -agentlib:jdwp=transport=dt_socket,address=<strong>8885</strong>,server=y,suspend=n"</p>
<p style="font-size:13px;line-height:1.5;">elif [ "$COMMAND" = "<strong>tasktracker</strong>" ] ; then</p>
<p style="font-size:13px;line-height:1.5;">  CLASS=org.apache.hadoop.mapred.TaskTracker</p>
<p style="font-size:13px;line-height:1.5;">  HADOOP_OPTS="$HADOOP_OPTS $HADOOP_TASKTRACKER_OPTS -agentlib:jdwp=transport=dt_socket,address=<strong>8884</strong>,server=y,suspend=n"</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">在进行一切实验之前，我们首先清空/data/hadoopdir/tmp以及logs文件夹。</p>
<p style="line-height:1.5;"> </p>
<h1 style="font-size:28px;line-height:56px;">二、格式化HDFS</h1>
<p style="line-height:1.5;">格式化HDFS需要运行命令：bin/hadoop namenode –format</p>
<p style="line-height:1.5;">于是打印出如下的日志：</p>
<table cellspacing="0" cellpadding="2" width="779" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="777" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;">10/11/20 19:52:21 INFO namenode.NameNode: STARTUP_MSG:<br>
/************************************************************<br>
STARTUP_MSG: Starting NameNode<br>
STARTUP_MSG:   host = namenode/192.168.1.104<br>
STARTUP_MSG:   args = [-format]<br>
STARTUP_MSG:   version = 0.19.2<br>
STARTUP_MSG:   build = <a href="https://svn.apache.org/repos/asf/hadoop/common/branches/branch-0.19" rel="nofollow" style="color:#000080;text-decoration:none;">https://svn.apache.org/repos/asf/hadoop/common/branches/branch-0.19</a> -r 789657; compiled by 'root'
 on Tue Jun 30 12:40:50 EDT 2009<br>
************************************************************/<br>
10/11/20 19:52:21 INFO namenode.FSNamesystem: fsOwner=admin,sambashare<br>
10/11/20 19:52:21 INFO namenode.FSNamesystem: supergroup=supergroup<br>
10/11/20 19:52:21 INFO namenode.FSNamesystem: isPermissionEnabled=true<br>
10/11/20 19:52:21 INFO common.Storage: Image file of size 97 saved in 0 seconds.<br>
10/11/20 19:52:21 INFO common.Storage: Storage directory <strong>/data/hadoopdir/tmp/dfs/name</strong> has been successfully formatted.<br>
10/11/20 19:52:21 INFO namenode.NameNode: SHUTDOWN_MSG:<br>
/************************************************************<br>
SHUTDOWN_MSG: Shutting down NameNode at namenode/192.168.1.104<br>
************************************************************/</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">这个时候在NameNode的/data/hadoopdir/tmp下面出现如下的文件树形结构：</p>
<table cellspacing="0" cellpadding="2" width="400" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="400" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
+- dfs<br>
       +- name<br>
              +--- current<br>
                         +---- edits<br>
                         +---- fsimage<br>
                         +---- fstime<br>
                         +---- VERSION<br>
              +---image<br>
                         +---- fsimage</td>
</tr></tbody></table><p style="line-height:1.5;">这个时候，DataNode的/data/hadoopdir/tmp中还是空的。</p>
<p style="line-height:1.5;"> </p>
<h1 style="font-size:28px;line-height:56px;">二、启动Hadoop</h1>
<p style="line-height:1.5;">启动Hadoop需要调用命令bin/start-all.sh，输出的日志如下：</p>
<table cellspacing="0" cellpadding="2" width="782" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="780" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;">starting namenode, logging to logs/hadoop-namenode-namenode.out</p>
<p style="font-size:13px;line-height:1.5;">192.168.1.106: starting datanode, logging to logs/hadoop-datanode-datanode02.out</p>
<p style="font-size:13px;line-height:1.5;">192.168.1.105: starting datanode, logging to logs/hadoop-datanode-datanode01.out</p>
<p style="font-size:13px;line-height:1.5;">192.168.1.107: starting datanode, logging to logs/hadoop-datanode-datanode03.out</p>
<p style="font-size:13px;line-height:1.5;">192.168.1.104: starting secondarynamenode, logging to logs/hadoop-secondarynamenode-namenode.out</p>
<p style="font-size:13px;line-height:1.5;">starting jobtracker, logging to logs/hadoop-jobtracker-namenode.out</p>
<p style="font-size:13px;line-height:1.5;">192.168.1.106: starting tasktracker, logging to logs/hadoop-tasktracker-datanode02.out</p>
<p style="font-size:13px;line-height:1.5;">192.168.1.105: starting tasktracker, logging to logs/hadoop-tasktracker-datanode01.out</p>
<p style="font-size:13px;line-height:1.5;">192.168.1.107: starting tasktracker, logging to logs/hadoop-tasktracker-datanode03.out</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">从日志中我们可以看出，此脚本启动了NameNode, 三个DataNode，SecondaryName，JobTracker以及三个TaskTracker.</p>
<p style="line-height:1.5;">下面我们分别从NameNode和三个DataNode中运行jps -l，看看到底运行了那些java程序：</p>
<p style="line-height:1.5;">在NameNode中：</p>
<table cellspacing="0" cellpadding="2" width="524" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="522" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;"> </p>
<p style="font-size:13px;line-height:1.5;">22214 org.apache.hadoop.hdfs.server.namenode.SecondaryNameNode</p>
<p style="font-size:13px;line-height:1.5;">22107 org.apache.hadoop.hdfs.server.namenode.NameNode</p>
<p style="font-size:13px;line-height:1.5;">22271 org.apache.hadoop.mapred.JobTracker</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">在datanode01中：</p>
<table cellspacing="0" cellpadding="2" width="523" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="521" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;">12580 org.apache.hadoop.mapred.TaskTracker</p>
<p style="font-size:13px;line-height:1.5;">12531 org.apache.hadoop.hdfs.server.datanode.DataNode</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">在datanode02中：</p>
<table cellspacing="0" cellpadding="2" width="523" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="521" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;">10548 org.apache.hadoop.hdfs.server.datanode.DataNode</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">在datanode03中：</p>
<table cellspacing="0" cellpadding="2" width="523" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="521" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;">12593 org.apache.hadoop.hdfs.server.datanode.DataNode</p>
<p style="font-size:13px;line-height:1.5;">12644 org.apache.hadoop.mapred.TaskTracker</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">同我们上面的配置完全符合。</p>
<p style="line-height:1.5;">当启动了Hadoop以后，/data/hadoopdir/tmp目录也发生了改变，通过ls -R我们可以看到。</p>
<p style="line-height:1.5;">对于NameNode：</p>
<ul style="margin-left:45px;"><li style="list-style: !important;">在name文件夹中，多了in_use.lock文件，说明NameNode已经启动了</li><li style="list-style: !important;">多了nameseondary文件夹，用于存放SecondaryNameNode的数据</li></ul><table cellspacing="0" cellpadding="2" width="518" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="516" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;"></p>
<p style="font-size:13px;line-height:1.5;">.:</p>
<p style="font-size:13px;line-height:1.5;">dfs</p>
<p style="font-size:13px;line-height:1.5;">./dfs:</p>
<p style="font-size:13px;line-height:1.5;">name  <strong>namesecondary</strong></p>
<p style="font-size:13px;line-height:1.5;">./dfs/name:</p>
<p style="font-size:13px;line-height:1.5;">current  image  <strong>in_use.lock</strong></p>
<p style="font-size:13px;line-height:1.5;">./dfs/name/current:</p>
<p style="font-size:13px;line-height:1.5;">edits  fsimage  fstime  VERSION</p>
<p style="font-size:13px;line-height:1.5;">./dfs/name/image:</p>
<p style="font-size:13px;line-height:1.5;">fsimage</p>
<p style="font-size:13px;line-height:1.5;">./dfs/namesecondary:</p>
<p style="font-size:13px;line-height:1.5;">current  image  <strong>in_use.lock</strong></p>
<p style="font-size:13px;line-height:1.5;">./dfs/namesecondary/current:</p>
<p style="font-size:13px;line-height:1.5;">edits  fsimage  fstime  VERSION</p>
<p style="font-size:13px;line-height:1.5;">./dfs/namesecondary/image:</p>
<p style="font-size:13px;line-height:1.5;">fsimage</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">对于DataNode:</p>
<ul style="margin-left:45px;"><li style="list-style: !important;">多了dfs和mapred两个文件夹</li><li style="list-style: !important;">dfs文件夹用于存放HDFS的block数据的</li><li style="list-style: !important;">mapred用于存放Map-Reduce Task任务执行所需要的数据的。</li></ul><table cellspacing="0" cellpadding="2" width="518" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="516" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;"></p>
<p style="font-size:13px;line-height:1.5;">.:</p>
<p style="font-size:13px;line-height:1.5;"><strong>dfs  mapred</strong></p>
<p style="font-size:13px;line-height:1.5;">./dfs:</p>
<p style="font-size:13px;line-height:1.5;">data</p>
<p style="font-size:13px;line-height:1.5;">./dfs/data:</p>
<p style="font-size:13px;line-height:1.5;">current  detach  in_use.lock  storage  tmp</p>
<p style="font-size:13px;line-height:1.5;">./dfs/data/current:</p>
<p style="font-size:13px;line-height:1.5;">dncp_block_verification.log.curr  VERSION</p>
<p style="font-size:13px;line-height:1.5;">./dfs/data/detach:</p>
<p style="font-size:13px;line-height:1.5;">./dfs/data/tmp:</p>
<p style="font-size:13px;line-height:1.5;">./mapred:</p>
<p style="font-size:13px;line-height:1.5;">local</p>
<p style="font-size:13px;line-height:1.5;">./mapred/local:</p>
</td>
</tr></tbody></table><p style="line-height:1.5;"> </p>
<p style="line-height:1.5;">当然随着Hadoop的启动，logs文件夹下也多个很多的日志：</p>
<p style="line-height:1.5;">在NameNode上，日志有：</p>
<ul style="margin-left:45px;"><li style="list-style: !important;">NameNode的日志：
<ul style="list-style-type:disc;margin-left:45px;"><li style="list-style: !important;">hadoop-namenode-namenode.log此为log4j的输出日志</li><li style="list-style: !important;">hadoop-namenode-namenode.out此为stdout和stderr的输出日志</li></ul></li><li style="list-style: !important;">SecondaryNameNode的日志：
<ul style="list-style-type:disc;margin-left:45px;"><li style="list-style: !important;">hadoop-secondarynamenode-namenode.log此为log4j的输出日志</li><li style="list-style: !important;">hadoop-secondarynamenode-namenode.out此为stdout和stderr的输出日志</li></ul></li><li style="list-style: !important;">JobTracker的日志：
<ul style="list-style-type:disc;margin-left:45px;"><li style="list-style: !important;">hadoop-jobtracker-namenode.log此为log4j的输出日志</li><li style="list-style: !important;">hadoop-jobtracker-namenode.out此为stdout和stderr的输出日志</li></ul></li></ul><p style="line-height:1.5;">在DataNode上的日志有(以datanode01为例子)：</p>
<ul style="margin-left:45px;"><li style="list-style: !important;">DataNode的日志
<ul style="list-style-type:disc;margin-left:45px;"><li style="list-style: !important;">hadoop-datanode-datanode01.log此为log4j的输出日志</li><li style="list-style: !important;">hadoop-datanode-datanode01.out此为stdout和stderr的输出日志</li></ul></li><li style="list-style: !important;">TaskTracker的日志
<ul style="list-style-type:disc;margin-left:45px;"><li style="list-style: !important;">hadoop-tasktracker-datanode01.log此为log4j的输出日志</li><li style="list-style: !important;">hadoop-tasktracker-datanode01.out此为stdout和stderr的输出日志</li></ul></li></ul><p style="line-height:1.5;">下面我们详细查看这些日志中的有重要意义的信息：</p>
<p style="line-height:1.5;">在hadoop-namenode-namenode.log文件中，我们可以看到NameNode启动的过程：</p>
<table cellspacing="0" cellpadding="2" width="709" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="707" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;">Namenode up at: namenode/192.168.1.104:9000</p>
<p style="font-size:13px;line-height:1.5;"><strong>//文件的数量</strong></p>
<p style="font-size:13px;line-height:1.5;">Number of files = 0</p>
<p style="font-size:13px;line-height:1.5;">Number of files under construction = 0</p>
<p style="font-size:13px;line-height:1.5;"><strong>//加载fsimage和edits文件形成FSNamesystem</strong></p>
<p style="font-size:13px;line-height:1.5;">Image file of size 97 loaded in 0 seconds.</p>
<p style="font-size:13px;line-height:1.5;">Edits file /data/hadoopdir/tmp/dfs/name/current/edits of size 4 edits # 0 loaded in 0 seconds.</p>
<p style="font-size:13px;line-height:1.5;">Image file of size 97 saved in 0 seconds.</p>
<p style="font-size:13px;line-height:1.5;">Finished loading FSImage in 12812 msecs</p>
<p style="font-size:13px;line-height:1.5;"><strong>//统计block的数量和状态</strong></p>
<p style="font-size:13px;line-height:1.5;">Total number of blocks = 0</p>
<p style="font-size:13px;line-height:1.5;">Number of invalid blocks = 0</p>
<p style="font-size:13px;line-height:1.5;">Number of under-replicated blocks = 0</p>
<p style="font-size:13px;line-height:1.5;">Number of  over-replicated blocks = 0</p>
<p style="font-size:13px;line-height:1.5;"><strong>//离开safe mode</strong></p>
<p style="font-size:13px;line-height:1.5;">Leaving safe mode after 12 secs.</p>
<p style="font-size:13px;line-height:1.5;"><strong>//注册DataNode</strong></p>
<p style="font-size:13px;line-height:1.5;">Adding a new node: /default-rack/192.168.1.106:50010</p>
<p style="font-size:13px;line-height:1.5;">Adding a new node: /default-rack/192.168.1.105:50010</p>
<p style="font-size:13px;line-height:1.5;">Adding a new node: /default-rack/192.168.1.107:50010</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">在hadoop-secondarynamenode-namenode.log文件中，我们可以看到SecondaryNameNode的启动过程：</p>
<table cellspacing="0" cellpadding="2" width="712" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="710" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;">Secondary Web-server up at: 0.0.0.0:50090</p>
<p style="font-size:13px;line-height:1.5;"><strong>//进行Checkpoint的周期</strong></p>
<p style="font-size:13px;line-height:1.5;">Checkpoint Period   :3600 secs (60 min)</p>
<p style="font-size:13px;line-height:1.5;">Log Size Trigger    :67108864 bytes (65536 KB)</p>
<p style="font-size:13px;line-height:1.5;"><strong>//进行一次checkpoint，从NameNode下载fsimage和edits</strong></p>
<p style="font-size:13px;line-height:1.5;">Downloaded file fsimage size 97 bytes.</p>
<p style="font-size:13px;line-height:1.5;">Downloaded file edits size 370 bytes.</p>
<p style="font-size:13px;line-height:1.5;"><strong>//加载edit文件，进行合并，将合并后的fsimage保存，我们可以看到fsimage变大了</strong></p>
<p style="font-size:13px;line-height:1.5;">Edits file /data/hadoopdir/tmp/dfs/namesecondary/current/edits of size 370 edits # 6 loaded in 0 seconds.</p>
<p style="font-size:13px;line-height:1.5;">Image file of size 540 saved in 0 seconds.</p>
<p style="font-size:13px;line-height:1.5;"><strong>//此次checkpoint结束</strong></p>
<p style="font-size:13px;line-height:1.5;">Checkpoint done. New Image Size: 540</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">在hadoop-jobtracker-namenode.log文件中，我们可以看到JobTracker的启动过程：</p>
<table cellspacing="0" cellpadding="2" width="711" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="709" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;">JobTracker up at: 9001</p>
<p style="font-size:13px;line-height:1.5;">JobTracker webserver: 50030</p>
<p style="font-size:13px;line-height:1.5;"><strong>//清除HDFS中的/data/hadoopdir/tmp/mapred/system文件夹，是用于Map-Reduce运行过程中保存数据的</strong></p>
<p style="font-size:13px;line-height:1.5;">Cleaning up the system directory</p>
<p style="font-size:13px;line-height:1.5;"><strong>//不断的从TaskTracker收到heartbeat，第一次是注册TaskTracker</strong></p>
<p style="font-size:13px;line-height:1.5;">Got heartbeat from: tracker_datanode01:localhost/127.0.0.1:58297</p>
<p style="font-size:13px;line-height:1.5;">Adding a new node: /default-rack/datanode01</p>
<p style="font-size:13px;line-height:1.5;">Got heartbeat from: tracker_datanode03:localhost/127.0.0.1:37546</p>
<p style="font-size:13px;line-height:1.5;">Adding a new node: /default-rack/datanode03</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">在hadoop-datanode-datanode01.log中，可以看到DataNode的启动过程：</p>
<table cellspacing="0" cellpadding="2" width="708" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="706" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;"><strong>//格式化DataNode存放block的文件夹</strong></p>
<p style="font-size:13px;line-height:1.5;">Storage directory /data/hadoopdir/tmp/dfs/data is not formatted.</p>
<p style="font-size:13px;line-height:1.5;">Formatting ...</p>
<p style="font-size:13px;line-height:1.5;"><strong>//启动DataNode</strong></p>
<p style="font-size:13px;line-height:1.5;">Opened info server at 50010</p>
<p style="font-size:13px;line-height:1.5;">Balancing bandwith is 1048576 bytes/s</p>
<p style="font-size:13px;line-height:1.5;">Initializing JVM Metrics with processName=DataNode, sessionId=null</p>
<p style="font-size:13px;line-height:1.5;"><strong>//向NameNode注册此DataNode</strong></p>
<p style="font-size:13px;line-height:1.5;">dnRegistration = DatanodeRegistration(datanode01:50010, storageID=, infoPort=50075, ipcPort=50020)</p>
<p style="font-size:13px;line-height:1.5;">New storage id DS-1042573498-192.168.1.105-50010-1290313555129 is assigned to data-node 192.168.1.105:5001</p>
<p style="font-size:13px;line-height:1.5;">DatanodeRegistration(192.168.1.105:50010, storageID=DS-1042573498-192.168.1.105-50010-1290313555129, infoPort=50075, ipcPort=50020)In DataNode.run, data = FSDataset{dirpath='/data/hadoopdir/tmp/dfs/data/current'}</p>
<p style="font-size:13px;line-height:1.5;"><strong>//启动block scanner</strong></p>
<p style="font-size:13px;line-height:1.5;">Starting Periodic block scanner.</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">在hadoop-tasktracker-datanode01.log中，可以看到TaskTracker的启动过程：</p>
<table cellspacing="0" cellpadding="2" width="709" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="707" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;"><strong>//启动TaskTracker</strong></p>
<p style="font-size:13px;line-height:1.5;">Initializing JVM Metrics with processName=TaskTracker, sessionId=</p>
<p style="font-size:13px;line-height:1.5;">TaskTracker up at: localhost/127.0.0.1:58297</p>
<p style="font-size:13px;line-height:1.5;">Starting tracker tracker_datanode01:localhost/127.0.0.1:58297</p>
<p style="font-size:13px;line-height:1.5;"><strong>//向JobTracker发送heartbeat</strong></p>
<p style="font-size:13px;line-height:1.5;">Got heartbeatResponse from JobTracker with responseId: 0 and 0 actions</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">一个特殊的log文件是hadoop-tasktracker-datanode02.log中，因为我们设置的最大Map Task数目和最大Reduce Task数据为0，而报了一个Exception，Can not start task tracker because java.lang.IllegalArgumentException，从而使得datanode02上的TaskTracker没有启动起来。</p>
<p style="line-height:1.5;"> </p>
<p style="line-height:1.5;">当Hadoop启动起来以后，在HDFS中也创建了一些文件夹/data/hadoopdir/tmp/mapred/system，用来保存Map-Reduce运行时候的共享资源。</p>
<h1 style="font-size:28px;line-height:56px;">三、向HDFS中放入文件</h1>
<p style="line-height:1.5;">向HDFS中放入文件，需要使用命令：bin/hadoop fs -put inputdata /data/input</p>
<p style="line-height:1.5;">放入文件完毕后，我们查看HDFS：bin/hadoop fs -ls /data/input，结果为：</p>
<p style="line-height:1.5;">-rw-r--r--   3 hadoop supergroup    6119928 2010-11-21 00:47 /data/input/inputdata</p>
<p style="line-height:1.5;">这个时候，我们查看DataNode下的/data/hadoopdir/tmp文件夹发生了变化：</p>
<ul style="margin-left:45px;"><li style="list-style: !important;">在datanode01, datanode02, datanode03上的/data/hadoopdir/tmp/dfs/data/current下面都多了如下的block文件</li><li style="list-style: !important;">可见block文件被复制了三份</li></ul><table cellspacing="0" cellpadding="2" width="769" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="767" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;"></p>
<p style="font-size:13px;line-height:1.5;">.:</p>
<p style="font-size:13px;line-height:1.5;">dfs  mapred</p>
<p style="font-size:13px;line-height:1.5;">./dfs:</p>
<p style="font-size:13px;line-height:1.5;">data</p>
<p style="font-size:13px;line-height:1.5;">./dfs/data:</p>
<p style="font-size:13px;line-height:1.5;">current  detach  in_use.lock  storage  tmp</p>
<p style="font-size:13px;line-height:1.5;">./dfs/data/current:</p>
<p style="font-size:13px;line-height:1.5;"><strong>blk_2672607439166801630  blk_2672607439166801630_1002.meta</strong>  dncp_block_verification.log.curr  VERSION</p>
<p style="font-size:13px;line-height:1.5;">./dfs/data/detach:</p>
<p style="font-size:13px;line-height:1.5;">./dfs/data/tmp:</p>
<p style="font-size:13px;line-height:1.5;">./mapred:</p>
<p style="font-size:13px;line-height:1.5;">local</p>
<p style="font-size:13px;line-height:1.5;">./mapred/local:</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">在放入文件的过程中，我们可以看log如下：</p>
<p style="line-height:1.5;">namenode的hadoop-namenode-namenode.log如下:</p>
<table cellspacing="0" cellpadding="2" width="770" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="768" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;"><strong>//创建/data/input/inputdata      </strong></p>
<p style="font-size:13px;line-height:1.5;">ugi=admin,sambashareip=/192.168.1.104       cmd=create      src<strong>=/data/input/inputdata</strong>       dst=null        perm=hadoop:supergroup:rw-r--r--</p>
<p style="font-size:13px;line-height:1.5;"><strong>//分配block</strong></p>
<p style="font-size:13px;line-height:1.5;">NameSystem.allocateBlock: /data/input/inputdata. <strong>blk_2672607439166801630_1002</strong></p>
<p style="font-size:13px;line-height:1.5;">NameSystem.addStoredBlock: blockMap updated: 192.168.1.107:50010 is added to<strong>blk_2672607439166801630_1002</strong> size 6119928</p>
<p style="font-size:13px;line-height:1.5;">NameSystem.addStoredBlock: blockMap updated: 192.168.1.105:50010 is added to<strong>blk_2672607439166801630_1002</strong> size 6119928</p>
<p style="font-size:13px;line-height:1.5;">NameSystem.addStoredBlock: blockMap updated: 192.168.1.106:50010 is added to<strong>blk_2672607439166801630_1002</strong> size 6119928</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">datanode01的hadoop-datanode-datanode01.log如下:</p>
<table cellspacing="0" cellpadding="2" width="768" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="766" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;"><strong>//datanode01从客户端接收一个block</strong></p>
<p style="font-size:13px;line-height:1.5;">Receiving block blk_2672607439166801630_1002 src: /192.168.1.104:41748 dest: /192.168.1.105:50010</p>
<p style="font-size:13px;line-height:1.5;">src: /192.168.1.104:41748, dest: /192.168.1.105:50010, bytes: 6119928, op: HDFS_WRITE, cliID: DFSClient_-1541812792, srvID: DS-1042573498-192.168.1.105-50010-1290313555129, blockid: blk_2672607439166801630_1002</p>
<p style="font-size:13px;line-height:1.5;">PacketResponder 2 for block blk_2672607439166801630_1002 terminating</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">datanode02的hadoop-datanode-datanode02.log如下：</p>
<table cellspacing="0" cellpadding="2" width="770" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="768" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;"><strong>//datanode02从datanode01接收一个block</strong></p>
<p style="font-size:13px;line-height:1.5;">Receiving block blk_2672607439166801630_1002 src: /192.168.1.105:60266 dest: /192.168.1.106:50010</p>
<p style="font-size:13px;line-height:1.5;">src: /192.168.1.105:60266, dest: /192.168.1.106:50010, bytes: 6119928, op: HDFS_WRITE, cliID: DFSClient_-1541812792, srvID: DS-1366730865-192.168.1.106-50010-1290313543717, blockid: blk_2672607439166801630_1002</p>
<p style="font-size:13px;line-height:1.5;">PacketResponder 1 for block blk_2672607439166801630_1002 terminating</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">datanode03的hadoop-datanode-datanode03.log如下：</p>
<table cellspacing="0" cellpadding="2" width="771" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="769" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;"><strong>//datanode03从datanode02接收一个block</strong></p>
<p style="font-size:13px;line-height:1.5;">Receiving block blk_2672607439166801630_1002 src: /192.168.1.106:58899 dest: /192.168.1.107:50010</p>
<p style="font-size:13px;line-height:1.5;">src: /192.168.1.106:58899, dest: /192.168.1.107:50010, bytes: 6119928, op: HDFS_WRITE, cliID: DFSClient_-1541812792, srvID: DS-765014609-192.168.1.107-50010-1290313555841, blockid: blk_2672607439166801630_1002</p>
<p style="font-size:13px;line-height:1.5;">PacketResponder 0 for block blk_2672607439166801630_1002 terminating</p>
<p style="font-size:13px;line-height:1.5;">Verification succeeded for blk_2672607439166801630_1002</p>
</td>
</tr></tbody></table><p style="line-height:1.5;"> </p>
<h1 style="font-size:28px;line-height:56px;">四、运行一个Map-Reduce程序</h1>
<p style="line-height:1.5;">运行Map-Reduce函数，需要运行命令：bin/hadoop jar hadoop-0.19.2-examples.jar wordcount /data/input /data/output</p>
<p style="line-height:1.5;">为了能够观察Map-Reduce一步步运行的情况，我们首先远程调试JobTracker，将断点设置在JobTracker.submitJob函数中。</p>
<p style="line-height:1.5;">按照我们上一篇文章讨论的那样，DFSClient向JobTracker提交任务之前，会将任务运行所需要的三类文件放入HDFS，从而可被JobTracker和TaskTracker得到：</p>
<ul style="margin-left:45px;"><li style="list-style: !important;">运行的jar文件：job.jar</li><li style="list-style: !important;">运行所需要的input split的信息：job.split</li><li style="list-style: !important;">运行所需的配置：job.xml</li></ul><p style="line-height:1.5;">当Map-Reduce程序停在JobTracker.submitJob函数中的时候，让我们查看HDFS中有如下的变化：</p>
<p style="line-height:1.5;">bin/hadoop fs -ls /data/hadoopdir/tmp/mapred/system</p>
<p style="line-height:1.5;">其中多了一个文件夹job_201011202025_0001，这是当前运行的Job的ID，在这个文件夹中有三个文件：</p>
<table cellspacing="0" cellpadding="2" width="772" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="770" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;">bin/hadoop fs -ls /data/hadoopdir/tmp/mapred/system/job_201011202025_0001</p>
<p style="font-size:13px;line-height:1.5;">Found 3 items</p>
<p style="font-size:13px;line-height:1.5;">-rw-r--r--  /data/hadoopdir/tmp/mapred/system/job_201011202025_0001/job.jar</p>
<p style="font-size:13px;line-height:1.5;">-rw-r--r--  /data/hadoopdir/tmp/mapred/system/job_201011202025_0001/job.split</p>
<p style="font-size:13px;line-height:1.5;">-rw-r--r--  /data/hadoopdir/tmp/mapred/system/job_201011202025_0001/job.xml</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">现在我们可以断开对JobTracker的远程调试。</p>
<p style="line-height:1.5;">在JobTracker.submitJob的函数中，会读取这些上传到HDFS的文件，从而将Job拆分成Map Task和Reduce Task。</p>
<p style="line-height:1.5;">当TaskTracker通过heartbeat向JobTracker请求一个Map Task或者Reduce Task来运行，按照我们上面的配置，显然datanode01会请求Map Task来执行，而datanode03会申请Reduce Task来执行。</p>
<p style="line-height:1.5;"> </p>
<p style="line-height:1.5;">下面我们首先来看datanode01上Map Task的执行过程：</p>
<p style="line-height:1.5;">当TaskTracker得到一个Task的时候，它会调用TaskTracker.localizeJob将job运行的三个文件从HDFS中拷贝到本地文件夹，然后调用TaskInProgress.localizeTask创建Task运行的本地工作目录。</p>
<p style="line-height:1.5;">我们来远程调试datanode01上的TaskTracker，分别将断点设在localizeJob和localizeTask函数中，当程序停在做完localizeTask后，我们来看datanode01上的/data/hadoopdir/tmp/mapred/local/taskTracker/jobcache下多了一个文件夹</p>
<p style="line-height:1.5;">job_201011202025_0001，在此文件夹下面有如下的结构：</p>
<table cellspacing="0" cellpadding="2" width="769" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="767" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;">datanode01:/data/hadoopdir/tmp/mapred/local/taskTracker/jobcache/job_201011202025_0001$ ls -R</p>
<p style="font-size:13px;line-height:1.5;">.:</p>
<p style="font-size:13px;line-height:1.5;">attempt_201011202025_0001_m_000000_0  attempt_201011202025_0001_m_000003_0  jars  job.xml  work</p>
<p style="font-size:13px;line-height:1.5;">./attempt_201011202025_0001_m_000000_0:</p>
<p style="font-size:13px;line-height:1.5;"><strong>job.xml  split.dta  work</strong></p>
<p style="font-size:13px;line-height:1.5;">./attempt_201011202025_0001_m_000000_0/work:</p>
<p style="font-size:13px;line-height:1.5;">./attempt_201011202025_0001_m_000003_0:</p>
<p style="font-size:13px;line-height:1.5;">pid  work</p>
<p style="font-size:13px;line-height:1.5;">./attempt_201011202025_0001_m_000003_0/work:</p>
<p style="font-size:13px;line-height:1.5;">tmp</p>
<p style="font-size:13px;line-height:1.5;">./attempt_201011202025_0001_m_000003_0/work/tmp:</p>
<p style="font-size:13px;line-height:1.5;">./jars:</p>
<p style="font-size:13px;line-height:1.5;">job.jar  META-INF  org</p>
<p style="font-size:13px;line-height:1.5;">./work:</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">其中，job.xml, job.jar，split.dta为配置文件和运行jar以及input split，jars文件夹下面为job.jar的解压缩。</p>
<p style="line-height:1.5;">接下来datanode01要创建Child JVM来执行Task，这时我们在datanode01上运行ps aux | grep java，可以发现各有一个新的JVM被创建：</p>
<table cellspacing="0" cellpadding="2" width="755" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="753" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;">/bin/java</p>
<p style="font-size:13px;line-height:1.5;">……</p>
<p style="font-size:13px;line-height:1.5;">-Xmx200m -agentlib:jdwp=transport=dt_socket,address=8883,server=y,suspend=y</p>
<p style="font-size:13px;line-height:1.5;">……</p>
<p style="font-size:13px;line-height:1.5;">org.apache.hadoop.mapred.Child</p>
<p style="font-size:13px;line-height:1.5;">127.0.0.1 58297</p>
<p style="font-size:13px;line-height:1.5;"><strong>attempt_201011202025_0001_m_000003_0</strong> 2093922206</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">从JVM的参数我们可以看出，这是一个map任务。从上面的文件我们可以看出，其实此TaskTracker已经在同一个Child JVM里面运行了两个map task，其中一个是attempt_201011202025_0001_m_000003_0，这个没有input split，后来发现他是一个job setup task，而另一个是attempt_201011202025_0001_m_000000_0，是一个真正处理数据的map
 task，当然如果需要处理的数据量足够大，会有多个处理数据的map task被运行。</p>
<p style="line-height:1.5;">我们可以对Child JVM进行远程调试，把断点设在MapTask.run函数中，从上一篇文章中我们知道，map的结果一开始都是保存在buffer中的，当数据量足够大，则spill到硬盘中，形成spill文件，在map task结束之前，我们查看attempt_201011202025_0001_m_000000_0文件夹，我们可以看到，大量的spill文件已经生成：</p>
<table cellspacing="0" cellpadding="2" width="753" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="751" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;">datanode01:/data/hadoopdir/tmp/mapred/local/taskTracker/jobcache/job_201011202025_0001/attempt_201011202025_0001_m_000000_0$ ls -R</p>
<p style="font-size:13px;line-height:1.5;">.:</p>
<p style="font-size:13px;line-height:1.5;">job.xml  output  split.dta  work</p>
<p style="font-size:13px;line-height:1.5;">./output:</p>
<p style="font-size:13px;line-height:1.5;"><strong>spill0.out   spill16.out  spill22.out  spill29.out  spill35.out  spill41.out  spill48.out  spill54.out  spill60.out  spill67.out  spill73.out  spill7.out</strong></p>
<p style="font-size:13px;line-height:1.5;"><strong>spill10.out  spill17.out  spill23.out  spill2.out   spill36.out  spill42.out  spill49.out  spill55.out  spill61.out  spill68.out  spill74.out  spill80.out</strong></p>
<p style="font-size:13px;line-height:1.5;"><strong>spill11.out  spill18.out  spill24.out  spill30.out  spill37.out  spill43.out  spill4.out   spill56.out  spill62.out  spill69.out  spill75.out  spill81.out</strong></p>
<p style="font-size:13px;line-height:1.5;"><strong>spill12.out  spill19.out  spill25.out  spill31.out  spill38.out  spill44.out  spill50.out  spill57.out  spill63.out  spill6.out   spill76.out  spill82.out</strong></p>
<p style="font-size:13px;line-height:1.5;"><strong>spill13.out  spill1.out   spill26.out  spill32.out  spill39.out  spill45.out  spill51.out  spill58.out  spill64.out  spill70.out  spill77.out  spill83.out</strong></p>
<p style="font-size:13px;line-height:1.5;"><strong>spill14.out  spill20.out  spill27.out  spill33.out  spill3.out   spill46.out  spill52.out  spill59.out  spill65.out  spill71.out  spill78.out  spill8.out</strong></p>
<p style="font-size:13px;line-height:1.5;"><strong>spill15.out  spill21.out  spill28.out  spill34.out  spill40.out  spill47.out  spill53.out  spill5.out   spill66.out  spill72.out  spill79.out  spill9.out</strong></p>
<p style="font-size:13px;line-height:1.5;">./work:</p>
<p style="font-size:13px;line-height:1.5;">tmp</p>
<p style="font-size:13px;line-height:1.5;">./work/tmp:</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">当整个map task结束后，所有的spill文件会合并成一个文件，这时候我们再查看attempt_201011202025_0001_m_000000_0文件夹：</p>
<table cellspacing="0" cellpadding="2" width="755" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="753" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;">datanode01:/data/hadoopdir/tmp/mapred/local/taskTracker/jobcache/job_201011202025_0001/attempt_201011202025_0001_m_000000_0$ ls -R<br>
.:<br>
job.xml  output  split.dta  work</p>
<p style="font-size:13px;line-height:1.5;">./output:<br><strong>file.out  file.out.index</strong></p>
<p style="font-size:13px;line-height:1.5;">./work:<br>
tmp</p>
<p style="font-size:13px;line-height:1.5;">./work/tmp:</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">当然如果有多个map task处理数据，就会生成多个file.out，在本例子中，一共只有两个map task处理数据，所以最后的结果为：</p>
<table cellspacing="0" cellpadding="2" width="761" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="759" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;">datanode01:/data/hadoopdir/tmp/mapred/local/taskTracker/jobcache/job_201011202025_0001$ ls -R attempt_201011202025_0001_m_00000*</p>
<p style="font-size:13px;line-height:1.5;">attempt_201011202025_0001_m_000000_0:</p>
<p style="font-size:13px;line-height:1.5;">job.xml  output  split.dta  work</p>
<p style="font-size:13px;line-height:1.5;">attempt_201011202025_0001_m_000000_0/output:</p>
<p style="font-size:13px;line-height:1.5;"><strong>file.out  file.out.index</strong></p>
<p style="font-size:13px;line-height:1.5;">attempt_201011202025_0001_m_000000_0/work:</p>
<p style="font-size:13px;line-height:1.5;">tmp</p>
<p style="font-size:13px;line-height:1.5;">attempt_201011202025_0001_m_000000_0/work/tmp:</p>
<p style="font-size:13px;line-height:1.5;">attempt_201011202025_0001_m_000001_0:</p>
<p style="font-size:13px;line-height:1.5;">job.xml  output  split.dta  work</p>
<p style="font-size:13px;line-height:1.5;">attempt_201011202025_0001_m_000001_0/output:</p>
<p style="font-size:13px;line-height:1.5;"><strong>file.out  file.out.index</strong></p>
<p style="font-size:13px;line-height:1.5;">attempt_201011202025_0001_m_000001_0/work:</p>
<p style="font-size:13px;line-height:1.5;">tmp</p>
<p style="font-size:13px;line-height:1.5;">attempt_201011202025_0001_m_000001_0/work/tmp:</p>
<p style="font-size:13px;line-height:1.5;">attempt_201011202025_0001_m_000003_0:</p>
<p style="font-size:13px;line-height:1.5;">pid  work</p>
<p style="font-size:13px;line-height:1.5;">attempt_201011202025_0001_m_000003_0/work:</p>
<p style="font-size:13px;line-height:1.5;">tmp</p>
<p style="font-size:13px;line-height:1.5;">attempt_201011202025_0001_m_000003_0/work/tmp:</p>
</td>
</tr></tbody></table><p style="line-height:1.5;"> </p>
<p style="line-height:1.5;">然后我们再来看datanode03上reduce task的运行情况：</p>
<p style="line-height:1.5;">我们同样远程调试datanode03上的TaskTracker，将断点设在localizeJob和localizeTask函数中，当程序停在做完localizeTask后，我们来看datanode03上的/data/hadoopdir/tmp/mapred/local/taskTracker/jobcache下也多了一个文件夹job_201011202025_0001，在此文件夹下面有如下的结构：</p>
<table cellspacing="0" cellpadding="2" width="767" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="765" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;">datanode03:/data/hadoopdir/tmp/mapred/local/taskTracker/jobcache/job_201011202025_0001$ ls -R attempt_201011202025_0001_r_00000*<br>
attempt_201011202025_0001_r_000000_0:<br>
job.xml  work</p>
<p style="font-size:13px;line-height:1.5;">attempt_201011202025_0001_r_000000_0/work:<br>
tmp</p>
<p style="font-size:13px;line-height:1.5;">attempt_201011202025_0001_r_000000_0/work/tmp:</p>
<p style="font-size:13px;line-height:1.5;">attempt_201011202025_0001_r_000002_0:<br>
pid  work</p>
<p style="font-size:13px;line-height:1.5;">attempt_201011202025_0001_r_000002_0/work:<br>
tmp</p>
<p style="font-size:13px;line-height:1.5;">attempt_201011202025_0001_r_000002_0/work/tmp:</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">上面的两个Reduce Task中，attempt_201011202025_0001_r_000002_0是一个job setup task，真正处理数据的是attempt_201011202025_0001_r_000000_0。</p>
<p style="line-height:1.5;">接下来datanode03要创建Child JVM来执行Task，这时我们在datanode03上运行ps aux | grep java，可以发现各有一个新的JVM被创建：</p>
<table cellspacing="0" cellpadding="2" width="763" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="761" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;">/bin/java</p>
<p style="font-size:13px;line-height:1.5;">……</p>
<p style="font-size:13px;line-height:1.5;">-Xmx200m -agentlib:jdwp=transport=dt_socket,address=8883,server=y,suspend=y -</p>
<p style="font-size:13px;line-height:1.5;">……</p>
<p style="font-size:13px;line-height:1.5;">org.apache.hadoop.mapred.Child</p>
<p style="font-size:13px;line-height:1.5;">127.0.0.1 37546</p>
<p style="font-size:13px;line-height:1.5;"><strong>attempt_201011202025_0001_r_000002_0</strong> 516504201</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">从JVM的参数我们可以看出，这是一个map任务。</p>
<p style="line-height:1.5;">从上一篇文章中我们知道，Reduce Task包括三个过程：copy，sort，reduce</p>
<p style="line-height:1.5;">拷贝过程即将所有的map结果复制到reduce task的本地</p>
<table cellspacing="0" cellpadding="2" width="752" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="750" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;">datanode03:/data/hadoopdir/tmp/mapred/local/taskTracker/jobcache/job_201011202025_0001/attempt_201011202025_0001_r_000000_0$ ls -R</p>
<p style="font-size:13px;line-height:1.5;">.:</p>
<p style="font-size:13px;line-height:1.5;">job.xml  output  pid  work</p>
<p style="font-size:13px;line-height:1.5;">./output:</p>
<p style="font-size:13px;line-height:1.5;"><strong>map_0.out  map_1.out  map_2.out  map_3.out</strong></p>
<p style="font-size:13px;line-height:1.5;">./work:</p>
<p style="font-size:13px;line-height:1.5;">tmp</p>
<p style="font-size:13px;line-height:1.5;">./work/tmp:</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">如图所示，如果共有4个map task，则共拷贝到本地4个map.out。</p>
<p style="line-height:1.5;">在拷贝的过程中，有一个背后的线程会对已经拷贝到本地的map.out进行预先的合并，形成map.merged文件，合并的规则是按照io.sort.factor来进行合并，对于我们的配置就是两两合并，下面我们看到的就是map_2.out和map_3.out合并成map_3.out.merged，在另外两个还没有合并的时候，拷贝过程结束了，则背后的合并进程也就结束了。</p>
<table cellspacing="0" cellpadding="2" width="754" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="752" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;">datanode03:/data/hadoopdir/tmp/mapred/local/taskTracker/jobcache/job_201011202025_0001/attempt_201011202025_0001_r_000000_0$ ls -R<br>
.:<br>
job.xml  output  pid  work</p>
<p style="font-size:13px;line-height:1.5;">./output:<br><strong>map_0.out  map_1.out  map_3.out.merged</strong></p>
<p style="font-size:13px;line-height:1.5;">./work:<br>
tmp</p>
<p style="font-size:13px;line-height:1.5;">./work/tmp:</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">sort过程就是将拷贝过来的map输出合并并排序，也是按照io.sort.factor来进行合并，也即两两合并。下面我们看到的就是map_0.out和map_1.out合并为一个intermediate.1，加上另外的map_3.out.merged，数目已经小于io.sort.factor了，于是不再合并。</p>
<table cellspacing="0" cellpadding="2" width="762" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="760" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;">datanode03:/data/hadoopdir/tmp/mapred/local/attempt_201011202025_0001_r_000000_0$ ls -r</p>
<p style="font-size:13px;line-height:1.5;"><strong>intermediate.1</strong></p>
</td>
</tr><tr><td valign="top" width="760" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;">datanode03:/data/hadoopdir/tmp/mapred/local/taskTracker/jobcache/job_201011202025_0001/attempt_201011202025_0001_r_000000_0$ ls -R<br>
.:<br>
job.xml  output  pid  work</p>
<p style="font-size:13px;line-height:1.5;">./output:<br><strong>map_3.out.merged</strong></p>
<p style="font-size:13px;line-height:1.5;">./work:<br>
tmp</p>
<p style="font-size:13px;line-height:1.5;">./work/tmp:</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">reduce的过程就是循环调用reducer的reduce函数，将结果输出到HDFS中。</p>
<table cellspacing="0" cellpadding="2" width="770" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="768" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;">namenode:/data/hadoop-0.19.2$ bin/hadoop fs -ls /data/output</p>
<p style="font-size:13px;line-height:1.5;">Found 2 items</p>
<p style="font-size:13px;line-height:1.5;">/data/output/_logs</p>
<p style="font-size:13px;line-height:1.5;">/data/output/part-00000</p>
</td>
</tr></tbody></table><p style="line-height:1.5;"> </p>
<p style="line-height:1.5;">当然我们通过log，也可以看到Map-Reduce的运行过程：</p>
<p style="line-height:1.5;">命令行输出的日志如下：</p>
<table cellspacing="0" cellpadding="2" width="770" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="768" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;">namenode:/data/hadoop-0.19.2$ bin/hadoop jar hadoop-0.19.2-examples.jar wordcount /data/input /data/output</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:38:44 INFO mapred.FileInputFormat: Total input paths to process : 4</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:38:45 INFO mapred.JobClient: Running job: job_201011202025_0001</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:38:46 INFO mapred.JobClient:  map 0% reduce 0%</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:39:14 INFO mapred.JobClient:  map 25% reduce 0%</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:39:23 INFO mapred.JobClient:  map 50% reduce 0%</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:39:27 INFO mapred.JobClient:  map 75% reduce 0%</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:39:30 INFO mapred.JobClient:  map 100% reduce 0%</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:39:31 INFO mapred.JobClient:  map 100% reduce 8%</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:39:36 INFO mapred.JobClient:  map 100% reduce 25%</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:39:40 INFO mapred.JobClient:  map 100% reduce 100%</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:39:41 INFO mapred.JobClient: Job complete: job_201011202025_0001</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:39:41 INFO mapred.JobClient: Counters: 16</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:39:41 INFO mapred.JobClient:   File Systems</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:39:41 INFO mapred.JobClient:     HDFS bytes read=61199280</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:39:41 INFO mapred.JobClient:     HDFS bytes written=534335</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:39:41 INFO mapred.JobClient:     Local bytes read=74505214</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:39:41 INFO mapred.JobClient:     Local bytes written=81308914</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:39:41 INFO mapred.JobClient:   Job Counters</p>
<p style="font-size:13px;line-height:1.5;">//四个map，一个reduce</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:39:41 INFO mapred.JobClient:     Launched reduce tasks=1</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:39:41 INFO mapred.JobClient:     Launched map tasks=4</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:39:41 INFO mapred.JobClient:     Data-local map tasks=4</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:39:41 INFO mapred.JobClient:   Map-Reduce Framework</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:39:41 INFO mapred.JobClient:     Reduce input groups=37475</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:39:41 INFO mapred.JobClient:     Combine output records=351108</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:39:41 INFO mapred.JobClient:     Map input records=133440</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:39:41 INFO mapred.JobClient:     Reduce output records=37475</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:39:41 INFO mapred.JobClient:     Map output bytes=31671148</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:39:41 INFO mapred.JobClient:     Map input bytes=24479712</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:39:41 INFO mapred.JobClient:     Combine input records=2001312</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:39:41 INFO mapred.JobClient:     Map output records=1800104</p>
<p style="font-size:13px;line-height:1.5;">10/11/22 07:39:41 INFO mapred.JobClient:     Reduce input records=149900</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">在namenode的hadoop-jobtracker-namenode.log中，我们可以看到JobTracker的运行情况：</p>
<table cellspacing="0" cellpadding="2" width="768" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="766" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;"><strong>//创建一个Job，分成四个map task</strong></p>
<p style="font-size:13px;line-height:1.5;">JobInProgress: Input size for job job_201011220735_0001 = 24479712</p>
<p style="font-size:13px;line-height:1.5;">JobInProgress: Split info for job:job_201011220735_0001</p>
<p style="font-size:13px;line-height:1.5;">JobInProgress: tip:task_201011220735_0001_m_000000 has split on node:/default-rack/datanode02</p>
<p style="font-size:13px;line-height:1.5;">JobInProgress: tip:task_201011220735_0001_m_000000 has split on node:/default-rack/datanode01</p>
<p style="font-size:13px;line-height:1.5;">JobInProgress: tip:task_201011220735_0001_m_000000 has split on node:/default-rack/datanode03</p>
<p style="font-size:13px;line-height:1.5;">JobInProgress: tip:task_201011220735_0001_m_000001 has split on node:/default-rack/datanode03</p>
<p style="font-size:13px;line-height:1.5;">JobInProgress: tip:task_201011220735_0001_m_000001 has split on node:/default-rack/datanode01</p>
<p style="font-size:13px;line-height:1.5;">JobInProgress: tip:task_201011220735_0001_m_000001 has split on node:/default-rack/datanode02</p>
<p style="font-size:13px;line-height:1.5;">JobInProgress: tip:task_201011220735_0001_m_000002 has split on node:/default-rack/datanode02</p>
<p style="font-size:13px;line-height:1.5;">JobInProgress: tip:task_201011220735_0001_m_000002 has split on node:/default-rack/datanode01</p>
<p style="font-size:13px;line-height:1.5;">JobInProgress: tip:task_201011220735_0001_m_000002 has split on node:/default-rack/datanode03</p>
<p style="font-size:13px;line-height:1.5;">JobInProgress: tip:task_201011220735_0001_m_000003 has split on node:/default-rack/datanode01</p>
<p style="font-size:13px;line-height:1.5;">JobInProgress: tip:task_201011220735_0001_m_000003 has split on node:/default-rack/datanode02</p>
<p style="font-size:13px;line-height:1.5;">JobInProgress: tip:task_201011220735_0001_m_000003 has split on node:/default-rack/datanode03</p>
<p style="font-size:13px;line-height:1.5;"> </p>
<p style="font-size:13px;line-height:1.5;"><strong>//datanode01通过heartbeat向JobTracker申请运行一个job setup task</strong></p>
<p style="font-size:13px;line-height:1.5;">JobTracker: Adding task 'attempt_201011220735_0001_m_000005_0' to tip task_201011220735_0001_m_000005, for tracker 'tracker_datanode01:localhost/127.0.0.1:48339'</p>
<p style="font-size:13px;line-height:1.5;">JobTracker: tracker_datanode01:localhost/127.0.0.1:48339 -&gt; LaunchTask: attempt_201011220735_0001_m_000005_0</p>
<p style="font-size:13px;line-height:1.5;">JobInProgress: Task 'attempt_201011220735_0001_m_000005_0' has completed task_201011220735_0001_m_000005 successfully.</p>
<p style="font-size:13px;line-height:1.5;"> </p>
<p style="font-size:13px;line-height:1.5;"><strong>//datanode01向JobTracker请求运行第一个map task</strong></p>
<p style="font-size:13px;line-height:1.5;">JobTracker: Adding task 'attempt_201011220735_0001_m_000000_0' to tip task_201011220735_0001_m_000000, for tracker 'tracker_datanode01:localhost/127.0.0.1:48339'</p>
<p style="font-size:13px;line-height:1.5;">JobInProgress: Choosing data-local task task_201011220735_0001_m_000000</p>
<p style="font-size:13px;line-height:1.5;">JobTracker: tracker_datanode01:localhost/127.0.0.1:48339 -&gt; LaunchTask: attempt_201011220735_0001_m_000000_0</p>
<p style="font-size:13px;line-height:1.5;">JobInProgress: Task 'attempt_201011220735_0001_m_000000_0' has completed task_201011220735_0001_m_000000 successfully.<br></p>
<p style="font-size:13px;line-height:1.5;"> </p>
<p style="font-size:13px;line-height:1.5;"><strong>//datanode01向JobTracker请求运行第二个map task</strong></p>
<p style="font-size:13px;line-height:1.5;">JobTracker: Adding task 'attempt_201011220735_0001_m_000001_0' to tip task_201011220735_0001_m_000001, for tracker 'tracker_datanode01:localhost/127.0.0.1:48339'</p>
<p style="font-size:13px;line-height:1.5;">JobInProgress: Choosing data-local task task_201011220735_0001_m_000001</p>
<p style="font-size:13px;line-height:1.5;">JobTracker: tracker_datanode01:localhost/127.0.0.1:48339 -&gt; LaunchTask: attempt_201011220735_0001_m_000001_0</p>
<p style="font-size:13px;line-height:1.5;">JobInProgress: Task 'attempt_201011220735_0001_m_000001_0' has completed task_201011220735_0001_m_000001 successfully.</p>
<p style="font-size:13px;line-height:1.5;"> </p>
<p style="font-size:13px;line-height:1.5;"><strong>//datanode01向JobTracker请求运行第三个map task</strong></p>
<p style="font-size:13px;line-height:1.5;">JobTracker: Adding task 'attempt_201011220735_0001_m_000002_0' to tip task_201011220735_0001_m_000002, for tracker 'tracker_datanode01:localhost/127.0.0.1:48339'</p>
<p style="font-size:13px;line-height:1.5;">JobInProgress: Choosing data-local task task_201011220735_0001_m_000002</p>
<p style="font-size:13px;line-height:1.5;">JobTracker: tracker_datanode01:localhost/127.0.0.1:48339 -&gt; LaunchTask: attempt_201011220735_0001_m_000002_0</p>
<p style="font-size:13px;line-height:1.5;">JobInProgress: Task 'attempt_201011220735_0001_m_000002_0' has completed task_201011220735_0001_m_000002 successfully.</p>
<p style="font-size:13px;line-height:1.5;"> </p>
<p style="font-size:13px;line-height:1.5;"><strong>//datanode01向JobTracker请求运行第四个map task</strong></p>
<p style="font-size:13px;line-height:1.5;">JobTracker: Adding task 'attempt_201011220735_0001_m_000003_0' to tip task_201011220735_0001_m_000003, for tracker 'tracker_datanode01:localhost/127.0.0.1:48339'</p>
<p style="font-size:13px;line-height:1.5;">JobInProgress: Choosing data-local task task_201011220735_0001_m_000003</p>
<p style="font-size:13px;line-height:1.5;">JobTracker: tracker_datanode01:localhost/127.0.0.1:48339 -&gt; LaunchTask: attempt_201011220735_0001_m_000003_0</p>
<p style="font-size:13px;line-height:1.5;">JobTracker: Got heartbeat from: tracker_datanode01:localhost/127.0.0.1:48339 (initialContact: false acceptNewTasks: true) with responseId: 39</p>
<p style="font-size:13px;line-height:1.5;">JobInProgress: Task 'attempt_201011220735_0001_m_000003_0' has completed task_201011220735_0001_m_000003 successfully.</p>
<br><br><p style="font-size:13px;line-height:1.5;"><strong>//datanode03向JobTracker申请运行一个commit task</strong></p>
<p style="font-size:13px;line-height:1.5;">JobTracker: Adding task 'attempt_201011220735_0001_r_000000_0' to tip task_201011220735_0001_r_000000, for tracker 'tracker_datanode03:localhost/127.0.0.1:44118'</p>
<p style="font-size:13px;line-height:1.5;">JobTracker: tracker_datanode03:localhost/127.0.0.1:44118 -&gt; LaunchTask: attempt_201011220735_0001_r_000000_0</p>
<p style="font-size:13px;line-height:1.5;">JobTracker: tracker_datanode03:localhost/127.0.0.1:44118 -&gt; CommitTaskAction: attempt_201011220735_0001_r_000000_0</p>
<p style="font-size:13px;line-height:1.5;">JobInProgress: Task 'attempt_201011220735_0001_r_000000_0' has completed task_201011220735_0001_r_000000 successfully.</p>
<p style="font-size:13px;line-height:1.5;"> </p>
<p style="font-size:13px;line-height:1.5;"><strong>//datanode03向JobTracker申请运行一个reduce task</strong></p>
<p style="font-size:13px;line-height:1.5;">JobTracker: Adding task 'attempt_201011220735_0001_r_000001_0' to tip task_201011220735_0001_r_000001, for tracker 'tracker_datanode03:localhost/127.0.0.1:44118'</p>
<p style="font-size:13px;line-height:1.5;">JobTracker: tracker_datanode03:localhost/127.0.0.1:44118 -&gt; LaunchTask: attempt_201011220735_0001_r_000001_0</p>
<p style="font-size:13px;line-height:1.5;">JobInProgress: Task 'attempt_201011220735_0001_r_000001_0' has completed task_201011220735_0001_r_000001 successfully.</p>
<p style="font-size:13px;line-height:1.5;"> </p>
<p style="font-size:13px;line-height:1.5;">JobInProgress: Job job_201011220735_0001 has completed successfully.</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">同样，在datanode01的hadoop-tasktracker-datanode01.log可以看到TaskTracker的运行过程。</p>
<p style="line-height:1.5;">在datanode01的logs/userlogs下面，我们可以看到为了运行map task所生成的Child JVM打印出的log，每个map task一个文件夹，在本例中，由于多个map task共用一个JVM，所以只输出了一组log文件</p>
<table cellspacing="0" cellpadding="2" width="767" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="765" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;">datanode01:/data/hadoop-0.19.2/logs/userlogs$ ls -R</p>
<p style="font-size:13px;line-height:1.5;">.:</p>
<p style="font-size:13px;line-height:1.5;"><strong>attempt_201011220735_0001_m_000000_0  attempt_201011220735_0001_m_000002_0  attempt_201011220735_0001_m_000005_0</strong></p>
<p style="font-size:13px;line-height:1.5;"><strong>attempt_201011220735_0001_m_000001_0  attempt_201011220735_0001_m_000003_0</strong></p>
<p style="font-size:13px;line-height:1.5;">./attempt_201011220735_0001_m_000000_0:</p>
<p style="font-size:13px;line-height:1.5;">log.index</p>
<p style="font-size:13px;line-height:1.5;">./attempt_201011220735_0001_m_000001_0:</p>
<p style="font-size:13px;line-height:1.5;">log.index</p>
<p style="font-size:13px;line-height:1.5;">./attempt_201011220735_0001_m_000002_0:</p>
<p style="font-size:13px;line-height:1.5;">log.index</p>
<p style="font-size:13px;line-height:1.5;">./attempt_201011220735_0001_m_000003_0:</p>
<p style="font-size:13px;line-height:1.5;">log.index</p>
<p style="font-size:13px;line-height:1.5;">./attempt_201011220735_0001_m_000005_0:</p>
<p style="font-size:13px;line-height:1.5;">log.index  stderr  stdout  syslog</p>
</td>
</tr></tbody></table><p style="line-height:1.5;">同样，在datanode03的hadoop-tasktracker-datanode03.log可以看到TaskTracker运行的过程。</p>
<p style="line-height:1.5;">在datanode03的logs/users下面，也有一组文件夹，每个reduce task一个文件夹，也是多个reduce task共用一个JVM:</p>
<table cellspacing="0" cellpadding="2" width="768" border="1" style="border:1px solid #C0C0C0;border-collapse:collapse;"><tbody><tr><td valign="top" width="766" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;border:1px solid #C0C0C0;border-collapse:collapse;">
<p style="font-size:13px;line-height:1.5;">datanode03:/data/hadoop-0.19.2/logs/userlogs$ ls -R</p>
<p style="font-size:13px;line-height:1.5;">.:</p>
<p style="font-size:13px;line-height:1.5;"><strong>attempt_201011220735_0001_r_000000_0  attempt_201011220735_0001_r_000001_0</strong></p>
<p style="font-size:13px;line-height:1.5;">./attempt_201011220735_0001_r_000000_0:</p>
<p style="font-size:13px;line-height:1.5;">log.index  stderr  stdout  syslog</p>
<p style="font-size:13px;line-height:1.5;">./attempt_201011220735_0001_r_000001_0:</p>
<p style="font-size:13px;line-height:1.5;">log.index</p>
</td>
</tr></tbody></table></div>
<div class="clear" style="clear:both;"></div>
<div id="blog_post_info_block">
<div id="BlogPostCategory"><br></div>
</div>
</div>
<br><p></p>
            </div>
                </div>