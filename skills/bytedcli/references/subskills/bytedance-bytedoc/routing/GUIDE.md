# ByteDoc 路由指南

当任务涉及数据库搜索、backend 消歧、site/vRegion 行为，以及 DMS/DBW 数据面路由时，使用本指南。

## 首屏规则

- 先按 `../references/selection-guards.md` 确认 `site`，再解析 backend/vregion；不要猜默认 backend 或 vregion。
- 首次搜索只传 `--site` + keyword/service，用结果确认候选唯一性；后续查看/数据面命令显式携带已确认的 `--backend` 和必要的 `--vregion`。
- `BYTEDOC_AMBIGUOUS` 或 `agent_protocol.next_state=ASK_USER` 是硬边界：展示候选给用户选择，不要自行挑选。
- `AUTH_REQUIRED` 是认证问题，不是 backend 错误；登录后重试同一条命令。
- `BYTEDOC_NOT_FOUND` 是当前 selector 下未找到；确认 PSM 拼写或 site，不要自动切换 site/backend。

## Backend 模型

- `backend=classic`：传统 ByteDoc；Mongo 数据面使用 DMS。
- `backend=cloud-native`：cloud-native ByteDoc；Mongo 数据面使用 DMS。
- `backend=volc`：DBW / Volc Mongo；Mongo 数据面使用 DBW。
- `deployMode=classic|cloud-native` 是历史元数据，不能表示 Volc。

## 搜索与候选确认

CLI 全站搜索（不限 VRegion、不限 backend）。用户提供 `--site` 后，Agent 需要用搜索结果确认 backend 和 vregion 是否唯一。

```bash
# 用户只提供关键词和 site，先搜索候选
bytedcli --json --site cn bytedoc search --keyword "example_db"

# 用户已确认特定 backend 后，可用 backend 过滤
bytedcli --json --site cn bytedoc search --keyword "example_db" --backend volc

# 海外站点
bytedcli --site i18n-tt --json bytedoc search --keyword "example_db"

# 查看详情（使用已确认 backend/vregion）
bytedcli --json --site cn --vregion China-North bytedoc get --service "example.bytedoc.demo_orders" --backend classic
```

- Cloud Service Search 可以返回 `backend=classic|cloud-native|volc` 和原始 `mode`。
- 海外控制面可能回退到 classic / cloud-native 平台搜索，没有独立 Volc 聚合来源。
- backend 和 vregion 由 CLI 从搜索结果中解析；只有 backend 与 vregion 都唯一，才可继续执行目标操作。
- 当搜索返回多个匹配项或单条匹配包含多个 vregion（`BYTEDOC_AMBIGUOUS`）时，必须让用户选择。
- 只有用户明确要求搜索/查找资源时，才可以用 search 展示候选；候选结果不能替用户自动选择。

## AI Agent 解析示例

```bash
# 场景：用户说"帮我看一下 example_db 这个库"
# Agent 先确认 site，再确认 backend/vregion 是否唯一

# Step 1: 搜索候选
bytedcli --json --site cn bytedoc search --keyword "example_db"

# Step 2: 如果唯一匹配，后续命令显式携带已确认的 --backend / --vregion
bytedcli --json --site cn --vregion China-North bytedoc collections --service "example.bytedoc.example_db" --backend classic

# Step 3: 如果 CLI 返回 BYTEDOC_AMBIGUOUS，展示候选表格让用户选择
# 候选表格包含 PSM、backend、vregion 信息
```

## Resolver 规则

- dotted PSM 不能自动推断为 cloud-native。
- CLI resolver 从搜索结果解析 backend 和 vregion；多 backend、多 vregion 或单条多 vregion 都需要用户介入。
- 将 `BYTEDOC_AMBIGUOUS` 和 `agent_protocol.next_state=ASK_USER` 视为必须询问用户的硬边界。
- 将 `AUTH_REQUIRED` 视为认证问题，不要把它当成尝试其他 backend 的证据。
- 将 `BYTEDOC_NOT_FOUND` 视为确定的否定结果，除非用户提供新的 selector。

## Site 与 DMS 规则

- DMS 必须使用当前全局 `--site` / `--vregion` 对应的 JWT、origin、referer 和 endpoint。
- i18n-bd DMS API host 是 `https://fedms-api.byteintl.net`；origin 是 `https://dms.byteintl.net`。
- i18n-tt DMS API host 是 `https://fedms-i18n-api.byteintl.net`；origin 是 `https://dms-i18n.byteintl.net`。
- 海外 classic 查询不要回退到 CN JWT、CN origin 或 `dc=cn`。
- 如果 `BYTEDOC_ROUTING_UNRESOLVED` 说明 region 无法判定，询问路由上下文或停止；不要盲试 CN。

## DO / DON'T

| 场景 | ✅ DO | ❌ DON'T |
| --- | --- | --- |
| 用户说"搜一下 xxx" | `bytedcli --json --site cn bytedoc search --keyword "xxx"`，展示候选 | 不要加猜测的 `--backend` 或 `--vregion` |
| 搜索返回 0 条结果 | 告诉用户未找到，确认 PSM 拼写或 site | 不要自动切换 site 或 backend 重试 |
| 搜索返回多条匹配或多个 vregion | 展示候选表格让用户选 | 不要自行挑一个继续执行 |
| 海外站点搜索 | 使用 `--site i18n-tt` 等正确 site | 不要回退到 `--site cn` |
| 命令返回 AUTH_REQUIRED | `bytedcli auth login` 后重试 | 不要把认证错误当 backend 错误处理 |

## 搜索结果为空时的处理

```bash
# 搜索未找到结果
bytedcli --json --site cn bytedoc search --keyword "nonexistent_db"
# → 返回空列表

# Agent 应该：
# 1. 告诉用户"在 cn 站点下未找到该数据库"
# 2. 询问：PSM 拼写是否正确？是否在其他 site（如 boe、i18n-bd）？
# 3. 不要自动尝试其他 site
```
