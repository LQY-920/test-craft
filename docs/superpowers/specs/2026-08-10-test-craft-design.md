# test-craft 设计文档（Spec）

- 日期：2026-08-10
- 状态：已确认
- 项目类型：开源 Agent Skill（SKILL.md 格式，兼容 superpowers 生态）

## 1. 背景与目标

用户的开发工作流为：superpowers `brainstorming` 澄清需求 → `writing-plans` 撰写需求/plan 文档 → Kimi Code 开发。测试环节不规范：没有专业的测试用例文档，接口测试与页面联调在会话中临时通过 Kimi WebBridge 完成，无据可依、无法复用、无法回归。

**test-craft** 是一个开源 skill，用于补齐这一环节：

1. **GENERATE**：根据需求文档（superpowers 产出的 spec/plan，或用户自写的 PRD），生成规范的、结构化的中文测试用例文档；
2. **EXECUTE**：Kimi Code 拿到该文档后逐条执行（接口用例发真实请求断言，页面用例通过 WebBridge 操作真实浏览器），当场回填执行结果与缺陷，更新汇总。

写与执行通过同一份 Markdown 文档闭环，不依赖外部测试管理平台。

## 2. 范围

### 覆盖

- 产品形态：网站、小程序、桌面程序
- 用例类型：接口测试、页面/交互测试（含界面检查子类：元素显示、文案、布局/响应式、交互反馈、跳转导航）
- 补充质量维度：按产品形态触发的可选检查清单（兼容性、弱网/离线、鉴权越权、安装卸载、多分辨率等）

### 不覆盖（YAGNI）

- ISO 25010 九特性全矩阵（只保留高价值维度，见 `references/checklists.md`）
- 性能专项（压测）、安全专项（渗透）——仅保留响应时间阈值、鉴权/越权等轻量检查
- 主观 UX 评价（美观度等）——客观显示问题进用例，主观判断留给用户
- 脚本工具、YAML 中间格式、TestLink 等平台对接

## 3. 总体架构

单 skill、双模式。`SKILL.md` 承载主流程与路由；细则按需加载 `references/`；模板与示例独立成文件。

```
test-craft/
├── SKILL.md                      # 英文。触发描述、模式路由、GENERATE 五步、EXECUTE 约定
├── references/
│   ├── writing-rules.md          # 编写规范细则（命名/编号/步骤与结果分离/模糊词黑名单/保留字/语法句式）
│   ├── design-methods.md         # 等价类、边界值、错误推测、场景法、判定表，各配例子
│   └── checklists.md             # 网站/小程序/桌面的补充检查清单
├── templates/
│   └── test-case-document.md     # 中文用例文档模板（四段式）
├── examples/
│   ├── sample-prd.md             # 中文示例 PRD（登录模块）
│   └── sample-testcases.md       # 生成的用例文档，含执行回填后的完整样子
├── README.md                     # 英文为主 + 中文简介段
└── LICENSE                       # MIT
```

语言约定：skill 本体（SKILL.md、references、README）英文撰写、关键术语附中文对照；生成的测试用例文档默认中文（跟随需求文档语言）。

## 4. 触发与路由

SKILL.md frontmatter（英文）：

```yaml
---
name: test-craft
description: Use when you need to write standardized project test cases from a requirements document (PRD, spec, or implementation plan), or execute an existing test-case document and record results. Covers API tests and UI/integration tests for websites, mini-programs, and desktop apps.
---
```

路由逻辑：

```
if 用户提供了已存在的用例文档且意图是执行 → EXECUTE 模式
elif 用户提供了需求文档（或可在仓库内找到 spec/plan/PRD） → GENERATE 模式
else → 先问用户要输入文档，不臆造
```

输入识别（GENERATE）：

- 显式路径：用户直接给的 PRD/spec 文件；
- 隐式发现：按序查找 `docs/superpowers/specs/`、`docs/superpowers/plans/`、仓库根目录 PRD/需求文档；找到多个时列出由用户选择；
- 需求文档不完整/有歧义时，先提问澄清再写用例。

## 5. GENERATE 流程（五步）

1. **需求解析**：通读需求文档，产出特性树（一级模块 → 二级模块 → 功能点），每个功能点标注来源章节，保证可追溯。
2. **测试点提取**：每个功能点提取正向流程、反向/异常、边界、权限、状态流转等测试点；隐性需求（登录态过期、弱网、并发重复提交等）由 `references/checklists.md` 按产品形态触发。
3. **测试点确认（人工 checkpoint）**：特性树 + 测试点清单呈现给用户确认后再展开；用户要求"直接生成"则跳过。
4. **用例设计**：加载 `references/design-methods.md`（等价类、边界值、错误推测、场景法、判定表）展开用例；每个测试点至少一条正向 + 必要反向；遵守 `references/writing-rules.md`。
5. **输出与自检**：按模板写入 `docs/tests/YYYY-MM-DD-<feature>-testcases.md`，跑自检清单（步骤 ≤7、无模糊词、步骤与预期一一对应、编号唯一、优先级分布合理），发现问题当场修正。文档末尾附需求覆盖矩阵。

## 6. 用例文档规范（模板核心）

四段式：文件头 → 用例列表 → 需求覆盖矩阵 → 执行汇总。

```markdown
# 用户登录模块 测试用例文档
> 来源需求: docs/superpowers/specs/2026-08-10-login-design.md
> 生成日期: 2026-08-10 | 产品形态: 网站 | 用例数: 12

## 用例列表

### TC-Login_001 登录_正确账号密码_登录成功
- 优先级: P0 | 类型: 接口
- 前置条件: 已注册用户 test@example.com / Passw0rd
- 测试数据: [username: test@example.com] [password: Passw0rd]
- 测试步骤:
  1. POST /api/login，body 为测试数据
- 预期结果:
  1. 返回 HTTP 200，body 中 code=0，且返回 token 字段非空
- 执行结果: 未执行

## 需求覆盖矩阵
| 功能点 | 来源章节 | 用例编号 |
|--------|---------|---------|
| 账号密码登录 | PRD 3.1 | TC-Login_001~007 |

## 执行汇总
- 总数 12 | 通过 - | 失败 - | 阻塞 - | 未执行 12 | 通过率 -
- 缺陷清单: 无
```

关键规则（完整版在 `references/writing-rules.md`，沉淀自华为 CodeArts 测试用例编写规范）：

- **编号**：`TC-<特性英文名>_<3位序号>`，全局唯一；
- **命名**：`功能_条件_观察点`，动宾结构，≤40 字符，无模糊词，同一特性内唯一；
- **优先级**：P0 冒烟（约 10%）/ P1 主流程正向（约 30%）/ P2 校验与边界（约 40%）/ P3 异常低频（约 20%）；
- **步骤**：每步一个动作，不含结果断言，≤7 步，页面用例写清入口与控件名；
- **预期结果**：可判定通过/失败；接口写状态码 + 关键字段，页面写具体提示文案/数据变化/路由；不允许"无错误"、"正常"这类无法判定的描述；
- **模糊词黑名单**：很多、一些、部分、大量、少量、多次、随机、任意、一段时间、正常、异常、正确、可能、大概……（完整列表进 references）；
- **保留字统一**：检查 / 设置 / 执行 / 重复；步骤用 `[操作者][动作][对象][参数]`，预置用 `设置[对象][属性]为[参数]`，预期用 `检查[对象][属性]为[参数]`；
- **步骤与预期一一对应**，编号一致；一个结果多个检查点时逐条列出；
- **测试数据**：步骤中明确列出，格式 `[字段名: 字段值]`。

## 7. EXECUTE 约定

执行方式按类型分派：

- `类型: 接口` → curl/脚本发真实请求，逐项断言预期结果（状态码、字段值）；
- `类型: 页面` → 通过 Kimi WebBridge 操作真实浏览器：导航、fill、click、snapshot 断言文案/元素/路由；视觉检查点截图并读图确认；
- 前置条件不满足先造数据/配环境；造不了标记"阻塞"并说明原因，不跳过、不假装执行。

逐条回填格式：

```markdown
- 执行结果: ✅ 通过 (2026-08-10)
- 执行结果: ❌ 失败 (2026-08-10) | 实际: 返回 code=1001 | 缺陷: 密码错误时未返回统一错误码
- 执行结果: ⛔ 阻塞 (2026-08-10) | 原因: 测试环境缺少短信网关
```

执行纪律：

- 严格按文档步骤执行；步骤本身有问题标"阻塞"并记录，**不现场改文档**（文档修正回 GENERATE 模式，保持单一职责）；
- 失败用例在"执行汇总 → 缺陷清单"追加一行：用例编号、现象、实际 vs 预期；
- 全部执行完更新汇总数字与通过率；
- 默认按 P0→P3 顺序执行；用户可指定只跑某模块/某优先级；
- 断点续跑：重进会话时说"继续执行 xx 文档"，从未执行条目继续。

## 8. 参考来源

- 华为 CodeArts TestPlan《测试用例编写规范》：https://support.huaweicloud.com/usermanual-testman/cloudtest_01_1301.html（含命名、编号、描述、测试类型、等级、文字表达共 14 个子页面，已完整阅读）
- ISO/IEC 25010:2023 产品质量模型（仅借鉴高价值维度，不套用全矩阵）：https://www.iso.org/obp/ui/#iso:std:iso-iec:25010:en
- 博客园《测试用例编写规范》（企业落地版）：https://www.cnblogs.com/testertechnology/p/10974726.html
- superpowers skill 格式约定：`C:/Users/Windows11/.agents/skills/superpowers-*/SKILL.md`

## 9. 成功标准

- 对任意一份 PRD/spec，skill 能产出符合第 6 节规范的用例文档，需求覆盖矩阵无遗漏；
- Kimi Code 能仅凭该文档逐条执行并回填，无需回头询问需求细节；
- 用例文档可回归复用：下个迭代需求变更后，重新走 GENERATE 增量更新；
- 开源用户按 README 能在 5 分钟内跑通 examples 示例。
