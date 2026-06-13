---
name: bytedance-safe
description: Content moderation platform operations via bytedcli. Router skill for safe domain sub-skills, including puzzle workflows, Safe disposal feature/action queries and copy tickets, Safe sample library queries, Safe BBQ app credential auth, Hawk scene metadata and ops queries, SafeMind graph lifecycle/testing/trace operations, OCTP content-safety entity traces, and Digital Employee workflows. Use for 中文意图：内容安全、实体ID链路、送审、机审、处置、沟通、举报、Hawk 平台、Hawk 场景、Hawk ops.
---

# Safe Domain

Content moderation platform commands for querying features, entities, datasources, tenants, packages, collections, disposal center features/actions, disposal feature/action copy tickets, sample libraries, BBQ app credential tokens, SafeMind graph instances, digital employee graph instances, and more.

## Authentication

Before using any safe command, authenticate first:

```bash
# SSO-based login (requires prior `bytedcli auth login --session`)
bytedcli safe login

# Or paste cookie directly
bytedcli safe login --cookie "session=xxx"

# Or set environment variable
export SAFE_COOKIE="your_cookie_here"
```

For agent / non-interactive flows, prefer the two-step pattern (Safe login natively supports `--begin` / `--complete <token>` so you do not need to chain `auth login` first):

```bash
# Step 1: returns complete_token, qr_image_path, qr_url and exits immediately
bytedcli --json safe login --begin

# Step 2: after the user scans, finish the login (status: pending / expired / success)
bytedcli --json safe login --complete <token>
```

The challenge is persisted to `~/.local/share/bytedcli/data/safe_login_challenges/<token>.json` with a 30-minute TTL. If a valid BDSSO session already exists, `--begin` derives Safe cookies inline and reports `reused_existing_session: true` without showing a QR code. `--qr-image` and `--no-terminal-qr` apply to QR-driven paths (default `safe login` and `--begin`); they are ignored with `--complete` or `--cookie`.

In JSON mode, every `safe login` success path returns `data.login_status` (`success` for default / `--begin` reused / `--complete success` / `--cookie`; `pending` for `--begin` with QR scan or `--complete` still waiting; `expired` for `--complete` after the 30-minute TTL). Agents can branch on `data.login_status` instead of parsing free-form messages.

## Configuration

Manage tenant, business, and other Safe settings:

```bash
bytedcli safe config get
bytedcli safe config get --key tenant
bytedcli safe config set --key tenant --value sample_tenant
bytedcli safe config clear --key tenant
```

## Sub-Domain References

每个 sub-domain 都已经合并到本 skill 的 `references/subskills/` 目录。需要某一类操作时再按需加载，不再单独发布 sub-skill。

| Sub-Domain         | Reference                                                          | When to load                                                                                                                                                   |
| ------------------ | ------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| bbq                | [references/subskills/bbq.md](references/subskills/bbq.md)                             | `safe bbq auth login` / `safe bbq group/topic/quota/lag/tos get`，BBQ token 缓存与配额计算                                                                      |
| puzzle             | [references/subskills/puzzle.md](references/subskills/puzzle.md)                       | Puzzle feature platform — features, entities, datasources, tenants, packages, collections                                                                       |
| disposal           | [references/subskills/disposal.md](references/subskills/disposal.md)                   | Disposal center — feature/action list、test、copy 工单                                                                                                           |
| hawk               | [references/subskills/hawk.md](references/subskills/hawk.md)                           | Hawk 平台 — services/scopes/scenes 元信息与 ops 查询                                                                                                              |
| hawkpro            | [references/subskills/hawkpro.md](references/subskills/hawkpro.md)                     | Hawkpro trace list/get、scene/rule/action 操作                                                                                                                   |
| sample             | [references/subskills/sample.md](references/subskills/sample.md)                       | Sample library — `safe sample list` / `safe sample query_samples` 工作流                                                                                          |
| digital-employee   | [references/subskills/digital-employee.md](references/subskills/digital-employee.md)   | Digital Employee — list/agent lookup、graph 验证、模拟、批量 sheet/CSV 任务                                                                                     |
| safemind           | [references/subskills/safemind.md](references/subskills/safemind.md)                   | SafeMind — model list、graph 生命周期、test、trace 分析                                                                                                          |
| eva                | [references/subskills/eva.md](references/subskills/eva.md)                             | EVA — model CRUD、feature/prompt 查询、evaluation search/create、scene、time range                                                                                |
| sparkinnovation    | [references/subskills/sparkinnovation.md](references/subskills/sparkinnovation.md)     | SparkInnovation 小改变工作流 — list/get/create/update/claim、业务线/部门、枚举                                                                                  |
| tcs                | [references/subskills/tcs.md](references/subskills/tcs.md)                             | TCS project & trace — get / clone / update-product-type / get-related-project-list / set-shared-project-split / query-object-ids                                |
| tcs-project-switch | [references/subskills/tcs-project-switch.md](references/subskills/tcs-project-switch.md) | TCS 队列用工模式切换编排 — 主审 → 众包 / 盲审分流 / ProductType 修复                                                                                                |
| octp               | [references/subskills/octp.md](references/subskills/octp.md)                           | OCTP entity trace — 按实体 ID 串联 review、machine-review、disposal、communication、report 链路                                                                  |
| bcp                | [references/subskills/bcp.md](references/subskills/bcp.md)                             | BCP reconciliation key list — 按 rule_id + 结果码查 bcp_key（Aeolus dashboard 263849）                                                                            |

## Common Options

- `--tenant <tenant>` — Tenant for API requests. Puzzle sub-commands, disposal feature/action queries, digital employee list/agent lookup, graph validation/update/simulation/result queries, digital employee batch simulation tasks, SafeMind queries, and sample queries support this option. Priority: `--tenant` > `SAFE_TENANT` env > config > default `ecology`.
  - Config: `bytedcli safe config set --key tenant --value <tenant>`
- `--business <business>` — Business ID (default: default)
- `--business-id <id>` / `--business-key <key>` — Sample query business headers. Priority: CLI > env (`SAFE_BUSINESS_ID`, `SAFE_BUSINESS_KEY`) > config (`business_id`, `business_key`).

## Digital Employee Quick Examples

```bash
bytedcli safe digital-employee list --name demo --page 1 --page-size 10
bytedcli safe digital-employee list --department-ids demo-department-id --project-ids demo-project-id
bytedcli safe digital-employee agent list --id demo-employee-id --page 1 --page-size 10
bytedcli safe digital-employee agent get --id demo-agent-id
```
