---
name: test-craft
description: Use when you need to write standardized project test cases from a requirements document (PRD, spec, or implementation plan), or execute an existing test-case document, record and backfill results. Covers API tests and UI/integration tests for websites, mini-programs, and desktop apps.
---

# test-craft

Turn a requirements document into a professional, executable test-case document — then execute it case by case and backfill results. Generated documents are written in **Chinese** (matching the requirement doc); this skill file is English.

**Announce at start:** "I'm using the test-craft skill to <generate test cases | execute the test document>."

## Mode routing

```
if the user points at an EXISTING test-case document (matches the template structure, contains 执行结果 fields) and wants it executed → EXECUTE mode
elif the user provides BOTH an existing test-case document and an updated requirement doc, intent = update → GENERATE update mode
elif a requirement doc is provided or discoverable → GENERATE mode
elif the user wants to update an existing test-case document but no updated requirement is available → ask which requirement the update should be based on
else → ask the user for the input document; never invent requirements
```

Requirement doc discovery order (GENERATE): explicit path from the user → `docs/superpowers/specs/` → `docs/superpowers/plans/` → PRD/requirement files in the repo root. If several candidates exist, list them and let the user pick. If the requirement is ambiguous or incomplete, ask clarifying questions BEFORE writing cases.

**GENERATE update mode:** when the requirement changed and a test-case document already exists, update that document in place instead of writing a fresh file: add cases for new 功能点, revise cases whose requirement changed, and mark obsolete cases as `- 执行结果: 🗑 已废弃 (YYYY-MM-DD) | 原因: <why>` (never delete them). A revised case's 执行结果 resets to `未执行`; preserve existing backfills only of cases whose steps and expected results did not change. Update the header's 用例数 and the 需求覆盖矩阵 — header 用例数 and 执行汇总 总数 exclude 已废弃 cases.

## GENERATE mode

Five steps:

1. **需求解析** — Read the requirement doc fully. Build the feature tree (一级模块 → 二级模块 → 功能点). Tag every 功能点 with its source section for traceability.
2. **测试点提取** — For each 功能点 extract test points: positive flow, negative/exception, boundaries, permissions, state transitions. Load `references/checklists.md` and add implicit test points for the product form (网站 / 小程序 / 桌面程序), noting skipped items and why.
3. **测试点确认** — Present the feature tree + test-point list to the user for confirmation BEFORE expanding into cases. Skip only if the user says to generate directly.
4. **用例设计** — Load `references/design-methods.md`. Expand each test point into cases (equivalence partitioning, boundary values, error guessing, scenario, decision table). Every test point gets at least one positive case plus the necessary negative ones. Load `references/writing-rules.md` and obey it for numbering, naming, steps, expected results, and vocabulary.
5. **输出与自检** — Write the document from `templates/test-case-document.md` to `docs/tests/YYYY-MM-DD-<feature>-testcases.md` (`<feature>` is the feature's English name). Every case starts with `执行结果: 未执行`. Then run the self-check list in `references/writing-rules.md` §12 and fix violations. The document MUST include the 需求覆盖矩阵 (section order follows the template: 用例列表 → 需求覆盖矩阵 → 执行汇总) proving every 功能点 is covered.

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
- 通过率 = 通过 ÷ 已执行（通过+失败+阻塞），annotated like `通过率 33%（已执行 3 条）`; ⛔ counts as executed, 🗑 已废弃 cases are excluded from all 执行汇总 counts.
- 页面 cases for non-browser product forms (小程序 / 桌面程序): if no automation driver is available, mark ⛔ 阻塞 with reason `需人工执行` — never fake execution.
- If the user gives no document path, discover test-case documents in `docs/tests/`; list candidates if several exist.
- Resume support: "继续执行 <文档>" picks up at the first 未执行 case in P0→P3 order. To re-run a ❌/⛔ case, the user asks for a re-run; then overwrite that case's 执行结果 line with the new result.
