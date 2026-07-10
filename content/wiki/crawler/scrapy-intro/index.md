---
title: "Scrapy框架入门"
date: 2026-07-10
description: "Python最强大的爬虫框架——从安装到第一个爬虫"
---

## 为什么选 Scrapy

Scrapy 是 Python 生态中最成熟的爬虫框架，自带请求调度、数据管道、中间件等一切你需要的功能。相比于 requests + BeautifulSoup 的手工作坊式爬虫，Scrapy 更适合规模化项目。

## 安装

```bash
pip install scrapy
```

验证安装：
```bash
scrapy version
```

## 创建第一个项目

```bash
scrapy startproject myproject
cd myproject
scrapy genspider example example.com
```

## 项目结构

```
myproject/
├── scrapy.cfg          # 部署配置
└── myproject/
    ├── __init__.py
    ├── items.py        # 定义要提取的数据结构
    ├── middlewares.py  # 中间件（请求/响应处理）
    ├── pipelines.py    # 数据管道（清洗、存储）
    ├── settings.py     # 全局配置
    └── spiders/        # 爬虫文件
        └── example.py
```

## 编写第一个 Spider

```python
import scrapy

class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = ["http://quotes.toscrape.com/"]

    def parse(self, response):
        for quote in response.css("div.quote"):
            yield {
                "text": quote.css("span.text::text").get(),
                "author": quote.css("small.author::text").get(),
                "tags": quote.css("div.tags a.tag::text").getall(),
            }

        # 翻页
        next_page = response.css("li.next a::attr(href)").get()
        if next_page:
            yield response.follow(next_page, self.parse)
```

运行：
```bash
scrapy crawl quotes -o quotes.json
```

## Scrapy 核心组件

| 组件 | 作用 |
|------|------|
| Engine | 控制数据流，触发事件 |
| Scheduler | 管理请求队列 |
| Downloader | 发送请求，获取响应 |
| Spider | 解析响应，提取数据 |
| Item Pipeline | 处理提取的数据（清洗、去重、入库）|
| Middlewares | 在请求/响应过程中插入自定义逻辑 |

## 数据管道 (Pipeline)

存储到 MySQL 的示例：

```python
import pymysql

class MySQLPipeline:
    def open_spider(self, spider):
        self.conn = pymysql.connect(host="localhost", user="root",
                                     password="", database="scrapy_db")
        self.cursor = self.conn.cursor()

    def process_item(self, item, spider):
        sql = "INSERT INTO quotes (text, author) VALUES (%s, %s)"
        self.cursor.execute(sql, (item["text"], item["author"]))
        self.conn.commit()
        return item

    def close_spider(self, spider):
        self.cursor.close()
        self.conn.close()
```

在 `settings.py` 中启用：
```python
ITEM_PIPELINES = {
    "myproject.pipelines.MySQLPipeline": 300,
}
```

## 下一步

掌握基础后，深入研究 Scrapy 的中间件系统、并发控制、分布式部署（Scrapy-Redis）。
