---
layout:     post
title:      shell脚本启动jar及修改jar内容文件操作
subtitle:   记录jar内部文件修改启动，停止等操作脚本编写
date:       2019-07-12
author:     lwj108
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - java
    - github
---
### 基本说明
在Linux服务器下/opt/micro-service/目录放置各类微服务jar文件并创建config文件夹将config.properties放入其中，jar包启动时会去读取其中的配置文件，如果需要指定文件可以使用upstart命令进行操作。

运行脚本命令：sh service.sh

```
在 opt/micro-service/目录文件存放格式必须如下
├── config 
│├── config.properties
├── service.sh
├── xxx.jar
├── xxx.jar
├── xxx.jar
...
```
### 执行脚本后相关命令注释：
* start：启动
* stop：停止
* status：状态
* restart：重启
* startAll：全部启动
* stopAll：全部停止
* restartAll：全部重启
* upstart：修改某一文件后启动


### 关于upstart的示例：
需要修改 message.jar中下/message/src/main/resources/kafka.properties文件
先输入kafka.properties，脚本回去查找jar下全部改名称的文件路径，选择你需要改动的路径下的文件，复制出来，再输入。之后输入你新的文件（需要替换到jar的）的路径，即完成文件的替换。之后会启动jar。

### shell脚本代码
```shell
cd /opt/micro-service/

#输入要进行的操作，启动jar
echo "you need todo?(start/stop/status/restart/startAll/stopAll/restartAll/upstart)"
read todo

#检查程序是否在运行
is_exist(){
  pid=`ps -ef|grep $jarName|grep -v grep|awk '{print $2}'`
  #如果不存在返回1，存在返回0     
  if [ -z "${pid}" ]; then
   return 1
  else
    return 0
  fi
}

#修改启动
upstart(){
  #输入要操作的jar包
  echo "Please Input jar Name:(servcie.jar)"
  read jarName

  is_exist
  if [ $? -eq 0 ]; then
    echo "${jarName} is already running. please stop"
  else
    jar tvf $jarName |grep $fileName
    #选择正确的文件路径进行修改
    echo "Query out as follows,Please select the copy you want"
    read oldFilePath
    #解压jar内文件
    jar xvf $jarName $oldFilePath
    sleep 1
    #输入jar外替换文件路径
    echo "Please Input replace file path:(like:config/config.properties)"
    read newFilePath
    #替换文件
    cp $newFilePath $oldFilePath
    echo "Replace success!"
    #压缩回jar内
    jar uvf $jarName $oldFilePath
    sleep 2
    #启动
    java -jar $jarName &
  fi
}

#启动
start(){
  #输入要操作的jar包
  echo "Please Input jar Name:(servcie.jar)"
  read jarName
  is_exist
  if [ $? -eq 0 ]; then
    echo "${jarName} is already running. pid=${pid}"
  else
    #解压jar内文件
    jar xvf $jarName BOOT-INF/classes/config/config.properties
    sleep 1
    #替换文件
    cp config.properties BOOT-INF/classes/config/config.properties
    #压缩回jar内
    jar uvf $jarName BOOT-INF/classes/config/config.properties
    sleep 2
    #启动
    java -jar $jarName &
  fi
}

#停止
stop(){
  #输入要操作的jar包
  echo "Please Input jar Name:(servcie.jar)"
  read jarName
  is_exist
  if [ $? -eq "0" ]; then
    kill -9 $pid
    ps -ef|grep java
  else
    echo "${jarName} is not running"
  fi 
}

#状态
status(){
  #输入要操作的jar包
  echo "Please Input jar Name:(servcie.jar)"
  read jarName
  is_exist
  if [ $? -eq "0" ]; then
    echo "${jarName} is running. Pid is ${pid}"
  else
    echo "${jarName} is NOT running."
  fi
}

#重启
restart(){
  #输入要操作的jar包
  echo "Please Input jar Name:(servcie.jar)"
  read jarName
  stop
  sleep 5
  start
}

#全部启动
startAll(){
  for FILENAME in $(ls *jar);
  do 
    pid=`ps -ef|grep $FILENAME|grep -v grep|awk '{print $2}'`
    java -jar $pid & ; 
  done
}

#全部停止
stopAll(){
  for FILENAME in $(ls *jar);
  do 
    pkill -9 $FILENAME
  done
}

#全部重启
restartAll(){
  stopAll
  sleep 10
  startAll
}

case $todo in
  start)
    start
    ;;
  stop)
    stop
    ;;
  status)
    status
    ;;
  restart)
    restart
    ;;
  startAll)
    startAll
    ;;
  stopAll)
    stopAll
    ;;
  restartAll)
    restartAll
    ;;
esac
```