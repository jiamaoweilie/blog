# 一个简单的负载均衡系统

[负载均衡](http://baike.baidu.com/link?url=IJriLUPAnM_nNxQw8eVoqfU4R9Tc8sl28d5QcZ7sOYp9jcNAA-HGq7RENalBfm-a)是由多台服务器以对称的方式组成一个服务器集合，每台服务器都具有等价的地位，都可以单独对外提供服务而无须其他服务器的辅助。通过某种负载分担技术，将外部发送来的请求分配到对称结构中的某一台服务器上，而接受到请求的服务器独立的回应客户的请求。负载均衡能够平均分配客户请求到服务器阵列，以提高相应速度，解决大量并发访问服务器问题。实现网络的负载均衡有很多方法，例如使用Nginx, LVS, HAProxy，f5等。本文将介绍如何使用Nginx配置一个简单的负载均衡系统。

[Nginx](http://nginx.org/)是一款轻量级的Web服务器/反向代理服务器以及电子邮件（IMAP/POP3）代理服务器。由俄罗斯的程序设计师Igor Sysoev所开发，供俄国大型的入口网站及搜索引擎Rambler（俄文：Рамблер）使用。其特点是占有内存少，并发能力强。本文会将Nginx封装在一个Docker Image中，将其作为Web服务器，分发来自用户的请求到不同的App服务器中，其中的App服务器是也是一个Docker Image封装的简单的Sinatra应用。然后将几个Image运行在一个Ubuntu机器上，模拟一个Web服务器和多台App服务器的环境，当然你的机器上首先需要[安装Docker](https://docs.docker.com/installation/ubuntulinux/)。

下面先来看看怎么实现这两个Docker Image。

## 创建Sinatra App服务

### Docker Image实现代码介绍

[上篇文章](http://jiamaoweilie.github.io/blog/2015/07/12/docker/)中介绍了如何使用Docker构建开发环境，这里不在赘述，直接看看它的Dockerfile文件的内容。

```
FROM ubuntu:14.04

MAINTAINER wei 20150629

RUN apt-get update
RUN apt-get install -y build-essential wget git
RUN apt-get install -y zlib1g-dev libssl-dev libreadline-dev libyaml-dev libxml2-dev libxslt-dev libmysqlclient-dev
RUN apt-get clean

RUN wget http://cache.ruby-lang.org/pub/ruby/2.2/ruby-2.2.2.tar.gz
RUN tar xvf ruby-2.2.2.tar.gz
RUN cd /ruby-2.2.2; ./configure; make install

RUN gem update --system

RUN gem install bundler

ADD app.rb /app.rb

ADD Gemfile /Gemfile

ADD config.ru /config.ru

RUN bundle install

EXPOSE 9292

CMD rackup --host 0.0.0.0
```

从上述代码可以看出，该Image使用ubuntu:14.04作为基础，然后使用apt-get安装一些工具，再安装了ruby环境，并将app.rb、Gemfile、和config.ru文件拷贝到Image中，然后执行`bundle install`安装应用所使用的gem依赖,再将9292端口暴露出来，最后使用rackup命令启动该应用。

### 应用实现代码介绍

该实例的`app.rb`实现了一个特别简单的Sinatra应用，代码如下：

```
require 'sinatra'

class App < Sinatra::Base
  get '/' do
    "Hello World!"
  end
end
```

`Gemfile`文件中定义了该应用的gem依赖：

```
source 'https://rubygems.org'

gem 'sinatra'
```

`config.ru`文件为使用Rack的gem中提供的工具rackup工具为该应用编写了一个启动项，具体内容为下：

```
require File.dirname(__FILE__) + '/app'

run App
```

完成上面代码之后，像Dockerfile文件中那样，执行`bundle install`安装依赖，然后执行`rackup`启动程序，访问`localhost://9292`就会显示**Hello World!**

完成代码的编写之后，运行`docker build`命令来build我们的Image：

```
sudo docker build -t docker-sinatra .
```

执行完该命令之后就生成了一个名为docker-sinatra的docker image，我们可以通过`docker images`命令查看当前机器上所有的docker image。

然后执行如下命令来生成一个Container来运行该image：

```
docker run -d -p 8080:9292 docker-sinatra
```

该命令运行名为docker-siatra的Image，并将Container中的9292端口映射到Ubuntu的8080端口，这样我们访问`localhost://8080`就可以看到我们想要的**Hello World!**了。

同样，如果我们想要8081端口也运行着这个app服务，只需执行如下命令：

```
sudo docker run -d -p 8081:9292 docker-sinatra
```

至此，我们已经实现了两个简单的Sinatra App服务，下面就来看看如何使用Nginx来分配用户的访问请求。

## 使用Nginx实现负载均衡

### 创建Nginx Docker Image

具体的Dockerfile文件如下：

```
FROM ubuntu:14.04

MAINTAINER wei 20150629

RUN apt-get update && \
    apt-get install -y nginx && \
    rm -rf /var/lib/apt/lists/* && \
    echo "\ndaemon off;" >> /etc/nginx/nginx.conf && \
    chown -R www-data:www-data /var/lib/nginx

ADD run.sh /run.sh

RUN chmod 755 /*.sh

VOLUME ["/etc/nginx/sites-enabled", "/etc/nginx/certs", "/etc/nginx/conf.d", "/var/log/nginx", "/var/www/html"]

WORKDIR /etc/nginx

CMD ["/run.sh"]

EXPOSE 80 
```

从上述代码可知，该Image安装了nginx，并配置了一些共享目录，最后run了一个脚本来启动nginx服务。

其中`run.sh`的内容为：

```
#!/bin/bash

# start nginx service
/usr/sbin/nginx
```

以上部分我们就完成了Nginx的Docker Image的基本编写，这时候`build`这个image：

```
sudo docker build -t docker-nginx .
```

然后run:

```
sudo docker run -d -p 80:80 docker-nginx
```

访问`localhost`，可以看到如下页面：

![](/images/img_for_LB/nginx.png)

这说明Nginx服务已经启动，下面我们就来更改它的配置文件使之可以作为Web服务器，分配用户访问请求到我们上面已经启动的两个app服务器上。

### Nginx配置文件

Nginx的配置文件为`nginx.conf`，一般的存储路径为`/etc/nginx/nginx.conf`。

``` yml
#运行用户
user www-data;
#启动进程,通常设置成和cpu的数量相等
worker_processes 1;
#全局错误日志及PID文件
error_log  /var/log/nginx/error.log;
pid /run/nginx.pid;

#工作模式及连接数上限
events {
  worker_connections 768;
}

#设定http服务器，利用它的反向代理功能提供负载均衡支持
http {
	
  #设定mime类型,类型由mime.type文件定义
  include /etc/nginx/mime.types;
  default_type  application/octet-stream;
 
  #sendfile指令指定nginx是否调用sendfile函数（zero copy 方式来输出文件，对于普通应用必须设为on,如果用来进行下载等应用磁盘IO重负载应用，可设置为 off，以平衡磁盘与网络I/O处理速度，降低系统的uptime.
  sendfile        on;
  keepalive_timeout  65;
	
  #设定负载均衡的服务器列表
  upstream allserver {
    server 192.168.59.104:8080;
    server 192.168.59.104:8081;
  }
  server {
    #侦听80端口
    listen       80;
    #定义使用localhist访问
    server_name  localhost;
    #默认请求
    location / {
      proxy_pass http://allserver;
    }
  }
}
daemon off;
```

上面代码展示了一个基本的Nginx配置文件，和关于该配置文件的简单介绍。其中，upstream节指定了运行在ip为192.168.59.104的机器上8080端口和8081端口的两个网络程序（就是我们上面起的两个sinatra应用）。server节中指定启动一个nginx进程，监听80端口，并将用户的访问全部转发到upstream中的两个程序上。

完成Nginx配置文件的更改之后，需要reload配置文件：

```
/usr/sbin/nginx -s reload
```

这样，我们的配置文件就可以生效了，当用户访问`http://localhost`就可以看到页面显示的**Hello World!**了。而且，只要这两个程序中的一个还在运行，用户就可以正常的访问。

至此，我们就是用Nginx实现了一个简单的负载均衡系统，该实例中的详细代码，可以参见[我的github](https://github.com/jiamaoweilie)。




