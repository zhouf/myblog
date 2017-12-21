---
title: 在CentOS下安装redis记录
date: 2017-12-21 22:46:32
tags:
---
下载redis安装包，写此文时最新安装包为4.0.6
```
# wget http://download.redis.io/releases/redis-4.0.6.tar.gz
```
解压到指定目录，如`/usr/local`
```
# tar -xvf redis-4.0.6.tar.gz -C /usr/local/
```
转到redis目录
```
# cd /usr/local/redis-4.0.6/
```
进行编译
```
# make
```
如果出现错误如下，说明缺少编译库
```
/bin/sh: cc: command not found
```
下载安装gcc编译库
```
# yum -y install gcc-c++
# make
```
如果出现如下错误，可能是之前编译的遗留问题
```
zmalloc.h:50:31: fatal error: jemalloc/jemalloc.h: No such file or directory
```
重新清理一下再编译
```
# make distclean
# make
```
出面下面的提示说明编译成功了，推荐使用`make test`测一下
```
Hint: It's a good idea to run 'make test' ;)
```
好吧，执行一个
```
# make test
```
提示需要tcl库
```
You need tcl 8.5 or newer in order to run the Redis test
```
下载安装一个
```
# yum -y install tcl
# make test
```
这个可能需要一点时间，等等吧，如果没有错误会看到如下消息
```
\o/ All tests passed without errors!
```

接下来再通过`install`把常用命令放在`/usr/local/bin`目录下
```
# make install
cd src && make install
make[1]: Entering directory `/usr/local/redis-4.0.6/src'

Hint: It's a good idea to run 'make test' ;)

    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
make[1]: Leaving directory `/usr/local/redis-4.0.6/src'
```
至此，安装完成，总结一下安装步骤，需要两个依赖库`gcc-c++`,`tcl`，那么理想的操作应该是先把库准备好再安装redis

## 安装小结
```
# yum -y install gcc-c++ tcl
# wget http://download.redis.io/releases/redis-4.0.6.tar.gz
# tar -xvf redis-4.0.6.tar.gz -C /usr/local/
# cd /usr/local/redis-4.0.6/
# make
# make install
# make test
```

## 启动服务
```
# redis-server &
```
因为启动会占用当前命令窗口，为了方便操作，可以使用&使其转入后台运行

## 启用客户端
```
# redis-cli
127.0.0.1:6379> 
```
单机模式已安装完成，可以开始redis学习了:D
