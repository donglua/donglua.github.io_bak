+++
title = "OpenGrok 部署"
draft = false
date = "2017-01-12T23:55:08+08:00"

+++

<!--more-->
> 环境：Debian 8

## 1. 安装 JDK 和 Tomcat

Add Java 8 PPA
```
sudo vim /etc/apt/sources.list.d/java-8-debian.list
```
添加以下内容
```
deb http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main
deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main
```
导入 GPG key
```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys EEA14886
```
安装 JDK
```
sudo apt-get update
sudo apt-get install oracle-java8-installer
```
验证是否安装成功
```
java -version
java version "1.8.0_111"
Java(TM) SE Runtime Environment (build 1.8.0_111-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.111-b14, mixed mode)
```
安装 Tomcat
```
sudo apt-get install tomcat8
```
配置环境变量才能启动 Tomcat
```
export JAVA_HOME=/usr/lib/jvm/java-8-oracle
export CATALINA_HOME=/usr/share/tomcat8/
export OPENGROK_TOMCAT_BASE=$CATALINA_HOME
```
安装 tomcat 管理程序、文档（可选）
```
sudo apt-get install tomcat8-admin tomcat8-examples tomcat8-docs
```
## 2. 安装 OpenGrok

下载
```
wget https://github.com/OpenGrok/OpenGrok/files/631110/opengrok-0.13-rc5.zip
```
解压
```
unzip opengrok-0.13-rc5.zip
tar xvf opengrok-0.13-rc5.tar.gz -C /var/
cd /var
mv /var/opengrok-0.13-rc5 opengrok
```
部署 opengrok
```
./var/opengrok/bin/OpenGrok deploy
```
建立索引
```
mkdir /var/opengrok/src
./var/opengrok/bin/OpenGrok index
```

最后 浏览器打开 <host>:8080/source 即可看到OpenGrok 的界面。

![](http://upload-images.jianshu.io/upload_images/11626-93a3a3294beed8d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 3. 使用 Nginx 作为反向代理

安装 Nginx
```
sudo apt-get install nginx
```
配置文件
```
upstream backend {
    server 127.0.0.1:8080;
}

server {
  listen 80;
  server_name source;
  root /var/lib/tomcat8/webapps/;
  index index.jsp index.html index.htm;

  location / {
     proxy_pass  http://backend/source/;
  }

  location  ~ ^/source/(.*) {
     return 301  /$1?$args;
  }
}
```

## 4. 参考
[搭建大型源码阅读环境——使用 OpenGrok](http://mazhuang.org/2016/12/14/rtfsc-with-opengrok/)
