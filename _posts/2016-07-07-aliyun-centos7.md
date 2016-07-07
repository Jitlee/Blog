---
layout: post
title: 阿里云CentOS7.2 部署Apache + PHP + MySQL
description: 本文讲述的是用yum命令在阿里云最新的CentOS7.2 环境下安装Apache、PHP和MySQL
categories: [Linux]
image:
geometry: 
tags: [阿里云 CentOS MySql Apache PHP]
---

> 买的几十块钱的服务器，我最新安装的是Mysql, 由于走了弯路，重置过了好几次。最后发现关键点就是没有安装阿里云镜像源，
连Perl都没办法安装正常。

本文参考：vfhky 
[https://typecodes.com/linux/yuminstallmysql5710.html](https://typecodes.com/linux/yuminstallmysql5710.html)

### 1. 安装阿里云镜像源

阿里云Linux安装镜像源地址：http://mirrors.aliyun.com/

#### 1.1. 备份原有的镜像

{% highlight bash %}
> mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
{% endhighlight %}

#### 1.2. 下载对应版本的镜像源

下载 CentOS 5

{% highlight bash %}
> wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-5.repo
{% endhighlight %}

下载 CentOS 6

{% highlight bash %}
> wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
{% endhighlight %}

下载 CentOS 7

{% highlight bash %}
> wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
{% endhighlight %}

#### 1.3. 生成缓存

{% highlight bash %}
> yum clean all
> yum makecache
{% endhighlight %}

### 2. 下载MySQL官方的yum repository

可以在MySql官方选择对应的版本，我选择的是最新的5.7版本。

[http://dev.mysql.com/downloads/repo/yum/](http://dev.mysql.com/downloads/repo/yum/)

*至于为什么要下载下来本地安装，而不直接在线安装，我用弯路告你，直接安装的将会是 MySQL在CentOS 7上面的一个分支版本MariaDB*

#### 2.1. 下载rpm

{% highlight bash %}
> wget -i http://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
{% endhighlight %}

#### 2.2. 本地安装

{% highlight bash %}
> yum -y localinstall mysql57-community-release-el7-7.noarch.rpm
{% endhighlight %}

#### 2.3. 安装MySQL服务器版本

{% highlight bash %}
> yum -y install mysql-community-server
{% endhighlight %}

#### 2.3. 启动服务

{% highlight bash %}
> systemctl start  mysqld.service
{% endhighlight %}

#### 2.4. 获取数据库的初始密码并更改

使用yum安装并启动MySQL服务后，MySQL进程会自动在进程日志中打印root用户的初始密码：

{% highlight bash %}
grep "password" /var/log/mysqld.log
{% endhighlight %}

利用上述密码进入mysql， 登陆Mysql，用SQL语句修改密码：

{% highlight bash %}
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'new password';
{% endhighlight %}

### 3. 安装Appache

#### 3.1. 在线安装

{% highlight bash %}
> yum install -y httpd
{% endhighlight %}

#### 3.2. 设置开机启动

{% highlight bash %}
> systemctl enable httpd.service
{% endhighlight %}

#### 3.2. 启动服务

{% highlight bash %}
> systemctl start httpd.service
{% endhighlight %}

apache的配置文件路径，和网站存放路径分别是：

{% highlight bash %}
> /etc/httpd/conf/httpd.conf
> /var/www/html
{% endhighlight %}

### 4. 安装PHP

{% highlight bash %}
> yum -y install php
{% endhighlight %}

根据需要安装所需的插件，后期也来得及用yum增加php插件，非常方便

{% highlight bash %}
> yum -y install php-gd php-ldap php-odbc php-pear php-xml php-xmlrpc php-mbstring php-snmp php-soap curl curl-devel php-mysql
{% endhighlight %}

重启httpd

{% highlight bash %}
> systemctl restart httpd.service
{% endhighlight %}

---

至此全部安装完毕，后续还有开放80,3306端口，修改Mysql外网可访问，配置Appache等工作就轻松多了。







