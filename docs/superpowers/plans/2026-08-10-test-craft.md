# test-craft Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the open-source `test-craft` skill that generates standardized Chinese test-case documents from requirement docs (PRD / superpowers spec / plan) and defines how Kimi Code executes them case-by-case with result backfill.

**Architecture:** Single skill, two modes (GENERATE / EXECUTE). `SKILL.md` (English) holds frontmatter, routing, the 5-step GENERATE pipeline, and the EXECUTE contract; details load on demand from `references/`; the Chinese document template lives in `templates/`; a worked example lives in `examples/`. Spec: `docs/superpowers/specs/2026-08-10-test-craft-design.md`.

**Tech Stack:** Markdown files only. No build, no code, no dependencies. Git for version control. Verification = file-existence checks, structural greps, and a final consistency review (no test framework applies to a pure-content skill).

**Working directory:** `D:/test-craft` (git repo already initialized, design spec committed).

---

### Task 1: 用例文档模板 `templates/test-case-document.md`

**Files:**
- Create: `templates/test-case-document.md`

- [ ] **Step 1: Write the template file**

Create `templates/test-case-document.md` with exactly this content:

````markdown
# <模块/特性名称> 测试用例文档

> 来源需求: <需求文档路径，如 docs/superpowers/specs/2026-08-10-xxx-design.md>
> 生成日期: <YYYY-MM-DD> | 产品形态: <网站|小程序|桌面程序> | 用例数: <N>
> 来源章节基线: <需求文档版本或 commit，可选>

## 用例列表

<!--
每条用例的完整字段如下（注释为填写说明，生成正式文档时删除本注释块）：
- 编号: TC-<特性英文名>_<3位序号>，全局唯一
- 标题: 功能_条件_观察点，动宾结构，≤40字符，无模糊词
- 优先级: P0 冒烟(~10%) / P1 主流程正向(~30%) / P2 校验与边界(~40%) / P3 异常低频(~20%)
- 类型: 接口 | 页面
- 前置条件: 环境、账号权限、数据准备；具体可执行，不依赖其他用例
- 测试数据: [字段名: 字段值] 格式逐项列出
- 测试步骤: 每步一个动作，≤7步，不含结果断言
- 预期结果: 与步骤一一对应，可判定通过/失败
-->

### TC-<Feature>_001 <标题：功能_条件_观察点>

- 优先级: <P0|P1|P2|P3> | 类型: <接口|页面>
- 前置条件: <执行前必须满足的状态>
- 测试数据: [<字段名>: <字段值>]
- 测试步骤:
  1. <操作步骤，每步一个动作>
- 预期结果:
  1. <与步骤对应、可判定的预期>
- 执行结果: 未执行

## 需求覆盖矩阵

| 功能点 | 来源章节 | 用例编号 |
|--------|----------|----------|
| <功能点名称> | <需求文档章节号> | TC-<Feature>_001~00N |

## 执行汇总

- 总数 <N> | 通过 - | 失败 - | 阻塞 - | 未执行 <N> | 通过率 -
- 缺陷清单: 无
````

- [ ] **Step 2: Verify structure**

Run: `grep -c "^## " templates/test-case-document.md`
Expected: `3`（用例列表 / 需求覆盖矩阵 / 执行汇总）
Run: `grep -n "执行结果: 未执行" templates/test-case-document.md`
Expected: 1 match

- [ ] **Step 3: Commit**

```bash
git add templates/test-case-document.md
git commit -m "feat: add test case document template"
```

---

### Task 2: 编写规范细则 `references/writing-rules.md`

**Files:**
- Create: `references/writing-rules.md`

- [ ] **Step 1: Write the rules file**

Create `references/writing-rules.md` (English body, Chinese terms quoted where they appear in generated documents) with exactly this content:

````markdown
# Test Case Writing Rules

Distilled from the Huawei CodeArts TestPlan "测试用例编写规范" (usermanual-testman/cloudtest_01_1301..1326). These rules are MANDATORY for every generated test case. Generated documents are written in Chinese; rule names below quote the Chinese terms the document must use.

## 1. General principles

- A test case is a set of inputs, execution conditions and expected results designed for a specific purpose. Cases must be precise (精准), concise with a single test logic (简洁单一), understandable (易懂), verifiable (易确认), consistent in style (风格一致) and vocabulary (用词一致), and free of duplicates (去重).
- Cases must be independent of each other. Never use another case as a precondition; using another case's outcome as a precondition is allowed.
- One case covers ONE test logic. If a flow needs more than 7 steps, split it into multiple cases.
- Do not explain basic product/testing knowledge inside a case.

## 2. Numbering (编号)

- Format: `TC-<Feature>_<NNN>` where `<Feature>` is the feature's English name from the feature tree (translate if the PRD is Chinese), `<NNN>` is a 3-digit sequence starting at 001.
- Globally unique within the document. Total length < 40 characters. Separator is `_`.

## 3. Naming (标题)

- Required, ≤ 40 characters, no vague wording.
- Structure: `功能_条件_观察点` (intent_precondition_observation), verb-object phrasing, segments joined by `_`.
- Unique within the feature: no duplicates, no containment relations, no names differing only by a sequence number or special character.
- No special characters other than `_`. Never reference unstable UI positions ("左上角" etc.) — use the control's objective name ("新建按钮").

## 4. Priority (优先级)

| Level | Meaning | Share |
|-------|---------|-------|
| P0 | Smoke: most basic function verification of the module | ~10% |
| P1 | Core path, positive flow with positive data | ~30% |
| P2 | Data validation, defaults, boundary values | ~40% |
| P3 | Exceptional flows, abnormal data, rarely-used functions | ~20% |

Priority reflects the feature's importance to the system, not the story's development priority.

## 5. Preconditions (前置条件)

- Every case should state its preconditions: environment, account type and permissions, data preparation.
- Concrete and actionable, never abstract. Bad: "XXX SDK、python 及 adb 已正常安装". Good: "python 安装版本为 3.4 以上；adb 版本为 1.3.6 以上".
- If steps depend on data, that data MUST appear in preconditions.

## 6. Steps (测试步骤)

- Required. One action per step; do not bundle multiple operations.
- ≤ 7 steps per case; split when longer.
- Instantiate every operation object and method so a stranger can execute without asking: page entry, link/button names written in full. Bad: "创造条件，使得A模块异常". Good: "拔掉A模块所在机器的网线".
- Parameters must be written out explicitly — never write just "边界值", "非法值", "遍历所有字符" as the instruction.
- Steps contain NO result assertions. Assertions belong to 预期结果 only.
- Page cases must state the entry path and exact control names.

## 7. Expected results (预期结果)

- Required. One expected result per step, numbering aligned with the step it checks. When one result has several checkpoints, list them all.
- Must be decidable pass/fail. Forbidden: "无错误", "无异常", "正常", "正确" without criteria.
- API cases: state the HTTP status code and key response fields/values, e.g. "返回 HTTP 200，body 中 code=0，token 字段非空".
- Page cases: state the exact message copy, data change, or route, e.g. "页面显示'用户名或密码错误'提示；停留在 /login 不跳转".
- Storage checks: name the table and key field value changes. Message checks: name the key content.
- No operation steps inside expected results.

## 8. Test data (测试数据)

- Listed in the case's own 测试数据 field, format: `[字段名: 字段值]`. Steps reference the data concretely (the actual value appears in the step where it is used).

## 9. Grammar patterns (语法句式)

| Category | Pattern | Example |
|----------|---------|---------|
| 操作类 (steps) | `[操作者][动作][对象][参数]` | 用户输入手机号 13800000000 |
| 赋值类 (preconditions) | `设置[对象][属性]为[参数]` | 设置登录失败锁定阈值为 5 次 |
| 检查类 (expected results) | `检查[对象][属性]为[参数]` | 检查响应字段 code 为 0 |
| 重复类 | `重复步骤[X]到步骤[Y]，重复N次` | 重复步骤2到步骤3，重复5次 |

Test logic and test data must be separated (data goes into 测试数据, not hard-coded prose).

## 10. Reserved words (保留字)

Use these consistently; avoid the listed variants:

| Use | Do NOT use |
|-----|-----------|
| 检查 | 观察、查询、确认、查看 |
| 设置 | 赋值、给予、标记 |
| 执行 | 运行、操作 |
| 重复 | 反复、循环 |

## 11. Fuzzy-word blacklist (模糊词黑名单)

These words must never appear in a generated case; replace with concrete values:

很多、一些、部分、大量、少量、多次、设备无异常、随机、任意、一段时间、一会、一点、特殊、很长、较长、很短、较短、多个、几块、多条、数次、变差、所有、大概、频繁、大约、某些、计数正常、长时间、多端口、异常、错误信息、死循环、正常、最大、最小、非法报文、非法字符、超长、超小、超短、长包、短包、边界值、过低、过高、全部、缺省、默认、各模块、可能、有的、一定范围、适量、多于、少于、左右、上下、假如、或许

## 12. Self-check before delivery (自检清单)

After generating the document, verify every case and fix violations on the spot:

1. 编号唯一且符合 `TC-<Feature>_<NNN>`；标题 ≤40 字符、动宾结构、无黑名单词。
2. 步骤 ≤7 步、每步单一动作、步骤内无断言；预期结果与步骤一一对应。
3. 每条预期结果可明确判定通过/失败（接口有状态码+字段；页面有具体文案/数据/路由）。
4. 前置条件具体可执行；测试数据以 `[字段名: 字段值]` 列出。
5. 优先级分布大致符合 10/30/40/20；需求覆盖矩阵覆盖需求文档全部功能点。
6. 全文无模糊词黑名单词汇；保留字统一（检查/设置/执行/重复）。
````

- [ ] **Step 2: Verify structure**

Run: `grep -c "^## " references/writing-rules.md`
Expected: `12`
Run: `grep -n "TC-<Feature>_<NNN>" references/writing-rules.md | head -3`
Expected: at least 1 match

- [ ] **Step 3: Commit**

```bash
git add references/writing-rules.md
git commit -m "feat: add test case writing rules reference"
```

> 修订记录（质量评审后）：§1 增需求覆盖矩阵要求；§2 长度改为 ≤40；§4 增 类型 声明要求；§7/§8/§9 测试数据表述对齐模板独立字段；§11 黑名单补"正确"；§12 自检清单扩为 8 条。以仓库中 `references/writing-rules.md` 最终内容为准。

---

### Task 3: 用例设计方法 `references/design-methods.md`

**Files:**
- Create: `references/design-methods.md`

- [ ] **Step 1: Write the methods file**

Create `references/design-methods.md` with exactly this content:

````markdown
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
| >20 位 | 无效 | 21 位串 |
| 纯字母 / 纯数字 / 含特殊字符 | 无效 | `abcdefgh` / `12345678` / `abc123!@` |

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
- Always layer 错误推测 on top for the triggers listed above.
- Merge cases that would be duplicates or full equivalents (去重).
````

- [ ] **Step 2: Verify structure**

Run: `grep -c "^## " references/design-methods.md`
Expected: `6`（5 个方法 + Selection guidance）

- [ ] **Step 3: Commit**

```bash
git add references/design-methods.md
git commit -m "feat: add test design methods reference"
```

---

### Task 4: 产品形态检查清单 `references/checklists.md`

**Files:**
- Create: `references/checklists.md`

- [ ] **Step 1: Write the checklists file**

Create `references/checklists.md` with exactly this content:

````markdown
# Product-Form Checklists

Load during GENERATE step 2 (测试点提取). Pick the section matching the project's 产品形态; turn applicable items into test points. These are the pragmatic subset of ISO/IEC 25010 for solo/small-team projects — NOT the full 9-characteristic matrix. Skip items the requirement doc explicitly rules out, and note why in the 测试点清单.

## 网站 (Website)

- 浏览器兼容: Chrome、Edge 最新版（Safari 视用户群）
- 响应式: 需求涉及的关键页面在 1920×1080 与 1366×768 下布局不破
- 鉴权与会话: 未登录访问受限 URL 跳登录页；登录态过期后操作被拦截；退出登录后回退浏览器不能看到受限内容
- 越权: 普通用户直接访问管理端 URL/接口被拒绝
- 浏览器行为: 刷新、前进、后退后页面状态正确
- 重复提交: 表单快速双击、网络慢时重复点击只产生一条记录
- 响应时间: 关键接口/页面在可接受阈值内（默认页面 < 3s，接口 < 1s，需求另有约定从其约定）

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

- 认证: 无 token / 过期 token / 伪造 token 调用受保护接口返回 401
- 越权: 用 A 用户身份请求 B 用户的资源被拒绝
- 参数校验: 缺必填参数、参数类型错误、超长字符串、SQL/XSS 注入字符
- 幂等: 同一请求重复提交不产生重复数据
- 状态码语义: 业务失败返回约定的错误码结构，不是一律 200
````

- [ ] **Step 2: Verify structure**

Run: `grep -c "^## " references/checklists.md`
Expected: `4`

- [ ] **Step 3: Commit**

```bash
git add references/checklists.md
git commit -m "feat: add product-form checklists reference"
```

---

### Task 5: 主文件 `SKILL.md`

**Files:**
- Create: `SKILL.md`

- [ ] **Step 1: Write SKILL.md**

Create `SKILL.md` with exactly this content:

````markdown
---
name: test-craft
description: Use when you need to write standardized project test cases from a requirements document (PRD, spec, or implementation plan), or execute an existing test-case document and record results. Covers API tests and UI/integration tests for websites, mini-programs, and desktop apps.
---

# test-craft

Turn a requirements document into a professional, executable test-case document — then execute it case by case and backfill results. Generated documents are written in **Chinese** (matching the requirement doc); this skill file is English.

**Announce at start:** "I'm using the test-craft skill to <generate test cases | execute the test document>."

## Mode routing

```
if the user points at an EXISTING test-case document and wants it executed → EXECUTE mode
elif a requirement doc is provided or discoverable → GENERATE mode
else → ask the user for the input document; never invent requirements
```

Requirement doc discovery order (GENERATE): explicit path from the user → `docs/superpowers/specs/` → `docs/superpowers/plans/` → PRD/requirement files in the repo root. If several candidates exist, list them and let the user pick. If the requirement is ambiguous or incomplete, ask clarifying questions BEFORE writing cases.

## GENERATE mode

Five steps:

1. **需求解析** — Read the requirement doc fully. Build the feature tree (一级模块 → 二级模块 → 功能点). Tag every 功能点 with its source section for traceability.
2. **测试点提取** — For each 功能点 extract test points: positive flow, negative/exception, boundaries, permissions, state transitions. Load `references/checklists.md` and add implicit test points for the product form (网站 / 小程序 / 桌面程序), noting skipped items and why.
3. **测试点确认** — Present the feature tree + test-point list to the user for confirmation BEFORE expanding into cases. Skip only if the user says to generate directly.
4. **用例设计** — Load `references/design-methods.md`. Expand each test point into cases (equivalence partitioning, boundary values, error guessing, scenario, decision table). Every test point gets at least one positive case plus the necessary negative ones. Load `references/writing-rules.md` and obey it for numbering, naming, steps, expected results, and vocabulary.
5. **输出与自检** — Write the document from `templates/test-case-document.md` to `docs/tests/YYYY-MM-DD-<feature>-testcases.md`. Every case starts with `执行结果: 未执行`. Then run the self-check list in `references/writing-rules.md` §12 and fix violations. The document MUST end with the 需求覆盖矩阵 proving every 功能点 is covered.

## EXECUTE mode

Execution contract — follow it exactly:

**Dispatch by case type:**
- `类型: 接口` → fire real requests (curl / scripts) and assert every expected result: status code, key fields and values.
- `类型: 页面` → drive the user's real browser via Kimi WebBridge: navigate, fill, click, snapshot to assert copy / elements / routes; screenshot and inspect for layout checkpoints.
- If a precondition is unmet, set up data/environment first. If that is impossible, mark the case 阻塞 with the reason — never skip silently, never pretend to have executed.

**Backfill immediately after each case**, replacing `执行结果: 未执行` with one of:

```
- 执行结果: ✅ 通过 (YYYY-MM-DD)
- 执行结果: ❌ 失败 (YYYY-MM-DD) | 实际: <what actually happened> | 缺陷: <defect summary>
- 执行结果: ⛔ 阻塞 (YYYY-MM-DD) | 原因: <why it cannot run>
```

**Discipline:**
- Execute exactly what the document says. If a step is wrong or the environment diverges, mark 阻塞 and record it — do NOT edit the document's steps on the spot. Document fixes go through GENERATE mode (single responsibility).
- For every ❌ case, append one line to 执行汇总 → 缺陷清单: case id, symptom, actual vs expected.
- After the run, update the 执行汇总 counts (通过/失败/阻塞/未执行) and 通过率.
- Default order: P0 → P3. The user may scope the run to a module or priority.
- Resume support: "继续执行 <文档>" picks up at the first 未执行 case.
````

- [ ] **Step 2: Verify frontmatter and structure**

Run:

```bash
python -c "
import re
t = open('SKILL.md', encoding='utf-8').read()
m = re.match(r'^---\n(.*?)\n---\n', t, re.S)
assert m, 'frontmatter missing'
fm = m.group(1)
assert 'name: test-craft' in fm, 'name missing'
assert 'description:' in fm, 'description missing'
assert '## GENERATE mode' in t and '## EXECUTE mode' in t and '## Mode routing' in t
print('SKILL.md OK')
"
```

Expected: `SKILL.md OK`

- [ ] **Step 3: Verify referenced files exist**

Run: `ls references/writing-rules.md references/design-methods.md references/checklists.md templates/test-case-document.md`
Expected: all four listed, no error

- [ ] **Step 4: Commit**

```bash
git add SKILL.md
git commit -m "feat: add SKILL.md with generate/execute modes"
```

---

### Task 6: 示例 PRD `examples/sample-prd.md`

**Files:**
- Create: `examples/sample-prd.md`

- [ ] **Step 1: Write the sample PRD**

Create `examples/sample-prd.md` with exactly this content:

```markdown
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
```

- [ ] **Step 2: Verify structure**

Run: `grep -c "^### 3\." examples/sample-prd.md`
Expected: `3`

- [ ] **Step 3: Commit**

```bash
git add examples/sample-prd.md
git commit -m "feat: add sample PRD example"
```

---

### Task 7: 示例用例文档 `examples/sample-testcases.md`

**Files:**
- Create: `examples/sample-testcases.md`

- [ ] **Step 1: Write the sample test-case document**

This file demonstrates BOTH the generated state and the post-execution backfilled state: 3 cases executed (one ✅, one ❌, one ⛔), the rest 未执行, summary consistent with those states.

Create `examples/sample-testcases.md` with exactly this content:

````markdown
# 用户登录模块 测试用例文档

> 来源需求: examples/sample-prd.md
> 生成日期: 2026-08-10 | 产品形态: 网站 | 用例数: 10

## 用例列表

### TC-Login_001 登录_正确账号密码_登录成功

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
- 测试数据: [username: lock@example.com] [password: Wrong1234]
- 测试步骤:
  1. POST /api/login 使用错误密码，重复执行本步骤，重复5次
  2. POST /api/login 使用正确密码 `{"username": "lock@example.com", "password": "Abc12345"}`
- 预期结果:
  1. 每次均返回 code 为 1001
  2. 返回 code 为 1002，message 为"账号已锁定，请10分钟后再试"
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
  1. 同步骤1
  2. 同步骤2
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
- 前置条件: 已成功登录过一次；设置 localStorage 中 token 为一个已过期的 token 值
- 测试数据: [token: expired-token-value]
- 测试步骤:
  1. 在地址栏访问 /dashboard
- 预期结果:
  1. 页面跳转到 /login
- 执行结果: 未执行

## 需求覆盖矩阵

| 功能点 | 来源章节 | 用例编号 |
|--------|----------|----------|
| 账号密码登录（成功/失败/锁定/参数校验） | PRD 3.1 | TC-Login_001~006 |
| 登录页交互（跳转/错误提示/回车提交） | PRD 3.2 | TC-Login_007~009 |
| 登录态保持与过期处理 | PRD 3.3 | TC-Login_010 |

## 执行汇总

- 总数 10 | 通过 1 | 失败 1 | 阻塞 1 | 未执行 7 | 通过率 33%（已执行 3 条）
- 缺陷清单:
  1. TC-Login_008：登录失败时前端显示"登录失败"，未使用接口返回的 message"用户名或密码错误"（实际 vs 预期：文案不一致）
````

- [ ] **Step 2: Verify consistency**

Run:

```bash
python -c "
import re
t = open('examples/sample-testcases.md', encoding='utf-8').read()
cases = re.findall(r'^### (TC-Login_\d{3})', t, re.M)
assert len(cases) == 10 and len(set(cases)) == 10, f'case count/unique fail: {cases}'
assert t.count('执行结果: 未执行') == 7, 'unexecuted count mismatch'
assert '✅ 通过' in t and '❌ 失败' in t and '⛔ 阻塞' in t, 'missing backfill samples'
for field in ['优先级:', '前置条件:', '测试数据:', '测试步骤:', '预期结果:', '执行结果:']:
    n = t.count(field)
    assert n >= 10, f'{field} appears {n} times'
print('sample-testcases OK')
"
```

Expected: `sample-testcases OK`

- [ ] **Step 3: Commit**

```bash
git add examples/sample-testcases.md
git commit -m "feat: add sample test-case document with backfill examples"
```

> 修订记录（评审后）：用例扩为 11 条（新增 TC-Login_011 覆盖 PRD 3.3 的 token 存储）；TC-Login_001 标题"正确"改"有效"（黑名单）；TC-Login_005 步骤具体化并使用重复类语法；TC-Login_008 预期结果 1-2 展开为可判定描述；汇总更新为 总数 11 | 未执行 8。以仓库中 `examples/sample-testcases.md` 最终内容为准。

---

### Task 8: README 与 LICENSE

**Files:**
- Create: `README.md`
- Create: `LICENSE`

- [ ] **Step 1: Write LICENSE**

Create `LICENSE` with the standard MIT text (replace `<year>` with `2026`, `<copyright holders>` with `test-craft contributors`):

```
MIT License

Copyright (c) 2026 test-craft contributors

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

- [ ] **Step 2: Write README.md**

Create `README.md` with exactly this content:

````markdown
# test-craft

An agent skill that turns a requirements document (PRD, spec, or implementation plan) into a professional, executable test-case document — and then executes it case by case, backfilling results into the same document.

Built for the [superpowers](https://github.com/obra/superpowers) workflow: `brainstorming` → `writing-plans` → development → **test-craft**.

> 中文简介：test-craft 是一个根据需求文档生成规范测试用例文档的 skill。生成的用例文档（中文）可直接交给 Kimi Code 逐条执行：接口用例发真实请求断言，页面用例通过 Kimi WebBridge 操作真实浏览器，执行结果当场回填到同一份文档，并自动汇总通过率与缺陷清单。规则沉淀自华为 CodeArts《测试用例编写规范》，质量维度取自 ISO/IEC 25010 的高价值子集。

## Features

- **GENERATE mode** — feature tree → test-point extraction → human checkpoint → case design (equivalence partitioning, boundary values, error guessing, scenario, decision table) → standardized Chinese Markdown document with a requirement-coverage matrix.
- **EXECUTE mode** — API cases via real HTTP requests, page cases via [Kimi WebBridge](https://www.kimi.com/features/webbridge); per-case result backfill (✅/❌/⛔), defect list, pass-rate summary, resume support.
- **Product forms** — websites, mini-programs, desktop apps; pragmatic ISO 25010 subset as optional checklists.
- **No platform lock-in** — one Markdown file is both the plan and the report.

## Install

Copy the repository into your agent's skills directory, e.g.:

```bash
git clone <repo-url> ~/.agents/skills/test-craft
```

## Usage

- Generate: "用 test-craft 根据 docs/superpowers/specs/xxx.md 生成测试用例"
- Execute: "用 test-craft 执行 docs/tests/2026-08-10-login-testcases.md"
- Resume: "继续执行 docs/tests/2026-08-10-login-testcases.md"

See `examples/sample-prd.md` → `examples/sample-testcases.md` for a complete before/after pair.

## Repository layout

```
SKILL.md                      # entry point: routing, GENERATE pipeline, EXECUTE contract
references/writing-rules.md   # naming, numbering, steps/expected-results rules, fuzzy-word blacklist
references/design-methods.md  # equivalence partitioning, boundary values, error guessing, scenario, decision table
references/checklists.md      # per-product-form quality checklists (website / mini-program / desktop)
templates/test-case-document.md
examples/                     # sample PRD + generated & backfilled test document
```

## License

MIT — see `LICENSE`.
````

- [ ] **Step 3: Verify**

Run: `head -1 README.md && head -1 LICENSE`
Expected: `# test-craft` and `MIT License`

- [ ] **Step 4: Commit**

```bash
git add README.md LICENSE
git commit -m "docs: add README and MIT license"
```

---

### Task 9: 整体验证与收尾

**Files:**
- Modify (if fixes needed): any file above

- [ ] **Step 1: Full repository structure check**

Run: `git ls-files`
Expected exactly:

```
LICENSE
README.md
SKILL.md
docs/superpowers/plans/2026-08-10-test-craft.md
docs/superpowers/specs/2026-08-10-test-craft-design.md
examples/sample-prd.md
examples/sample-testcases.md
references/checklists.md
references/design-methods.md
references/writing-rules.md
templates/test-case-document.md
```

- [ ] **Step 2: Cross-reference check — every file path mentioned in SKILL.md exists**

Run:

```bash
for f in references/checklists.md references/design-methods.md references/writing-rules.md templates/test-case-document.md; do
  test -f "$f" || echo "MISSING: $f"
done; echo "cross-ref check done"
```

Expected: only `cross-ref check done`, no MISSING lines

- [ ] **Step 3: Fuzzy-word self-audit on the example document**

The generated example must not itself violate the blacklist. Run (the authoritative list is §11 of `references/writing-rules.md`; this audit extracts it programmatically):

```bash
python -c "
import re
rules = open('references/writing-rules.md', encoding='utf-8').read()
m = re.search(r'## 11\.[^\n]*\n\n[^\n]*\n\n([^\n]+)', rules)
blacklist = [w for w in m.group(1).split('、') if w]
t = open('examples/sample-testcases.md', encoding='utf-8').read()
hits = [w for w in blacklist if w in t]
assert '无异常' not in t and '无错误' not in t
print('blacklist hits:', hits if hits else 'none')
"
```

Expected: `blacklist hits: none`. If hits appear, edit the example to use concrete wording and re-run. (Note: the blacklist contains multi-char terms like 错误信息/非法字符 that legitimately appear in checklist/design-method teaching text — this audit applies to the generated example only.)

- [ ] **Step 4: Spec coverage review**

Re-read `docs/superpowers/specs/2026-08-10-test-craft-design.md` sections 4–7 and confirm each maps to a file: routing → SKILL.md; GENERATE 5 steps → SKILL.md; document spec → templates + writing-rules.md; EXECUTE contract → SKILL.md; checklists → references/checklists.md; design methods → references/design-methods.md. Fix any gap found.

- [ ] **Step 5: Final commit (only if steps 3–4 produced changes)**

```bash
git add -A
git commit -m "chore: final consistency fixes from self-audit"
```

Run: `git log --oneline`
Expected: one commit per task (plus the earlier spec and plan commits)
