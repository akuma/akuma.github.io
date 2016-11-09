---
title: 使用 include 指令简化 Nginx 配置
date: 2013-11-02 16:03:59
tags: nginx
---

在查看公司生产环境的 nginx 配置文件时，经常可以看到大段重复的代码，导致修改一个相同的配置，往往要同时修改好几个地方（比如使用了多个虚拟主机的情况），可维护性非常差。

<!--more-->

上述问题其实可以很简单的通过 include 指令来简化配置文件，下面就通过一个配置文件的例子来说明如何简化配置。

## 简化前的配置 conf/nginx.conf

```nginx
user              nobody nobody;
worker_processes  4;

pid               logs/nginx.pid;
error_log         logs/error.log error;

events {
    use                 epoll;
    worker_connections  1024;
}

http {
    include           mime.types;
    include           proxy.conf;

    sendfile          on;
    tcp_nopush        on;
    tcp_nodelay       off;
    keepalive_timeout 0;
    default_type      application/octet-stream;

    upstream backend {
        server        192.168.1.188:8080 srun_id=c1 weight=1;
        jvm_route     $cookie_JSESSIONID reverse;
    }

    server {
        listen        80;
        server_name   foo.com;
        index         index.html index.htm;
        access_log    logs/access.log;

        location / {
            proxy_pass http://backend;
            proxy_redirect              off;
            proxy_set_header            Host            $host;
            proxy_set_header            X-Real-IP       $remote_addr;
            proxy_set_header            X-Forwarded-For $proxy_add_x_forwarded_for;
            client_max_body_size        50M;
            client_body_buffer_size     256k;
            proxy_connect_timeout       600;
            proxy_send_timeout          300;
            proxy_read_timeout          300;
            proxy_buffer_size           4k;
            proxy_buffers               4 32k;
            proxy_busy_buffers_size     64k;
            proxy_temp_file_write_size  64k;
        }

        location /status {
            stub_status    on;
            access_log     off;
            allow          all;
            deny           all;
        }
    }
}
```

## 使用 include 指令简化配置文件

### 抽取 proxy 设置到单独文件中（`conf/proxy.conf`）

```nginx
proxy_redirect              off;
proxy_set_header            Host            $host;
proxy_set_header            X-Real-IP       $remote_addr;
proxy_set_header            X-Forwarded-For $proxy_add_x_forwarded_for;
client_max_body_size        50M;
client_body_buffer_size     256k;
proxy_connect_timeout       600;
proxy_send_timeout          300;
proxy_read_timeout          300;
proxy_buffer_size           4k;
proxy_buffers               4 32k;
proxy_busy_buffers_size     64k;
proxy_temp_file_write_size  64k;
```

### 抽取 status 设置到单独文件中（`conf/status.conf`）

```nginx
location /status {
    stub_status    on;
    access_log     off;
    allow          all;
    deny           all;
}
```

### 抽取杂项配置到单独文件中（`conf/misc.conf`）

```nginx
sendfile          on;
tcp_nopush        on;
tcp_nodelay       off;
keepalive_timeout 0;
default_type      application/octet-stream;
```

### 最终简化后的配置

```nginx
user              nobody nobody;
worker_processes  4;

pid               logs/nginx.pid;
error_log         logs/error.log error;

events {
    use                 epoll;
    worker_connections  1024;
}

http {
    include   mime.types;
    include   proxy.conf;
    incluee   misc.conf;

    upstream backend {
        server      192.168.1.188:8080 srun_id=c1 weight=1;
        jvm_route   $cookie_JSESSIONID reverse;
    }

    server {
        listen      80;
        server_name foo.com;
        charset     utf-8;
        index       index.html index.htm;
        access_log  logs/access.log;

        location / {
            proxy_pass http://backend;
        }

        include     status.conf;
    }
}
```
