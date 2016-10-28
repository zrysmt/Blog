---
title: 写给前端er的TCP/IP知识及《图解TCP/IP》读书笔记
tags: 
- TCP/IP
- 图解TCP/IP
categories: HTTP/TCP/IP
---
# 1.分层
OSI参考模型分为7层，TCP/IP分为四层。
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/tcp-ip/1.jpg)

# 2.物理设备介绍
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/tcp-ip/2.jpg)

# 3.传输过程
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/tcp-ip/3.jpg)

# 4.分层介绍

## 4.1 数据链路层

几个关键的相关技术
- MAC地址：用于识别数据链路层中互连的节点，在使用网卡（NIC）的情况下，MAC地址会烧入在ROM中
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/tcp-ip/4.png)
- 以太网（Ethernet）
以太网帧式，前端是前导码部分，后面是帧的本体
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/tcp-ip/5.jpg)
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/tcp-ip/6.jpg)
帧尾叫做FCS，用来检测帧信息是否完整

## 4.2 网路层

### 4.2.1 IP协议--无连接型

- **数据链路层和IP层的区别：**
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/tcp-ip/7.jpg)
#### 1. **IP地址的分类**
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/tcp-ip/8.jpg)
A类：0.0.0.0    ~   127.0.0.0      【127为回环测试地址，如127.0.0.0为本机地址】
B类：128.0.0.1  ~   191.255.0.0
C类：192.0.0.0  ~   233.255.255.0
D类：224.0.0.0  ~   239.255.255.0   【用于多播】
#### 2. **单播、广播、多播**
单播：一对一
广播：会被路由器屏蔽
【例如：192.168.0.0/24广播地址为192.168.0.255/24】
多播：能通过路由器，D类IP地址，从224.0.0.0 ~ 239.255.255.255
其中224.0.0.0到224.0.0.255不需要路由控制，在同一个链路中能实现多播。
#### 3. **解决IP地址有限：**
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/tcp-ip/9.jpg)
标识方法：
**方法1：**
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/tcp-ip/10.jpg)
**方法2：**
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/tcp-ip/11.jpg)
#### 4. **IP分片：**
数据链路不同，最大的传输单元（MTU）不同，所以需要对IP分片进行处理。分片只能在目标主机中进行重组。
- **ICMP通知MTU大小**
路径MTU发现机制（UDP情况下）
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/tcp-ip/12.png)
路径MTU发现机制（TCP情况下）不同于上
#### 5. **IPv6**
IP地址长度为128位，以每18比特为一组进行标记，如果出现连续的0，用“::”代替
- **IPv6地址结构：**
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/tcp-ip/13.png)
全局单播地址是世界上唯一的地址
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/tcp-ip/14.png)
#### 6. **IPv4首部**
IP首部+IP载荷（数据）组成：
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/tcp-ip/16.png)
#### 7. **IPv6首部**
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/tcp-ip/17.png)

### 4.2.2 IP协议相关技术

#### 1. DNS
管理主机名和IP地址之间对应关系的系统，叫做DNS系统。
- **DNS查询：**
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/tcp-ip/18.png)
第三步 会将IP地址信息暂时保存到缓存中，减少每次查询时的性能消耗。
DNS的主要记录包括很多类型的数据，比如类型A值主机名的IP地址，PTR指IP地址的反向解析，即IP地址检索的主机名。
#### 2. ARP
IP地址到Mac地址解析
#### 3.ICMP
主要功能是确认IP包是否成功送达目的地址，通知在发送过程当中IP包被废弃的原因，改善网络的设置等。
#### 4.DHCP
动态设置ip地址

## 4.3 TCP/UDP

- TCP首部格式
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/tcp-ip/19.jpg)
- 三次握手
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/tcp-ip/20.jpg)
- 识别多个请求
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/tcp-ip/21.jpg)
- 套接口
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/tcp-ip/22.jpg)

## 4.4 应用层
应用层有SSH，FTP，HTTP，TLS/SSL等
- ftp使用两条TCP连接
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/tcp-ip/23.jpg)
- javascript，CGI
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/tcp-ip/24.jpg)












































