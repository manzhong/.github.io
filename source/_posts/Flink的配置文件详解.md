---
title: Flink的配置文件详解
tags: Flink
categories: Flink
abbrlink: 22766
date: 2020-09-25 20:44:07
summary_img:
encrypt:
enc_pwd:
---

# Flink的配置文件详解

## 1 概述

 	flink的配置文件,不经常配的话,有些参数不好记忆,为了以后方便,就记录一下,flink的配置文件,方便以后查

安装目录下主要有 flink-conf.yaml 配置、日志的配置文件、zk 配置、Flink SQL Client 配置。

```shell
hongfei-MacBook-Pro:conf manzhong$ ls
flink-conf.yaml			
logback-yarn.xml
log4j-cli.properties		
logback.xml
log4j-console.properties	
masters
log4j-yarn-session.properties	
slaves
log4j.properties		
sql-client-defaults.yaml
logback-console.xml		
zoo.cfg
hongfei-MacBook-Pro:conf manzhong$
```

## 2 flink-conf.yaml

### 2.1 基础配置

```yaml
# jobManager 的IP地址
jobmanager.rpc.address: localhost

# JobManager 的端口号 内部与taskmanager通信的端口号 
jobmanager.rpc.port: 6123

# JobManager JVM heap 内存大小
jobmanager.heap.size: 1024m

# TaskManager JVM heap 内存大小
taskmanager.heap.size: 1024m

# 每个 TaskManager 提供的任务 slots 数量大小

taskmanager.numberOfTaskSlots: 1

# 程序默认并行计算的个数
parallelism.default: 1

# 文件系统来源
# fs.default-scheme
```

### 2.2 高可用配置

```yaml
# 可以选择 'NONE' 或者 'zookeeper'.
# high-availability: zookeeper

# 文件系统路径，让 Flink 在高可用性设置中持久保存元数据
# high-availability.storageDir: hdfs:///flink/ha/

# zookeeper 集群中仲裁者的机器 ip 和 port 端口号
# high-availability.zookeeper.quorum: localhost:2181

# 默认是 open，如果 zookeeper security 启用了该值会更改成 creator
# high-availability.zookeeper.client.acl: open
```

### 2.3 容错和检查点 配置

```yaml
# 用于存储和检查点状态
# state.backend: filesystem

# 存储检查点的数据文件和元数据的默认目录
# state.checkpoints.dir: hdfs://namenode-host:port/flink-checkpoints

# savepoints 的默认目标目录(可选)
# state.savepoints.dir: hdfs://namenode-host:port/flink-checkpoints

# 用于启用/禁用增量 checkpoints 的标志
# state.backend.incremental: false
```

### 2.4 web 前端配置

```yaml
# 基于 Web 的运行时监视器侦听的地址.
#jobmanager.web.address: 0.0.0.0

#  Web 的运行时监视器端口
rest.port: 8081

# 是否从基于 Web 的 jobmanager 启用作业提交
# jobmanager.web.submit.enable: false
```

### 2.5 高级配置

```yaml
# io.tmp.dirs: /tmp

# 是否应在 TaskManager 启动时预先分配 TaskManager 管理的内存
# taskmanager.memory.preallocate: false

# 类加载解析顺序，是先检查用户代码 jar（“child-first”）还是应用程序类路径（“parent-first”）。 默认设置指示首先从用户代码 jar 加载类
# classloader.resolve-order: child-first


# 用于网络缓冲区的 JVM 内存的分数。 这决定了 TaskManager 可以同时拥有多少流数据交换通道以及通道缓冲的程度。 如果作业被拒绝或者您收到系统没有足够缓冲区的警告，请增加此值或下面的最小/最大值。 另请注意，“taskmanager.network.memory.min”和“taskmanager.network.memory.max”可能会覆盖此分数

# taskmanager.network.memory.fraction: 0.1
# taskmanager.network.memory.min: 67108864
# taskmanager.network.memory.max: 1073741824
```

### 2.6 Flink 集群安全配置

```yaml
# 指示是否从 Kerberos ticket 缓存中读取
# security.kerberos.login.use-ticket-cache: true

# 包含用户凭据的 Kerberos 密钥表文件的绝对路径
# security.kerberos.login.keytab: /path/to/kerberos/keytab

# 与 keytab 关联的 Kerberos 主体名称
# security.kerberos.login.principal: flink-user

# 以逗号分隔的登录上下文列表，用于提供 Kerberos 凭据（例如，`Client，KafkaClient`使用凭证进行 ZooKeeper 身份验证和 Kafka 身份验证）
# security.kerberos.login.contexts: Client,KafkaClient
```

### 2.7 zk的安全配置

```yaml
# 覆盖以下配置以提供自定义 ZK 服务名称
# zookeeper.sasl.service-name: zookeeper

# 该配置必须匹配 "security.kerberos.login.contexts" 中的列表（含有一个）
# zookeeper.sasl.login-context-name: Client
```

### 2.8 HistoryServer

```yaml
# 你可以通过 bin/historyserver.sh (start|stop) 命令启动和关闭 HistoryServer

# 将已完成的作业上传到的目录
# jobmanager.archive.fs.dir: hdfs:///completed-jobs/

# 基于 Web 的 HistoryServer 的地址
# historyserver.web.address: 0.0.0.0

# 基于 Web 的 HistoryServer 的端口号
# historyserver.web.port: 8082

# 以逗号分隔的目录列表，用于监视已完成的作业
# historyserver.archive.fs.dir: hdfs:///completed-jobs/

# 刷新受监控目录的时间间隔（以毫秒为单位）
# historyserver.archive.fs.refresh-interval: 10000
```

## 3 slaves / master

```shell
hongfei-MacBook-Pro:conf manzhong$ cat slaves
localhost
hongfei-MacBook-Pro:conf manzhong$ cat masters
localhost:8081
hongfei-MacBook-Pro:conf manzhong$
```



```
slaves
里面是每个 worker 节点的 IP/Hostname，每一个 worker 结点之后都会运行一个 TaskManager，一个一行。
master
host:port
```

## 4 zoo.cfg

```yaml
# 每个 tick 的毫秒数
tickTime=2000

# 初始同步阶段可以采用的 tick 数
initLimit=10

# 在发送请求和获取确认之间可以传递的 tick 数
syncLimit=5

# 存储快照的目录
# dataDir=/tmp/zookeeper

# 客户端将连接的端口
clientPort=2181

# ZooKeeper quorum peers
server.1=localhost:2888:3888
# server.2=host:peer-port:leader-port
```

## 5 日志配置

Flink 在不同平台下运行的日志文件

```
log4j-cli.properties
log4j-console.properties
log4j-yarn-session.properties
log4j.properties
logback-console.xml
logback-yarn.xml
logback.xml
```

## 6 sql-client-defaults.yaml

```yaml
execution:
  # 'batch' or 'streaming' execution
  type: streaming
  # allow 'event-time' or only 'processing-time' in sources
  time-characteristic: event-time
  # interval in ms for emitting periodic watermarks
  periodic-watermarks-interval: 200
  # 'changelog' or 'table' presentation of results
  result-mode: changelog
  # parallelism of the program
  parallelism: 1
  # maximum parallelism
  max-parallelism: 128
  # minimum idle state retention in ms
  min-idle-state-retention: 0
  # maximum idle state retention in ms
  max-idle-state-retention: 0
  
deployment:
  # general cluster communication timeout in ms
  response-timeout: 5000
  # (optional) address from cluster to gateway
  gateway-address: ""
  # (optional) port from cluster to gateway
  gateway-port: 0
```

## 7 总结

以上记录都是安装目录下的配置文件

官网的配置文件:

```url
https://ci.apache.org/projects/flink/flink-docs-stable/ops/config.html
```



