# Product-Form Checklists

Load during GENERATE step 2 (测试点提取). Pick the section matching the project's 产品形态; turn applicable items into test points. These are the pragmatic subset of ISO/IEC 25010 for solo/small-team projects — NOT the full 9-characteristic matrix. Skip items the requirement doc explicitly rules out, and note why in the 测试点清单.

Note: checklist wording is test-point vocabulary, not case text. When expanding into cases, per writing-rules §11 instantiate concrete, decidable wording (e.g. turn "页面状态正确" into the specific expected state).

## 网站 (Website)

- 浏览器兼容: Chrome、Edge 最新版（Safari 视用户群）
- 响应式: 需求涉及的关键页面在 1920×1080 与 1366×768 下布局不破
- 鉴权与会话: 未登录访问受限 URL 跳登录页；登录态过期后操作被拦截；退出登录后回退浏览器不能看到受限内容
- 越权: 普通用户直接访问管理端 URL/接口被拒绝
- 浏览器行为: 刷新、前进、后退后页面状态正确
- 重复提交: 表单快速双击、网络慢时重复点击只产生一条记录
- 响应时间: 关键接口/页面在可接受阈值内（默认页面 < 3s，接口 < 1s，需求另有约定从其约定）
- 界面检查: 元素显示、文案拼写、交互反馈（加载态/空态/错误提示/禁用态）、跳转与导航

## 小程序 (Mini-program)

- 微信版本与机型: 最新稳定版微信；iOS 与 Android 各至少一款真机
- 授权与登录态: 用户拒绝授权后的降级表现；token 过期自动续期或重登
- 网络异常: 弱网下加载提示与超时处理；断网恢复后数据自动刷新
- 列表交互: 下拉刷新、上拉加载更多、空列表态
- 页面栈: tab 切换、navigateBack 后的页面状态
- 分享: 触发分享的页面标题/路径正确（需求涉及时）
- 支付: 支付取消、支付失败回调（需求涉及支付时）

## 桌面程序 (Desktop app)

- 安装/卸载: 全新安装成功；卸载后无残留进程；覆盖安装（升级）保留用户数据
- 显示: 1920×1080 与 1366×768 分辨率、125%/150% DPI 缩放下布局不破
- 离线行为: 断网时核心功能的表现符合需求
- 文件权限: 无权限目录读写时给出明确提示
- 异常恢复: 进程被杀后重启，未保存数据的处理符合需求
- 窗口行为: 最小化到托盘/任务栏恢复、多显示器拖动（需求涉及时）

## 接口（所有形态通用）

- 认证: 无 token / 过期 token / 伪造 token 调用受保护接口返回 401（需求另有鉴权约定的从其约定）
- 越权: 用 A 用户身份请求 B 用户的资源被拒绝
- 参数校验: 缺必填参数、参数类型错误、超长字符串、SQL/XSS 注入字符
- 幂等: 同一请求重复提交不产生重复数据
- 状态码语义: 业务失败返回约定的错误码结构（HTTP 层与业务码的约定以需求文档为准）
