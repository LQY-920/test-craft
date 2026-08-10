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
