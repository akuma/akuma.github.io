---
title: 关于 IE6 处理压缩 JavaScript 的 Bug
date: 2011-10-18 12:01:37
tags: javascript bug ie
---

## 问题描述

为了提高客户端 javascript 文件的加载速度，我们将所有的 javascript 代码通过 nginx、tomcat 等进行 gzip 压缩后再发给浏览器。这样原来可能 200 KB 多的脚本文件压缩后之只有 30 KB 多，浏览加载速度提高之后，用户体验就会更好。

但是最近工作中遇到了奇怪的问题，在 IE7、Firefox、Chrome 等浏览器下运行的非常良好的脚本到了 IE6 下有时候就会没有反应，需要刷新网页后才脚本才能运行。

<!--more-->

Google 之后发现微软的官方文档里对 IE6 有提到一点："Do not enable HTTP compression for the script files"。

## 解决办法

如果应用程序只采用了 `tomcat` 部署，那么可以通过配置 Connector，只关闭对 IE6 的 javascript 压缩。
以下是一个以 `Http11NioProtocol Connector` 为例的配置：

```xml
<Connector port="80" protocol="org.apache.coyote.http11.Http11NioProtocol"
           maxHttpHeaderSize="8192" redirectPort="8443"
           enableLookups="false" useBodyEncodingForURI="true"
           connectionTimeout="30000" disableUploadTimeout="true"
           maxThreads="500" acceptCount="1000"
           compression="on" compressionMinSize="2048"
           noCompressionUserAgents="gozilla, traviata, .*MSIE [1-6].*"
           compressableMimeType="text/html,text/xml,text/javascript,text/css,text/plain" />
```

如果应用程序采用了 `nginx + tomcat` 部署，那么可以通过以下配置只关闭对 IE6 的 javascript 压缩。

```nginx
gzip_types    text/plain text/css  text/xml application/xml application/xml+rss application/x-javascript text/javascript;
gzip_disable  ".*MSIE [1-6].*";
```

## 结论

虽然 IE6 在国外已经被埋葬了，但国内仍然有大量的用户在使用它，做为 Web 开发人员的我们仍需要对它有足够的认识。
