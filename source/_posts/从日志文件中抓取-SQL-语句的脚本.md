---
title: 从日志文件中抓取 SQL 语句的脚本
date: 2012-05-04 12:45:14
tags: python sql log
---

为了获取系统日志中的 SQL 语句并进行性能分析，我写了一个简单的 python 脚本，用于从日志中抓取 SQL。

<!--more-->

简单特性：

- 语法相同参数不同的 SQL 会只保留一条。
- 默认输出到控制台，如果希望保存到文件可以采用重定向命令。

```python
#!/usr/bin/python
#coding=utf-8
# A python script for extracting sql statments from file then print them to console.

import sys
import re

__version__ = "0.2"

def extract_sqls(filenames, for_explain=False):
    for filename in filenames:
        extract_sql(filename, for_explain)

def extract_sql(filename, for_explain=False):
    '''Extract sql statments from the file and print them to std.'''

    f = None
    try:
        f = file(filename)
    except IOError:
        print '%s: %s: No such file' % (sys.argv[0], filename)
        return

    print '\n%s> %s <%s\n' % ('=' * 25, filename, '=' * 25)
    sql_set = set()
    while True:
        line = f.readline()
        if len(line) == 0:
            break

        # If the string is a dml sql, then record it
        explain_sql_pres = ['(^| )(SELECT )', '(^| )(UPDATE )', '(^| )(DELETE )']
        all_sql_pres = explain_sql_pres + ['(^| )(INSERT INTO )']
        sql_pres = explain_sql_pres if for_explain is True else all_sql_pres
        for sql_pre in sql_pres:
            m = re.search(sql_pre, line, re.I)
            if m is not None:
                break

        # If the string is not a dml sql, ignore it
        if m is None:
            continue

        raw_sql = line[m.start(2):-1]
        base_sql = re.sub("'.*'|\d+", '?', raw_sql)

        if for_explain is True:
            if base_sql not in sql_set:
                sql_set.add(base_sql)
                print_sql(raw_sql)
        else:
            print_sql(raw_sql)

    f.close()
    print

def print_sql(sql):
    if sql[-1] == ';':
        print '%s' % sql
    else:
        print '%s;' % sql

# Script starts from here
if len(sys.argv) < 2:
    print '''\
%s: too few arguments
Try `%s --help\' for more information.''' % (sys.argv[0], sys.argv[0])
    sys.exit()

if sys.argv[1].startswith('--'):
    option = sys.argv[1][2:]
    # Fetch sys.argv[1] but without the first two characters
    if option == 'for-explain':
        extract_sqls(sys.argv[2:], True)
    elif option == 'version':
        print 'sql_extractor', __version__
        sys.exit()
    elif option == 'help':
        print '''
Usage: %s [OPTION] FILE...
Extract sql statments from file then print them to console.

Options include:
  --for-explain    Only print one sql for which with different parameters
  --version        Prints the version number
  --help           Display this help''' % sys.argv[0]
    else:
        print '''\
%s: invalid option -- %s
Try `%s --help\' for more information.''' % (sys.argv[0], option, sys.argv[0])
        sys.exit()
else:
    extract_sqls(sys.argv[1:])
```

使用举例：

```sh
./sql_extractor.py app.log > app.sql
```
