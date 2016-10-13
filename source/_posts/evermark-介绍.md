---
title: Evermark 介绍
date: 2016-10-10 15:56:04
tags: evernote markdown
---

[**Evermark**](https://github.com/akuma/evermark) 是我用 JavaScript 开发的命令行工具，支持以 Markdown 格式写 Evernote 笔记，简单实用。

目前支持的特性如下：

- 支持基于命令行添加、发布 Markdown 格式的笔记
- 支持自动添加在笔记内容中指定的笔记本和标签
- 支持发布或撤销某个目录下的所有 Markdown 笔记
- 支持高亮代码块、图片引用、表格等
- 支持任务列表
- 支持数学公式
- 支持流程图、序列图、甘特图

<!--more-->

## 安装方法

```bash
npm install -g evermark
```

## 命令说明

### 初始化 Evermark 文件夹

执行完后需要修改该目录下的 `evermark.json`，填写你的 `developerToken`。

```bash
evermark init <destination>
```

`developerToken` 的生成链接：

- [Evernote International](https://www.evernote.com/api/DeveloperToken.action)
- [印象笔记](https://app.yinxiang.com/api/DeveloperToken.action)

### 查看或修改配置

```bash
evermark config [name] [value]
```

### 添加笔记文件

创建一个 markdown 文件，存放在 Evermark 文件夹的 `notes` 目录下。

```bash
evermark new <title>
```

### 发布笔记

将 markdown 文件发布到 Evernote，对于已发布过的文件会采取更新操作。

```bash
evermark publish <file_or_directory>
```

### 撤销笔记

在 Evernote 中删除 markdown 文件对应的笔记，markdown 文件不会删除。

```bash
evermark unpublish <file_or_directory>
```

### 查看帮助

```bash
evermark help [command]
```

## Evermark 支持的 Markdown 语法

### Headers

```
# H1
## H2
### H3
#### H4
##### H5
###### H6
```

### Emphasis

```
*This text will be italic*
_This will also be italic_

**This text will be bold**
__This will also be bold__

~~This text will be crossed~~

_You **can** combine ~~them~~_
```

### Sups & Subs

```
19^th^
H~2~O
```

### Emoji

```
:smile: :heart: :sunny: :watermelon: :cn:
```

### Links

```
http://github.com - automatic!
[GitHub](http://github.com)
```

### Blockquotes

```
As Kanye West said:

> We're living the future so
> the present is our past.
```

### Lists

#### Unordered

```
- Item 1
- Item 2
  - Item 2a
  - Item 2b
```

#### Ordered

```
1. Item 1
1. Item 2
1. Item 3
   - Item 3a
   - Item 3b
```

### Task Lists

```
- [x] Write blog post with :heart:
- [x] Create sample **gist**
- [ ] Take screenshots for blog post
```

### Tables

```
First Header | Second Header
------------ | -------------
Content from cell 1 | Content from cell 2
Content in the first column | Content in the second column
```

### Images

```
![Image of Test](img/test.png "Image of Test")
![GitHub Logo](https://assets-cdn.github.com/images/modules/logos_page/Octocat.png "GitHub Logo")
```

### Inline Code

```
This is an inline code: `var example = true`
```

### Block Code

```
```js
console.log('Hello world!')
``````

## Diagrams

这部分功能使用 [Mermaid](https://github.com/knsv/mermaid) 实现，具体可以参考它的[文档](http://knsv.github.io/mermaid/)。

```
```
graph LR
  A-->B
  B-->C
  C-->A
  D-->C
``````

```
```
sequenceDiagram
  Alice->>John: Hello John, how are you?
  John-->>Alice: Great!
``````

```
```
gantt
  title A Gantt Diagram

  section Section
  A task           :a1, 2014-01-01, 30d
  Another task     :after a1  , 20d
  section Another
  Task in sec      :2014-01-12  , 12d
  anther task      : 24d
``````

### Raw HTML

```
<div style="color: red;">This is a <strong>html</strong> code.</div>
```

### Notebooks & Tags

- **Evermark** 自动使用文档内出现的第一个标题作为笔记标题。
- **Evermark** 支持 `@(笔记本)[标签A|标签B]` 语法, 以选择笔记本和添加标签。
