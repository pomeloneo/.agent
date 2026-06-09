# ByteDoc Mongo 操作指南

当任务涉及集合查看、安全 Mongo shell 风格查询，以及 classic、cloud-native、Volc Mongo 的结构化文档操作时，使用本指南。

## 首屏规则

- classic / cloud-native Mongo 操作走 DMS。
- Volc Mongo 操作走 DBW。
- 读取和写入前都必须确认 `site`，并按 `../references/selection-guards.md` 解析目标数据库的 `backend` 和 `vregion`。
- `backend` 和 `vregion` 没有默认值：只能由用户明确提供，或由搜索/resolver 证明唯一；任一维度多值时必须让用户选择。
- `BYTEDOC_AMBIGUOUS` / `agent_protocol.next_state=ASK_USER` 是硬边界：展示候选表，不要自行选择后继续。
- 不要用 `--deploy-mode` 解决 Volc 歧义；`deployMode` 无法表示 Volc。
- `backend=volc` 的集合查看和 Mongo 查询走 DBW。`collections` 失败时不要改用 classic IAM `access role list`；只有返回 `BYTEDOC_ACCESS_REQUIRED` 或 setup_commands 时才进入授权流程。
- `bytedoc shell` 支持库级 `db.*` 只读命令；例如 `db.getCollectionNames()` 不需要 `--collection`，但它走 ExecuteSQL 权限，不能替代 `collections` 主链路。未以 `db.` 开头的查询才需要 `--collection`。
- 默认只做只读探查；`collection create`、`document insert`、`document update` 是真实写操作，Agent 必须先口语化展示 service、collection、filter/doc 摘要并获得用户明确确认。
- `BYTEDOC_UNSAFE_OPERATION_BLOCKED` 是终止态；不要换 BOE、DBW、DMS direct API、query file 或脚本绕过。

## 查询示例

```bash
# 用户说"查一下 example_db 库的 users 集合"
bytedcli --json --site cn bytedoc search --keyword "example.bytedoc.example_db"

# 只有确认唯一候选后，后续命令才显式携带已确认的 backend/vregion
bytedcli --json --site cn --vregion China-North bytedoc shell --service "example.bytedoc.example_db" --backend classic --collection "users" --query 'find().limit(5)'

# 如果搜索或 resolver 返回 BYTEDOC_AMBIGUOUS，展示候选给用户选择，不执行查询
```

## 安全命令

```bash
# 列出集合（使用已确认 backend/vregion）
bytedcli --json --site cn --vregion China-North bytedoc collections --service "example.bytedoc.demo_orders" --backend classic
bytedcli --json --site boe --vregion China-BOE bytedoc collections --service "example.bytedoc.demo_volc" --backend volc

# shell 查询（使用已确认 backend/vregion）
bytedcli --json --site cn --vregion China-North bytedoc shell --service "example.bytedoc.demo_orders" --backend classic --collection "demo_items" --query 'find().limit(10)'
bytedcli --json --site cn --vregion China-North bytedoc shell --service "example.bytedoc.demo_orders" --backend classic --collection "demo_items" --query-file ./query.mongo
bytedcli --json --site boe --vregion China-BOE bytedoc shell --service "example.bytedoc.demo_volc" --backend volc --query 'db.getCollectionNames()'

# 文档查询（使用已确认 backend/vregion）
bytedcli --json --site cn --vregion China-North bytedoc document list --service "example.bytedoc.demo_orders" --backend classic --collection "demo_items" --filter-json '{"tenant":"demo"}' --limit 10
```

## 危险操作停止规则

- CLI 会在到达 DMS 或 DBW 前拦截破坏性 delete/drop/rename 操作。
- 被拦截的例子包括 collection drop/rename、document delete、`drop`、`renameCollection`、`deleteOne`、`deleteMany`、`remove`、`findOneAndDelete`、drop index 和包含 delete 的 `bulkWrite`。
- `BYTEDOC_UNSAFE_OPERATION_BLOCKED` 是终止态。不要改用 BOE、DBW、DMS direct API、query file 或脚本重试。
- 如果用户确实需要破坏性管理操作，请引导用户走 bytedcli 之外的专门审核管理流程。

## 访问错误

- 如果 Mongo 命令返回 `BYTEDOC_ACCESS_REQUIRED`，切换到 `access-workflows/GUIDE.md` 并遵循 `error.details.setup_commands`。
- 不要把访问失败解释成 schema、collection 或 routing 猜测。
- 如果缺少 site/backend/vregion 或权限 role，先澄清或展示候选，不要切换 backend/site/vregion/role 重试。

## DO / DON'T

| 场景 | ✅ DO | ❌ DON'T |
| --- | --- | --- |
| 查询文档 | `bytedoc shell --query 'find({"status":"active"}).limit(10)'` | 不要省略 `.limit()`，避免全表扫描 |
| 写入文档 | 先展示待写 service/collection/doc 摘要并等用户确认 | 不要把写命令当 dry-run 执行 |
| 收到 UNSAFE_OPERATION_BLOCKED | 停止，告知用户走管理流程 | 不要换 BOE 或 DMS 直连绕过 |
| 收到 ACCESS_REQUIRED | 执行 `setup_commands` | 不要切换 backend 重试 |
| Volc `collections` 失败 | 读取 DBW 错误；必要时用 `shell --query 'db.getCollectionNames()'` 复核，但注意它需要 ExecuteSQL 权限 | 不要改用 classic `access role list` 证明权限 |
| 用户要删数据 | 告知被 CLI 安全拦截，引导审核流程 | 不要尝试用 `updateMany + $unset` 绕过 |
| 需要 aggregate 查询 | 用 `shell --query 'aggregate([...])'` | 不要因为没有 aggregate 子命令就拒绝 |

## 端到端示例：写入文档流程

```bash
# 场景：用户说"往 demo_orders 的 logs 集合插入一条测试数据"

# Step 1: 确认 site 后先查看集合是否存在
bytedcli --json --site cn bytedoc search --keyword "example.bytedoc.demo_orders"
# → 确认唯一候选：backend=classic, vregion=China-North
bytedcli --json --site cn --vregion China-North bytedoc collections --service "example.bytedoc.demo_orders" --backend classic
# → 确认 "logs" 集合存在

# Step 2: Agent 先展示写入摘要并等待用户明确确认
# 将向 example.bytedoc.demo_orders.logs 插入 1 条文档：{"type":"test","message":"hello","ts":"2024-01-01T00:00:00Z"}。是否继续？

# Step 3: 用户确认后才执行真实写入
bytedcli --json --site cn --vregion China-North bytedoc document insert --service "example.bytedoc.demo_orders" --backend classic --collection "logs" --doc-json '{"type":"test","message":"hello","ts":"2024-01-01T00:00:00Z"}'

# Step 4: 如果返回 BYTEDOC_ACCESS_REQUIRED
# → 遵循 error.details.setup_commands 申请权限
```
