# rabbitmq 安装

标签（空格分隔）： rabbitmq

---
# rabbitmq简介
  MQ全称为Message Queue, 消息队列（MQ）是一种应用程序对应用程序的通信方法。应用程序通过读写出入队列的消息（针对应用程序的数据）来通信，而无需专用连接来链接它们。消息传递指的是程序之间通过在消息中发送数据进行通信，而不是通过直接调用彼此来通信，直接调用通常是用于诸如远程过程调用的技术。排队指的是应用程序通过 队列来通信。队列的使用除去了接收和发送应用程序同时执行的要求。其中较为成熟的MQ产品有IBM WEBSPHERE MQ等等。
# 安装
## 需要先装erlang
 下载地址：http://www.erlang.org/downloads
 编译erl,加压压缩包，cd到解压的压缩包内
 yum install -y gcc ncurses-devel openssl-devel unixODBC-devel wxGTK-devel
 ./configure --prefix=/usr/local/erlang
 

