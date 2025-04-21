老铁，太懂了你想要啥了😎  
下面我就给你写一个 **适用于 Kali Linux 的 Burp Suite + Firefox 自动配置脚本**，你只需要运行它，就能完成：

- 启动 Burp
- 配置 Firefox 使用 Burp 代理
- 自动导入 Burp 的 CA 证书
- 一键开启抓包环境！

---

## 🐚 Kali 自动配置脚本：`burp_firefox_setup.sh`

```bash
#!/bin/bash

# ============ 配置参数 ============
BURP_PROXY_HOST="127.0.0.1"
BURP_PROXY_PORT="8080"
FIREFOX_PROFILE_NAME="burp-profile"
CA_CERT_PATH="$HOME/burp_cert.der"
# ==================================

echo "✅ [1/5] 确保 Burp Suite 已运行，并监听在 $BURP_PROXY_HOST:$BURP_PROXY_PORT"
echo "💡 如果还没开启 Burp，请现在手动打开它"

read -p "👉 打开 Burp 后，按 Enter 继续..."

echo "✅ [2/5] 访问 Burp 提供的证书页面，下载 CA 证书..."
echo "📦 正在使用 curl 抓取 Burp CA..."

curl -sk http://burp/cert -o "$CA_CERT_PATH"

if [ ! -f "$CA_CERT_PATH" ]; then
  echo "❌ 下载证书失败，请手动访问 http://burp 下载并保存为 burp_cert.der"
  exit 1
fi

echo "✅ 证书已保存为：$CA_CERT_PATH"

# 检查是否已有该 Firefox profile
echo "✅ [3/5] 创建 Firefox 专用抓包配置 profile：$FIREFOX_PROFILE_NAME"
firefox -CreateProfile "$FIREFOX_PROFILE_NAME" >/dev/null

# 获取 profile 路径
PROFILE_DIR=$(find ~/.mozilla/firefox -type d -name "*.$FIREFOX_PROFILE_NAME")

if [ ! -d "$PROFILE_DIR" ]; then
  echo "❌ 没找到 profile 目录，创建失败"
  exit 1
fi

echo "📂 profile 路径：$PROFILE_DIR"

# ============ 修改 Firefox prefs.js ============
echo "✅ [4/5] 修改 Firefox 配置文件 prefs.js 添加代理设置..."

cat >> "$PROFILE_DIR/prefs.js" <<EOF

// === Burp 代理配置 ===
user_pref("network.proxy.http", "$BURP_PROXY_HOST");
user_pref("network.proxy.http_port", $BURP_PROXY_PORT);
user_pref("network.proxy.ssl", "$BURP_PROXY_HOST");
user_pref("network.proxy.ssl_port", $BURP_PROXY_PORT);
user_pref("network.proxy.type", 1);
user_pref("network.proxy.no_proxies_on", "");
EOF

# ============ 导入证书 ============
echo "✅ [5/5] 使用 certutil 导入 CA 到 Firefox..."

# 检查并安装必要工具
if ! command -v certutil &>/dev/null; then
  echo "🔧 certutil 未安装，正在安装 libnss3-tools..."
  sudo apt update && sudo apt install -y libnss3-tools
fi

certutil -A -n "BurpSuiteCA" -t "C,," -d sql:"$PROFILE_DIR" -i "$CA_CERT_PATH"

echo "✅ 证书导入成功"

# 启动 Firefox
echo "🚀 启动 Firefox + Burp 抓包 profile！"

firefox --no-remote -P "$FIREFOX_PROFILE_NAME" &

echo "🎉 配置完成，开始抓包吧！"
```

---

## ✅ 使用方法

1. **保存脚本为** `burp_firefox_setup.sh`
2. 给它加执行权限：

```bash
chmod +x burp_firefox_setup.sh
```

3. **运行它**：

```bash
./burp_firefox_setup.sh
```

---

## 🧪 效果展示

✅ 自动下载 Burp 的 CA 证书  
✅ 自动创建 Firefox 配置  
✅ 自动配置代理：`127.0.0.1:8080`  
✅ 自动导入证书到 Firefox  
✅ 自动启动 Firefox + 专用抓包 profile

---

## 💡 小贴士

| 场景 | 建议做法 |
|------|---------|
| 你切换多个测试环境 | 使用不同的 Firefox profile |
| 抓不到 HTTPS 请求 | 检查证书是否成功导入 |
| Burp 拦不到请求 | 确保 Intercept 打开且 Firefox 用了代理 |
| 想改代理端口 | 改脚本最前面的变量 `BURP_PROXY_PORT` |

---

要不要我再给你写个 **Curl+Burp脚本抓包攻击演练小测试？** 比如 CSRF 或登录爆破的脚本配套一套给你整整齐齐？🚀