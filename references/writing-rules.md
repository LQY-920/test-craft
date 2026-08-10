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
