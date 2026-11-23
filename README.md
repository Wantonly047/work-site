# 工作记录台

一个带登录和云数据库的工作网站。前端是静态文件，放在 GitHub 仓库里做版本管理；登录、数据存储由 WorkBuddy 云服务提供。

## 架构

- 前端：`index.html`（无构建步骤，通过 CDN 加载 WorkBuddy Cloud SDK）
- 后端：WorkBuddy 云服务（用户认证 + PostgreSQL 数据库）
- 数据表：`work_notes`，开启行级安全（RLS），每个用户只能读写自己的记录
- 代码托管：GitHub 仓库；协作通过 Issue / Pull Request / 分支保护

## 本地查看

```bash
cd work-site
python -m http.server 8080
```

浏览器打开 `http://localhost:8080`。注意：手机验证码和微信登录需要在正式发布的 HTTPS 域名上才能完整验证，本地预览主要用于看界面。

## 放到 GitHub

```bash
git init
git add .
git commit -m "init work site"
git remote add origin https://github.com/<你的用户名>/<仓库名>.git
git branch -M main
git push -u origin main
```

后续修改走Pull Request：

```bash
git checkout -b feature/xxx
# 修改代码
git add . && git commit -m "describe change"
git push -u origin feature/xxx
# 在 GitHub 上开 Pull Request，合并到 main
```

建议在 GitHub 仓库 Settings → Branches 给 `main` 加分支保护，并要求 PR 通过检查后再合并。仓库里带了 `.github/workflows/ci.yml`，会在 push 和 PR 时做基础检查。

## 发布

这个应用有后端和登录，推荐用 WorkBuddy 发布，并且必须复用创建云服务时的同一个 applicationId（`wbapp_5Th4W1gnzzjiFcsN9W2hTo`），否则域名变化会导致登录因 Origin 不匹配而失败。

GitHub Pages 可以托管纯静态页面，但**不能**直接承载这个带登录和数据库的版本：云服务校验 Origin，GitHub Pages 的域名不是该应用绑定的域名，验证码/微信登录和数据库请求会被拒绝。GitHub 在这里的角色是代码托管、版本历史和团队协作。

## 登录方式

按通用登录场景默认提供：

- 手机号 + 短信验证码登录/注册（无密码）
- 微信登录（网站授权/扫码）

需要在 WorkBuddy 管理面板打开对应登录开关；微信登录还要求站点已发布到正式 HTTPS 域名。

## 数据库

表结构（已创建）：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | BIGINT | 自增主键 |
| owner_id | TEXT | 默认 `auth.uid()`，客户端不传 |
| owner_name | TEXT | 昵称快照，可为空 |
| title | TEXT | 标题 |
| content | TEXT | 内容 |
| status | TEXT | draft / done |
| created_at | TIMESTAMPTZ | 创建时间 |

RLS 策略：`SELECT/INSERT/UPDATE/DELETE` 均限制 `owner_id = auth.uid()`，跨用户访问返回空结果或错误。

## 配置说明

`index.html` 里的 `endpoint`、`oauthRelayBaseUrl`、`publishableKey` 来自云服务的 publicConfig，属于前端可公开的标识，不包含长期密钥。不要提交任何服务端密钥到仓库。
