---
title: 如何禁用不常用的 HTTP Method
date: 2011-09-01 15:07:42
tags: http
---

## 目的

出于安全性的考虑，我们想要屏蔽掉客户端以 OPTIONS、TRACE、HEAD、DELETE、PUT 等方式请求 Web 应用，只保留最常用的 GET、POST。

<!--more-->

## 实现方式

### 修改 nginx 配置文件

nginx 里设置这个比较简单，只需要在 `location` 节点中添加如下配置即可：

```nginx
limit_except GET POST {
  deny   all;
}
```

如果想要对个别 IP 开放，可以这样设置：

```nginx
limit_except GET POST {
  allow  192.168.1.0/32; # 对客户端 192.168.1.0 - 192.168.1.32 开放
  deny   all;
}
```

> 具体可以参考 nginx 文档：http://wiki.nginx.org/NginxHttpCoreModule#limit_except

这种方式适用于应用系统是通过 nginx 做反向代理访问的情况，同时后端的 tomcat 不暴露外网 IP。

经过测试，对于只要允许 GET、POST 请求，HEAD 请求也是支持的，所以这种方式无法在允许 GET、POST 的前提下屏蔽 HEAD。

### 修改 tomcat（或应用系统）的 web.xml

如果是 tomcat 直接暴露外网 IP 的情况，那么只能通过修改 `web.xml` 来实现屏蔽相关 method 的请求。
在 `web.xml` 中添加如下代码：

```xml
<security-constraint>
    <web-resource-collection>
        <web-resource-name>disabledMethods</web-resource-name>
        <url-pattern>/*</url-pattern>
        <http-method>TRACE</http-method>
        <http-method>OPTIONS</http-method>
        <http-method>HEAD</http-method>
        <http-method>PUT</http-method>
        <http-method>DELETE</http-method>
    </web-resource-collection>
    <auth-constraint />
</security-constraint>
```

这样应用系统就只会接受 GET、POST 方式的请求了。

经过测试，如果 `<security-constraint>` 中没有 `<auth-constraint>` 子元素的话，配置实际上是不起中用的。
如果加入了 `<auth-constraint>` 子元素，但是其内容为空，这表示所有身份的用户都被禁止访问相应的资源。
