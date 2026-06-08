---
name: bytedance-safe
description: Content moderation platform operations via bytedcli. Router skill for safe domain sub-skills, including puzzle workflows, Safe disposal feature/action queries and copy tickets, Safe sample library queries, SafeMind graph lifecycle/testing/trace operations, OCTP content-safety entity traces, and Digital Employee workflows. Use for 中文意图：内容安全、实体ID链路、送审、机审、处置、沟通、举报.
---

# Safe Domain

Content moderation platform commands for querying features, entities, datasources, tenants, packages, collections, disposal center features/actions, disposal feature/action copy tickets, sample libraries, SafeMind graph instances, digital employee graph instances, and more.

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

## Sub-Domain Skills

| Sub-Domain         | Skill                             | Description                                                                                                              |
| ------------------ | --------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| puzzle             | bytedance-safe-puzzle             | Feature platform — features, entities, datasources, tenants, packages, collections                                       |
| disposal           | bytedance-safe-disposal           | Disposal center — feature/action list queries and copy tickets by platform type, scene, and key                          |
| hawkpro            | bytedance-safe-hawkpro            | Trace query — list and get hawkpro moderation traces                                                                     |
| sample             | bytedance-safe-sample             | Sample library query workflow via `safe sample list` / `safe sample query_samples`                                       |
| digital-employee   | bytedance-safe-digital-employee   | Digital Employee list/agent lookup, graph validation/update, agent simulation, result queries, and sheet/CSV batch task management |
| safemind           | bytedance-safe-safemind           | Model list, graph lifecycle/test operations, and trace analysis for AI reasoning engine                                  |
| eva                | bytedance-safe-eva                | EVA platform — model CRUD, feature/prompt queries and management, prompt create for re-training, evaluation search/create, scene list, and time range queries |
| tcs                | bytedance-safe-tcs                | TCS project & trace operations — get / clone / update-product-type / get-related-project-list / set-shared-project-split / query-object-ids |
| tcs-project-switch | bytedance-safe-tcs-project-switch | 队列用工模式切换编排 skill — 主审 → 众包 / 盲审分流 / ProductType 修复                                                   |
| octp               | bytedance-safe-octp               | OCTP entity trace — query content-safety events by entity ID across review, machine-review, disposal, communication, and report links |
| bcp                | bytedance-safe-bcp                | BCP reconciliation key list — query bcp_key by rule_id and result code via the Aeolus dashboard (263849)                  |

Load the corresponding sub-skill for detailed command syntax, parameters, and usage patterns.

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

