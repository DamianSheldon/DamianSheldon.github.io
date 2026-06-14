----
layout: post
title:  "Welcome to Jekyll!"
date:   2025-02-04 17:40:38 +0800
categories: [Web]
---

下面是一个完整的测试用 Markdown 示例，**覆盖了技术博客中常见的所有 Markdown 元素**，适用于 GitHub Pages（GFM/kramdown）、Jekyll 博客等环境。

# Markdown 全元素测试文档

> 这是一个覆盖常用 Markdown 语法的示例，用于测试博客渲染效果。

---

## 目录（Table of Contents）

- [标题](#标题)
- [段落与换行](#段落与换行)
- [强调文本](#强调文本)
- [列表](#列表)
- [链接](#链接)
- [图片](#图片)
- [表格](#表格)
- [代码块](#代码块)
- [引用](#引用)
- [注脚](#注脚)
- [数学公式](#数学公式)
- [折叠内容（details）](#折叠内容details)

---

## 标题

# H1 标题
## H2 标题
### H3 标题
#### H4 标题
##### H5 标题
###### H6 标题

---

## 段落与换行

这是第一段。  
这是第二段，使用了两个空格结尾手动换行。  
第三段是空行后自动换行。

---

## 强调文本

**加粗文本**  
*斜体文本*  
~~删除线文本~~  
`行内代码`

---

## 列表

### 无序列表

- 项目 1
  - 子项 1.1
  - 子项 1.2
- 项目 2

### 有序列表

1. 第一项
2. 第二项
   1. 子项 2.1
   2. 子项 2.2

---

## 链接

- [GitHub](https://github.com)
- [站内链接到图片](#图片)
- <https://jekyllrb.com/docs/home>

---

## 图片

### 外链图片

![示例图片](https://unsplash.com/photos/abstract-depiction-of-female-reproductive-organs-AV6idRPxKK4)

### 本地图片（Jekyll 示例）

![logo](/images/bird_32_gray.png)

---

## 表格

| 编号 | 名称     | 描述         |
|------|----------|--------------|
| 1    | Apple    | 苹果公司     |
| 2    | Microsoft| 微软公司     |
| 3    | Google   | 谷歌公司     |

---

## 代码块

### 行内代码

使用 `System.out.println("Hello")` 打印。

### 多语言代码块

#### Java

```java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello Markdown");
    }
}
````

#### Bash

```bash
echo "Hello World"
```

#### JSON

```json
{
  "name": "markdown",
  "type": "test"
}
```

---

## 引用

> 这是一段引用。
> 可以多行显示。
>
> > 嵌套引用

---

## 注脚（kramdown 支持）

这是带注脚的文字[^1]。

[^1]: 注脚内容在此处说明。

---

## 数学公式（支持 MathJax 时生效）

行内公式：\$E = mc^2\$

块级公式：

$$
\int_{a}^{b} x^2 dx = \frac{b^3 - a^3}{3}
$$

---

## 折叠内容（details）

<details>
<summary>点击展开折叠内容</summary>

这是折叠的内容，可以写说明文档、FAQ 等。

</details>

---

## Task List（GFM 特有）

* [x] 已完成任务
* [ ] 待完成任务
* [ ] ~~取消的任务~~

---

## Emoji 表情

支持的 emoji 示例：🚀 🎉 ✅ ❌ 🔥

---

## 引用脚本或高亮代码（Jekyll 代码高亮）

{% highlight ruby %}
def hello
puts 'Hello, world!'
end
{% endhighlight %}

> *注：需要开启 Jekyll 的 Rouge 代码高亮功能。*

---

## License

MIT © 2025 by 测试作者

```

---

## 📌 说明

- 支持 Jekyll 时，YAML Front Matter（`---` 区块）是必须的。
- 内容适配了 GitHub Pages 的默认 kramdown/GFM 渲染器。
- 包括图片、代码块、数学公式、注脚、引用、折叠区域等全部常见博客元素。


