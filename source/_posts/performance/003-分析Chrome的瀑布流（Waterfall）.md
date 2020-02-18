---
title: 003-分析Chrome的瀑布流（Waterfall）
categories:
  - performance
abbrlink: 961373d8
date: 2020-02-13 15:16:53
---

摘要：当需要调试网页或分析网站性能时，我们往往会F12打开浏览器控制台，查看网络请求，看网页加载了哪些资源，以及对应的请求方式（Method）、状态码（Status）、资源类型（Type）、大小（Size）、耗费的时间（Time）等。

如果某个资源耗费的时间比较长，需要深入分析时，则需要看：瀑布流（Waterfall），在Waterfall中可以看出时间具体花在了哪些部分。

<!-- maore -->

# 解说

打开用chrome console，可以看到如下

![](/images/post/performance-web/chrome-console.jpg)

可以看到页面加载的时间窗口。此时可以将鼠标放置 右侧的waterfall上，可以查看具体耗时

![](/images/post/performance-web/chrome-console2.jpg)

## 瀑布流中各项指标含义如下：
```text
Queueing：浏览器将资源放入队列时间，比如：遇到更高优先级的请求或请求并发超过6了。
Stalled：因放入队列时间而发生的停滞时间。
Proxy negotiation：与代理服务器协商时间。
DNS Lookup：DNS解析时间，浏览器需要将域名转换成IP。
Initial Connection：在浏览器发送请求前，需要建立HTTP连接的时间。
SSL：如果网站使用了HTTPS，这个就是浏览器与服务器建立安全性连接的时间。
Request sent：请求发送的时间。
Waiting (TTFB)：等待服务端返回数据的时间，这个时间受制于服务端处理性能。
Content Download：浏览器下载资源的时间，这个时间受制于文件大小和带宽。
```

如何优化，请参看：http://blog.bjlhx.top/categories/performance-web
