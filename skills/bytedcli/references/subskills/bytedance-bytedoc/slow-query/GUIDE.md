# ByteDoc 慢查询指南

当任务涉及 slow-query overview、detail、subscribers、metrics 和 index recommend 时，使用本指南。

## 命令

```bash
# 慢查询概览（使用已确认 backend/vregion）
bytedcli --json --site cn --vregion China-North bytedoc slow-query overview --service "example.bytedoc.demo_orders" --backend classic --millis 100

# 慢查询指标
bytedcli --json --site cn --vregion China-North bytedoc slow-query metrics --service "example.bytedoc.demo_orders" --backend classic --interval 5m

# 订阅者
bytedcli --json --site cn --vregion China-North bytedoc slow-query subscribers --service "example.bytedoc.demo_orders" --backend classic

# 索引推荐
bytedcli --json --site cn --vregion China-North bytedoc slow-query index-recommend --service "example.bytedoc.demo_orders" --backend classic
```

## 慢查询诊断示例

```bash
# 场景：用户说"看一下 example_db 最近的慢查询"
bytedcli --json --site cn bytedoc search --keyword "example.bytedoc.example_db"
# → 确认唯一候选：backend=classic, vregion=China-North

bytedcli --json --site cn --vregion China-North bytedoc slow-query overview --service "example.bytedoc.example_db" --backend classic
# 如果搜索或 resolver 返回 BYTEDOC_AMBIGUOUS，展示候选给用户选择，不执行慢查询
```

## 约束

- slow-query 前必须确认 `site`、`backend` 和 `vregion`；缺少 `site` 时按 `../references/selection-guards.md` 询问。
- `backend` 和 `vregion` 没有默认值：只能由用户明确提供，或由搜索/resolver 证明唯一；任一维度多值时必须让用户选择。
- `BYTEDOC_AMBIGUOUS` / `agent_protocol.next_state=ASK_USER` 是硬边界：展示候选表，不要自行选择后继续。
- slow-query 平台 API 只覆盖 classic 和 cloud-native，不覆盖独立 Volc DBW 日志。
- Volc 页面上的日志、备份、参数视图属于 DBW / Volc 详情面，不等同于 `bytedoc slow-query`；用户确认 `backend=volc` 后直接说明不适用，不要尝试 classic/cloud-native fallback。
- classic detail 接受 ObjectId 风格的 id，并可自动展开 overview fingerprint id。
- 如果用户在获得 SDK 代码后追问查询为什么慢，将本指南和 `mongodb-query-optimizer/GUIDE.md` 结合使用。

## DO / DON'T

| 场景 | ✅ DO | ❌ DON'T |
| --- | --- | --- |
| 用户说"查慢查询" | 先搜索/解析 backend 与 vregion，唯一后执行 `slow-query overview --backend <backend>` | 不要直接按 classic 或任意 vregion 查询 |
| 需要查看某条慢查询详情 | 先从 overview 拿到 id，再用 `slow-query detail --ids <id>` | 不要猜测 id 格式 |
| backend=volc 的慢查询 | 告诉用户 slow-query 平台不覆盖 Volc DBW | 不要尝试 classic fallback |
| 用户问"怎么优化" | 结合 `mongodb-query-optimizer/GUIDE.md` 给建议 | 不要只说"加索引"而不给证据 |
| `--millis` 参数 | `--millis 100` 表示过滤执行时间 ≥100ms 的查询 | 不要把它理解为采样窗口 |

## 端到端示例：慢查询诊断到优化建议

```bash
# 场景：用户说"demo_orders 最近查询很慢，帮我看看"

# Step 1: 解析目标数据库
bytedcli --json --site cn bytedoc search --keyword "example.bytedoc.demo_orders"
# → 确认唯一候选：backend=classic, vregion=China-North

# Step 2: 查看慢查询概览
bytedcli --json --site cn --vregion China-North bytedoc slow-query overview --service "example.bytedoc.demo_orders" --backend classic --millis 100
# → 返回 fingerprint 列表，如 [{ "id": "abc123", "namespace": "demo_orders.orders", "avgMs": 2500, "count": 340 }]

# Step 3: 查看最慢的 fingerprint 详情
bytedcli --json --site cn --vregion China-North bytedoc slow-query detail --service "example.bytedoc.demo_orders" --backend classic --ids "abc123"
# → 返回该 query shape 的详细信息（filter、sort、scanType 等）

# Step 4: 查看索引推荐
bytedcli --json --site cn --vregion China-North bytedoc slow-query index-recommend --service "example.bytedoc.demo_orders" --backend classic
# → 返回推荐索引列表

# Step 5: 结合证据给出优化建议
# Agent 解读：该 query 在 orders 集合上做了全表扫描（COLLSCAN），建议加 index
# 加载 mongodb-query-optimizer/GUIDE.md 给出 ESR 建议
```
