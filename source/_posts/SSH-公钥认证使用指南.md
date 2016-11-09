---
title: SSH 公钥认证使用指南
date: 2010-11-01 21:27:59
tags: linux ssh
---

## 为什么要使用公钥认证

通常，通过 ssh 登录远程服务器时，使用密码认证，分别输入用户名和密码，两者和服务器端匹配就可以登录。<br>
但是密码认证有以下的缺点：

- 用户无法设置空密码（即使系统允许空密码，也会十分危险）。
- 密码容易被人偷窥或猜到。
- 服务器上的一个帐户若要给多人使用，则必须让所有使用者都知道密码，导致密码容易泄露，而且修改密码时必须通知所有人。

<!--more-->

而使用公钥认证则可以解决上述问题。

- 公钥认证允许使用空密码，省去每次登录都需要输入密码的麻烦。
- 多个使用者可以通过各自的密钥登录到系统上的同一个用户。

## 公钥认证的原理

所谓的公钥认证，实际上是使用一对加密字符串，一个称为公钥（public key），任何人都可以看到其内容，用于加密；
另一个称为密钥（private key），只有拥有者才能看到，用于解密。
通过公钥加密过的密文使用密钥可以轻松解密，但根据公钥来猜测密钥却十分困难。

ssh 的公钥认证就是使用了这一特性。服务器和客户端都各自拥有自己的公钥和密钥。为了说明方便，以下将使用这些符号。

客户端公钥	Ac
客户端密钥	Bc
服务器公钥	As
服务器密钥	Bs

在认证之前，客户端需要通过某种方法将公钥 Ac 登录到服务器上。

认证过程分为两个步骤:

1. 会话密钥（session key）生成
  1. 客户端请求连接服务器，服务器将 As 发送给客户端。
  2. 服务器生成会话ID(session id)，设为 p，发送给客户端。
  3. 客户端生成会话密钥(session key)，设为 q，并计算 r = p xor q。
  4. 客户端将 r 用 As 进行加密，结果发送给服务器。
  5. 服务器用 Bs 进行解密，获得 r。
  6. 服务器进行 r xor p 的运算，获得 q。
  7. 至此服务器和客户端都知道了会话密钥 q，以后的传输都将被 q 加密。
2. 认证
  1. 服务器生成随机数 x，并用 Ac 加密后生成结果 S(x) ，发送给客户端。
  2. 客户端使用 Bc 解密 S(x) 得到 x。
  3. 客户端计算 q + x 的 md5 值 n(q+x)，q为上一步得到的会话密钥。
  4. 服务器计算 q + x 的 md5 值 m(q+x)。
  4. 客户端将 n(q+x) 发送给服务器。
  5. 服务器比较 m(q+x) 和 n(q+x)，两者相同则认证成功。

## 服务器端设置

使用公钥认证需要对服务器进行一些设置，以下将以 Ubuntu 系统为例说明。
修改服务器端的 `/etc/ssh/sshd_config` 配置文件，确保以下配置开启：

```
RSAAuthentication yes        # 启用 RSA 认证
PubkeyAuthentication yes     # 启用公钥认证
PasswordAuthentication no    # 如果对服务器安全性要求比较高，可以设置用户只允许通过公钥认证，禁止用户用密码方式登录
```

然后重新启动 sshd：

```sh
$ sudo /etc/init.d/ssh restart
```

## 客户端设置

假设客户端的用户 jerry 要以 tom 用户登录到服务器上。首先在客户端执行下面的命令：

```
[jerry@client: ~] $ ssh-keygen -t rsa
Generating public/private rsa1 key pair.
Enter file in which to save the key (/home/jerry/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):  输入密码
Enter same passphrase again:   再次输入密码
Your identification has been sabed in /home/jerry/.ssh/id_rsa
Your public key has been saved in /home/jerry/.ssh/id_rsa.pub
```

其中，`passphrase` 可以为空，如果为空，登录时将不需要密码。
生成的文件保存在主目录的 .ssh 目录下，`id_rsa` 为客户端密钥，`id_rsa.pub` 为客户端公钥。
之后，通过 scp 方式将公钥 `id_rsa.pub` 传到到服务器上，并执行下列命令。

```
[tom@server:~]$ cat id_rsa.pub >> .ssh/authorized_keys
```

其中 `id_rsa.pub` 是客户端的用户 jerry 的公钥。注意：

- .ssh 目录的权限是 700。
- .ssh/authorized_keys 文件权限是 600。

这样在客户端即可通过以下的命令连接服务器（假设 server 的 ip 是 192.168.0.1）：

```
[jerry@client:~]$ ssh tom@192.168.0.1
```

如果生成公钥的时候输入了密码，又不想每次登录服务器时都输入密码，可以先执行下列命令：

```
[jerry@client:~]$ ssh-add
Enter passphrase for /home/jerry/.ssh/id_rsa: 输入密码
Identity added: /home/jerry/.ssh/id_rsa (/home/jerry/.ssh/id_rsa)
```

以后登录服务器就不需要输入密码了。
