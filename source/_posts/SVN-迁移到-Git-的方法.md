---
title: SVN 迁移到 Git 的方法
date: 2012-01-08 15:44:18
tags: git svn
---

工作中需要将一个老的 SVN 库迁移到 Git，网上收集了一些操作方法，总结出来备忘一下。

<!--more-->

## 创建 user.txt

通过以下命令获得 Subversion 代码库的作者列表：

```
$ svn log --xml | grep author | sort -u | perl -pe 's/.>(.?)<./$1 = /'
```

在此输出结果的基础之上，创建 `user.txt` 文件，将 Subversion 中的用户映射到 Git 中的提交者。为 `git svn` 提供该文件可以使它更精确的映射作者数据，例如：

```
akuma = Huang Weijia <ihuangwj@gmail.net>
```

## 执行 git svn clone 命令

在 `git svn clone` 后面添加 `--no-metadata` 来阻止 `git svn` 包含那些 Subversion 的附加信息。
可以指定 `-s` 参数来告诉 Git 该 Subversion 仓库遵循了基本的分支和标签命名法则，即：`trunk/branches/tags`

示例：

```
$ git svn clone http://server/svn/repos/demo/ --authors-file=user.txt --no-metadata -s demo
```

## clone 后的清理工作

最后一步要清理一下 `git svn` 创建的那些怪异的索引结构。

首先要移动标签，把它们从奇怪的远程分支变成实际的标签，然后把剩下的分支移动到本地。进入到刚刚 clone 的项目跟目录下执行如下命令：

```
$ cp -Rf .git/refs/remotes/tags/* .git/refs/tags/
$ rm -Rf .git/refs/remotes/tags
```

该命令将原本以 `tag/` 开头的远程分支的索引变成真正的（轻巧的）标签。

接下来，把 `refs/remotes` 下面剩下的索引变成本地分支：

```
$ cp -Rf .git/refs/remotes/* .git/refs/heads/
$ rm -Rf .git/refs/remotes
```

现在所有的旧分支都变成真正的 Git 分支，所有的旧标签也变成真正的 Git 标签。

最后一项工作就是把新建的 Git 服务器添加为远程服务器并且向它推送。下面是新增远程服务器的例子：

```
$ git remote add origin git@server:demo.git
```

为了让所有的分支和标签都得到上传，使用这条命令：

```
$ git push origin --all
```

现在，所有的分支和标签都应该整齐干净的躺在新的 Git 服务器里了。

## 用户授权

生成公钥，然后添加到 Git 服务器的 `authorized_keys` 文件中，示例命令如下：

```
$ cat /tmp/id_rsa.akuma.pub >> ~/.ssh/authorized_keys
```

关于公钥认证参考：[SSH 公钥认证使用指南](../../../../2010/11/01/SSH-公钥认证使用指南/)

## 参考资料

[Pro Git](http://progit.org/book/zh/)
