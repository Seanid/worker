# JDK版本不同导致的SSL异常:javax.net.ssl.SSLHandshakeException: Remote host closed connection during .....


#### 公司：YLZF
#### 职位：Java开发工程师
#### 项目代号：E

## 前言

遇到这个问题得说一下笔者的开发环境，笔者所在公司，平时开发用的web容器是jboss，使用的JDK是oracle的JDK，但是测试和生产环境用的是WAS，JDK用的是IBM的JDK，由于项目的不同，测试环境所安装的web容器和JDK版本都并不相同。这个也是笔者遇到问题的原因所在。

## 问题描述

笔者在公司负责一个项目的开发，该项目有一个功能需要调用到公司另一个项目的接口，而那个项目提供的接口是基于HTTPS的，所以笔者在进行调用的时候，使用了JDK提供的SSL连接进行了请求。


    SSLContext.getInstance(“SSL”);


该代码，笔者在自己本机的JBoss上测试过，并没有问题。可是，发布到was里面之后，程序一直没法成功调用接口，通过日志追踪后，发现程序在服务器运行的时候后台报了异常，该异常如下：

    javax.net.ssl.SSLHandshakeException: Remote host closed connection during handshank

握手失败，网上有人说，该异常是因为证书的原因，可是笔者尝试过后发现，该说法并不对应笔者所描述的场景，最后在同事的帮助下找到了原因。

## 问题分析

该问题出现的原因在于SSL协议的版本不同。发现问题之后，笔者分析了一下具体情况。由于项目的原因，笔者现在用的JDK版本还是1.5的，同时，笔者去调的项目也是个老项目，用的JDK版本也是1.5的。所以当笔者通过本地测试的时候，SSL的请求方和接收方的版本是一致的，并没有问题。可是由于项目的扩容，笔者项目的测试环境前阵子进行了迁移，新的测试环境采用的是IBM的JDK1.6。因为JDK版本不同，默认采用的握手协议不同，所以导致两边程序进行握手的时候会失败。

仔细查询了一下资料后发现，原来oracle的JDK默认采用的是TLSv1和TLSv2进行尝试握手连接的，而IBM新的JDK采用的是SSLv3，新老版本还不一样。

详细的资料地址：http://www-01.ibm.com/support/docview.wss?uid=swg21687173

问题解决

该问题的解决方法有两个：

方法一：修改获取SSL连接的的代码（笔者使用的方法，简单）

SSLContext.getInstance(“TLS”);

方法二：更新IBM的JDK

在上面给出的链接中，其实IBM已经意识到自己的问题，也给出了另外一个补丁，据说更新后就能成功（笔者没尝试过(￣▽￣)”）