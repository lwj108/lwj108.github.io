---
layout:     post
title:      kafka部署及相关操作命令
subtitle:   kafka在服务器上需要记住的操作
date:       2019-06-12
author:     lwj108
header-img: img/post-bg-unix-linux.jpg
catalog: true
tags:
    - kafka
---
#解压kafka安装包
cd /kafka_2.12-2.2.0/bin

#启动kafka
sh kafka-server-start.sh ../config/server.properties

#config/server.properties在搭建kafka集群时，可以配置多个监听（多个端口需配置多个配置文件在kafka启动时选定对应的配置文件即可）
#如
#server1.properties
advertised.listeners=PLAINTEXT://192.168.1.xxx:9092 
port=9092
#server2.properties
advertised.listeners=PLAINTEXT://192.168.1.xxx:9093 
port=9093

#启动zookeeper
bin/zookeeper-server-start.sh config/zookeeper.properties &

#启动kafka
bin/kafka-server-start.sh config/server.properties &

#停止kafka
bin/kafka-server-stop.sh

#停止zookeeper
bin/zookeeper-server-stop.sh

#开启zookeeper端口
firewall-cmd --zone=public --add-port=2181/tcp --permanent

#开启kafka端口
firewall-cmd --zone=public --add-port=9092/tcp --permanent

#重启firewall
firewall-cmd --reload 

#创建topic
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test

#展示topic
bin/kafka-topics.sh --list --zookeeper localhost:2181

#描述topic
bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic

#生产者：
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my-replicated-topic

#消费者：
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginnin
#如果创建消费者报错 consumer zookeeper is not a recognized option请使用下面命令
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning