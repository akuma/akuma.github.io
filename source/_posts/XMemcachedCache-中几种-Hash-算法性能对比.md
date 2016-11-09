---
title: XMemcachedCache 中几种 Hash 算法性能对比
date: 2011-05-26 19:34:01
tags: cache memcached performance hash
---

## 测试用例

在一台 Memcached 服务器上开启了 3 个不同端口的进程，Memcached 客户端以相同的权重访问这三个进程。

测试客户端使用 2000 个并发线程，其中 1000 个执行 PUT 操作，另外 1000 个执行 GET 操作，每个线程中的 PUT 和 GET 操作均执行 1000 次。存入缓存的值为长度在 20 到 256 之间的随机字符串。

### 机器配置表

| 类型             | CPU                                            | 内存  | IP            |
|:---------------:|:----------------------------------------------:|:-----:|:-------------:|
| Memcached 服务端 | Intel(R) Pentium(R) 4 CPU 1.80GHz              | 1G    | 192.168.1.174 |
| 测试客户端        | Intel(R) Pentium(R) Dual-Core CPU E2210 2.2GHz | 2.75G | 192.168.0.68 |

### Memcached 进程启动参数

```
/usr/local/memcached/bin/memcached -u root -m 16 -l 192.168.1.174 -p 11212 -d -P /root/memcached_1.pid
/usr/local/memcached/bin/memcached -u root -m 16 -l 192.168.1.174 -p 11213 -d -P /root/memcached_2.pid
/usr/local/memcached/bin/memcached -u root -m 16 -l 192.168.1.174 -p 11214 -d -P /root/memcached_3.pid
```

## 测试结果

| 批次 | 通讯协议 | 哈希算法   | 200并发耗时 (ms) | 2000并发耗时 (ms) |
|:---:|:-------:|:---------:|:---------------:|:------:|
| 1   | Binary  | Native    | 1547            | 127719 |
| 2   | Binary  | Native    | 1546            | 128313 |
| 3   | Binary  | Native    | 1531            | 127465 |
| 4   | Binary  | Ketama    | 2046            | 164797 |
| 5   | Binary  | Ketama    | 2015            | 170344 |
| 6   | Binary  | Ketama    | 2063            | 167788 |
| 7   | Binary  | Election  | 2625            | 238531 |
| 8   | Binary  | Election  | 2718            | 236359 |
| 9   | Binary  | Election  | 2641            | 232672 |
| 10  | Text    | Native    | 1578            | 133639 |
| 11  | Text    | Native    | 1547            | 137562 |
| 12  | Text    | Native    | 1594            | 142497 |
| 13  | Text    | Ketama    | 2016            | 177922 |
| 14  | Text    | Ketama    | 2110            | 170917 |
| 15  | Text    | Ketama    | 2078            | 165625 |
| 16  | Text    | Election  | 2735            | 244797 |
| 17  | Text    | Election  | 2859            | 231329 |
| 18  | Text    | Election  | 2781            | 243453 |

## 测试结果分析

- 毫无疑问，Native Hash（即使用 Java 中的 hashCode() 方法生成），在性能上是最优的，但是在多节点的情况下，增加、删除节点后，所有缓存全部会失效。
- Election Hash 因为对于每个节点都要进行一次 Hash 值计算来选举出可以存放内容的节点，所以在处理时间上会比 Ketama Hash 更耗时。
- 在高并发、大批量操作下，二进制协议比文本协议在性能上会更优一些，但不是非常明显。

## 结论

- 在只有一个 Memcached 节点的情况下，可以使用 Native Hash，这样性能最好。
- 在有多个 Memcached 节点的情况下，一般优先采用 Ketama Hash，这样缓存一致性可以保证，性能也优于 Election Hash。
- 一般来说倾向于优先使用二进制协议，性能上比文本协议好一些。
