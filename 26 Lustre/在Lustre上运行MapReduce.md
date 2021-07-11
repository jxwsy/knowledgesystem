# 在Lustre上运行MapReduce

[TOC]

## 1、现有集群状况

主机名  |  角色
---|:---
node1   | MGS MGT MDS MDT
node2   | OSS OST
node3   | OSS OST
node5   | NameNode、 ResoucerManager、Client(lustre)
node6   | DataNode、 NodeManager、 SecondaryNameNode
node7   | DataNode、 NodeManager

## 2、集群调整

将 Lustre 挂载到 Hadoop 集群的每个节点上，挂载目录相同。（也就是，将 Hadoop 集群的每个节点都作为 Lustre 的一个客户端节点。）

方法参考：[https://github.com/ZGG2016/knowledgesystem/blob/master/26%20Lustre/%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA.md](https://github.com/ZGG2016/knowledgesystem/blob/master/26%20Lustre/%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA.md)

```sh
[root@node5 lustre]# pwd
/mnt/lustre

[root@node6 lustre]# pwd
/mnt/lustre

[root@node7 lustre]# pwd
/mnt/lustre
```

调整 Hadoop 的配置文件

```xml
[root@node5 hadoop]# cat core-site.xml
<configuration>
    <property>
          <name>fs.defaultFS</name>
          <value>file:///mnt/lustre</value>
    </property>
    <property>
         <name>hadoop.tmp.dir</name>
         <value>/mnt/lustre/mr/tmp/</value>
    </property> 
</configuration>

[root@node5 hadoop]# cat mapred-site.xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.cluster.local.dir</name>
        <value>/mnt/lustre/mr/interfile/</value>
    </property>
    <property>
        <name>yarn.app.mapreduce.am.staging-dir</name>
        <value>/mnt/lustre/mr/hadoop-yarn/staging/</value>
    </property>
</configuration>

[root@node5 hadoop]# cat yarn-site.xml
<configuration>
	...
    <property>
        <name>yarn.nodemanager.local-dirs</name>  
        <value>/mnt/lustre/mr/interfile/</value>  
    </property>
    ...
</configuration>
```

同步到 node6 和 node7 节点。

## 3、测试

```sh
hadoop jar /opt/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar wordcount /mnt/lustre/wc/wc.txt /mnt/lustre/wc/out

[root@node5 out]# pwd
/mnt/lustre/wc/out

[root@node5 out]# ls
part-r-00000  _SUCCESS

[root@node5 out]# cat part-r-00000
aa      3
bb      2

[root@node5 out]# lfs getstripe part-r-00000
part-r-00000
lmm_stripe_count:  1
lmm_stripe_size:   1048576
lmm_pattern:       raid0
lmm_layout_gen:    0
lmm_stripe_offset: 1
        obdidx           objid           objid           group
             1              70           0x46                0
```

## 4、问题

执行任务出现 `File file:/tmp/hadoop-yarn/staging/root/.staging/job_1622686375131_0001/job.splitmetainfo does not exist
java.io.FileNotFoundException: File file:/tmp/hadoop-yarn/staging/root/.staging/job_1622686375131_0001/job.splitmetainfo does not exist`。

因为 node6 和 node7 节点没有此目录，而 node5 有。

所以在 `mapred-site.xml` 中，将属性 `yarn.app.mapreduce.am.staging-dir` 设置到挂载目录下，这样这三个节点都能访问到此目录。

```xml
<property>
    <name>yarn.app.mapreduce.am.staging-dir</name>
    <value>/mnt/lustre/mr/hadoop-yarn/staging/</value>
</property>
```