---
layout:     post
title:      Flume(一) Flume初识 环境的搭建和配置
---
<div id="article_content" class="article_content clearfix csdn-tracking-statistics" data-pid="blog" data-mod="popu_307" data-dsm="post">
								<div class="article-copyright">
					版权声明：欢迎转载，转载请说明出处https://csdn.yanxml.com。大数据Github项目地址https://github.com/SeanYanxml/bigdata。					https://blog.csdn.net/u010416101/article/details/54265984				</div>
								            <div id="content_views" class="markdown_views prism-atom-one-dark">
							<!-- flowchart 箭头图标 勿删 -->
							<svg xmlns="http://www.w3.org/2000/svg" style="display: none;"><path stroke-linecap="round" d="M5,0 0,2.5 5,5z" id="raphael-marker-block" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"></path></svg>
							<h1 id="flume初识">Flume初识</h1>

<p>本文主要包括如下的几个部分：</p>

<ul>
<li><strong>下载Flume</strong></li>
<li><strong>配置Flume</strong></li>
<li><strong>启动Flume 及其命令解析</strong></li>
</ul>

<hr>



<h2 id="1-下载flume">1. 下载Flume</h2>

<p>到Flume的官方网站下载相关文件<a href="http://flume.apache.org/download.html" rel="nofollow">Flume官网</a>，本人下载的是最新的稳定版本：1.7.0。(ps:下载编译后的文件，不要下载源文件。我们的目标是使用Flume，而不是研究Flume的源码。) <br>
下载图示如下： <br>
<img src="https://img-blog.csdn.net/20170109002812996?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDQxNjEwMQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" alt="下载bin，也就是binary编译后的二进制文件，开箱即用。" title=""></p>



<h2 id="2-配置flume的配置文件">2. 配置Flume的配置文件</h2>

<blockquote>
  <p>为什么需要配置Flume的文件呢？ <br>
  因为各个机器都不相同，配置也就不相同。其中，还有将Flume的log输出到控制台，输出到hdfs，输出到kafka，输出到物理文件和mongodb等等等… 这些都需要配置进行，Flume运行时会将这些配置进行解析，以达到简单配置即可使用的目的。大大降低了软件的使用难度和部署、维护难度。</p>
</blockquote>

<p>首先，我们当然要解压之前下载好的flume文件包。(windows/mac直接点击解压，Linux可以使用tar -zxvf 命令进行解压。) <br>
解压后的我的文件目录为：</p>



<pre class="prettyprint"><code class=" hljs lasso">localhost:Flume Sean$ ls
apache<span class="hljs-attribute">-flume</span><span class="hljs-subst">-</span><span class="hljs-number">1.7</span><span class="hljs-number">.0</span><span class="hljs-attribute">-bin</span>      apache<span class="hljs-attribute">-flume</span><span class="hljs-subst">-</span><span class="hljs-number">1.7</span><span class="hljs-number">.0</span><span class="hljs-attribute">-bin</span><span class="hljs-built_in">.</span>tar<span class="hljs-built_in">.</span>gz</code></pre>

<p>随后，我们便要进入Flume文件夹，对于配置文件进行编辑了。当然和往常一样，在进行编辑之前，我们需要将原文件进行保存备份。 <br>
其操作命令如下所示：</p>



<pre class="prettyprint"><code class=" hljs lasso">cd apache<span class="hljs-attribute">-flume</span><span class="hljs-subst">-</span><span class="hljs-number">1.7</span><span class="hljs-number">.0</span><span class="hljs-attribute">-bin</span>/conf
scp flume<span class="hljs-attribute">-conf</span><span class="hljs-built_in">.</span>properties<span class="hljs-built_in">.</span>template flume<span class="hljs-attribute">-conf</span><span class="hljs-built_in">.</span>properties</code></pre>

<p>这样在conf目录就有flume-conf.properties这个文件了，也是我们接下来需要配置的这个文件，启动Flume的时候会使用到。 <br>
接着，我们将官方的配置写入其中，官方配置如下：</p>



<pre class="prettyprint"><code class=" hljs avrasm"><span class="hljs-preprocessor"># example.conf: A single-node Flume configuration</span>

<span class="hljs-preprocessor"># Name the components on this agent</span>
<span class="hljs-preprocessor"># The name of the agent is defined as a1.</span>
a1<span class="hljs-preprocessor">.sources</span> = <span class="hljs-built_in">r1</span>
a1<span class="hljs-preprocessor">.sinks</span> = k1
a1<span class="hljs-preprocessor">.channels</span> = c1

<span class="hljs-preprocessor"># Describe/configure the source</span>
a1<span class="hljs-preprocessor">.sources</span><span class="hljs-preprocessor">.r</span>1<span class="hljs-preprocessor">.type</span> = netcat
a1<span class="hljs-preprocessor">.sources</span><span class="hljs-preprocessor">.r</span>1<span class="hljs-preprocessor">.bind</span> = localhost
a1<span class="hljs-preprocessor">.sources</span><span class="hljs-preprocessor">.r</span>1<span class="hljs-preprocessor">.port</span> = <span class="hljs-number">44444</span>

<span class="hljs-preprocessor"># Describe the sink</span>
a1<span class="hljs-preprocessor">.sinks</span><span class="hljs-preprocessor">.k</span>1<span class="hljs-preprocessor">.type</span> = logger

<span class="hljs-preprocessor"># Use a channel which buffers events in memory</span>
<span class="hljs-preprocessor"># 内存模式 </span>
a1<span class="hljs-preprocessor">.channels</span><span class="hljs-preprocessor">.c</span>1<span class="hljs-preprocessor">.type</span> = memory
a1<span class="hljs-preprocessor">.channels</span><span class="hljs-preprocessor">.c</span>1<span class="hljs-preprocessor">.capacity</span> = <span class="hljs-number">1000</span>
<span class="hljs-preprocessor"># 传输参数设置。</span>
a1<span class="hljs-preprocessor">.channels</span><span class="hljs-preprocessor">.c</span>1<span class="hljs-preprocessor">.transactionCapacity</span> = <span class="hljs-number">100</span>

<span class="hljs-preprocessor"># Bind the source and sink to the channel</span>
<span class="hljs-preprocessor"># The sources-&gt;channels-&gt;sinks config , the channels and the channel name must be defined.</span>
a1<span class="hljs-preprocessor">.sources</span><span class="hljs-preprocessor">.r</span>1<span class="hljs-preprocessor">.channels</span> = c1
a1<span class="hljs-preprocessor">.sinks</span><span class="hljs-preprocessor">.k</span>1<span class="hljs-preprocessor">.channel</span> = c1</code></pre>

<p>没有接触过Linux的小伙伴，可以通过vim 命令进行操作Esc－&gt;:x <br>
，或者Esc－&gt;:wq进行保存。(不懂的，可以搜索下vim命令，反正我们的目的是在里面创建一个叫做“flume-conf.properties“的文件，并且将上述的配置内容写入即可。)</p>



<h2 id="3-flume启动命令及其解析">3. Flume启动命令及其解析</h2>

<p>配置好步骤2的一切之后，就可以启动Flume了。 <br>
我们先退回到根目录下： <br>
也就是”/Users/xxx/Software/Flume/apache-flume-1.7.0-bin/”下 <br>
具体看flume文件的放置位置。 <br>
然后输入启动命令：</p>

<blockquote>
  <p>./bin/flume-ng agent –conf conf –conf-file conf/example.conf <br>
  –name a1  -Dflume.root.logger=INFO,console</p>
</blockquote>

<p>如果看到如下的输出，就代表启动成功啦。</p>



<pre class="prettyprint"><code class=" hljs mel">localhost:apache-flume-<span class="hljs-number">1.7</span><span class="hljs-number">.0</span>-bin Sean$ ./bin/flume-ng agent --conf conf --conf-<span class="hljs-keyword">file</span> conf/example.conf --name a1  -Dflume.root.logger=INFO,console
Info: Including Hadoop libraries found via (/Users/Sean/Software/hadoop/hadoop-<span class="hljs-number">2.2</span><span class="hljs-number">.0</span>/bin/hadoop) <span class="hljs-keyword">for</span> HDFS access
 + <span class="hljs-keyword">exec</span> /Library/Java/JavaVirtualMachines/jdk1<span class="hljs-number">.8</span><span class="hljs-number">.0</span>_102.jdk/Contents/Home/bin/java -Xmx20m -Dflume.root.logger=INFO,console -cp <span class="hljs-string">'/Users/Sean/Software/Flume/apache-flume-1.7.0-bin/conf:/Users/Sean/Software/Flume/apache-flume-1.7.0-bin/lib/*:/Users/Sean/Software/hadoop/hadoop-2.2.0/etc/hadoop:/Users/Sean/Software/hadoop/hadoop-2.2.0/share/hadoop/common/lib/*:/Users/Sean/Software/hadoop/hadoop-2.2.0/share/hadoop/common/*:/Users/Sean/Software/hadoop/hadoop-2.2.0/share/hadoop/hdfs:/Users/Sean/Software/hadoop/hadoop-2.2.0/share/hadoop/hdfs/lib/*:/Users/Sean/Software/hadoop/hadoop-2.2.0/share/hadoop/hdfs/*:/Users/Sean/Software/hadoop/hadoop-2.2.0/share/hadoop/yarn/lib/*:/Users/Sean/Software/hadoop/hadoop-2.2.0/share/hadoop/yarn/*:/Users/Sean/Software/hadoop/hadoop-2.2.0/share/hadoop/mapreduce/lib/*:/Users/Sean/Software/hadoop/hadoop-2.2.0/share/hadoop/mapreduce/*:/Users/Sean/Software/hadoop/hadoop-2.2.0/contrib/capacity-scheduler/*.jar'</span> -Djava.library.path=:/Users/Sean/Software/hadoop/hadoop-<span class="hljs-number">2.2</span><span class="hljs-number">.0</span>/lib/native org.apache.flume.node.Application --conf-<span class="hljs-keyword">file</span> conf/example.conf --name a1
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding <span class="hljs-keyword">in</span> [jar:<span class="hljs-keyword">file</span>:/Users/Sean/Software/Flume/apache-flume-<span class="hljs-number">1.7</span><span class="hljs-number">.0</span>-bin/lib/slf4j-log4j12-<span class="hljs-number">1.6</span><span class="hljs-number">.1</span>.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding <span class="hljs-keyword">in</span> [jar:<span class="hljs-keyword">file</span>:/Users/Sean/Software/hadoop/hadoop-<span class="hljs-number">2.2</span><span class="hljs-number">.0</span>/share/hadoop/common/lib/slf4j-log4j12-<span class="hljs-number">1.7</span><span class="hljs-number">.5</span>.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http:<span class="hljs-comment">//www.slf4j.org/codes.html#multiple_bindings for an explanation.</span>
<span class="hljs-number">2017</span>-<span class="hljs-number">01</span>-<span class="hljs-number">09</span> <span class="hljs-number">00</span>:<span class="hljs-number">46</span>:<span class="hljs-number">14</span>,<span class="hljs-number">796</span> (lifecycleSupervisor-<span class="hljs-number">1</span>-<span class="hljs-number">0</span>) [INFO - org.apache.flume.node.PollingPropertiesFileConfigurationProvider.start(PollingPropertiesFileConfigurationProvider.java:<span class="hljs-number">62</span>)] Configuration provider starting
<span class="hljs-number">2017</span>-<span class="hljs-number">01</span>-<span class="hljs-number">09</span> <span class="hljs-number">00</span>:<span class="hljs-number">46</span>:<span class="hljs-number">14</span>,<span class="hljs-number">802</span> (conf-<span class="hljs-keyword">file</span>-poller-<span class="hljs-number">0</span>) [INFO - org.apache.flume.node.PollingPropertiesFileConfigurationProvider<span class="hljs-variable">$FileWatcherRunnable</span>.run(PollingPropertiesFileConfigurationProvider.java:<span class="hljs-number">134</span>)] Reloading configuration <span class="hljs-keyword">file</span>:conf/example.conf
<span class="hljs-number">2017</span>-<span class="hljs-number">01</span>-<span class="hljs-number">09</span> <span class="hljs-number">00</span>:<span class="hljs-number">46</span>:<span class="hljs-number">14</span>,<span class="hljs-number">810</span> (conf-<span class="hljs-keyword">file</span>-poller-<span class="hljs-number">0</span>) [INFO - org.apache.flume.conf.FlumeConfiguration<span class="hljs-variable">$AgentConfiguration</span>.addProperty(FlumeConfiguration.java:<span class="hljs-number">930</span>)] Added sinks: k1 Agent: a1
<span class="hljs-number">2017</span>-<span class="hljs-number">01</span>-<span class="hljs-number">09</span> <span class="hljs-number">00</span>:<span class="hljs-number">46</span>:<span class="hljs-number">14</span>,<span class="hljs-number">811</span> (conf-<span class="hljs-keyword">file</span>-poller-<span class="hljs-number">0</span>) [INFO - org.apache.flume.conf.FlumeConfiguration<span class="hljs-variable">$AgentConfiguration</span>.addProperty(FlumeConfiguration.java:<span class="hljs-number">1016</span>)] Processing:k1
<span class="hljs-number">2017</span>-<span class="hljs-number">01</span>-<span class="hljs-number">09</span> <span class="hljs-number">00</span>:<span class="hljs-number">46</span>:<span class="hljs-number">14</span>,<span class="hljs-number">811</span> (conf-<span class="hljs-keyword">file</span>-poller-<span class="hljs-number">0</span>) [INFO - org.apache.flume.conf.FlumeConfiguration<span class="hljs-variable">$AgentConfiguration</span>.addProperty(FlumeConfiguration.java:<span class="hljs-number">1016</span>)] Processing:k1
<span class="hljs-number">2017</span>-<span class="hljs-number">01</span>-<span class="hljs-number">09</span> <span class="hljs-number">00</span>:<span class="hljs-number">46</span>:<span class="hljs-number">14</span>,<span class="hljs-number">824</span> (conf-<span class="hljs-keyword">file</span>-poller-<span class="hljs-number">0</span>) [INFO - org.apache.flume.conf.FlumeConfiguration.validateConfiguration(FlumeConfiguration.java:<span class="hljs-number">140</span>)] Post-validation flume configuration contains configuration <span class="hljs-keyword">for</span> agents: [a1]
<span class="hljs-number">2017</span>-<span class="hljs-number">01</span>-<span class="hljs-number">09</span> <span class="hljs-number">00</span>:<span class="hljs-number">46</span>:<span class="hljs-number">14</span>,<span class="hljs-number">824</span> (conf-<span class="hljs-keyword">file</span>-poller-<span class="hljs-number">0</span>) [INFO - org.apache.flume.node.AbstractConfigurationProvider.loadChannels(AbstractConfigurationProvider.java:<span class="hljs-number">147</span>)] Creating channels
<span class="hljs-number">2017</span>-<span class="hljs-number">01</span>-<span class="hljs-number">09</span> <span class="hljs-number">00</span>:<span class="hljs-number">46</span>:<span class="hljs-number">14</span>,<span class="hljs-number">829</span> (conf-<span class="hljs-keyword">file</span>-poller-<span class="hljs-number">0</span>) [INFO - org.apache.flume.channel.DefaultChannelFactory.create(DefaultChannelFactory.java:<span class="hljs-number">42</span>)] Creating <span class="hljs-keyword">instance</span> of channel c1 type <span class="hljs-keyword">memory</span>
<span class="hljs-number">2017</span>-<span class="hljs-number">01</span>-<span class="hljs-number">09</span> <span class="hljs-number">00</span>:<span class="hljs-number">46</span>:<span class="hljs-number">14</span>,<span class="hljs-number">835</span> (conf-<span class="hljs-keyword">file</span>-poller-<span class="hljs-number">0</span>) [INFO - org.apache.flume.node.AbstractConfigurationProvider.loadChannels(AbstractConfigurationProvider.java:<span class="hljs-number">201</span>)] Created channel c1
<span class="hljs-number">2017</span>-<span class="hljs-number">01</span>-<span class="hljs-number">09</span> <span class="hljs-number">00</span>:<span class="hljs-number">46</span>:<span class="hljs-number">14</span>,<span class="hljs-number">835</span> (conf-<span class="hljs-keyword">file</span>-poller-<span class="hljs-number">0</span>) [INFO - org.apache.flume.<span class="hljs-keyword">source</span>.DefaultSourceFactory.create(DefaultSourceFactory.java:<span class="hljs-number">41</span>)] Creating <span class="hljs-keyword">instance</span> of <span class="hljs-keyword">source</span> r1, type netcat
<span class="hljs-number">2017</span>-<span class="hljs-number">01</span>-<span class="hljs-number">09</span> <span class="hljs-number">00</span>:<span class="hljs-number">46</span>:<span class="hljs-number">14</span>,<span class="hljs-number">845</span> (conf-<span class="hljs-keyword">file</span>-poller-<span class="hljs-number">0</span>) [INFO - org.apache.flume.sink.DefaultSinkFactory.create(DefaultSinkFactory.java:<span class="hljs-number">42</span>)] Creating <span class="hljs-keyword">instance</span> of sink: k1, type: logger
<span class="hljs-number">2017</span>-<span class="hljs-number">01</span>-<span class="hljs-number">09</span> <span class="hljs-number">00</span>:<span class="hljs-number">46</span>:<span class="hljs-number">14</span>,<span class="hljs-number">850</span> (conf-<span class="hljs-keyword">file</span>-poller-<span class="hljs-number">0</span>) [INFO - org.apache.flume.node.AbstractConfigurationProvider.getConfiguration(AbstractConfigurationProvider.java:<span class="hljs-number">116</span>)] Channel c1 connected to [r1, k1]
<span class="hljs-number">2017</span>-<span class="hljs-number">01</span>-<span class="hljs-number">09</span> <span class="hljs-number">00</span>:<span class="hljs-number">46</span>:<span class="hljs-number">14</span>,<span class="hljs-number">856</span> (conf-<span class="hljs-keyword">file</span>-poller-<span class="hljs-number">0</span>) [INFO - org.apache.flume.node.Application.startAllComponents(Application.java:<span class="hljs-number">137</span>)] Starting new configuration:{ sourceRunners:{r1=EventDrivenSourceRunner: { <span class="hljs-keyword">source</span>:org.apache.flume.<span class="hljs-keyword">source</span>.NetcatSource{name:r1,state:IDLE} }} sinkRunners:{k1=SinkRunner: { policy:org.apache.flume.sink.DefaultSinkProcessor<span class="hljs-variable">@720b314f</span> counterGroup:{ name:null counters:{} } }} channels:{c1=org.apache.flume.channel.MemoryChannel{name: c1}} }
<span class="hljs-number">2017</span>-<span class="hljs-number">01</span>-<span class="hljs-number">09</span> <span class="hljs-number">00</span>:<span class="hljs-number">46</span>:<span class="hljs-number">14</span>,<span class="hljs-number">865</span> (conf-<span class="hljs-keyword">file</span>-poller-<span class="hljs-number">0</span>) [INFO - org.apache.flume.node.Application.startAllComponents(Application.java:<span class="hljs-number">144</span>)] Starting Channel c1
<span class="hljs-number">2017</span>-<span class="hljs-number">01</span>-<span class="hljs-number">09</span> <span class="hljs-number">00</span>:<span class="hljs-number">46</span>:<span class="hljs-number">14</span>,<span class="hljs-number">930</span> (lifecycleSupervisor-<span class="hljs-number">1</span>-<span class="hljs-number">0</span>) [INFO - org.apache.flume.instrumentation.MonitoredCounterGroup.register(MonitoredCounterGroup.java:<span class="hljs-number">119</span>)] Monitored counter <span class="hljs-keyword">group</span> <span class="hljs-keyword">for</span> type: CHANNEL, name: c1: Successfully registered new MBean.
<span class="hljs-number">2017</span>-<span class="hljs-number">01</span>-<span class="hljs-number">09</span> <span class="hljs-number">00</span>:<span class="hljs-number">46</span>:<span class="hljs-number">14</span>,<span class="hljs-number">930</span> (lifecycleSupervisor-<span class="hljs-number">1</span>-<span class="hljs-number">0</span>) [INFO - org.apache.flume.instrumentation.MonitoredCounterGroup.start(MonitoredCounterGroup.java:<span class="hljs-number">95</span>)] Component type: CHANNEL, name: c1 started
<span class="hljs-number">2017</span>-<span class="hljs-number">01</span>-<span class="hljs-number">09</span> <span class="hljs-number">00</span>:<span class="hljs-number">46</span>:<span class="hljs-number">14</span>,<span class="hljs-number">932</span> (conf-<span class="hljs-keyword">file</span>-poller-<span class="hljs-number">0</span>) [INFO - org.apache.flume.node.Application.startAllComponents(Application.java:<span class="hljs-number">171</span>)] Starting Sink k1
<span class="hljs-number">2017</span>-<span class="hljs-number">01</span>-<span class="hljs-number">09</span> <span class="hljs-number">00</span>:<span class="hljs-number">46</span>:<span class="hljs-number">14</span>,<span class="hljs-number">933</span> (conf-<span class="hljs-keyword">file</span>-poller-<span class="hljs-number">0</span>) [INFO - org.apache.flume.node.Application.startAllComponents(Application.java:<span class="hljs-number">182</span>)] Starting Source r1
<span class="hljs-number">2017</span>-<span class="hljs-number">01</span>-<span class="hljs-number">09</span> <span class="hljs-number">00</span>:<span class="hljs-number">46</span>:<span class="hljs-number">14</span>,<span class="hljs-number">934</span> (lifecycleSupervisor-<span class="hljs-number">1</span>-<span class="hljs-number">0</span>) [INFO - org.apache.flume.<span class="hljs-keyword">source</span>.NetcatSource.start(NetcatSource.java:<span class="hljs-number">155</span>)] Source starting
<span class="hljs-number">2017</span>-<span class="hljs-number">01</span>-<span class="hljs-number">09</span> <span class="hljs-number">00</span>:<span class="hljs-number">46</span>:<span class="hljs-number">14</span>,<span class="hljs-number">961</span> (lifecycleSupervisor-<span class="hljs-number">1</span>-<span class="hljs-number">0</span>) [INFO - org.apache.flume.<span class="hljs-keyword">source</span>.NetcatSource.start(NetcatSource.java:<span class="hljs-number">169</span>)] Created serverSocket:sun.nio.ch.ServerSocketChannelImpl[/<span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">44444</span>]</code></pre>

<p>这边可以解析下启动命令：</p>

<blockquote>
  <p>./bin/flume-ng agent –conf conf –conf-file conf/example.conf  –name a1  -Dflume.root.logger=INFO,console</p>
</blockquote>

<ul>
<li>最前面是通过bin下的flume-ng这个脚本进行启动；</li>
<li>下面就是启动时候需要的一系列参数，–conf表示是配置 conf文件，conf-file 表示配置文件的位置，可以是相对位置也可以是绝对位置，–name 就是之前配置文件内的”a1.channels = c1”，猜测是flume服务器配置的名称， -Dflume.root.logger 表示输出形式，console是命令窗口输出，logging是log/flume.log文件输出。</li>
</ul>



<h2 id="4-输出-和-测试">4. 输出 和 测试</h2>

<p>为了测试，我们重新打开一个命令框，输入命令”telnet localhost 44444”，预计得到下述结果： <br>
当前命令窗：</p>



<pre class="prettyprint"><code class=" hljs r">localhost:~ xxx$ telnet localhost <span class="hljs-number">44444</span>
Trying <span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span><span class="hljs-keyword">...</span>
Connected to localhost.
Escape character is <span class="hljs-string">'^]'</span>.
hello
OK</code></pre>

<p>Flume命令窗：</p>



<pre class="prettyprint"><code class=" hljs css">2017<span class="hljs-tag">-01-09</span> 00<span class="hljs-pseudo">:56</span><span class="hljs-pseudo">:54</span>,946 (<span class="hljs-tag">SinkRunner-PollingRunner-DefaultSinkProcessor</span>) <span class="hljs-attr_selector">[INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)]</span> <span class="hljs-tag">Event</span>: <span class="hljs-rules">{ <span class="hljs-rule"><span class="hljs-attribute">headers</span>:<span class="hljs-value">{</span></span></span>} <span class="hljs-tag">body</span>: 68 65 6<span class="hljs-tag">C</span> 6<span class="hljs-tag">C</span> 6<span class="hljs-tag">F</span> 0<span class="hljs-tag">D</span>                               <span class="hljs-tag">hello</span>. }</code></pre>

<p>这边便是监控到了，在另一个页面输入到44444端口到hello信息。</p>

<p>好了，第一节到Flume介绍就到此为止了。</p>            </div>
						<link href="https://csdnimg.cn/release/phoenix/mdeditor/markdown_views-9e5741c4b9.css" rel="stylesheet">
                </div>