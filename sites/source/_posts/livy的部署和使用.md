---
title: livy的部署和使用
tags: ['livy','spark']
date: 2022-04-20
categories: ['livy']
---
## Livy概述
Livy是Apache Spark的一个REST服务，通过Livy，可以实现在任意的平台上通过Http请求提交Spark任务。
* 通过livy提交的spark任务，对原始的spark脚本没有任何入侵
* livy支持多用户、多任务并行的和spark集群进行交互
* 可以在python、scala、java中通过livy进行spark批任务的处理
<!--more-->


## Livy部署
Livy的部署只需要在spark集群的master节点上进行部署即可。
这里默认依赖的spark和hadoop都已经就位，并不需要从头安装。

### 下载livy
```shell
# 下载地址：http://livy.incubator.apache.org/download/
wget https://www.apache.org/dyn/closer.lua/incubator/livy/0.7.1-incubating/apache-livy-0.7.1-incubating-bin.zip
```

### 修改配置文件
1. 修改livy.conf
```shell
cp livy.conf.template livy.conf & vim livy.conf
```
需要调整的基础信息如下：
```yml
#Livy会话所使用的Spark集群运行模式
livy.spark.master = yarn

#######其他可以根据自身项目需求进行设置，不想修改默认就好########

#Livy会话所使用的Spark集群部署模式
livy.spark.deploy-mode=cluster
#默认使用hiveContext
livy.repl.enable-hive-context = true
#开启用户代理
livy.impersonation.enabled = true
#设置session空闲过期时间
livy.server.session.timeout = 1h
#yarn/local本地模式或者yarn模式
livy.server.session.factory = yarn
```

2. 修改livy-env.sh
```shell
cp livy-env.sh.template livy-env.sh & vim livy-env.sh
```
需要调整的基础信息如下：
```yml
#Spark安装目录
HADOOP_CONF_DIR=/opt/software/hadoop-3.0.0/etc/hadoop
#Hadoo配置目录
SPARK_HOME=/opt/software/spark-2.5.2-bin-without-hadoop
```

### 启动Livy
```shell
# 启动livy
./apache-livy-0.7.1-incubating-bin/bin/livy-server start

# 关闭livy
./apache-livy-0.7.1-incubating-bin/bin/livy-server stop
```

## Livy使用
livy默认的接口为8998, 在配置文件没有显式的声明其他端口的时候，可以直接向当前的端口提交任务。
```shell
# 提交任务
curl -H "Content-Type: application/json" -X POST -d '{"driverMemory": "20G","numExecutors": 60,"executorCores": 4,"executorMemory": "25G","conf": {"spark.dynamicAllocation.enabled": "False","spark.default.arallelism": "2000","spark.sql.shuffle.partitions": "200","spark.memory.fraction": "0.8","spark.port.maxRetries": "256"},"file": "${file_path_on_hdfs}","args": ["--date=20220420"]}' http://${spark-master-url}:8998/batches/ | python -m json.tool
```
通过上述命令提交后，返回如下，会包含生成的livy-session的id。
![livy-submit-return](../images/deploy-usage-of-livy/livy-submit-return.png)

```shell
# 获取指定batch_id的session状态
curl -X GET http://${spark-master-url}:8998/batches/${batch_id}/state | python -m json.tool
# 获取指定batch_id的session-log
curl -X GET http://${spark-master-url}:8998/batches/${batch_id}/log | python -m json.tool
# 中断指定batch_id的session，同时会中断application
curl -X DELETE http://${spark-master-url}:8998/batches/${batch_id} | python -m json.tool
```