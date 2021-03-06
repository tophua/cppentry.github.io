---
layout:     post
title:      hadoop2 hdfs命令
---
<div id="article_content" class="article_content clearfix csdn-tracking-statistics" data-pid="blog" data-mod="popu_307" data-dsm="post">
								            <link rel="stylesheet" href="https://csdnimg.cn/release/phoenix/template/css/ck_htmledit_views-f76675cdea.css">
						<div class="htmledit_views" id="content_views">
                
<p>Hadoop2 HDFS shell命令</p>
<p> </p>
<p>1. <tt>hdfs dfs -appendToFile &lt;localsrc&gt; ... &lt;dst&gt;</tt></p>
<p> </p>
<p>可同时上传多个文件到HDFS里面</p>
<p> </p>
<p>2.  <tt>hdfs dfs -cat URI [URI ...]</tt></p>
<p> </p>
<p>查看文件内容</p>
<p> </p>
<p>3. hdfs dfs -chgrp [-R] GROUP URI [URI ...]</p>
<p> </p>
<p>修改文件所属组</p>
<p> </p>
<p>4.  <tt>hdfs dfs -chmod [-R] &lt;MODE[,MODE]... | OCTALMODE&gt; URI [URI ...]</tt></p>
<p> </p>
<p>修改文件权限</p>
<p> </p>
<p>5. hdfs dfs -chown [-R] [OWNER][:[GROUP]] URI [URI ]</p>
<p> </p>
<p>修改文件所有者，文件所属组，其他用户的读、写、执行权限</p>
<p> </p>
<p>6. hdfs dfs -copyFromLocal &lt;localsrc&gt; URI</p>
<p> </p>
<p>复制文件到hdfs</p>
<p> </p>
<p>7.  <tt>hdfs dfs -copyToLocal [-ignorecrc] [-crc] URI &lt;localdst&gt;</tt></p>
<p><tt> </tt></p>
<p>复制文件到本地</p>
<p><tt> </tt></p>
<p><tt>8.</tt> hdfs dfs -count [-q] &lt;paths&gt;</p>
<p> </p>
<p>统计文件及文件夹数目</p>
<p> </p>
<p>9.  <tt>hdfs dfs -cp [-f] URI [URI ...] &lt;dest&gt;</tt></p>
<p> </p>
<p>Hadoop HDFS 文件系统间的文件复制</p>
<p> </p>
<p>10. hdfs dfs -du [-s] [-h] URI [URI ...]</p>
<p> </p>
<p>统计目录下的文件及大小</p>
<p> </p>
<p> </p>
<p> </p>
<p>11. hdfs dfs -dus &lt;args&gt;</p>
<p> </p>
<p>汇总目录下的文件总大小</p>
<p> </p>
<p>12.  <tt>hdfs dfs -expunge</tt></p>
<p> </p>
<p><span style="color:rgb(51,51,51);">清空回收站，文件被删除时，它首先会移到临时目录</span><span style="color:rgb(51,51,51);">.Trash/</span><span style="color:rgb(51,51,51);">中，当超过延迟时间之后，文件才会被永久删除</span></p>
<p> </p>
<p>13. hdfs dfs -get [-ignorecrc] [-crc] &lt;src&gt; &lt;localdst&gt;</p>
<p> </p>
<p>下载文件到本地</p>
<p> </p>
<p>14. hdfs dfs -getfacl [-R] &lt;path&gt;</p>
<p> </p>
<p>查看ACL （访问权限组拥有者）</p>
<p> </p>
<p>15. hdfs dfs -getmerge &lt;src&gt; &lt;localdst&gt; [addnl]</p>
<p> </p>
<p>合并下载文件到本地</p>
<p> </p>
<p>16. hdfs dfs -ls &lt;args&gt;</p>
<p> </p>
<p>查看目录</p>
<p> </p>
<p>17. hdfs dfs -lsr &lt;args&gt;</p>
<p> </p>
<p><span style="color:rgb(51,51,51);">循环列出目录、子目录及文件信息</span><span style="color:rgb(51,51,51);"> </span></p>
<p> </p>
<p>18. hdfs dfs -mkdir [-p] &lt;paths&gt;</p>
<p> </p>
<p>创建空白文件夹</p>
<p> </p>
<p>19.  <tt>dfs -moveFromLocal &lt;localsrc&gt; &lt;dst&gt;</tt></p>
<p> </p>
<p>剪切文件到hdfs</p>
<p> </p>
<p>20.  <tt>hdfs dfs -moveToLocal [-crc] &lt;src&gt; &lt;dst&gt;</tt></p>
<p> </p>
<p>剪切文件到本地</p>
<p> </p>
<p>21. hdfs dfs -mv URI [URI ...] &lt;dest&gt;</p>
<p> </p>
<p>剪切hdfs文件</p>
<p> </p>
<p>22. hdfs dfs -put &lt;localsrc&gt; ... &lt;dst&gt;</p>
<p> </p>
<p>上传文件</p>
<p> </p>
<p>23. hdfs dfs -rm [-skipTrash] URI [URI ...]</p>
<p> </p>
<p>删除文件/空白文件夹</p>
<p> </p>
<p>24.  <tt>hdfs dfs -rmr [-skipTrash] URI [URI ...]</tt></p>
<p> </p>
<p>递归删除  删除文件及文件夹下的所有文件</p>
<p> </p>
<p>25. hdfs dfs -setfacl [-R] [<a name="a-b-k" style="color:rgb(16,138,198);"></a>-b|-k <a name="a-m-x_acl_spec" style="color:rgb(16,138,198);"></a>-m|-x &lt;acl_spec&gt; &lt;path&gt;]|[--set &lt;acl_spec&gt; &lt;path&gt;]</p>
<p> </p>
<p>Sets Access Control Lists (ACLs) of files and directories.</p>
<p>Options:</p>
<ul><li style="color:rgb(51,51,51);">-b: Remove all but the base ACL entries. The entries for user, group and others are retained for compatibility with permission bits.</li><li style="color:rgb(51,51,51);">-k: Remove the default ACL.</li><li style="color:rgb(51,51,51);">-R: Apply operations to all files and directories recursively.</li><li style="color:rgb(51,51,51);">-m: Modify ACL. New entries are added to the ACL, and existing entries are retained.</li><li style="color:rgb(51,51,51);">-x: Remove specified ACL entries. Other ACL entries are retained.</li><li style="color:rgb(51,51,51);">--set: Fully replace the ACL, discarding all existing entries. The <em>acl_spec</em> must include entries for user, group, and others for compatibility with permission bits.</li><li style="color:rgb(51,51,51);"><em>acl_spec</em>: Comma separated list of ACL entries.</li><li style="color:rgb(51,51,51);"><em>path</em>: File or directory to modify.</li></ul><p>Examples:</p>
<ul><li style="color:rgb(51,51,51);">hdfs dfs -setfacl -m user:hadoop:rw- /file</li><li style="color:rgb(51,51,51);">hdfs dfs -setfacl -x user:hadoop /file</li><li style="color:rgb(51,51,51);">hdfs dfs -setfacl -b /file</li><li style="color:rgb(51,51,51);">hdfs dfs -setfacl -k /dir</li><li style="color:rgb(51,51,51);">hdfs dfs -setfacl --set user::rw-,user:hadoop:rw-,group::r--,other::r-- /file</li><li style="color:rgb(51,51,51);">hdfs dfs -setfacl -R -m user:hadoop:r-x /dir</li><li style="color:rgb(51,51,51);">hdfs dfs -setfacl -m default:user:hadoop:r-x /dir</li></ul><p>Exit Code:</p>
<p>Returns 0 on success and non-zero on error.</p>
<p> </p>
<p> </p>
<p>26.  <tt>hdfs dfs -setrep [-R] [-w] &lt;numReplicas&gt; &lt;path&gt;</tt></p>
<p> </p>
<p>修改副本数</p>
<p> </p>
<p>27. hdfs dfs -stat URI [URI ...]</p>
<p> </p>
<p>显示文件统计信息</p>
<p> </p>
<p>28.  <tt>hdfs dfs -tail [-f] URI</tt></p>
<p> </p>
<p>查看文件尾部信息</p>
<p> </p>
<p>29. hdfs dfs -test -[ezd] URI</p>
<p> </p>
<p><span style="color:rgb(51,51,51);">对PATH进行如下类型的检查： </span></p>
<p><span style="color:rgb(51,51,51);">-e PATH</span><span style="color:rgb(51,51,51);">是否存在，如果PATH存在，返回0，否则返回1 </span></p>
<p><span style="color:rgb(51,51,51);">-z </span><span style="color:rgb(51,51,51);">文件是否为空，如果长度为0，返回0，否则返回1 </span></p>
<p><span style="color:rgb(51,51,51);">-d </span><span style="color:rgb(51,51,51);">是否为目录，如果PATH为目录，返回0，否则返回1 </span></p>
<p> </p>
<p>30. hdfs dfs -text &lt;src&gt;</p>
<p> </p>
<p>查看文件内容</p>
<p> </p>
<p>31.  <tt>hdfs dfs -touchz URI [URI ...]</tt></p>
<p> </p>
<p><span style="color:rgb(51,51,51);">创建长度为</span><span style="color:rgb(51,51,51);">0</span><span style="color:rgb(51,51,51);">的空文件</span></p>
            </div>
                </div>