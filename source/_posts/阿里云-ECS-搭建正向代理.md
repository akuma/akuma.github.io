---
title: 阿里云 ECS 搭建正向代理
date: 2016-12-03 12:30:06
tags: 阿里云 ecs proxy
---

目前公司的产品采用了多个 ECS 加 SLB 的方式负载均衡部署。出于节省成本和安全方面的考虑，我们只有一台作为 CDN 源站的 ECS 开通了外网带宽。

<!--more-->

由于业务上有调用互联网 API 的需求（比如调用微信开放平台的接口），我用 Node.js 写了一个简单的 HTTP API 代理，部署在开通外网带宽的 ECS 上。其他 ECS 上的业务系统调用互联网 API 时均通过这个代理进行，功能上虽然满足了但是不够通用。

为了解决这个问题，我今天换了一种实现方案：在开通外网带宽的 ECS 上安装 `Squid` 实现正向代理，让业务系统通过 Squid 请求互联网 API。选用 Squid 的原因是它是一个高性能的代理缓存服务器，支持 FTP、gopher 和 HTTP 协议，它使用一个单独的、非模块化的、I/O 驱动的进程来处理所有的客户端请求。

## 正向代理的安装

### 安装并启动 Squid

```
$ sudo yum -y squid                  # 安装
$ sudo squid -z                      # 初始化
$ sudo /etc/init.d/squid start       # 启动
```

### 测试 Squid

在没有外网带宽的 ECS 添加配置文件 http-proxy.sh：

```
$ sudo vi /etc/profile.d/http-proxy.sh
$ source /etc/profile.d/http-proxy.sh
```

文件内容如下：

```sh
#!/bin/sh

export http_proxy=http://proxy.mydomain.com:3128
export https_proxy=http://proxy.mydomain.com:3128
```

其中的 `proxy.mydomain.com` 是作为 HTTP 代理的 ECS 服务器名称。

```
$ curl -v http://www.douban.com/
```

测试结果：

```
* About to connect() to proxy proxy.mydomain.com port 3128 (#0)
*   Trying xxx.xxx.xxx.xxx...
* Connected to proxy.mydomain.com (xxx.xxx.xxx.xxx) port 3128 (#0)
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
< X-Cache: MISS from proxy.mydomain.com
< X-Cache-Lookup: MISS from proxy.mydomain.com:3128
< Via: 1.0 proxy.mydomain.com (squid/3.1.23)
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
* Connection #0 to host proxy.mydomain.com left intact
```

从结果中可以看出请求时通过 `proxy.mydomain.com` 服务器完成的。

## 应用端的配置

### Java 应用的配置

对于 Java 应用，最简单的方式是修改 JRE 安装目录下的网络配置文件 `lib/net.properties`，启用系统代理：

```properties
java.net.useSystemProxies=true
http.nonProxyHosts=localhost|127.*|10.*.*.*
```

配置项 `http.nonProxyHosts` 的目的是对本地接口的调用不通过代理。

比上述方式更灵活的方式是在应用的启动命令上添加相关参数：

```
$ java -Dhttp.proxyHost=proxy.mydomain.com \
  -Dhttp.proxyPort=3128 \
  -Dhttps.proxyHost=proxy.mydomain.com \
  -Dhttps.proxyPort=3128 \
  -Dhttp.nonProxyHosts="localhost|127.*|10.*.*.*" \
  ...
```

最灵活的方式是在程序中指定代理，示例代码如下：

```java
import java.net.InetSocketAddress;
import java.net.Proxy;
import java.net.SocketAddress;
import java.net.URL;
import java.net.URLConnection;


// 创建一个 HTTP proxy 对象
SocketAddress addr = new InetSocketAddress("proxy.mydomain.com", 3128);
Proxy proxy = new Proxy(Proxy.Type.HTTP, addr);

// 使用代理请求
URL url = new URL("http://foo.example.com/");
URLConnection conn = url.openConnection(proxy);

// 不使用代理请求
URL url2 = new URL("http://bar.example.com/");
URLConnection conn2 = url2.openConnection(Proxy.NO_PROXY);
```

### Node.js 应用的修改

对于 Node.js 应用，如果使用了 `http` 进行接口调用，则代码类似如下：

```js
const options = {
  host: 'proxy.mydomain.com', // 设置为代理服务器的域名
  port: 3128,               // 设置为代理服务器的端口
  path: apiUrl,             // 真正调用的接口地址
}
const request = http.request(options, (res) => {
  let str = ''
  res.on('data', (chunk) => {
    str += chunk
  })
  res.on('end', () => {
    console.log('No more data in response.')
  })
})
request.on('error', (e) => {
  console.log(`problem with request: ${e.message}`)
})
request.end()
```
