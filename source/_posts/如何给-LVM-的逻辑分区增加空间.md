---
title: 如何给 LVM 的逻辑分区增加空间
date: 2012-02-27 22:22:19
tags: linux lvm
---

使用 LVM 的好处就是可以在不重启系统的前提下方便的动态调整 Linux 分区大小，这对服务器来特别好用。下面我以工作中用到的一台开发服务器 192.168.0.222 为例进行说明。

<!--more-->

先用 `df -h` 命令查看磁盘分区使用情况：

```
$ df -h
```

显示结果如下：

```
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--builder-root
                      220G  690M  208G   1% /
none                  1.8G  188K  1.8G   1% /dev
none                  1.9G     0  1.9G   0% /dev/shm
none                  1.9G   72K  1.9G   1% /var/run
none                  1.9G     0  1.9G   0% /var/lock
/dev/mapper/nvidia_acfacaaf1
                      228M   64M  153M  30% /boot
/dev/mapper/ubuntu--builder-tmp
                      2.0G   90M  1.8G   5% /tmp
/dev/mapper/ubuntu--builder-usr
                      9.9G  4.1G  5.3G  44% /usr
/dev/mapper/ubuntu--builder-home
                      9.9G  1.3G  8.2G  14% /home
/dev/mapper/ubuntu--builder-opt
                       99G   93G  992M  99% /opt
/dev/mapper/ubuntu--builder-var
                      9.9G  2.2G  7.3G  23% /var
```

我们发现 /opt 分区只有 99% 了，需要马上增大空间。
出于安全考虑，最好将所有相关的服务停止之后再操作：

```
$ sudo /etc/init.d/mysql stop
$ sudo /etc/init.d/proftpd stop
$ sudo /etc/init.d/apache stop
$ sudo /etc/init.d/bind9 stop
$ killall -9 java
```

之后执行如下命令：

```
$ sudo lvextend -L200G /dev/ubuntu-builder/opt
```

我们先将 /opt 扩展到 200G，然后我们需要将 /opt `umount` 之后再调整大小：

```
$ sudo umount /dev/ubuntu-builder/opt
$ sudo resize2fs /dev/ubuntu-builder/opt
$ sudo mount /dev/ubuntu-builder/opt /opt
```

最后我们再运行 `df -h` 命令查看，结果显示如下：

```
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--builder-root
                      220G  690M  208G   1% /
none                  1.8G  188K  1.8G   1% /dev
none                  1.9G     0  1.9G   0% /dev/shm
none                  1.9G   72K  1.9G   1% /var/run
none                  1.9G     0  1.9G   0% /var/lock
/dev/mapper/nvidia_acfacaaf1
                      228M   64M  153M  30% /boot
/dev/mapper/ubuntu--builder-tmp
                      2.0G   90M  1.8G   5% /tmp
/dev/mapper/ubuntu--builder-usr
                      9.9G  4.1G  5.3G  44% /usr
/dev/mapper/ubuntu--builder-home
                      9.9G  1.3G  8.2G  14% /home
/dev/mapper/ubuntu--builder-opt
                      197G   93G   95G  50% /opt
/dev/mapper/ubuntu--builder-var
                      9.9G  2.2G  7.3G  23% /var
```

成功了！

最后把停止的服务重新启动即可：

```
$ sudo /etc/init.d/mysql start
$ sudo /etc/init.d/proftpd start
$ sudo /etc/init.d/apache start
$ sudo /etc/init.d/bind9 start
$ $CATALINA_HOME/bin/tm start
```
