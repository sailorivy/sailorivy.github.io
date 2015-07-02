---
title: Cassandra起步
layout: post
categories: 源码分析
tags: Cassandra
comments: yes
---

<h2>Cassandra</h2>
Cassandra是一套高可扩展、最终一致、分布式的结构化键值存储系统。

Cassandra结合了Dynamo的分布技术和Google的BigTable数据模型，具有Dynamo的最终一致性，并提供了与Google的Bigtable相似的、基于ColumnFamily的数据模型。

<h2>编译</h2>
<h3>工具</h3>
Cassandra的源码使用Git管理，使用Ant进行编译。下面是v2.2编译使用工具的版本要求：

| 工具 | 版本 |
|:--------|:-------|
| Git | 1.8.0_45 |
| JDK | 1.9.4 |
| Ant | 2.3.5 |

<h3>编译</h3>
1. 获取源码

		git clone git://git.apache.org/cassandra.git cassandra
		cd cassandra
	
	执行上面的命令获取到的是Cassandra的trunk，trunk比较活跃，每天都有更新，不太利于学习和源码分析。不过要是已经对Cassandra比较熟悉了，可以跟进trunk：
	
		git pull
	
	Cassandra目前最新的正式版本是v2.1.7，也推出了v2.2.0的RC1版本。后面选择v2.1.x进行学习和分析，所以检出v2.1.x对应的分支：
	
		git checkout cassandra-2.1
		
2.	调用Ant进行编译

		ant
		
3. 调用Ant编译cassandra-stress工具

		ant stress-build        

<h2>快速开始</h2>

<h3>配置</h3>

Cassandra的配置文件位于conf目录下。主要的配置文件有：

* cassandra.yaml——存储配置。里面会设置集群名称、失败处理策略、监听地址、传输端口等信息，以及Cassandra使用的目录，包括数据文件、提交日志、缓存的存放目录。这些目录默认在Cassandra的data目录下，如果要自己设置，需要确保设置的目录存在并有写权限。
* logback.xml——日志的相关配置。
* cassandra-env.sh——可以配置JVM的设置。

<h3>运行节点</h3>

高可扩展的集群也是由多个单节点组成的，先看看单节点的操作。

<h4>启动</h4>
按照手册，执行下面的命令就可以在前台启动Cassandra：

	./bin/cassandra -f
	
bin/cassandra默认会读取bin/cassandra.in.sh和conf/cassandra-env.sh中的内容。

<h4>Mac下的问题</h4>
20150613 pull下来的代码编译后，在Mac下运行上面的命令会有下面的错误：

	-bash: ./bin/cassandra: /bin/sh^M: bad interpreter: No such file or directory
	
这是因为bin/cassandra的换行符是Dos格式的，在Mac下无法执行。所以先修改换行符：

	vi bin/cassandra
	:set ff?
	（结果为fileformat=dos）
	:set ff=unix
	:wq
	
-------------

再次启动会遇到下面的错误：

	: command not foundsh: line 16: 
	'/bin/cassandra.in.sh: line 43: syntax error near unexpected token `do
	'/bin/cassandra.in.sh: line 43: `for jar in "$CASSANDRA_HOME"/lib/*.jar; do
	You must set the CASSANDRA_CONF and CLASSPATH vars

这是因为bin/cassandra默认会调用bin/cassandra.in.sh，bin/cassandra.in.sh的换行符也是Dos格式的，在Mac下执行会出错。同样，采用上面的步骤修改换行符。

------------
再次启动，标准输出一开始会有下面的内容：

	: command not foundndra-env.sh: line 16: 
	'/bin/../conf/cassandra-env.sh: line 17: syntax error near unexpected token `
	'/bin/../conf/cassandra-env.sh: line 17: `calculate_heap_sizes()

conf/cassandra-env.sh里设置了JVM的配置，bin/cassandra会调用conf/cassandra-env.sh。而conf/cassandra-env.sh的换行符也是Dos格式的，同样，采用上面的步骤修改换行符。

------------
继续启动，标准输出里还是有WARN和ERROR：

	WARN  13:02:48 JNA link failure, one or more native method will be unavailable.
	WARN  13:02:48 JMX is not enabled to receive remote connections. Please see cassandra-env.sh for more info.
	ERROR 13:02:48 Error starting local jmx server: 
	java.io.IOException: Cannot bind to URL [rmi://localhost:7199/jmxrmi]: 	javax.naming.ServiceUnavailableException [Root exception is 	java.rmi.ConnectException: Connection refused to host: localhost; nested exception is: 
		java.net.ConnectException: Connection refused]
		......
	Caused by: javax.naming.ServiceUnavailableException: null
		......
	Caused by: java.rmi.ConnectException: Connection refused to host: localhost; nested exception is: 
		java.net.ConnectException: Connection refused
		......
	Caused by: java.net.ConnectException: Connection refused
		......

JNA的警告暂时没有解决。

至于JMX，我们对conf/cassandra-env.sh进行如下修改：

	1. 注释掉LOCAL_JMX=yes。Cassandra默认只能从localhost访问JMX，要允许远程连接，就注释掉LOCAL_JMX=yes。
	2. 如果允许远程连接，Cassandra默认不启用ssl、但启用认证。我们不进行安全处理，所以：
		* 将JVM_OPTS="$JVM_OPTS -Dcom.sun.management.jmxremote.authenticate=true"改成JVM_OPTS="$JVM_OPTS -Dcom.sun.management.jmxremote.authenticate=false"
		* 注释JVM_OPTS="$JVM_OPTS -Dcom.sun.management.jmxremote.password.file=/etc/cassandra/jmxremote.password"
	

------------
继续启动，标准输出里还是有ERROR：

	ERROR 02:44:32 Exception encountered during startup
	java.lang.RuntimeException: Unable to gossip with any seeds
		......
	java.lang.RuntimeException: Unable to gossip with any seeds
		......
	Exception encountered during startup: Unable to gossip with any seeds

这个错误是因为节点监听的地址没有纳入集群，节点监听地址用listen_address配置，而集群中的节点要互相发现，就要把节点的地址配置到seeds中。cassandra.yaml中的相关配置如下：

	listen_address: localhost
	......
	seed_provider:
		- class_name: org.apache.cassandra.locator.SimpleSeedProvider
		  parameters:
             - seeds: "127.0.0.1"
	
	将 localhost 和 127.0.0.1 都改成IP地址 

<h4>停止</h4>
Cassandra手册上说如果指定-f选项启动的话，按"Control-C"就可以停止节点，但v2.1并不支持这种方式。

在bin目录下有一个stop-server脚本，修改换行符后执行，它会让我们阅读脚本内容，实际上启动不指定-p选项的话，Cassandra节点的停止就是用kill -9命令去杀进程。


<h3>cqlsh的使用</h3>

Cassandra提供了查询语言CQL（Cassandra Query Language），使用CQL可以定义Schema、插入数据、执行查询。

bin/cqlsh提供了交互式执行方式，我们可以运行cqlsh脚本来执行CQL。Mac上先修改bin/cqlsh的换行符，并将里面DEFAULT_HOST的值改成节点所在机器的IP地址。

	bin/cqlsh
	
连接成功的话会有下面的提示：

	Connected to Test Cluster at 192.168.6.166:9042.
	[cqlsh 5.0.1 | Cassandra 2.1.7-SNAPSHOT | CQL spec 3.2.0 | Native protocol v3]
	Use HELP for help.
	cqlsh> 

可以参照Cassandra官方Getting Started里的[CQL示例](http://wiki.apache.org/cassandra/GettingStarted#Step_4:_Using_cqlsh)进行测试。
