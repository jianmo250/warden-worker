# Warden：一个运行在 Cloudflare Workers 上的 Bitwarden 兼容服务器

本项目提供了一个自托管、兼容 Bitwarden 的服务器，可以**免费**部署在 Cloudflare Workers 上。它专为低维护设计，让你实现“一键部署，高枕无忧”，无需担心服务器管理或持续性费用。

## 为什么选择这个项目？

虽然像 [Vaultwarden](https://github.com/dani-garcia/vaultwarden) 这样的项目提供了优秀的自托管方案，但它们仍然需要你管理服务器或 VPS。这可能会带来麻烦，而且如果你忘记为服务器续费，可能会失去对密码的访问权限。

Warden 旨在通过利用 Cloudflare Workers 生态系统来解决这个问题。通过将 Warden 部署到 Cloudflare Worker 并使用 Cloudflare D1 进行存储，你可以拥有一个完全免费、无服务器且低维护的 Bitwarden 服务器。

## 功能特性

- **核心保险库功能：** 创建、读取、更新和删除密码条目（Ciphers）及文件夹。
    
- **文件附件：** 可选支持 Cloudflare R2 存储附件。
    
- **TOTP 支持：** 存储并生成二次验证动态密码（TOTP）。
    
- **兼容 Bitwarden：** 支持官方 Bitwarden 客户端。
    
- **免费托管：** 运行在 Cloudflare 的免费层级（Free Tier）内。
    
- **低维护：** 部署一次即可，几乎不需要维护。
    
- **安全：** 你的加密数据存储在你自己的 Cloudflare D1 数据库中。
    
- **易于部署：** 使用 Wrangler CLI 几分钟内即可完成部署。
    

### 附件支持

Warden 支持使用 Cloudflare R2 存储文件附件。附件功能是可选的，需要手动配置才能开启。详见部署指南。请注意，R2 可能会产生额外费用；请参考 [Cloudflare R2 价格页面](https://developers.cloudflare.com/r2/pricing/)。

## 项目现状

**本项目尚未实现全部功能**，~~且可能永远不会完全实现~~。目前它支持个人保险库的核心功能（包括 TOTP）。但是，它**不支持**以下功能：

- 分享/协作
    
- 2FA 登录（除 TOTP 外）
    
- Bitwarden Send
    
- 设备和会话管理
    
- 紧急访问
    
- 管理员操作
    
- 组织（Organizations）
    
- 其他 Bitwarden 高级功能
    

目前没有计划实现这些功能。本项目的首要目标是提供一个简单、免费且低维护的个人密码管理器。

## 兼容性

- **浏览器扩展：** Chrome, Firefox, Safari 等（已在 Chrome 2025.11.1 版本测试）。
    
- **Android 应用：** 官方 Bitwarden Android App（已在 2025.11.0 版本测试）。
    
- **iOS 应用：** 官方 Bitwarden iOS App（已在 2025.11.0 版本测试）。
    

## 演示版

演示实例位于：[warden.qqnt.de](https://warden.qqnt.de/)。

你可以使用以 `@warden-worker.demo` 结尾的邮箱注册新账号（邮箱无需验证）。

如果你决定停止使用演示实例，请删除你的账号以腾出空间。

**强烈建议部署你自己的实例**，因为演示版可能会触发 Cloudflare 的频率限制并被禁用。

## 快速入门

- 选择部署路径：[CLI 部署](https://www.google.com/search?q=docs/deployment.md%23cli-deployment) 或 [Github Actions 自动部署](https://www.google.com/search?q=docs/deployment.md%23cicd-deployment-with-github-actions)。
    
- 根据部署文档设置密钥（Secrets）和可选的附件配置。
    
- 配置 Bitwarden 客户端，将其服务器 URL 指向你的 Worker 网址。
    

## 前端 (Web Vault)

前端使用 [Cloudflare Workers Static Assets](https://developers.cloudflare.com/workers/static-assets/) 与 Worker 绑定。GitHub Actions 工作流会自动下载最新的 [bw_web_builds](https://github.com/dani-garcia/bw_web_builds)（Vaultwarden 网页版）并与后端一起部署。

**运行机制：**

- 静态文件（HTML, CSS, JS）由 Cloudflare 边缘网络直接提供。
    
- API 请求（`/api/*`, `/identity/*`）会被路由到 Rust 编写的 Worker。
    
- 无需单独的 Pages 部署或域名配置。
    

> [!NOTE]
> 
> 正在从旧的独立前端部署迁移？如果你之前将前端单独部署到 Cloudflare Pages，现在可以删除 warden-frontend Pages 项目并重新设置 Worker 路由。前端现在已集成在 Worker 中。

> [!WARNING] Web Vault 前端源自 Vaultwarden，因此会显示许多高级 UI 功能，但其中大部分是不可用的。请参考 [项目现状](https://www.google.com/search?q=%23%E9%A1%B9%E7%9B%AE%E7%8E%B0%E7%8A%B6)。

## 配置自定义域名（可选）

默认的 `*.workers.dev` 域名默认是禁用的，因为它可能会抛出 1101 错误。你可以通过在 `wrangler.toml` 中设置 `workers_dev = true` 来启用它。

如果你想使用自定义域名，请按照以下步骤操作：

### 第一步：添加 DNS 记录

1. 登录 [Cloudflare 控制面板](https://dash.cloudflare.com/)。
    
2. 选择你的域名（例如 `example.com`）。
    
3. 前往 **DNS** → **Records**。
    
4. 点击 **Add record**:
    
    - **Type:** `A` (或 `AAAA`)
        
    - **Name:** 你的子域名 (例如 `vault`)
        
    - **IPv4 address:** `192.0.2.1` (这是一个占位符，实际路由由 Worker 处理)
        
    - **Proxy status:** **Proxied** (橙色小云朵 - **必须开启**！)
        
5. 点击 **Save**。
    

> ⚠️ **重要：** **Proxy status 必须为 "Proxied"**。如果显示为 "DNS only"（灰色云朵），Worker 路由将无法生效。

### 第二步：添加 Worker 路由

1. 前往 **Workers & Pages** → 选择你的 `warden-worker`。
    
2. 点击 **Settings** → **Domains & Routes**。
    
3. 点击 **Add** → **Route**。
    
4. 配置路由：
    
    - **Route:** `vault.example.com/*` (替换为你的域名)
        
    - **Zone:** 选择你的域名区域
        
    - **Worker:** `warden-worker`
        
5. 点击 **Add route**。
    

## 内置频率限制 (Rate Limiting)

本项目集成了由 [Cloudflare Rate Limiting API](https://developers.cloudflare.com/workers/runtime-apis/bindings/rate-limit/) 提供的频率限制功能，以保护敏感接口：

|**接口**|**限制频率**|**识别标识**|**目的**|
|---|---|---|---|
|`/identity/connect/token`|5 次/分钟|邮箱地址|防止密码暴力破解|
|`/api/accounts/register`|5 次/分钟|IP 地址|防止批量注册和邮箱穷举|
|`/api/accounts/prelogin`|5 次/分钟|IP 地址|防止邮箱穷举|

你可以在 `wrangler.toml` 中调整频率限制设置：

Ini, TOML

```
[[ratelimits]]
name = "LOGIN_RATE_LIMITER"
namespace_id = "1001"
# 调整限制次数 (limit) 和周期 (period, 10 或 60 秒)
simple = { limit = 5, period = 60 }
```

> [!NOTE] `period` 必须是 `10` 或 `60` 秒。详情参阅 [Cloudflare 文档](https://developers.cloudflare.com/workers/runtime-apis/bindings/rate-limit/)。

如果缺少相关绑定，请求将直接通过而不受限制（优雅降级）。

## 配置项

在 `wrangler.toml` 的 `[vars]` 下配置环境变量，或通过 Cloudflare 控制面板设置：

- **`TRASH_AUTO_DELETE_DAYS`** (可选, 默认: `30`):
    
    - 自动清理回收站的天数。
        
    - 设置为 `0` 或负数则禁用。
        
- **`IMPORT_BATCH_SIZE`** (可选, 默认: `30`):
    
    - 导入/删除操作的批处理大小。
        
    - `0` 禁用批处理。
        
- **`DISABLE_USER_REGISTRATION`** (可选, 默认: `true`):
    
    - 控制是否在客户端 UI 显示注册按钮（不改变服务器实际行为）。
        
- **`AUTHENTICATOR_DISABLE_TIME_DRIFT`** (可选, 默认: `false`):
    
    - 设置为 `true` 则禁用 TOTP 校验的 ±1 个时间步长偏差。
        
- **`ATTACHMENT_MAX_BYTES`** (可选):
    
    - 单个附件的最大字节数。
        
    - 示例：`104857600` 代表 100MB。
        
- **`ATTACHMENT_TOTAL_LIMIT_KB`** (可选):
    
    - 每个用户的附件总存储限制 (KB)。
        
    - 示例：`1048576` 代表 1GB。
        
- **`ATTACHMENT_TTL_SECS`** (可选, 默认: `300`):
    
    - 附件上传/下载 URL 的有效期。
        

### 定时任务 (Cron)

Worker 会运行定时任务来清理被删除的项目。默认每天 UTC 时间 03:00 运行一次。

## 数据库操作

- **备份与恢复：** 参考 [数据库备份与恢复文档](https://www.google.com/search?q=docs/db-backup-recovery.md#github-actions-backups)。
    
- **时间旅行 (Time Travel)：** 参见 [D1 Time Travel](https://www.google.com/search?q=docs/db-backup-recovery.md#d1-time-travel-point-in-time-recovery) 进行数据回滚。
    
- **本地开发：**
    
    - 快速开始：`wrangler dev --persist`
        
    - 在本地查看数据库：SQLite 文件位于 `.wrangler/state/v3/d1/` 下。
        

## 本地开发

使用 Wrangler 在本地运行支持 D1 的 Worker。

**快速开始 (仅 API):**

Bash

```
wrangler dev --persist
```

**全栈开发 (包含 Web Vault):**

1. 下载前端资源。
    
2. 启动本地环境：`wrangler dev --persist`
    
3. 访问 `http://localhost:8787`。
    

## 贡献

欢迎提交 Issue 和 PR。请在提交前运行 `cargo fmt` 和 `cargo clippy --target wasm32-unknown-unknown --no-deps`。

## 许可证

本项目采用 MIT 许可证。详见 `LICENSE` 文件。
