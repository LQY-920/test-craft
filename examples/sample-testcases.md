# 用户登录模块 测试用例文档

> 来源需求: examples/sample-prd.md
> 生成日期: 2026-08-10 | 产品形态: 网站 | 用例数: 11

## 用例列表

### TC-Login_001 登录_有效账号密码_登录成功

- 优先级: P0 | 类型: 接口
- 前置条件: 已注册用户，[username: test@example.com]，[password: Abc12345]，账号未锁定
- 测试数据: [username: test@example.com] [password: Abc12345]
- 测试步骤:
  1. POST /api/login，body 为 `{"username": "test@example.com", "password": "Abc12345"}`
- 预期结果:
  1. 返回 HTTP 200，body 中 code 为 0，data.token 字段非空
- 执行结果: ✅ 通过 (2026-08-10)

### TC-Login_002 登录_密码错误_返回统一错误文案

- 优先级: P1 | 类型: 接口
- 前置条件: 已注册用户，[username: test@example.com]，账号未锁定
- 测试数据: [username: test@example.com] [password: Wrong1234]
- 测试步骤:
  1. POST /api/login，body 为 `{"username": "test@example.com", "password": "Wrong1234"}`
- 预期结果:
  1. 返回 HTTP 200，body 中 code 为 1001，message 为"用户名或密码错误"
- 执行结果: 未执行

### TC-Login_003 登录_用户名不存在_返回统一错误文案

- 优先级: P1 | 类型: 接口
- 前置条件: 不存在用户 ghost@example.com
- 测试数据: [username: ghost@example.com] [password: Abc12345]
- 测试步骤:
  1. POST /api/login，body 为 `{"username": "ghost@example.com", "password": "Abc12345"}`
- 预期结果:
  1. 返回 HTTP 200，body 中 code 为 1001，message 为"用户名或密码错误"
- 执行结果: 未执行

### TC-Login_004 登录_缺少password参数_返回参数缺失

- 优先级: P2 | 类型: 接口
- 前置条件: 无
- 测试数据: [username: test@example.com]
- 测试步骤:
  1. POST /api/login，body 为 `{"username": "test@example.com"}`（不含 password 字段）
- 预期结果:
  1. 返回 HTTP 400，body 中 code 为 4000，message 为"参数缺失"
- 执行结果: 未执行

### TC-Login_005 登录_连续失败5次后_账号锁定10分钟

- 优先级: P2 | 类型: 接口
- 前置条件: 已注册用户，[username: lock@example.com]，[password: Abc12345]，账号未锁定
- 测试数据: [username: lock@example.com] [错误密码: Wrong1234] [有效密码: Abc12345]
- 测试步骤:
  1. POST /api/login，body 为 `{"username": "lock@example.com", "password": "Wrong1234"}`
  2. 重复步骤1到步骤1，重复4次
  3. POST /api/login，body 为 `{"username": "lock@example.com", "password": "Abc12345"}`
- 预期结果:
  1. 返回 HTTP 200，body 中 code 为 1001，message 为"用户名或密码错误"
  2. 每次均返回 code 为 1001
  3. 返回 code 为 1002，message 为"账号已锁定，请10分钟后再试"
- 执行结果: 未执行

### TC-Login_006 登录_密码含注入字符_返回统一错误文案

- 优先级: P3 | 类型: 接口
- 前置条件: 已注册用户，[username: test@example.com]，账号未锁定
- 测试数据: [username: test@example.com] [password: ' or 1=1--]
- 测试步骤:
  1. POST /api/login，body 为 `{"username": "test@example.com", "password": "' or 1=1--"}`
- 预期结果:
  1. 返回 HTTP 200，body 中 code 为 1001，message 为"用户名或密码错误"；服务端无 5xx 错误
- 执行结果: 未执行

### TC-Login_007 登录页_登录成功_跳转dashboard

- 优先级: P0 | 类型: 页面
- 前置条件: 浏览器已打开 /login；已注册用户，[username: test@example.com]，[password: Abc12345]，账号未锁定
- 测试数据: [username: test@example.com] [password: Abc12345]
- 测试步骤:
  1. 在用户名输入框输入 test@example.com
  2. 在密码输入框输入 Abc12345
  3. 单击"登录"按钮
- 预期结果:
  1. 用户名输入框显示已输入内容
  2. 密码以掩码显示
  3. 页面跳转到 /dashboard
- 执行结果: 未执行

### TC-Login_008 登录页_登录失败_按钮恢复并显示错误文案

- 优先级: P1 | 类型: 页面
- 前置条件: 浏览器已打开 /login；已注册用户，[username: test@example.com]，账号未锁定
- 测试数据: [username: test@example.com] [password: Wrong1234]
- 测试步骤:
  1. 在用户名输入框输入 test@example.com
  2. 在密码输入框输入 Wrong1234
  3. 单击"登录"按钮
  4. 等待接口返回
- 预期结果:
  1. 用户名输入框显示已输入内容
  2. 密码以掩码显示
  3. "登录"按钮进入 loading 态且不可点击
  4. 按钮恢复可点击；密码框下方显示"用户名或密码错误"；页面停留在 /login
- 执行结果: ❌ 失败 (2026-08-10) | 实际: 错误文案显示为"登录失败" | 缺陷: 前端未使用接口返回的 message 字段，而是写死文案

### TC-Login_009 登录页_输入框回车_提交登录

- 优先级: P2 | 类型: 页面
- 前置条件: 浏览器已打开 /login；已注册用户，[username: test@example.com]，[password: Abc12345]，账号未锁定
- 测试数据: [username: test@example.com] [password: Abc12345]
- 测试步骤:
  1. 在用户名输入框输入 test@example.com
  2. 在密码输入框输入 Abc12345
  3. 在密码输入框按回车键
- 预期结果:
  1. 用户名输入框显示已输入内容
  2. 密码以掩码显示
  3. 页面跳转到 /dashboard
- 执行结果: ⛔ 阻塞 (2026-08-10) | 原因: WebBridge 无法向输入框派发可信回车键事件，需人工执行本条

### TC-Login_010 登录态_token过期访问dashboard_跳回登录页

- 优先级: P1 | 类型: 页面
- 前置条件: 已成功登录过一次；设置 localStorage 中 token 为 expired-token-value
- 测试数据: [token: expired-token-value]
- 测试步骤:
  1. 在地址栏访问 /dashboard
- 预期结果:
  1. 页面跳转到 /login
- 执行结果: 未执行

### TC-Login_011 登录态_登录成功_token存入localStorage

- 优先级: P1 | 类型: 页面
- 前置条件: 浏览器已打开 /login；已注册用户，[username: test@example.com]，[password: Abc12345]，账号未锁定
- 测试数据: [username: test@example.com] [password: Abc12345]
- 测试步骤:
  1. 在用户名输入框输入 test@example.com
  2. 在密码输入框输入 Abc12345
  3. 单击"登录"按钮
- 预期结果:
  1. 用户名输入框显示已输入内容
  2. 密码以掩码显示
  3. 检查 localStorage 中键 token 的值非空
- 执行结果: 未执行

## 需求覆盖矩阵

| 功能点 | 来源章节 | 用例编号 |
|--------|----------|----------|
| 账号密码登录（成功/失败/锁定/参数校验） | PRD 3.1 | TC-Login_001~006 |
| 登录页交互（跳转/错误提示/回车提交） | PRD 3.2 | TC-Login_007~009 |
| 登录态保持与过期处理 | PRD 3.3 | TC-Login_010~011 |

## 执行汇总

- 总数 11 | 通过 1 | 失败 1 | 阻塞 1 | 未执行 8 | 通过率 33%（已执行 3 条）
- 缺陷清单:
  1. TC-Login_008：登录失败时前端显示"登录失败"，未使用接口返回的 message"用户名或密码错误"（实际 vs 预期：文案不一致）
