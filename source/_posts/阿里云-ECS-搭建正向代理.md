---
title: 阿里云 ECS 搭建正向代理
date: 2016-12-03 12:30:06
tags: 阿里云 ecs proxy
---

目前公司的产品采用了多个 ECS 加 SLB 的方式负载均衡部署。出于节省成本和安全方面的考虑，我们只有一台作为 CDN 源站的 ECS 开通了外网带宽。

<!--more-->

由于业务上有调用互联网 API 的需求（比如调用微信开放平台的接口），我用 Node.js 写了一个简单的 HTTP 正向代理，部署在开通外网带宽的 ECS 上。其他 ECS 调用互联网 API 时均通过这个代理进行，功能上虽然满足了但是性能不是很好。

为了解决这个问题，我今天换了一种实现方案：在开通外网带宽的 ECS 上安装 `Squid` 实现正向代理。

## 安装、初始化和启动 Squid

```
sudo yum -y squid
sudo squid -z
sudo /etc/init.d/squid start
```

## 测试 Squid

在没有外网带宽的 ECS 添加配置文件 http-proxy.sh：

```
sudo vi /etc/profile.d/http-proxy.sh
source /etc/profile.d/http-proxy.sh
```

文件内容如下：

```sh
#!/bin/sh

export http_proxy=http://proxy.server:3128
export https_proxy=http://proxy.server:3128
```

其中的 `proxy.server` 是作为 HTTP 代理的 ECS 服务器名称。

```
curl -v http://www.douban.com/
```

测试结果：

```
* About to connect() to proxy proxy.server port 3128 (#0)
*   Trying xxx.xxx.xxx.xxx...
* Connected to proxy.server (xxx.xxx.xxx.xxx) port 3128 (#0)
> GET http://www.douban.com/ HTTP/1.1
> User-Agent: curl/7.29.0
> Host: www.douban.com
> Accept: */*
> Proxy-Connection: Keep-Alive
>
* HTTP 1.0, assume close after body
< HTTP/1.0 301 Moved Permanently
< Date: Sat, 03 Dec 2016 05:49:58 GMT
< Content-Type: text/html
< Content-Length: 178
< Location: https://www.douban.com/
< Server: dae
< X-Cache: MISS from proxy.server
< X-Cache-Lookup: MISS from proxy.server:3128
< Via: 1.0 proxy.server (squid/3.1.23)
* HTTP/1.0 connection set to keep alive!
< Connection: keep-alive
<
<html>
<head><title>301 Moved Permanently</title></head>
<body bgcolor="white">
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>
* Connection #0 to host proxy.server left intact
```

从结果中可以看出请求时通过 `proxy.server` 服务器完成的。
