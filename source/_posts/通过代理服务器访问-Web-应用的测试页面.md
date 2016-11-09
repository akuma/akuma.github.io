---
title: 通过代理服务器访问 Web 应用的测试页面
date: 2009-11-04 11:32:02
tags: web proxy test
---

通常基于性能方面的考虑，会采用动静态页面分离、负载均衡等方式部署 Web 应用。而这些方式都会用到代理服务器，比如 nginx、apache、varnish 等等。

<!--more-->

以下是一个做此类代理配置时方便问题调试的 JSP 页面。

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" %>
<%@ page import="java.util.Enumeration" %>

<h1>This is a page for testing proxy webapp.</h1>
<h3>Webapp context path: <%=request.getContextPath()%></h3>

<b>sessionId:</b> <%=request.getSession().getId()%><br />
<b>serverName:</b> <%=request.getServerName()%><br />
<b>remoteAddr:</b> <%=request.getRemoteAddr()%><br />
<b>realRemoteAddr:</b> <%=getRealRemoteAddr(request)%><br />

<h3>Request Headers</h3>
<%
Enumeration<String> em = request.getHeaderNames();
while (em.hasMoreElements()) {
    String name = em.nextElement();
    StringBuilder sb = new StringBuilder();
    Enumeration<String> e = request.getHeaders(name);
    while (e.hasMoreElements()) {
        sb.append(e.nextElement() + " | ");
    }
    out.println("<b>" + name + ":</b> " + sb + "<br />");
}
%>

<%!
String getRealRemoteAddr(HttpServletRequest request) {
    String remoteAddr = request.getHeader("x-forwarded-for");

    if (remoteAddr == null || remoteAddr.trim().length() == 0 || "unknown".equalsIgnoreCase(remoteAddr)) {
        remoteAddr = request.getHeader("x-real-ip");
    }

    if (remoteAddr == null || remoteAddr.trim().length() == 0 || "unknown".equalsIgnoreCase(remoteAddr)) {
        remoteAddr = request.getHeader("Proxy-Client-IP");
    }

    if (remoteAddr == null || remoteAddr.trim().length() == 0 || "unknown".equalsIgnoreCase(remoteAddr)) {
        remoteAddr = request.getHeader("WL-Proxy-Client-IP");
    }

    if (remoteAddr == null || remoteAddr.trim().length() == 0 || "unknown".equalsIgnoreCase(remoteAddr)) {
        remoteAddr = request.getRemoteAddr();
    }

    if (remoteAddr != null && remoteAddr.trim().length() != 0) {
        String[] remoteAddrs = remoteAddr.split(",");
        remoteAddr = remoteAddrs[0].trim();
    }

    return remoteAddr;
}
%>
```
