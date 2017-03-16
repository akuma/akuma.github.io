---
title: macOS 开发环境搭建笔记
date: 2017-01-05 12:44:31
tags: macOS dev
---

## 初始准备工作

### 通过 AppStore 安装 XCode

### 修改 sudo 权限（管理员不需要输入密码）

打开 shell，执行 `sudo visudo`，修改以下部分内容：

```
%admin          ALL = (ALL) NOPASSWD: ALL
```

<!-- more -->

## 安装命令行工具

### Homebrew

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

### zsh 相关

```
brew install zsh zsh-completions
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

### 安装 JDK

```
brew cask install java
```

### 安装 Maven

```
brew install maven
```

编辑文件 `$HOME/.m2/settings.xml`，修改成适合自己的内容（比如包含公司私服设置等）。

### 安装 NVM

`NVM` 是一个 Node.js 版本管理，通过它可以方便的切换 Node.js 版本。

```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.0/install.sh | bash
```

### 安装其他命令行工具

```
brew install coreutils moreutils autojump htop wget git git-extras diff-so-fancy mycli redis cloc tmux tree unrar
```

一些工具说明：

- coreutils：[GNU core utilities](https://www.gnu.org/software/coreutils/)
- moreutils：[GNU more utilities](https://www.gnu.org/software/moreutils/)
- autojump：[目录导航命令](https://github.com/wting/autojump)
- htop：[类似 top 的命令](https://hisham.hm/htop/)
- git-extras：[git 扩展命令](https://github.com/tj/git-extras/blob/master/Commands.md)
- diff-so-fancy：[更好看的 diff 工具](https://github.com/so-fancy/diff-so-fancy)
- mycli：[MySQL 命令行客户端](https://github.com/dbcli/mycli)
- cloc：统计代码行数的工具
- tmux：多终端模拟工具
- tree：显示目录树的命令
- unrar：解压 rar 文件的命令

### 修改 zsh 配置

编辑 `~/.zshrc`，添加如下内容：

```sh
# Config zsh completions
fpath=(/usr/local/share/zsh-completions $fpath)

# Config autojump
[ -f /usr/local/etc/profile.d/autojump.sh ] && . /usr/local/etc/profile.d/autojump.sh

# Config JAVA HOME
export JAVA_HOME=`/usr/libexec/java_home`

# Config NVM for Node.js
export NVM_HOME="$HOME/.nvm"
export NVM_NODEJS_ORG_MIRROR=https://npm.taobao.org/mirrors/node
export PHANTOMJS_CDNURL=http://npm.taobao.org/mirrors/phantomjs
export SELENIUM_CDNURL=http://npm.taobao.org/mirrorss/selenium
export CHROMEDRIVER_CDNURL=http://npm.taobao.org/mirrors/chromedriver
[ -s "$NVM_HOME/nvm.sh" ] && . "$NVM_HOME/nvm.sh"
```

**注意**：保存 zsh 配置后，需要执行 `source ~/.zshrc` 或重建 Shell 窗口才会生效。

### 配置 git

编辑 `~/.gitconfig`，内容如下：

```
[user]
    name = <your name>
    email = <your email>

[alias]
    ci = commit -m
    co = checkout
    st = status
    br = branch
    wc = whatchanged
    df = diff
    pom = push origin master
    unstage = reset HEAD
    last = log -1 HEAD
    sl = shortlog -s
    lg = log -p
    gl = log --all --graph --pretty=format:'%Cred%h%Creset -%C(auto)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative
    praise = blame

[core]
    autocrlf = input
    quotepath = false
    ignorecase = false
    pager = diff-so-fancy | less --tabs=4 -RFX
    whitespace = trailing-space,space-before-tab,indent-with-non-tab
    editor = /usr/bin/vim

[merge]
    ff = false

[pull]
    rebase = true

[push]
    default = matching

[color]
    status = auto
    branch = auto
    diff = auto
    ui = true
    pager = true

[color "branch"]
    current = green reverse
    local = yellow
    remote = red

[color "diff"]
    meta = yellow bold
    frag = magenta bold
    old = red bold
    new = green bold

[color "status"]
    added = yellow bold
    changed = red bold
    untracked = white bold

[color "diff-highlight"]
    oldNormal = red bold
    oldHighlight = red bold 52
    newNormal = green bold
    newHighlight = green bold 22
```

## 安装 APP

### 开发调试

- Atom（文本编辑器）及插件
  - file-icons
  - highlight-line
  - highlight-selected
  - minimap
  - minimap-highlight-selected
  - Sublime-Style-Column-Selection
  - editorconfig
  - css-snippets
  - javascript-snippets
  - linter
  - linter-xo
  - linter-eslint
  - local-history
- IntelliJ IDEA（Java IDE）
- Postman（接口调试工具）
- GitKraken（Git 客户端）
- Charles（HTTP 抓包工具）
- Wireshark（TCP 抓包工具）
- Docker（Docker Mac 版）

### 生产效率

- Alfred
- 1Password（密码管理）
- Spectacle（窗口管理）
- f.lux（显示器色温调整）
- MacDown（Markdown 编辑器）
- OmniGraffle（作图工具）
