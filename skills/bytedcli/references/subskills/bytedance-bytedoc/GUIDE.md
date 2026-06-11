---
name: bytedance-bytedoc
description: "Use when tasks mention ByteDoc, bytedoc, ByteDoc SDK access, multi-language SDK generation (Go/Python/Java/Node.js/C++), ByteDoc database authorization, PSM authorization, IAM user permission management, Mongo collections/documents in ByteDoc, ByteDoc slow queries, or ByteDoc/Mongo errors such as not authorized, access denied, forbidden, no permission, BYTEDOC_ACCESS_REQUIRED, BYTEDOC_AMBIGUOUS, or BYTEDOC_UNSAFE_OPERATION_BLOCKED."
---

# bytedcli ByteDoc Skill

ByteDoc 路由器。不同 backend（classic / cloud-native / volc）和不同 site 使用完全不同的 API 基础设施。**不要猜测参数；先停下来确认，再执行命令。**

---

## 🛑 STOP — 执行任何命令前必须通过

以下条件未满足时，**禁止执行任何 ByteDoc 命令**：

| # | 条件 | 未满足时的行为 |
|---|------|---------------|
| 1 | `site` 已确认 | **立即询问用户**，给出选项：`cn / boe / i18n-bd / i18n-tt / us-ttp / eu-ttp` |
| 2 | `backend` 和 `vregion` 已确认（用户提供或 CLI 解析出唯一候选） | **先执行 `bytedoc search` 解析**；多值时展示候选表让用户选择 |
| 3 | CLI 未返回 `BYTEDOC_AMBIGUOUS` | **展示候选表格给用户选择**，不要自行挑选 |
| 4 | CLI 未返回 `BYTEDOC_UNSAFE_OPERATION_BLOCKED` | **立即停止**，不要换路径绕过 |
| 5 | 权限 role 来自固定枚举 | **展示枚举让用户选**，不要猜 `write`/`rw` |

**硬性约束：同一类错误不得通过切换 backend、site 或 role 反复重试。收到 AMBIGUOUS/STOP 后只接受用户的明确选择。**

---

## AI Agent 决策树

```text
用户请求到达
│
├─ 1. site 已知？
│   ├─ 否 → 🛑 立即询问（展示 site 选项表）→ 拿到 site 后继续
│   └─ 是 ↓
│
├─ 2. 搜索/解析目标（确认 backend + vregion 唯一性）
│   │   bytedcli --json --site <site> bytedoc search --keyword "<PSM或关键词>"
│   │
│   ├─ 唯一候选 → 记录 backend + vregion，后续命令显式携带
│   ├─ BYTEDOC_AMBIGUOUS → 🛑 展示候选表给用户选择
│   ├─ BYTEDOC_ACCESS_REQUIRED → 执行 setup_commands
│   ├─ AUTH_REQUIRED → auth login 后重试
│   └─ 无匹配 → 告知用户，确认关键词/site 是否正确
│
├─ 3. 确定意图类型，执行目标命令（显式带 --backend + --vregion）
│   ├─ 搜索/查看      → bytedoc get / collections
│   ├─ 读写文档       → bytedoc shell / document list|insert|update
│   ├─ 权限申请/检查  → bytedoc access psm create / ticket create / role list
│   ├─ SDK 接入       → bytedoc sdk plan / generate / doctor
│   └─ 慢查询诊断    → bytedoc slow-query overview / detail
│
└─ 4. 响应用户
```

**关键：步骤 1 和步骤 2 不可跳过。即使用户看起来意图明确，也必须先解析确认。**

---

## DO / DON'T

| 场景 | ✅ DO | ❌ DON'T |
|------|-------|---------|
| 用户未提供 site | 立即询问，给出选项表 | 假设 `cn` 或从上下文猜测 |
| 用户说"帮我看 xxx 库" | 先确认 site → 再 `bytedoc search` 确认唯一性 | 跳过解析直接 `get --backend classic` |
| 首次执行命令 | 先解析 backend/vregion；唯一后显式带入后续命令 | 猜一个 `--backend classic` 加上去 |
| CLI 返回 BYTEDOC_AMBIGUOUS | 原样展示候选表格，问用户选哪个 | 从候选中自行挑选一个执行 |
| CLI 返回 BYTEDOC_ACCESS_REQUIRED | 执行 `error.details.setup_commands` | 换一个 backend 或 site 重试 |
| CLI 返回 HTTP 400/500 错误 | 读取结构化错误信息，按错误码处理 | 猜测"可能是权限问题"并跳到 access 流程 |
| 用户说"写权限" | 展示 PSM role 和 IAM role 两类选项让用户确认 | 直接用 `--roles write` |
| 用户未提供 collection | 先用 `bytedoc collections` 列出可用集合 | 猜 collection 名 |
| 命令成功但 data 为空 | 告诉用户"该条件下没有匹配数据" | 认为命令出错而切换 backend 重试 |
| 需要写入操作 | 先走 dry-run 预览，等用户确认 | 直接提交写入 |
| `BYTEDOC_UNSAFE_OPERATION_BLOCKED` | 停止，告知用户被拦截 | 换 BOE/DMS/DBW/脚本绕过 |

---

## 端到端示例：从零开始查询一个数据库

```text
用户："帮我查一下 order_service 这个库最近的订单"

Agent 内部决策：
  → site 未知 → 必须先问
```

**Agent → 用户：** "请确认目标站点（site）：cn / boe / i18n-bd / i18n-tt / us-ttp / eu-ttp"

```text
用户："cn"

Agent 内部决策：
  → site=cn 已确认
  → backend/vregion 未知 → 必须先搜索解析
```

```bash
# 步骤 1：搜索确认 backend 和 vregion 唯一性
bytedcli --json --site cn bytedoc search --keyword "order_service"
# → 返回唯一匹配：example.bytedoc.order_service (classic, China-North)
```

```text
Agent 内部决策：
  → 唯一候选：backend=classic, vregion=China-North
  → 后续命令显式携带这两个参数
```

```bash
# 步骤 2：列出集合
bytedcli --json --site cn --vregion China-North bytedoc collections --service "example.bytedoc.order_service" --backend classic
# → ["orders", "users", "payments"]

# 步骤 3：查询文档（用户关心 "最近的订单"）
bytedcli --json --site cn --vregion China-North bytedoc shell --service "example.bytedoc.order_service" --backend classic --collection "orders" --query 'find({}).sort({"created_at":-1}).limit(5)'
# → 返回最近 5 条订单文档
```

**Agent → 用户：** 展示查询结果。

---

## 调用方式

- 默认命令前缀：`bytedcli`。
- 自动化场景使用结构化输出：`--json` 放在 `bytedoc` 前，例如 `bytedcli --json bytedoc search --keyword demo_orders`。
- 全局路由参数放在 `bytedoc` 前：`--site`、`--vregion`。
- 如果需要登录，针对目标站点执行 `bytedcli auth login`。

## Agent 协议

- 成功的 access 预览使用 camelCase：`agentProtocol` 和 `agentProtocol.nextState`。
- 错误详情使用 snake_case：`agent_protocol` 和 `agent_protocol.next_state`。
- `safeToExecuteAutomatically=false` → 获得用户决策前必须停止自动化执行。
- `requiresUserConfirmation=true` → 渲染 review table 并请求用户明确确认。
- `nextState=COLLECT_REQUIRED_FIELDS` → 向用户补齐 CLI 返回的缺失字段。
- `next_state=RUN_SETUP_COMMANDS` → 先执行或展示 `setup_commands`。
- `next_state=ASK_USER` → 🛑 用户消歧。
- `next_state=STOP` → 🛑 不要重试，不要绕路。

## 不可违背的规则

- `classic`、`cloud-native`、`volc` 是三个独立 backend。`volc` 是 DBW / Volc Mongo，不是 cloud-native。
- `deployMode` 是历史两态元数据：`classic|cloud-native`。严格消歧使用 `--backend classic|cloud-native|volc`。
- 授权和写入场景中，`site`、权限枚举、backend 和 vregion 都必须已确认；不允许用 classic 或任意 vregion 作为隐式默认值。
- ByteDoc access 写操作必须先 dry-run。展示 `confirmation.reviewTable`、`payload`、`missingFields` 和 `nextActions`；不要展示隐藏执行材料。
- 不要向用户暴露 `agentProtocol.doNotDisplay` 中列出的隐藏执行字段。
- Volc PSM 授权真源是 ByteDoc multi-cloud account 数据，不是 classic IAM role bindings 或 BPM ticket 历史。

---

## 按需加载指南

| 用户意图或错误 | 下一步加载 |
| --- | --- |
| 搜索/列表/详情、backend 歧义、site/vRegion、DMS/DBW 路由 | `routing/GUIDE.md` |
| 集合查看、Mongo shell 查询、文档 list/insert/update、危险操作拦截 | `mongo-ops/GUIDE.md` |
| `BYTEDOC_ACCESS_REQUIRED`、角色选项、权限预检、PSM/IAM 授权 | `access-workflows/GUIDE.md` |
| SDK 接入计划、多语言 SDK 素材、SDK 连接/鉴权/网络失败 | `sdk-access/GUIDE.md` |
| 慢查询 overview/detail/metrics/index recommend | `slow-query/GUIDE.md` |
| 错误码分诊与常见失败恢复 | `troubleshooting/GUIDE.md` |
| ByteDoc SDK 素材确定后的 MongoDB client/query/schema 质量优化 | `mongodb-connection/GUIDE.md`、`mongodb-natural-language-querying/GUIDE.md`、`mongodb-query-optimizer/GUIDE.md`、`mongodb-schema-design/GUIDE.md` |

---

## 命令速查（参考区）

```bash
# 解析候选：先确认 site，再确认 backend/vregion 是否唯一
bytedcli --json --site cn bytedoc search --keyword "demo_orders"

# 唯一候选或用户确认后，后续命令显式携带 backend/vregion
bytedcli --json --site cn --vregion China-North bytedoc get --service "example.bytedoc.demo_orders" --backend classic

# Mongo 读取链路
bytedcli --json --site cn --vregion China-North bytedoc collections --service "example.bytedoc.demo_orders" --backend classic
bytedcli --json --site cn --vregion China-North bytedoc document list --service "example.bytedoc.demo_orders" --backend classic --collection "demo_items" --limit 10

# 授权发现与 dry-run 预览
bytedcli --json --site cn --vregion China-North bytedoc access role list --service "example.bytedoc.demo_orders" --backend classic
bytedcli --json --site cn --vregion China-North bytedoc access permission get --service "example.bytedoc.demo_orders" --backend classic --role-name "bytedoc.data_reader.cn"
bytedcli --json --site cn --vregion China-North bytedoc access ticket create --service "example.bytedoc.demo_orders" --backend classic --account "example.caller.psm"

# SDK 计划与诊断
bytedcli --json --site cn --vregion China-North bytedoc sdk plan --service "example.bytedoc.demo_orders" --backend classic --access-env auto --language go
# --language 支持 go|python|java|nodejs|cpp（默认 go），sdk generate 同理
bytedcli --json --site cn --vregion China-North bytedoc sdk doctor --service "example.bytedoc.demo_orders" --backend classic --error-text "not authorized on demo_orders"

# 慢查询
bytedcli --json --site cn --vregion China-North bytedoc slow-query overview --service "example.bytedoc.demo_orders" --backend classic
```

## 参考资料

- `references/bytedoc.md`：更完整的命令矩阵。
- `references/selection-guards.md`：backend、site、区域和权限枚举的执行前澄清门禁。
