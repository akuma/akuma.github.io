---
title: 使用 Servlet Filter 实现页面缓存
date: 2012-09-18 14:08:21
tags: java servlet filter cache performance
---

最近在公司内部基础库中用 filter 实现了一个简单的页面缓存组件。和数据缓存相比，这种过滤器方式的缓存粒度更粗一些，但性能更好一些，特别适用于那些数据更新不是很频繁的页面。当然，这个组件适合小项目中使用，大型项目还是应该用 nginx 或者缓存中间件来处理页面缓存。

<!--more-->

在实际使用中，内容缓存可以和数据缓存搭配使用，这种使用方式其实很能够体现缓存应该是分层次的思想。例如缓存的层次可以如下：

```
浏览器缓存 -> Web 服务器缓存 -> 应用内容缓存 -> 应用数据缓存 -> 数据库缓存
```

## Spring 中需要添加的配置

```xml
<bean id="cacheFilter" class="demo.CacheFilter">
    <property name="time" value="43200" />
    <property name="cache" ref="cacheBean" />
    <property name="cacheManager" ref="cacheManager" />
</bean>
```

可用参数解释：

- time：缓存过期时间，以秒为单位
- cache：引用的缓存类
- cacheManager：引用的缓存管理类

## web.xml 中需要添加的配置

```xml
<filter>
    <filter-name>cacheFilter</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    <init-param>
        <param-name>targetFilterLifecycle</param-name>
        <param-value>true</param-value>
    </init-param>
    <init-param>
        <param-name>targetBeanName</param-name>
        <param-value>cacheFilter</param-value>
    </init-param>
    <init-param>
        <param-name>lastModified</param-name>
        <param-value>initial</param-value>
    </init-param>
    <init-param>
        <param-name>expires</param-name>
        <param-value>14400</param-value>
    </init-param>
    <init-param>
        <param-name>max-age</param-name>
        <param-value>14400</param-value>
    </init-param>
    <init-param>
        <param-name>enable</param-name>
        <param-value>true</param-value>
    </init-param>
    <init-param>
        <param-name>pattern</param-name>
        <param-value>^/cachedPage.htm$</param-value>
    </init-param>
</filter>

<filter-mapping>
    <filter-name>cacheFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

可用参数解释：

- lastModified：是否在响应中设置 Last-Modified 头，可选值：off（不添加）、on（交由其他模块处理）、initial（系统自动生成）
- expires：是否在响应中设置 Expires 头，可选值：off（不添加）、on（交由其他模块处理）、time（具体的过期时间，以秒为单位）
- max-age：是否在响应中设置 Cache-Control 头，具体的过期时间，以秒为单位
- enable：是否开启缓存过滤器
- pattern：允许被缓存的页面的 URI 正则表达式

上述配置中，只针对页面 `/cachedPage.htm` 的内容进行缓存，并且设置 `Last-Modified` 由系统自动生成，客户端缓存有效时间为 4 小时。
