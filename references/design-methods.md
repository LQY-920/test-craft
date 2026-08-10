# Test Case Design Methods

Load this file during GENERATE step 4 (用例设计). Each method below turns one test point into concrete cases. Generated cases are in Chinese; examples below show the expected style.

## 1. 等价类划分 (Equivalence partitioning)

Divide all inputs into classes where every member behaves the same; pick one representative per class. Always design both:

- 有效等价类: inputs the system should accept
- 无效等价类: inputs the system should reject

Example — password field requires 8-20 chars of letters+digits:

| Class | Type | Representative |
|-------|------|----------------|
| 8-20 位字母+数字 | 有效 | `abc12345` |
| <8 位 | 无效 | `ab12` |
| >20 位 | 无效 | `abc12345abc12345abc12` |
| 纯字母 | 无效 | `abcdefgh` |
| 纯数字 | 无效 | `12345678` |
| 含特殊字符 | 无效 | `abc123!@` |

Note: method names and class labels above are teaching vocabulary. Per writing-rules §11 they must not appear in generated case text — always instantiate concrete values.

## 2. 边界值分析 (Boundary value analysis)

Defects cluster at boundaries. For every range or length limit, test: 边界值、边界值+1、边界值-1、最大个数、最大个数+1、最小个数、最小个数-1.

Example — field allows 8-20 characters → test lengths: 7, 8, 9, 19, 20, 21.

Also cover: 空值、空表、0 值、负数、非法字符、日期时间边界（跨零点、跨年度）、数值精度与汇总误差。

## 3. 错误推测 (Error guessing)

From experience, target likely defects. Common triggers for web/mini-program/desktop projects:

- 并发重复提交（快速双击提交按钮、重复点击支付）
- 登录态过期后继续操作
- 直接访问需要权限的 URL（越权）
- 刷新/浏览器前进后退后的状态
- 网络中断后恢复
- SQL/XSS 特殊字符输入（`' or 1=1--`、`<script>alert(1)</script>`）

## 4. 场景法 (Scenario testing)

From the requirement's basic flow and alternative flows, build one case per path. Use for multi-step business processes (注册-登录-下单-支付).

- 基本流: the happy path end to end
- 备选流: each branch (库存不足、支付失败重试、取消订单)

## 5. 判定表 (Decision table)

Use when output depends on a combination of conditions. List condition stubs, enumerate meaningful combinations, one case per column.

Example — 登录: 账号存在? × 密码正确? × 账号锁定? →

| 账号存在 | 密码正确 | 已锁定 | 预期 |
|----------|----------|--------|------|
| 是 | 是 | 否 | 登录成功 |
| 是 | 是 | 是 | 提示已锁定 |
| 是 | 否 | 否 | 提示密码错误，失败次数+1 |
| 否 | - | - | 提示用户名或密码错误 |

## Selection guidance

- Every test point: at least one positive case + necessary negative cases.
- Input fields → 等价类 + 边界值 first.
- Multi-condition logic → 判定表.
- Multi-step flows → 场景法.
- Layer 错误推测 on top, applying the triggers listed above where they are relevant to the test point.
- Merge cases that would be duplicates or full equivalents (去重).
