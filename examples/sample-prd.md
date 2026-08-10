# 示例 PRD：用户登录模块（网站）

## 1. 背景

为示例网站提供账号密码登录能力。本 PRD 仅用于演示 test-craft 的用例生成。

## 2. 术语

- 登录态: 登录成功后服务端签发的 token，有效期 2 小时。

## 3. 功能需求

### 3.1 账号密码登录（接口）

- 接口: `POST /api/login`，body: `{ "username": string, "password": string }`
- 成功: 返回 HTTP 200，`{ "code": 0, "data": { "token": string } }`
- 失败: 返回 HTTP 200，`{ "code": 1001, "message": "用户名或密码错误" }`（用户名不存在与密码错误返回相同文案，防账号枚举）
- 连续失败 5 次后账号锁定 10 分钟，期间即使密码正确也返回 `{ "code": 1002, "message": "账号已锁定，请10分钟后再试" }`
- username/password 为必填，缺失时返回 HTTP 400，`{ "code": 4000, "message": "参数缺失" }`

### 3.2 登录页交互（页面）

- 路由 `/login`，包含用户名输入框、密码输入框（掩码显示）、"登录"按钮
- 点击"登录"后按钮进入 loading 态并禁用，直到接口返回
- 登录成功跳转到 `/dashboard`
- 登录失败在密码框下方显示接口返回的 message 文案
- 在任意输入框按回车等价于点击"登录"

### 3.3 登录态保持

- 登录成功后 token 存入 localStorage，键名 `token`
- 携带过期 token 访问 `/dashboard` 时跳转回 `/login`
