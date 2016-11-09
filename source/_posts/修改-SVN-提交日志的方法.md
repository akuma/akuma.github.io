---
title: 修改 SVN 提交日志的方法
date: 2009-08-15 13:29:56
tags: svn log
---

在我们使用 SVN 的时候，有时候会遇到这样的问题：对某次的修改做了提交，但发现提交时的日志写的不对，于是想要修改这个日志。

SVN 默认是不支持日志修改的，而只要将下面的脚本文件放到源代码服务器仓库的 hooks 目录就可以得到支持了。

<!--more-->

## Windows 版

```bat
@ECHO OFF
:: Set all parameters. Even though most are not used, in case you want to add
:: changes that allow, for example, editing of the author or addition of log messages.
set repository=%1
set revision=%2
set userName=%3
set propertyName=%4
set action=%5

:: Only allow the log message to be changed, but not author, etc.
if /I not "%propertyName%" == "svn:log" goto ERROR_PROPNAME

:: Only allow modification of a log message, not addition or deletion.
if /I not "%action%" == "M" goto ERROR_ACTION

:: Make sure that the new svn:log message is not empty.
set bIsEmpty=true
for /f "tokens=*" %%g in ('find /V ""') do (
set bIsEmpty=false
)
if "%bIsEmpty%" == "true" goto ERROR_EMPTY

goto :eof

:ERROR_EMPTY
echo Empty svn:log messages are not allowed. >&2
goto ERROR_EXIT

:ERROR_PROPNAME
echo Only changes to svn:log messages are allowed. >&2
goto ERROR_EXIT

:ERROR_ACTION
echo Only modifications to svn:log revision properties are allowed. >&2
goto ERROR_EXIT

:ERROR_EXIT
exit /b 1
```

## Linux 版

注意：要将脚本设置为可执行。

```sh
#!/bin/sh
# Set up the list of possible parameters.
REPOS="$1"
REV="$2"
USER="$3"
PROPNAME="$4"
ACTION="$5"

# Allow log modifications
if [ "$ACTION" = "M" -a "$PROPNAME" = "svn:log" ]; then
    exit 0
else
    echo "Changing revision properties other than svn:log is prohibited" >&2
    exit 1
fi
```
