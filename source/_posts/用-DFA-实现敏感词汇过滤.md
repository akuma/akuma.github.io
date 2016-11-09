---
title: 用 DFA 实现敏感词汇过滤
date: 2013-03-15 14:54:32
tags: dfa 词汇过滤
---

因为项目需要重写了一个简单的过滤敏感词汇的工具类，因为使用了 `DFA` 算法，所以比较高效。具体实现时在 DFA 算法基础上做了一些改进，如英文词汇的检测改造：单词 `have` 不会检测出 `av`，而单词 `av` 则能检测到。

<!--more-->

## 具体的使用方法

```java
String content = "敏感词汇"; // 这是需要过滤的文本
List<String> tabooedList = TabooedUtils.getTabooedWords(content); // 这是过滤出来的敏感词汇列表
```

另外，如果想自己定义敏感词汇，可以在项目的 src 下面建立一个名为 tabooed.words 的敏感词汇文件。文件内容为每行一个敏感词，例如：

```
zhuanfalun
bitch
三个代表
一党
多党
```

这样 `TabooedUtils` 就会使用自定义的词汇文件，而不用框架中默认的文件。

敏感词汇会在工具类 `TabooedUtils` 被 JVM 加载的时候载入并缓存。如果想临时修改词汇文件而又不想重启应用就生效，可以在程序中调用 reload 敏感词汇文件的方法，例如：

```
TabooedUtils.reloadTabooedWords(); // 重新加载 tabooed.words 文件
String content = "敏感词汇"; // 这是需要过滤的文本
List<String> tabooedList = TabooedUtils.getTabooedWords(content); // 这是过滤出来的敏感词汇列表
```

## 关于性能

根据初步的测试结果，在 1000 多敏感词汇的情况下，连续过滤 1378 字的文章 1000 次，耗时 200 ms 多。

## 参考资料

- [使用 DFA 实现文字过滤](http://www.iteye.com/topic/336577/)
- [维基百科 DFA](https://zh.wikipedia.org/wiki/dfa)
