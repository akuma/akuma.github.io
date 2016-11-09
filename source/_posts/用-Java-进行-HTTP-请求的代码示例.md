---
title: 用 Java 进行 HTTP 请求的代码示例
date: 2009-06-16 17:51:35
tags: java http
---

这里只以 GET 请求为例，对比分别使用 JDK 和 Commons HttpClient 实现的代码。

<!--more-->

采用 JDK API 的方式：

```java
InputStream in = null;

try {
    URLConnection connection = new URL(url).openConnection();
    connection.setConnectTimeout(1000 * 3); // 设置连接超时时间: 3s
    connection.setReadTimeout(1000 * 3); // 设置读取超时时间: 3s
    connection.connect();

    in = connection.getInputStream();

    // 获取 HTTP 响应结果
    int byteRead;
    byte[] buffer = new byte[1024 * 4];
    StringBuilder stringBuilder = new StringBuilder();
    while ((byteRead = in.read(buffer)) != -1) {
        stringBuilder.append(new String(buffer, 0, byteRead));
    }
    String response = stringBuilder.toString();
}
catch (IOException e) {
    // 出错处理代码...
}
finally {
    // 关闭输入流
    FileUtils.close(in);
}
```

采用 Commons HttpClient 类库的方式：

```java
HttpClient client = new HttpClient();
client.getHttpConnectionManager().getParams().setConnectionTimeout(1000 * 3); // 设置连接超时时间: 3s
client.getHttpConnectionManager().getParams().setSoTimeout(1000 * 3); // 设置读取超时时间: 3s

HttpMethod method = new GetMethod(url); // 以 GET 方式请求
method.setQueryString("id=1"); // 设置请求参数
client.executeMethod(method);

// 获取 HTTP 响应结果
String response = method.getResponseBodyAsString();

method.releaseConnection(); // 关闭连接
```
