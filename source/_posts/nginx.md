---
title: Introduction of Nginx
date: 2018-04-09 15:26:12
tags:
---

# 发音

- 这是官网Frequently Asked Questions的第一个问题 https://www.nginx.com/resources/wiki/community/faq/
- correct
  - en-juhn-eks*
  - Engine-X
- Incorrect
  - en-jingks

# 反向代理(Reverse Proxy)

- 在电脑网络中，反向代理是代理服务器的一种。服务器根据客户端的请求，从其关系的一组或多组后端服务器（如Web服务器）上获取资源，然后再将这些资源返回给客户端，客户端只会得知反向代理的IP地址，而不知道在代理服务器后面的服务器簇的存在。

- CDN就是一个典型的反向代理。

## 正向代理代理客户端，反向代理代理服务器

- 与正向代理不同: 前向代理作为客户端的代理，将从互联网上获取的资源返回给一个或多个的客户端，服务器端（如Web服务器）只知道代理的IP地址而不知道客户端的IP地址；而反向代理是作为服务器端（如Web服务器）的代理使用，而不是客户端。客户端借由前向代理可以间接访问很多不同互联网服务器（簇）的资源，而反向代理是供很多客户端都通过它间接访问不同后端服务器上的资源，而不需要知道这些后端服务器的存在，而以为所有资源都来自于这个反向代理服务器。

- 举一个可能不太准确的例子：我们平时说到的代理就是代购，我拜托某个去某个国家的朋友帮我代购东西，外国的商家并不知道最终是谁在买这个商品，甚至可能有很多朋友都帮忙请他代购，或者说我请了不同的朋友帮我买不同的东西——商家不关心这些的，他只和向他掏钱的外卖小哥打交道。最终是谁在消费都被代理掉了。当然，代理也分透明和不透明。反向代理呢？超市更类似反向代理，我们都会亲自去逛超市，但是我们不必太去关心我们买到的商品从原料到加工到批发的复杂过程，和之前“我们”被代理掉相反，超市把提供这些商品的过程全都“代理”掉了。

## nginx反向代理

- load balance
- multiple services

# How To Install Nginx

以下操作在Centos7.4的机器上完成 对CentOS 7都适用。

## Step One—Add Nginx Repository

```bash
sudo yum install epel-release
```

## Step Two-Install Nginx

```bash
sudo yum install nginx
```

## Step Three—Start Nginx

```bash
sudo systemctl start nginx
```

ruanyf大神的这篇[blog](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)很好的解释了为什么要推荐这种启动方式（systemd）而不是直接init进程。

如果你的机器上有firewall正在跑，需要额外步骤来允许http和https访问

```bash
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
```

# 基本操作

```bash
nginx -s signal
```

signal可以是以下几种之一：

- stop: 暴力的关闭
- quit: 温柔的关闭
- reload: 修改了配置文件之后很容易忘的东西
- reopen: 平时玩耍时基本没怎么用到（重新打开日志，其实是重置了log的写指针）

```bash
nginx -t
```
测试配置是否有问题。强烈建议修改配置文件后执行。

# nginx module

nginx的模块从结构上分为3种：

- 核心模块：HTTP模块，EVENT模块，MAIL模块。

- 基础模块：HTTPAccess模块，HTTP FastCGI模块，HTTP Proxy模块，HTTP Rewrite模块。

- 第三方模块：HTTP Upstream Hash模块，Notice模块，HTTP Access Key模块。

从功能上分为3种：

- Handles（处理器模块）：直接处理请求，并进行输出内容和修改headers信息，Handles处理器模块一般只能有一个。

- Filters（过滤器模块）：对其他处理器模块输出内容进行修改操作。最后由Nginx输出。

- Proxies（代理类模块）：与后端一些服务，比如FastCGI进行交互。实现服务代理和负载均衡等功能。

Nginx模块属于静态编译方式，启动Nginx后，会直接加载所有模块。


# 基本配置


```conf
# user字段表明了Nginx服务是由哪个用户哪个群组来负责维护进程的
# 我这里用了cainengtian用户，staff组来启动并维护进程
# 查看当前用户命令： whoami
# 查看当前用户所属组命令： groups ，当前用户可能有多个所属组，选第一个即可
user nginx;
# worker_processes字段表示Nginx服务占用的内核数量
# 为了充分利用服务器性能你可以直接写你本机最高内核
# 查看本机最高内核数量命令： sysctl -n hw.ncpu
worker_processes auto;
# error_log字段表示Nginx错误日志记录的位置
# 模式选择：debug/info/notice/warn/error/crit
# 上面模式从左到右记录的信息从最详细到最少
error_log /var/log/nginx/error.log;
# Nginx执行的进程id,默认配置文件是注释了
# 如果上面worker_processes的数量大于1那Nginx就会启动多个进程
# 而发信号的时候需要知道要向哪个进程发信息，不同进程有不同的pid，所以写进文件发信号比较简单
# 你只需要手动创建，比如我下面的位置： touch /usr/local/var/run/nginx.pid
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    # 每一个worker进程能并发处理的最大连接数
    # 当作为反向代理服务器，计算公式为： `worker_processes * worker_connections / 4`
    # 当作为HTTP服务器时，公式是除以2
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    # 在一个数据包里发送所有头文件，而不是一个接一个的发送
    tcp_nopush          on;
    # 不要缓存
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

# Settings for a TLS enabled server.
#
#    server {
#        listen       443 ssl http2 default_server;
#        listen       [::]:443 ssl http2 default_server;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```

# 配置反向代理

```conf
upstream atlas {
    server 127.0.0.1:9000;
    keepalive 64;
}

server {
    listen 80;
    server_name yucong.xyz;

    location / {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-Nginx-Proxy true;
        proxy_set_header Connection "";
        proxy_pass http://atlas;
    }
}
```

## Nginx proxy_set_header

允许重新定义或添加字段传递给代理服务器的请求头。

## upstream模块

作用：是用来定义服务器组的模块
将使nginx跨越单机的限制，完成网络数据的接收、处理和转发。

结构：

upstream groupName {

  server serverName1 [param1=value1] [param2=value2] [param3];

  server serverName2;

}

server的类型可以是：

1. 域名：如:webserver.website.com;

2. IP：如:192.168.0.239:80;

3. Unix套接字文件：如unix:/tmp/webserver;

# 配置临时跳转

``` conf
location /from-path/ {
    # 当匹配到{your-domain}/from-path的时候会跳转{your-domain}/to-path
    return 302 {your-domain}/to-path
}
```

# 配置访问限制

```conf
# 当匹配到/info的时候只允许x.x.x.x访问，其它的全部限制
# 同时改写为/info.php
location = /info {
    allow x.x.x.x;
    deny all;
    rewrite (.*) /info.php
}
```

rewrite功能就是，使用nginx提供的全局变量或自己设置的变量，结合正则表达式和标志位实现url重写以及重定向。rewrite只能放在server{},location{},if{}中，并且只能对域名后边的除去传递的参数外的字符串起作用。语法rewrite regex replacement [flag];

# default-ssl 配置
我们都知道HTTP在传输的过程中都是明文的，这直接导致了在传输的任何一个过程中都容易被窃取信息，所以才有了SSL（安全套接层）以及升级版TLS（传输层安全协议）的出现，其实就是在HTTP应用层给TCP/IP传输层的中间增加了TLS/SSL层，统称为HTTPS。

那如何通过Nginx配置HTTPS站点呢，下面就是default-ssl配置文件的内容

```conf
server {
    # 默认情况下HTTPS监听443端口
    listen  443 ssl;
    server_name  localhost;
    root  /var/www/;
    # 下面这些都是配置SSL需要的
    ssl on;
    # 下面两个字段需要的crt利用openssl生成，具体可以看[这里](http://nginx.org/en/docs/http/configuring_https_servers.html)
    ssl_certificate ssl/localhost.crt;
    ssl_certificate_key ssl/localhost.key;
    ssl_session_timeout 10m;
    ssl_protocols SSLv2 SSLv3 TLSv1;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    location = /info {
        allow 127.0.0.1;
        deny all;
        rewrite (.*) /info.php;
    }
    location /phpmyadmin/ {
        root /usr/local/share/phpmyadmin;
        index index.php index.html index.htm;
    }
    location / {
        include /usr/local/etc/nginx/conf.d/php-fpm;
    }
    error_page 403 /403.html;
    error_page 404 /404.html;
}
```

# 配置load balance

## 内置

Nginx负载均衡是通过upstream模块来实现的，内置实现了三种负载策略，配置还是比较简单的。官网负载均衡配置说明：http://nginx.org/en/docs/http/load_balancing.html

- 轮循(default)

Nginx根据请求次数，将每个请求均匀分配到每台服务器

- 最少连接 least_con

将请求分配给连接数最少的服务器。Nginx会统计哪些服务器的连接数最少。

- IP Hash ip_hash

绑定处理请求的服务器。第一次请求时，根据该客户端的IP算出一个HASH值，将请求分配到集群中的某一台服务器上。后面该客户端的所有请求，都将通过HASH算法，找到之前处理这台客户端请求的服务器，然后将请求交给它来处理

```config
http {

    # ... other config

    upstream myapp {
        # least_con
        # ip_hash
        server 192.168.0.100:8080;
        server 192.168.0.101:8080;
        server example.com:8080;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://myapp;
        }
    }

    # ... 省略其它配置
}
```

### server 参数：

- weight： 默认为1，将请求平均分配给每台server
- max_fails： 默认为1。某台Server允许请求失败的次数，超过最大次数后，在fail_timeout时间内，新的请求将不会分配给这台机器。如果设置为0，Nginx会将这台Server置为永久无效状态，然后将请求发给定义了proxy_next_upstream, fastcgi_next_upstream, uwsgi_next_upstream, scgi_next_upstream, and memcached_next_upstream指令来处理这次错误的请求。
- fail_timeout: 默认为10秒。某台Server达到max_fails次失败请求后，在fail_timeout期间内，nginx会认为这台Server暂时不可用，不会将请求分配给它
- backup: 备份机，所有服务器挂了之后才会生效
- down: 标识不可用（我不知道有啥用，可能有一些技术动态的激活？没用过。。。）
- max_conns: 限制分配给某台Server处理的最大连接数量，超过这个数量，将不会分配新的连接给它。默认为0，表示不限制
- resolve: 将server指令配置的域名，指定域名解析服务器。需要在http模块下配置resolver指令，指定域名解析服务

```config
upstream tomcats {
    server 192.168.0.100:8080 weight=2 max_fails=3 fail_timeout=15;
    server 192.168.0.101:8080 weight=3;
    server 192.168.0.102:8080 backup;
}
```

```config
http {
    resolver 10.0.0.1;

    upstream u {
        zone ...;
        ...
        server example.com resolve;
    }
}
```

## 第三方

- fair: 根据服务器的响应时间来分配请求，响应时间短的优先分配，即负载压力小的优先会分配。
- url_hash: 按请求url的hash结果来分配请求，使每个url定向到同一个后端服务器，服务器做缓存时比较有效。

基本用法：
1. Google and download source code.(wget xxx -> unzip xxx)
2. Compile nginx.(make)
3. Copy and replace old nginx. (ps aux | grep nginx -> kill -g {nginx pid} -> cp xxx yyy -> start nginx)