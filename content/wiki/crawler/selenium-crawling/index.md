---
title: "Selenium动态渲染页面爬取"
date: 2026-07-10
description: "用Selenium处理JS渲染页面——当静态爬虫不够用时"
---

## 什么时候需要 Selenium

当目标网站的数据是通过 JavaScript 动态加载的，且你无法直接找到背后的 API 时，Selenium 是备选方案。

> **能用 requests 就不要用 Selenium**——Selenium 慢、吃资源、不稳定。但有时候，它是唯一的选择。

## 环境准备

```bash
pip install selenium webdriver-manager
```

使用 `webdriver-manager` 自动管理 ChromeDriver 版本：
```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager

service = Service(ChromeDriverManager().install())
driver = webdriver.Chrome(service=service)
```

## 基本操作

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

driver = webdriver.Chrome()

# 访问页面
driver.get("https://example.com")

# 查找元素
element = driver.find_element(By.CSS_SELECTOR, ".title")
elements = driver.find_elements(By.CSS_SELECTOR, ".item")

# 获取文本/属性
text = element.text
href = element.get_attribute("href")

# 等待元素加载（显式等待——推荐）
wait = WebDriverWait(driver, 10)
element = wait.until(
    EC.presence_of_element_located((By.CSS_SELECTOR, ".content"))
)

# 执行 JS
driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")

# 截屏
driver.save_screenshot("page.png")

driver.quit()
```

## 处理滚动加载

```python
import time

last_height = driver.execute_script("return document.body.scrollHeight")
scroll_count = 0
max_scrolls = 20

while scroll_count < max_scrolls:
    driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
    time.sleep(2)  # 等待新内容加载
    new_height = driver.execute_script("return document.body.scrollHeight")
    if new_height == last_height:
        break
    last_height = new_height
    scroll_count += 1
```

## 无头模式（Headless）

服务器上运行不需要打开浏览器窗口：

```python
from selenium.webdriver.chrome.options import Options

options = Options()
options.add_argument("--headless=new")  # 新版无头模式
options.add_argument("--no-sandbox")
options.add_argument("--disable-dev-shm-usage")
options.add_argument("--disable-blink-features=AutomationControlled")  # 隐藏自动化标记
options.add_argument("user-agent=Mozilla/5.0 ...")  # 自定义UA

driver = webdriver.Chrome(options=options)
```

## 隐藏 Selenium 特征

网站能检测到你在用 Selenium（`navigator.webdriver` 为 true）。

```python
# 隐藏 webdriver 标记
driver.execute_cdp_cmd("Page.addScriptToEvaluateOnNewDocument", {
    "source": """
        Object.defineProperty(navigator, 'webdriver', {get: () => undefined})
    """
})
```

> 更彻底的方案：使用 `undetected-chromedriver` 库，专门包装了反检测逻辑。

## Selenium 最佳实践

1. **显式等待 > 隐式等待 > time.sleep()**
2. **用 CSS 选择器**，不用复杂的 XPath
3. **及时 quit()**——否则 Chrome 进程残留吃内存
4. **异常处理**——网络超时、元素缺失要 catch
5. **日志记录**——爬虫挂了要知道在哪一步挂的

## 对比：Selenium vs Playwright vs Puppeteer

| | Selenium | Playwright | Puppeteer |
|---|---------|------------|-----------|
| 语言 | 多语言 | JS/Python/.NET/Java | JS |
| 速度 | 较慢 | 较快 | 较快 |
| 浏览器支持 | Chrome/Firefox/Safari | Chrome/Firefox/Safari | Chrome/Firefox |
| 反检测 | 需要额外处理 | 内置较好 | 内置较好 |
| 社区 | 最大最成熟 | 增长最快 | Google维护 |

> 如果可以选择，2024年后推荐用 Playwright 代替 Selenium——性能更好、API 更现代。但 Selenium 的社区资源和历史积累仍然最大。
