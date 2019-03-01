---
layout: post
title: hadoop Springboot集成fsshell
category: hadoop学习
tags: hadoop
keywords: hadoop
top: 2
---

### HDFS文件系统操作
HDFS是一个和我们平时的文件系统有很大区别的系统，我们对于HDFS的编程应用其实相当于是一个HDFS的指令集合的应用
下面是结合<a href=https://docs.spring.io/spring-hadoop/docs/2.5.1.BUILD-SNAPSHOT/reference/html/index.html>Spring for Apache 
Hadoop</a>与<a href=https://github.com/spring-projects/spring-hadoop-samples>github的示例</a>学习spring与hadoop结合使用

# 目标

使用Spring Boot构建一个简单的Spring Hadoop应用程序，从hdfs获取一个简单的文件及目录清单。

### pom增加导入
```
 <dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-hadoop-boot</artifactId>
    <version>2.2.0.RELEASE</version>
</dependency>
```
### application代码
```
@SpringBootApplication
public class DemoApplication implements CommandLineRunner {

	@Autowired
	private FsShell shell;

	@Override
	public void run(String... args) {
		for (FileStatus s : shell.lsr("hdfs://localhost:9000/")) {
			System.out.println("> " + s.getPath());
		}
	}

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}
```

在这之前使用我使用dfs指令创建了几个目录`/HADOOP/CREATE`，上传了一个文件`LICENSE.txt`，可以在WEB上看到
<img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/20190218.png"></img>

jar包执行

```
java -jar target/boot-fsshell-0.1.0.jar
```

执行jar包得到结果：
```
[root@xxx jar_repo]# java -jar boot-fsshell-0.1.0.jar 
> hdfs://localhost:9000/
> hdfs://localhost:9000/HADOOP
> hdfs://localhost:9000/HADOOP/CREATE
> hdfs://localhost:9000/HADOOP/CREATE/LICENSE.txt

```
#### 注意
在这期间执行`start-dfs.sh`发现DataNode一直起不来，查看日志
```
vim /opt/HADOOP/hadoop-2.9.2/logs/hadoop-root-datanode-server25.log
```
```
2019-02-15 17:31:17,544 INFO org.apache.hadoop.hdfs.server.datanode.DataNode: registered UNIX signal handlers for [TERM, HUP, INT]
2019-02-15 17:31:18,933 INFO org.apache.hadoop.hdfs.server.datanode.checker.ThrottledAsyncChecker: Scheduling a check for [DISK]file:/opt/HADOOP/datanode/store/
2019-02-15 17:31:19,074 INFO org.apache.hadoop.metrics2.impl.MetricsConfig: loaded properties from hadoop-metrics2.properties
2019-02-15 17:31:19,234 INFO org.apache.hadoop.metrics2.impl.MetricsSystemImpl: Scheduled Metric snapshot period at 10 second(s).
2019-02-15 17:31:19,234 INFO org.apache.hadoop.metrics2.impl.MetricsSystemImpl: DataNode metrics system started
2019-02-15 17:31:19,250 INFO org.apache.hadoop.hdfs.server.common.Util: dfs.datanode.fileio.profiling.sampling.percentage set to 0. Disabling file IO profiling
2019-02-15 17:31:19,256 INFO org.apache.hadoop.hdfs.server.datanode.BlockScanner: Initialized block scanner with targetBytesPerSec 1048576
2019-02-15 17:31:19,262 INFO org.apache.hadoop.hdfs.server.datanode.DataNode: Configured hostname is localhost
2019-02-15 17:31:19,263 INFO org.apache.hadoop.hdfs.server.common.Util: dfs.datanode.fileio.profiling.sampling.percentage set to 0. Disabling file IO profiling
2019-02-15 17:31:19,263 WARN org.apache.hadoop.conf.Configuration: No unit for dfs.datanode.outliers.report.interval(1800000) assuming MILLISECONDS
2019-02-15 17:31:19,271 INFO org.apache.hadoop.hdfs.server.datanode.DataNode: Starting DataNode with maxLockedMemory = 0
2019-02-15 17:31:19,322 INFO org.apache.hadoop.hdfs.server.datanode.DataNode: Opened streaming server at /0.0.0.0:50107
2019-02-15 17:31:19,326 INFO org.apache.hadoop.hdfs.server.datanode.DataNode: Balancing bandwidth is 10485760 bytes/s
2019-02-15 17:31:19,326 INFO org.apache.hadoop.hdfs.server.datanode.DataNode: Number threads for balancing is 50
2019-02-15 17:31:19,543 INFO org.mortbay.log: Logging to org.slf4j.impl.Log4jLoggerAdapter(org.mortbay.log) via org.mortbay.log.Slf4jLog
2019-02-15 17:31:19,564 INFO org.apache.hadoop.security.authentication.server.AuthenticationFilter: Unable to initialize FileSignerSecretProvider, falling back to use random secrets.
2019-02-15 17:31:19,597 INFO org.apache.hadoop.http.HttpRequestLog: Http request log for http.requests.datanode is not defined
2019-02-15 17:31:19,609 INFO org.apache.hadoop.http.HttpServer2: Added global filter 'safety' (class=org.apache.hadoop.http.HttpServer2$QuotingInputFilter)
2019-02-15 17:31:19,613 INFO org.apache.hadoop.http.HttpServer2: Added filter static_user_filter (class=org.apache.hadoop.http.lib.StaticUserWebFilter$StaticUserFilter) to context datanode
2019-02-15 17:31:19,614 INFO org.apache.hadoop.http.HttpServer2: Added filter static_user_filter (class=org.apache.hadoop.http.lib.StaticUserWebFilter$StaticUserFilter) to context static
2019-02-15 17:31:19,614 INFO org.apache.hadoop.http.HttpServer2: Added filter static_user_filter (class=org.apache.hadoop.http.lib.StaticUserWebFilter$StaticUserFilter) to context logs
       at org.apache.hadoop.hdfs.server.datanode.web.DatanodeHttpServer.start(DatanodeHttpServer.java:250)
       at org.apache.hadoop.hdfs.server.datanode.DataNode.startInfoServer(DataNode.java:963)
       at org.apache.hadoop.hdfs.server.datanode.DataNode.startDataNode(DataNode.java:1370)
       at org.apache.hadoop.hdfs.server.datanode.DataNode.<init>(DataNode.java:495)
       at org.apache.hadoop.hdfs.server.datanode.DataNode.makeInstance(DataNode.java:2695)
       at org.apache.hadoop.hdfs.server.datanode.DataNode.instantiateDataNode(DataNode.java:2598)
       at org.apache.hadoop.hdfs.server.datanode.DataNode.createDataNode(DataNode.java:2645)
       at org.apache.hadoop.hdfs.server.datanode.DataNode.secureMain(DataNode.java:2789)
       at org.apache.hadoop.hdfs.server.datanode.DataNode.main(DataNode.java:2813)
Caused by: java.net.BindException: Address already in use
       at sun.nio.ch.Net.bind0(Native Method)
       at sun.nio.ch.Net.bind(Net.java:433)
       at sun.nio.ch.Net.bind(Net.java:425)
       at sun.nio.ch.ServerSocketChannelImpl.bind(ServerSocketChannelImpl.java:223)
       at sun.nio.ch.ServerSocketAdaptor.bind(ServerSocketAdaptor.java:74)
       at io.netty.channel.socket.nio.NioServerSocketChannel.doBind(NioServerSocketChannel.java:125)
       at io.netty.channel.AbstractChannel$AbstractUnsafe.bind(AbstractChannel.java:475)
       at io.netty.channel.DefaultChannelPipeline$HeadContext.bind(DefaultChannelPipeline.java:1021)
       at io.netty.channel.AbstractChannelHandlerContext.invokeBind(AbstractChannelHandlerContext.java:455)
       at io.netty.channel.AbstractChannelHandlerContext.bind(AbstractChannelHandlerContext.java:440)
       at io.netty.channel.DefaultChannelPipeline.bind(DefaultChannelPipeline.java:844)
       at io.netty.channel.AbstractChannel.bind(AbstractChannel.java:194)
       at io.netty.bootstrap.AbstractBootstrap$2.run(AbstractBootstrap.java:340)
       at io.netty.util.concurrent.SingleThreadEventExecutor.runAllTasks(SingleThreadEventExecutor.java:380)
       at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:357)
       at io.netty.util.concurrent.SingleThreadEventExecutor$2.run(SingleThreadEventExecutor.java:116)
       at io.netty.util.concurrent.DefaultThreadFactory$DefaultRunnableDecorator.run(DefaultThreadFactory.java:137)
       at java.lang.Thread.run(Thread.java:748)
2019-02-15 17:31:20,398 INFO org.apache.hadoop.util.ExitUtil: Exiting with status 1: java.net.BindException: Problem binding to [0.0.0.0:50012] java.net.BindException: Address already in use; For more details see:  http://wiki.apache.org/hadoop/BindException
2019-02-15 17:31:20,405 INFO org.apache.hadoop.hdfs.server.datanode.DataNode: SHUTDOWN_MSG:
/************************************************************
SHUTDOWN_MSG: Shutting down DataNode at localhost/127.0.0.1
```
发现是端口被占用，找到被占用的端口，干掉之后重启就正常了
```
netstat -atunlp|grep 50107
kill -9 xxx
```
