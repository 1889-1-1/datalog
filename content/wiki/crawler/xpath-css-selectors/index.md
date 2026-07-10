---
title: "XPath与CSS选择器"
date: 2026-07-10
description: "网页数据提取的两大利器——XPath和CSS选择器完全指南"
---

## 选择器的作用

爬虫的核心任务是从 HTML 中提取数据。XPath 和 CSS 选择器是两种主流的定位方式。Scrapy 两者都支持，但各有优劣。

## CSS 选择器速查

| 选择器 | 语法 | 示例 |
|--------|------|------|
| 标签 | `tag` | `div` → 所有 div |
| 类 | `.class` | `.title` → class="title" 的元素 |
| ID | `#id` | `#main` → id="main" 的元素 |
| 子元素 | `parent > child` | `div > p` → div 的直接 p 子元素 |
| 后代 | `parent child` | `div p` → div 内所有 p |
| 属性 | `[attr=val]` | `[href]` → 有 href 属性的元素 |
| 第n个 | `:nth-child(n)` | `li:nth-child(2)` → 第2个 li |
| 文本内容 | `::text` | `p::text` → p 的文本节点 |
| 属性值 | `::attr(name)` | `a::attr(href)` → a 的 href 值 |

## XPath 速查

| 表达式 | 说明 |
|--------|------|
| `/html/body/div` | 绝对路径 |
| `//div` | 任意位置的 div |
| `//div[@class="title"]` | class="title" 的 div |
| `//a/@href` | 所有 a 标签的 href 属性 |
| `//h1/text()` | 所有 h1 的文本 |
| `//div[contains(@class, "item")]` | class 包含 "item" 的 div |
| `//a[starts-with(@href, "https")]` | href 以 https 开头的 a |
| `//div/parent::*` | div 的父元素 |
| `//div/following-sibling::*` | div 的后续兄弟元素 |
| `//tr[position()>1]` | 第2行及之后的 tr（跳过表头）|

## CSS vs XPath

| | CSS | XPath |
|---|-----|-------|
| 可读性 | ⭐⭐⭐⭐⭐ 简洁 | ⭐⭐⭐ 略繁琐 |
| 功能 | 有限（只能向下查找）| 强大（支持向上、兄弟节点）|
| 按文本匹配 | 不支持 | 支持 `//a[contains(text(),'登录')]` |
| 性能 | 快 | 略慢（大多数场景可忽略）|

> **建议**：优先用 CSS（简洁），CSS 做不到的用 XPath（如按文本内容匹配）。Scrapy 中 `response.css()` 和 `response.xpath()` 可以混用。

## 实战技巧

### 提取表格数据
```python
for row in response.css("table tr"):
    yield {
        "name": row.css("td:nth-child(1)::text").get(),
        "price": row.css("td:nth-child(2)::text").get(),
    }
```

### 按文本找元素（XPath 独有）
```python
# 找到文字为"下一页"的按钮
next_btn = response.xpath('//a[contains(text(), "下一页")]/@href').get()
```

### 使用正则辅助
```python
# CSS选择器 + 正则
response.css("div.content *::text").re(r"\d{4}-\d{2}-\d{2}")  # 提取日期
```

## 浏览器工具

- **Chrome DevTools**：右键 → 检查 → 右键元素 → Copy → Copy selector / Copy XPath
- **XPath Helper 插件**：实时测试 XPath 表达式
- **Scrapy Shell**：`scrapy shell "url"` 后在命令行交互式测试选择器
