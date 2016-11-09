---
title: 使用 Servlet Filter 动态添加 html 代码
date: 2012-08-08 21:39:34
tags: java servlet filter http
---

最近一个同事问我，怎么快速实现在一个老系统的页面上统一添加用于页面性能分析的 JavaScript 脚本。确实，老系统这么多页面，如果一个一个去修改是非常费时而且笨的做法。

<!--more-->

我想了一下，写了一个 Servlet Filter，可以实现在 HTTP 响应内容中的某个节点（比如 `</body>` ）前添加 HTML 代码，优雅的解决了这个问题。

## 使用样例

### 插入 html 代码的类

```java
package demo;

/**
 * 继承 HtmlContentProvider，实现提供插入内容的方法。
 */
public class HtmlContentProviderImpl implements HtmlContentProvider {
    public String getContent(HttpServletRequest request) {
        return "<div style=\"display:none\">"
                + "<script src=\"http://s16.cnzz.com/stat.php?id=xxxxxxx&web_id=xxxxxxx\""
                + " language=\"javascript\"></script></div>";
    }
}
```

### Spring 配置

```xml
<bean id="htmlContentAppendFilter" class="demo.HtmlContentAppendFilter">
    <!-- 插入代码的节点名称，本例是 </head>，表示将代码添加到 </head> 节点之前 -->
    <property name="htmlNode" value="&lt;/head&gt;" />
    <!-- 在响应内容中查找节点时，是否采用反向查找的方式，本例采用正向查找 -->
    <property name="reverseLookup" value="false" />
    <!-- 定义需要进行过滤的 requestPath 的正则表达式，本例只过滤 .htm 后缀请求 -->
    <property name="includes" value=".+\.htm" />
    <!-- 定义需要忽略过滤的 requestPath 的正则表达式，本例排除对 ReplyAction 请求的过滤 -->
    <property name="excludes" value="/.+/remote.+" />
    <!-- 提供 HTML 代码的实现类，该内容会被添加到指定节点之前 -->
    <property name="htmlContentProvider">
        <bean class="demo.HtmlContentProviderImpl" />
    </property>
</bean>
```

### web.xml 配置

```xml
<filter>
    <filter-name>htmlContentAppendFilter</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    <init-param>
        <param-name>targetBeanName</param-name>
        <param-value>htmlContentAppendFilter</param-value>
    </init-param>
</filter>

<filter-mapping>
    <filter-name>htmlContentAppendFilter</filter-name>
    <url-pattern>*.htm</url-pattern>
</filter-mapping>
```

这样，对于所有的 `.htm` 请求，只要响应消息中包含了 `</head>` 节点，都会自动添加 `HtmlContentProviderImpl.getContent(request)` 中返回的内容，即一段 CNZZ 脚本。

## 性能和不足之处

在使用此过滤器和不使用此过滤器的情况下，对常规的页面（1M 以内）做了压力测试，发现吞吐量相差不大，基本可以忽略添加过滤器的影响。对于数据量比较大的页面，比如超过 1M 的页面，那么可能比较服务器消耗内存，同时在查找节点时速度也会慢一些，从而对页面性能造成一些影响。从另外一方面来说，出于性能的考虑，也应该尽可能保持页面小一些，超过 100K 的页面都值得考虑是否只能这么大。
