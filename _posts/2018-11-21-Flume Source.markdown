---
layout:     post
title:      Flume Source
---
<div id="article_content" class="article_content clearfix csdn-tracking-statistics" data-pid="blog" data-mod="popu_307" data-dsm="post">
								            <link rel="stylesheet" href="https://csdnimg.cn/release/phoenix/template/css/ck_htmledit_views-f76675cdea.css">
						<div class="htmledit_views" id="content_views">
                
<div id="cnblogs_post_body" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;line-height:21px;">
<p><span style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;line-height:21px;"><span style="line-height:24px;font-size:18px;">Flume Source</span></span><br></p>
<p>1、Flume’s Tiered Event Sources</p>
<table width="684" cellpadding="0" border="1" style="border:1px solid rgb(192,192,192);border-collapse:collapse;"><tbody><tr><td width="216" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>collectorSource[(<em>port</em>)]</p>
</td>
<td width="466" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>Collector source，监听端口汇聚数据</p>
</td>
</tr><tr><td width="216" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>autoCollectorSource</p>
</td>
<td width="466" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>通过master协调物理节点自动汇聚数据</p>
</td>
</tr><tr><td width="216" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>logicalSource</p>
</td>
<td width="466" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>逻辑source，由master分配端口并监听rpcSink</p>
</td>
</tr></tbody></table><p>   </p>
<p>   </p>
<p><strong> </strong></p>
<p>2、Flume’s Basic Sources</p>
<table width="685" cellpadding="1" border="1" style="border:1px solid rgb(192,192,192);border-collapse:collapse;"><tbody><tr><td width="214" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>null</p>
</td>
<td width="469" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
 </td>
</tr><tr><td width="214" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>console</p>
</td>
<td width="469" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>监听用户编辑历史和快捷键输入，只在node_nowatch模式下可用</p>
</td>
</tr><tr><td width="214" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>stdin</p>
</td>
<td width="469" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>监听标准输入，只在node_nowatch模式下可用，每行将作为一个event source</p>
</td>
</tr><tr><td width="214" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>rpcSource(<em>port</em>)</p>
</td>
<td width="469" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>由rpc框架(thrift/avro)监听tcp端口</p>
</td>
</tr><tr><td width="214" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>text("<em>filename</em>")</p>
</td>
<td width="469" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>一次性读取一个文本，每行为一个event</p>
</td>
</tr><tr><td width="214" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>tail("<em>filename</em>"[, <em>startFromEnd</em>=false])</p>
</td>
<td width="469" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>每行为一个event。监听文件尾部的追加行，如果startFromEnd为true，tail将从文件尾读取，如果为false，tail将从文件开始读取全部数据</p>
</td>
</tr><tr><td width="214" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>multitail("<em>filename</em>"[, <em>file2</em> [,<em>file3</em>… ] ])</p>
</td>
<td width="469" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>同上，同时监听多个文件的末尾</p>
</td>
</tr><tr><td width="214" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>tailDir("<em>dirname</em>"[, fileregex=".*"[, startFromEnd=false[, recurseDepth=0]]])</p>
</td>
<td width="469" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>监听目录中的文件末尾，使用正则去选定需要监听的文件（不包含目录），recurseDepth为递归监听其下子目录的深度</p>
</td>
</tr><tr><td width="214" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>seqfile("<em>filename</em>")</p>
</td>
<td width="469" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>监听hdfs的sequencefile，全路径</p>
</td>
</tr><tr><td width="214" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>syslogUdp(<em>port</em>)</p>
</td>
<td width="469" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>监听Udp端口</p>
</td>
</tr><tr><td width="214" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>syslogTcp(<em>port</em>)</p>
</td>
<td width="469" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>监听Tcp端口</p>
</td>
</tr><tr><td width="214" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>syslogTcp1(<em>port</em>)</p>
</td>
<td width="469" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>只监听Tcp端口的一个链接</p>
</td>
</tr><tr><td width="214" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>execPeriodic("<em>cmdline</em>", <em>ms</em>)</p>
</td>
<td width="469" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>周期执行指令，监听指令的输出，整个输出都被作为一个event</p>
</td>
</tr><tr><td width="214" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>execStream("<em>cmdline</em>")</p>
</td>
<td width="469" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>执行指令，监听指令的输出，输出的每一行被作为一个event</p>
</td>
</tr><tr><td width="214" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>exec("<em>cmdline</em>"[,<em>aggregate</em>=false[,<em>restart</em>=false[,<em>period</em>=0]]])</p>
</td>
<td width="469" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>执行指令，监听指令的输出，aggregate如果为true，整个输出作为一个event如果为false，则每行作为一个event。如果restart为true，则按period为周期重新运行</p>
</td>
</tr><tr><td width="214" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>synth(<em>msgCount</em>,<em>msgSize</em>)</p>
</td>
<td width="469" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>随即产生字符串event,msgCount为产生数量，msgSize为串长度</p>
</td>
</tr><tr><td width="214" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>synthrndsize(<em>msgCount</em>,<em>minSize</em>,<em>maxSize</em>)</p>
</td>
<td width="469" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>同上，minSize – maxSize</p>
</td>
</tr><tr><td width="214" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>nonlsynth(<em>msgCount</em>,<em>msgSize</em>)</p>
</td>
<td width="469" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
 </td>
</tr><tr><td width="214" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>asciisynth(<em>msgCount</em>,<em>msgSize</em>)</p>
</td>
<td width="469" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>Ascii码字符</p>
</td>
</tr><tr><td width="214" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>twitter("<em>username</em>","<em>pw</em>"[,"<em>url</em>"])</p>
</td>
<td width="469" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>尼玛twitter的插件啊</p>
</td>
</tr><tr><td width="214" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>irc("<em>server</em>",<em>port</em>, "<em>nick</em>","<em>chan</em>")</p>
</td>
<td width="469" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
 </td>
</tr><tr><td width="214" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>scribe[(+<em>port</em>)]</p>
</td>
<td width="469" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>Scribe插件</p>
</td>
</tr><tr><td width="214" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>report[(periodMillis)]</p>
</td>
<td width="469" valign="top" style="border:1px solid rgb(192,192,192);border-collapse:collapse;">
<p>生成所有physical node报告为事件源</p>
</td>
</tr></tbody></table><br></div>
<div id="MySignature" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;line-height:21px;">
</div>
<div id="blog_post_info_block" style="font-family:Verdana, Geneva, Arial, Helvetica, sans-serif;line-height:21px;">
<div id="blog_post_info"></div>
</div>
            </div>
                </div>