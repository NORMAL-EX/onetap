# OneTap

> Cloudflare-Turnstile 风格的「点一下」人机验证 · 单文件自托管 · 内嵌 SQLite

OneTap 是一个自托管的人机验证服务:一个独立的 Go 二进制即提供可嵌入的验证组件、公开校验 API、服务端 `/siteverify` 校验端点,以及一个带深浅色 / 多语言的管理后台。

> 📦 **本仓库仅用于分发**:这里只发布**预编译二进制**与 **Docker 镜像**,不包含源代码(项目闭源以降低被逆向风险)。

---

## 反机器人能力

- **服务端二次校验是信任根**:token 由服务器私钥 HMAC 签名、单次有效、短时效、绑定 sitekey,客户端无法伪造或重放。
- **每 6 小时轮换的字节码 VM + 每 epoch 多态混淆 WASM 求解器(WASM-only)**:服务器**不下发任何明文算法或字节码**,答案只能由执行随 epoch 自动换新、自动混淆的 WebAssembly 求解器得出 —— 纯脚本没有可照抄的算法。
- **最基本的浏览器环境绑定**:无 DOM / 空 userAgent 的非浏览器请求直接拒绝(误判率极低)。
- **自适应难度 + IP 风险段(CIDR) + 行为评分 + 限频 + 审计日志**。

> ⚠️ 与所有客户端验证码一样,无法 100% 阻止驱动真实浏览器的高级自动化 / AI Agent。最终判定务必在你的**后端**调用 `/siteverify`。

---

## 快速开始

### Docker(推荐)

```bash
docker run -d --name onetap \
  -p 8080:8080 \
  -v onetap-data:/data \
  -e CS_MASTER_SECRET=$(openssl rand -hex 32) \
  ghcr.io/normal-ex/onetap:latest
```

首次启动会在日志中打印随机生成的 `admin` 密码。管理后台:`http://localhost:8080/admin/`。

### 预编译二进制

从 [Releases](https://github.com/NORMAL-EX/onetap/releases) 下载对应平台的压缩包,解压后运行:

```bash
./onetap --addr :8080 --db ./onetap.db
```

支持平台:Linux(amd64/arm64)、macOS(amd64/arm64)、Windows(amd64)。

---

## 接入(三步)

1. 后台「站点 / 接入」新建站点,拿到 `sitekey`(公开)与 `secret`(私密)。
2. 前端引入组件:

```html
<script src="https://<你的部署域名>/widget.js" defer></script>
<div class="cs-captcha" data-sitekey="你的_sitekey"></div>
```

组件会把 token 写入表单隐藏字段 `cs-token`(可配置)。

3. 后端用 `secret` 校验 token(单次有效):

```bash
curl -X POST https://<你的部署域名>/siteverify \
  -d "secret=你的_secret" \
  -d "response=前端拿到的_token"
# => {"success":true,...}
```

---

## 常用配置

| 环境变量 / 参数 | 说明 |
| --- | --- |
| `CS_MASTER_SECRET` | **生产必填**:hex 编码、≥32 字节的主密钥(信任根,独立于数据库)。 |
| `CS_TRUSTED_PROXIES` | 受信任反向代理的 CIDR;仅此时才采信 `X-Forwarded-For` / `X-Real-IP`。 |
| `--addr` / `CS_ADDR` | 监听地址(默认 `:8080`)。 |
| `--db` / `CS_DB` | SQLite 数据库路径。 |
| `--reset-password` | 重置 admin 密码并打印,然后退出。 |
| `--version` | 打印版本与构建日期。 |

> 接入页若启用严格 CSP,需放行 `script-src 'wasm-unsafe-eval'`(求解器为 WASM-only)。

---

## License

MIT License。

© 2026-Present Cloud-PE Dev.
© 2026-Present NORMAL-EX (dddffgg)
