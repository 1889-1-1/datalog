---
title: "SQL注入原理与防御"
date: 2026-07-10
description: "深入理解SQL注入攻击的原理和防御方法"
---

## 什么是 SQL 注入

SQL 注入是最常见的 Web 攻击之一，攻击者通过在输入中插入恶意 SQL 代码，操纵数据库执行非预期的查询。

## 攻击原理

假设登录验证的代码是这样写的：

```python
# 危险的写法
username = request.form["username"]
password = request.form["password"]
sql = f"SELECT * FROM users WHERE username='{username}' AND password='{password}'"
cursor.execute(sql)
```

攻击者在用户名输入框中输入：
```
admin' --
```

拼接后的 SQL 变为：
```sql
SELECT * FROM users WHERE username='admin' --' AND password=''
```

`--` 是 SQL 注释，后面的密码验证被注释掉了！攻击者无需密码即可登录为 admin。

## 常见攻击手法

### 1. 绕过登录
```
输入：admin' OR '1'='1
结果：WHERE username='admin' OR '1'='1' AND password='...'
```

### 2. UNION 查询窃取数据
```
输入：' UNION SELECT username, password FROM users --
结果：在正常查询结果后拼接出所有用户名密码
```

### 3. 盲注（Boolean-Based Blind）
```
输入：' AND SUBSTRING((SELECT password FROM users LIMIT 1), 1, 1) = 'a
利用页面是否返回正常，逐字符猜解密码
```

### 4. 时间盲注（Time-Based Blind）
```
输入：' AND IF(SUBSTRING(password,1,1)='a', SLEEP(5), 0) --
如果页面延迟5秒响应，说明第一位密码是'a'
```

## 防御方案

### 1. 参数化查询（最重要！）

```python
# 安全的写法：参数化查询
cursor.execute(
    "SELECT * FROM users WHERE username = %s AND password = %s",
    (username, password)
)
```

### 2. ORM 框架

```python
# Django ORM
User.objects.filter(username=username, password=password)

# SQLAlchemy
session.query(User).filter(User.username == username)
```

### 3. 输入验证 + 白名单

```python
import re

def is_safe_username(username):
    return bool(re.match(r'^[a-zA-Z0-9_]{3,20}$', username))
```

### 4. 最小权限原则

```sql
-- 应用的数据库用户只需要 CRUD 权限
-- 永远不要给 DROP、ALTER、GRANT 权限
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'xxx';
GRANT SELECT, INSERT, UPDATE, DELETE ON mydb.* TO 'app_user'@'localhost';
```

### 5. WAF（Web 应用防火墙）

在应用层前面加一道 WAF 过滤明显的 SQL 注入 payload。

## 自查清单

- [ ] 代码中是否有任何字符串拼接 SQL？
- [ ] 所有数据库查询是否使用参数化？
- [ ] 数据库用户是否只有最小必需权限？
- [ ] 错误信息是否不暴露数据库结构？
- [ ] 是否部署了 WAF 作为外围防护？
