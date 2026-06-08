---
name: bytedance-safe-tcs
description: TCS project, trace, and object-id query operations via bytedcli safe domain. Query TCS project detail, related project list, trace information, and review object IDs that entered review for a project within a time range; update a project's ProductType by migrating it to a target project, set the shared project split allocation, and clone a TCS project (queue).
---

# Safe TCS — Project, Trace & Object ID Operations

Query TCS project metadata, related project list, trace information and review object IDs, and perform limited write operations (such as ProductType migration and queue cloning) on the TCS platform via `bytedcli safe tcs`.

## Authentication

Requires Safe authentication. See the parent skill `bytedance-safe` for login instructions:

```bash
bytedcli auth login --session
bytedcli safe login
```

## Commands

### project

```bash
# Get TCS project by project id
bytedcli safe tcs project get --project-id <project-id>

# Explicit JSON output
bytedcli --json safe tcs project get --project-id <project-id>

# Update a TCS project's ProductType by migrating it from a target project
bytedcli safe tcs project update-product-type --project-id <project-id> --target-project-id <target-project-id>

# Explicit JSON output
bytedcli --json safe tcs project update-product-type --project-id <project-id> --target-project-id <target-project-id>

# List related projects that share with the given TCS project id
bytedcli safe tcs project get-related-project-list --shared-project-id <shared-project-id>

# Explicit JSON output
bytedcli --json safe tcs project get-related-project-list --shared-project-id <shared-project-id>

# Set the share split allocation for a TCS shared project (use JSON string or @file)
bytedcli safe tcs project set-shared-project-split --shared-project-id <shared-project-id> --split-list <json|@file> [--action <number>] [--yes]

# Explicit JSON output (automation must include --yes)
bytedcli --json safe tcs project set-shared-project-split --shared-project-id <shared-project-id> --split-list <json|@file> [--action <number>] --yes

# Query object IDs that entered review for a project within a time range
bytedcli safe tcs project query-object-ids --project-id <project-id> --start <time> --end <time>

# Explicit JSON output with a custom limit
bytedcli --json safe tcs project query-object-ids --project-id <project-id> --start <time> --end <time> --limit 50
```

### trace

```bash
# Get TCS trace of a project by project id
bytedcli safe tcs trace get --project-id <project-id>

# Explicit JSON output
bytedcli --json safe tcs trace get --project-id <project-id>
```

## Options

### project get / trace get

| Option              | Default    | Description    |
| ------------------- | ---------- | -------------- |
| `--project-id <id>` | (required) | TCS project id |

### project update-product-type

| Option                     | Default    | Description                                                                   |
| -------------------------- | ---------- | ----------------------------------------------------------------------------- |
| `--project-id <id>`        | (required) | Source TCS project id whose ProductType will be updated                       |
| `--target-project-id <id>` | (required) | Target TCS project id whose ProductType will be applied to the source project |

### project get-related-project-list

| Option                     | Default    | Description                                                        |
| -------------------------- | ---------- | ------------------------------------------------------------------ |
| `--shared-project-id <id>` | (required) | Shared TCS project id whose related project list will be retrieved |

### project set-shared-project-split

| Option                     | Default    | Description                                                                                                                                                                                                                                                                                            |
| -------------------------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `--shared-project-id <id>` | (required) | Shared TCS project id whose split allocation will be set                                                                                                                                                                                                                                               |
| `--split-list <json>`      | (required) | JSON string of split items, or `@/path/to/file.json`; each item: `{ projectId, sharedTaskPercent, isSupportProject?, action?, projectVerifyType? }`. `projectVerifyType` is optional and should come from the child queue's `safe tcs project get` field `project_verify_type`; omit when unavailable. |
| `--action <number>`        | `1`        | Outer action value sent to the upstream API; per-item `action` defaults to this when omitted                                                                                                                                                                                                           |
| `-y`, `--yes`              | (off)      | Skip the interactive preview & confirm step (use in automation; required when running with `--json` or non-TTY shell)                                                                                                                                                                                  |

### project query-object-ids

| Option              | Default    | Description                                                                                     |
| ------------------- | ---------- | ----------------------------------------------------------------------------------------------- |
| `--project-id <id>` | (required) | TCS project id                                                                                  |
| `--start <time>`    | (required) | Start time of the query window                                                                  |
| `--end <time>`      | (required) | End time of the query window                                                                    |
| `--limit <n>`       | `50`       | Max object IDs to return; must be a positive integer and cannot exceed backend-supported limits |

## Output Modes

- **默认文本**: `project get`、`project update-product-type`、`project get-related-project-list`、`project set-shared-project-split` 与 `trace get` 会输出可读文本；对象结果会渲染成 KV 表。`project query-object-ids` 会先输出命中数量，再逐行输出 object ID。
- **显式 JSON (`--json`)**: 在 `data` 字段返回结构化 JSON，适合脚本消费。

## Common Patterns

**Inspect a project's metadata then pull its trace:**

```bash
bytedcli safe tcs project get --project-id example-project-id
bytedcli safe tcs trace get --project-id example-project-id
```

**Update a project's ProductType to match another project:**

```bash
bytedcli safe tcs project update-product-type \
  --project-id example-source-id \
  --target-project-id example-target-id
```

**List related projects that share with a given TCS project id:**

```bash
bytedcli safe tcs project get-related-project-list \
  --shared-project-id example-shared-id
```

**Set split allocation for a shared project (JSON inline):**

```bash
bytedcli safe tcs project set-shared-project-split \
  --shared-project-id example-shared-id \
  --split-list '[{"projectId":"example-p1","sharedTaskPercent":0.8},{"projectId":"example-p2","sharedTaskPercent":0.2}]'
```

**Set split allocation from file:**

```bash
bytedcli safe tcs project set-shared-project-split \
  --shared-project-id example-shared-id \
  --split-list @/path/to/splits.json \
  --action 1
```

**Clone a TCS project (queue) to a new queue:**

```bash
# Clone as a regular queue
bytedcli safe tcs project clone --project-id 100000

# Clone as a crowd-source (hands) queue (must specify --group-name)
bytedcli safe tcs project clone --project-id 100000 --is-hands-project --group-name "demo-group"

# Clone as a shared (parent) queue
bytedcli safe tcs project clone --project-id 100000 --is-shared-project

# Clone with a custom title (optional; passed through to clone_project)
bytedcli safe tcs project clone --project-id 100000 --is-hands-project --group-name "demo-group" --title "demo-hands"
```

> `--title <title>` 可选；用户明确提供「克隆后队列名称」时透传，未提及则不写入请求体，沿用后端默认命名。
> `--group-name <name>` 透传到后端 `belongGroupName`；当 `--is-hands-project=true` 时必须指定。取值参考飞书表格 <https://bytedance.larkoffice.com/wiki/TZoBw7G72iaX7RkTtoXcxNvSnif?table=tblH8ScsAFYsdk02&view=vewdWVvBH5>。
> 缺省 `--group-name` 时：交互式 TTY 会追问一行；`--json` 或非 TTY 模式直接抛 `SAFE_INPUT_ERROR`，自动化场景必须显式追加 `--group-name`。

**High-risk write commands — CLI auto preview & require confirmation:**

`set-shared-project-split` 与 `update-product-type` 是高危写命令，CLI handler 层会自动拦截：拉取队列名称、把改动以中文 preview 渲染到 stderr，然后通过 readline 单行提示等用户输入 `y` / `yes`（忽略大小写）才真正调用上游 API。任何 skill / MCP / shell 调用都会走同一道护栏，无需 Agent 重复实现。

行为约定：

- 交互式 TTY 未带 `--yes` → 打印 preview，readline 单行提示等待确认；用户输入 `y` / `yes` 之外的内容时 CLI 抛 `SAFE_INPUT_ERROR`（message：`操作已取消`）。
- `--json` 模式或非 TTY 未带 `--yes` → CLI 立即抛 `SAFE_INPUT_ERROR`（message：`高危操作需要交互确认，非交互/JSON 模式请追加 --yes 跳过提示`），避免脚本死锁；自动化调用必须显式追加 `--yes`。
- name lookup 失败时该条目降级为 `<id> (name unavailable)`，preview 仍会打印并继续要求确认。

CLI 实际打印的 preview 模板：

```text
# set-shared-project-split
本次提交的分流配置是：
• 父队列：example-shared-id+父队列名称
• 子队列 example-p1+子队列名称：80%
• 子队列 example-p2+子队列名称：20%

# update-product-type
本次即将执行高危操作：迁移 TCS 队列 ProductType
• 源队列（被修改）：example-source-id+源队列名称
• 目标队列（提供 ProductType）：example-target-id+目标队列名称
```

调用示例：

```bash
# 交互式：CLI 自动 preview 并等待用户确认
bytedcli safe tcs project set-shared-project-split \
  --shared-project-id example-shared-id \
  --split-list '[{"projectId":"example-p1","sharedTaskPercent":0.8},{"projectId":"example-p2","sharedTaskPercent":0.2}]'

# 自动化 / JSON 模式：必须显式追加 --yes 跳过 preview
bytedcli --json safe tcs project set-shared-project-split \
  --shared-project-id example-shared-id \
  --split-list @/path/to/splits.json \
  --yes

# update-product-type 同样受拦截
bytedcli safe tcs project update-product-type \
  --project-id example-source-id \
  --target-project-id example-target-id

bytedcli --json safe tcs project update-product-type \
  --project-id example-source-id \
  --target-project-id example-target-id \
  --yes
```

**Query object IDs that entered review in a time window:**

```bash
bytedcli safe tcs project query-object-ids \
  --project-id example-project-id \
  --start 2026-01-01T00:00:00Z \
  --end 2026-01-01T01:00:00Z \
  --limit 50
```

## References

- [invocation.md](../../invocation.md) — bytedcli 通用调用方式
- [tcs-api.md](references/tcs-api.md) — 命令与 API 映射
- [troubleshooting.md](../../troubleshooting.md) — 常见问题与处理
- [bytedance-safe-tcs-project-switch](../bytedance-safe-tcs-project-switch/SKILL.md) — 队列用工模式切换（主审 → 众包 / 盲审）编排 skill
