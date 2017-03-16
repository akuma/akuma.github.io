---
title: sed 和 awk 学习
date: 2017-03-11 22:59:47
tags: awk sed linux shell
---

最近工作中经常会涉及到一些日志处理，这个时候就体现出 sed、awk 这两个命令行工具的强大威力了。
书到用时方恨少，所以最近在看《sed & awk》这本书来补充知识。这篇博客就是学习笔记。

<!-- more -->

## sed

sed 是 `Stream EDitor` 的缩写，它是一个 **非交互式** 的、**面向字符流** 的编辑器。（例如 vim 则正好和 sed 相反）。
这两个特点使 sed 成为命令行下非常有用的一个处理工具。

sed 的典型用法是在一个或多个文件上自动实现编辑操作。

## awk

awk 是一种 **模式匹配** 的编程语言，也是一个 **面向字符流** 的命令行工具，它编写于 1977 年。
awk 的名称是三个主要作者的姓的首字母缩写：Drs. A. Aho、P. Weinberger 和 B. Kernighan。

awk 非常适合处理那些拥有某种结构的数据，比如将某种结构的数据转换成格式化的报表。


### AWK 编程语言的基础知识

记录、字段和规则

#### 了解规则

```
/pattern/ { action }
```

#### 使用 BEGIN 和 END 模式

#### 一些基本的操作

- exit：停止程序的执行，并且退出。
- next：停止处理当前记录，并且前进到下一条记录。
- nextfile：停止处理当前文件，并且前进到下一个文件。
- print：打印使用引号括起来的文本、记录、字段和变量。（缺省情况下是打印出当前整行记录。）
- printf：打印格式化文本，类似于它的 C 语言对等成分，但必须指定结尾的换行。
- sprintf：返回格式化文本字符串，与 printf 使用相同的格式。

### 运行 AWK 的不同方式

在命令行中、作为筛选器、从文件或作为脚本。

#### 命令行

```
awk 'program' file
```

```
awk '{ print }' sample
```

#### 作为筛选器

```
awk 'program'
```

```
$ cat sample | awk '{ print }'
```

#### 从文件

```
{ program }
```

```
$ awk -f progfile sample
```

#### 作为脚本

```$ cat > awkat#!/usr/bin/awk -f{ print }Ctrl-d
$ chmod u+x awkat
$ ./awkat sample
```

### 操作记录和字段

```
# 打印特定的字段
awk ' { print $1 } ' sample
awk ' { print $7, $3, $1 } ' sample
awk ' { print $7 $8 } ' sample
awk ' { print "Field 2: " $2 } ' sample

# 更改字段分隔符
awk ' BEGIN { FS = "!" } { print $1 } ' sample
awk ' BEGIN { FS = "Heigh-ho" } { print $2 } ' sample
awk ' BEGIN { FS = "[Hh]eigh-ho" } { print $2 } ' sample
awk -F ":" ' { print $5 } ' /etc/passwd

# 更改记录分隔符
awk ' BEGIN { RS = "," } // ' sample

# 更改输出
awk ' BEGIN { ORS = "" } // { print } END { print "\n" } ' sample
awk ' END { print "Input contains " NR " lines." } ' sample
```

- NF：该变量包含每个记录的字段个数。
- NR：该变量包含当前的记录个数。
- FS：该变量是字段分隔符。
- RS：该变量是记录分隔符。
- OFS：该变量是输出字段分隔符。
- ORS：该变量是输出记录分隔符。
- FILENAME：该变量包含所读取的输入文件的名称。
- IGNORECASE：当 IGNORECASE 设置为非空值，GAWK 将忽略模式匹配中的大小写。

### 模式匹配

```
# 在输入记录中匹配模式
awk '/green/ { print }' sample
awk '/!.*!/' sample

# 在字段中匹配模式
awk '$3 ~ /s/' sample

# 使用布尔操作符
awk '/in/ && /the/ && /to/ {print $4}' sample
awk 'BEGIN { OFS="-" } ! /holly/ || /most/ { print $1, $2 }' sample

# 使用范围模式
awk '/Heigh/,/folly/' sample

# 输出匹配的整段内容
awk 'BEGIN { FS="\n"; RS="" } /green/' sample
```
