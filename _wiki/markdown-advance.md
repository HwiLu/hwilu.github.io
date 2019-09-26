---
layout: wiki
title: Markdown语法进阶
categories: [markdown, 转载]
description: Markdown语法进阶，那些简单的就不记录了。
keywords: Markdown
---



### 超链接

Markdown 支持行内式链接和引用式链接。

**Markdown：**

```
行内式 [博客](https://mazhuang.org "我的个人博客") 链接，带 title。

行内式 [GitHub](https://github.com/mzlogin) 链接。

引用式 [博客][1] 链接。

引用式 [GitHub][2] 链接，带 title。

[1]: https://mazhuang.org
[2]: https://github.com/mzlogin "我的 GitHub 主页"
```

**预览效果：**

行内式 [博客](https://mazhuang.org "我的个人博客") 链接，带 title。

行内式 [GitHub](https://github.com/mzlogin) 链接。

引用式 [博客][1] 链接。

引用式 [GitHub][2] 链接，带 title。

[1]: https://mazhuang.org
[2]: https://github.com/mzlogin	"我的 GitHub 主页"

### 自动链接

自动链接扩展，即：当识别到 URL，或用 `<`、`>` 包括的 URL 时，会自动为其生成 `a` 标签。

**Markdown：**

```
https://github.com

<example@gmail.com>
```

**预览效果：**

https://github.com

<example@gmail.com>

**对应 HTML：**

```html
<p><a href="https://github.com">https://github.com</a></p>

<p><a href="mailto:example@gmail.com">example@gmail.com</a></p>
```

### emoji

以 GitHub Pages 为例。

**Markdown：**

```
:camel: :blush: :smile:
```

**预览效果：**

:camel: :blush: :smile:

**对应 HTML：**

```html
<p>
  <img class="emoji" title=":camel:" alt=":camel:" src="https://assets-cdn.github.com/images/icons/emoji/unicode/1f42b.png" height="20" width="20">
  <img class="emoji" title=":blush:" alt=":blush:" src="https://assets-cdn.github.com/images/icons/emoji/unicode/1f60a.png" height="20" width="20">
  <img class="emoji" title=":smile:" alt=":smile:" src="https://assets-cdn.github.com/images/icons/emoji/unicode/1f604.png" height="20" width="20">
</p>
```

### 表格

**Markdown：**

    | 编号  | 姓名（左） | 年龄（右） | 性别（中） |
    | ----- | :--------  | ---------: | :------:   |
    | 0     | 张三       | 28         | 男         |
    | 1     | 李四       | 29         | 男         |

**预览效果：**

| 编号 | 姓名（左） | 年龄（右） | 性别（中） |
| ---- | :--------- | ---------: | :--------: |
| 0    | 张三       |         28 |     男     |
| 1    | 李四       |         29 |     男     |

**对应 HTML：**

```html
<table>
  <thead>
    <tr>
      <th>编号</th>
      <th align="left">姓名（左）</th>
      <th align="right">年龄（右）</th>
      <th align="center">性别（中）</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td align="left">张三</td>
      <td align="right">28</td>
      <td align="center">男</td>
    </tr>
    <tr>
      <td>1</td>
      <td align="left">李四</td>
      <td align="right">29</td>
      <td align="center">男</td>
    </tr>
  </tbody>
</table>
```

### 任务列表

在 GitHub / GitLab 里有较好的支持。

**Markdown：**

```
- [x] 洗碗
- [ ] 清洗油烟机
- [ ] 拖地
```

**预览效果：**

- [x] 洗碗
- [ ] 清洗油烟机
- [ ] 拖地

**对应 HTML：**

```html
<ul class="contains-task-list">
  <li class="task-list-item"><input type="checkbox" id="" disabled="" class="task-list-item-checkbox" checked=""> 洗碗</li>
  <li class="task-list-item"><input type="checkbox" id="" disabled="" class="task-list-item-checkbox"> 清洗油烟机</li>
  <li class="task-list-item"><input type="checkbox" id="" disabled="" class="task-list-item-checkbox"> 拖地</li>
</ul>
```



转载于：[https://github.com/mzlogin/markdown-intro](https://github.com/mzlogin/markdown-intro)