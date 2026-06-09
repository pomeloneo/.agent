# ByteDoc

ByteDoc 数据库搜索、列表、详情、关注、集合查看、允许的文档读写、安全 Mongo shell 风格命令与慢查询分析统一收敛在 `bytedoc <action>`。

旧的 `bytedoc db <action>` 兼容入口仍可使用；新示例和新脚本优先使用 flat 命令。

## 常见命令

```bash
# 搜索数据库（全站搜索，用于确认 backend/vregion 是否唯一）
bytedcli --site cn bytedoc search --keyword "demo_orders"
bytedcli --site cn bytedoc search --keyword "bytedoc.demo_catalog" --backend cloud-native
bytedcli --site cn bytedoc search --keyword "bytedoc.demo_catalog" --backend volc
bytedcli --site i18n-tt bytedoc search --keyword "demo_orders"

# 查看我关注的库 / 数据库详情
bytedcli --json --site cn bytedoc list
bytedcli --json --site cn bytedoc list --all --deploy-mode cloud-native
bytedcli --json --site cn --vregion China-North bytedoc get --service "demo_orders" --backend classic

# 数据库访问授权角色 / 权限检查（跟随全局 --site / --vregion）
bytedcli --json --site cn --vregion China-North bytedoc access role list --service "example.bytedoc.demo_orders" --backend classic
bytedcli --json --site cn --vregion China-North bytedoc access permission get --db-name "demo_orders" --backend classic --role-name "bytedoc.data_reader.cn"
bytedcli --site i18n-tt --json --vregion VA bytedoc access role list --db-name "demo_orders" --backend classic

# PSM/User 接入授权工单（只示例 dry-run；用户确认后按 CLI 协议内部提交）
bytedcli --json --site cn --vregion China-North bytedoc access ticket create --db-name "demo_orders" --backend classic --account "example.caller.psm"
bytedcli --json --site cn --vregion China-North bytedoc access ticket create --db-name "demo_orders" --backend classic --account "example.caller.psm" --reviewers "alice" --roles "read" --reason "Need read access for service integration"

# PSM 授权管理：classic 走 ByteDoc ticket，Volc / 多云走 BPM workflow 11975
bytedcli --site boe --json --vregion China-BOE bytedoc access psm list --service "example.bytedoc.demo_volc" --backend volc
bytedcli --json --site cn --vregion China-North bytedoc access psm create --db-name "demo_orders" --backend classic --operation apply --account "example.biz.psm" --reviewers "alice" --roles "read" --reason "Need PSM read authorization"
bytedcli --json --site cn --vregion China-North bytedoc access psm create --db-name "demo_orders" --backend classic --operation modify --account "example.biz.psm" --reviewers "alice" --roles "readWrite" --reason "Update PSM authorization"
bytedcli --json --site cn --vregion China-North bytedoc access psm create --db-name "demo_orders" --backend classic --operation delete --account "example.biz.psm" --reviewers "alice" --roles "read" --reason "Remove PSM authorization"
bytedcli --site boe --json --vregion China-BOE bytedoc access psm create --service "example.bytedoc.demo_volc" --backend volc --operation apply --account "example.biz.psm" --roles "readWrite" --reason "Need confirmed Volc PSM authorization"

# IAM 用户权限管理：授权、改期、回收、自助申请
bytedcli --json --site cn --vregion China-North bytedoc access user grant --db-name "demo_orders" --backend classic --role-name "bytedoc.viewer.cn" --principal "demo.user" --principal-type user --duration 30d --reason "Need temporary view access"
bytedcli --json --site cn --vregion China-North bytedoc access user update --db-name "demo_orders" --backend classic --role-name "bytedoc.viewer.cn" --principal "demo.user" --duration 7d --reason "Shorten authorization"
bytedcli --json --site cn --vregion China-North bytedoc access user revoke --db-name "demo_orders" --backend classic --role-name "bytedoc.viewer.cn" --principal "demo.user" --reason "Access no longer needed"
bytedcli --json --site cn --vregion China-North bytedoc access user apply --db-name "demo_orders" --backend classic --role-name "bytedoc.viewer.cn" --duration 180d --reason "Need self-service view access"

# 集合与慢查询（使用已确认 backend/vregion）
bytedcli --json --site cn --vregion China-North bytedoc collections --service "demo_orders" --backend classic
bytedcli --json --site cn --vregion China-North bytedoc collections --db-name demo_catalog --backend cloud-native
bytedcli --json --site boe --vregion China-BOE bytedoc collections --service "example.bytedoc.demo_volc" --backend volc
bytedcli --json --site cn --vregion China-North bytedoc collection create --service "example.bytedoc.demo_catalog" --backend cloud-native --collection "demo_items"
bytedcli --json --site cn --vregion China-North bytedoc slow-query overview --service "demo_orders" --backend classic --millis 100
bytedcli --json --site cn --vregion China-North bytedoc slow-query metrics --service "demo_orders" --backend classic --interval 5m

# Mongo shell 风格命令（site/backend/vregion 必须确认）
bytedcli --json --site cn --vregion China-North bytedoc shell --service "demo_orders" --backend classic --collection "demo_records" --query 'find().limit(10)'
bytedcli --json --site cn --vregion China-North bytedoc shell --service "demo_orders" --backend classic --collection "demo_records" --query-file ./query.mongo
bytedcli --json --site cn --vregion China-North bytedoc shell --db-name demo_catalog --backend cloud-native --collection "demo_items" --query 'find().limit(10)'
bytedcli --json --site boe --vregion China-BOE bytedoc shell --service "example.bytedoc.demo_catalog" --backend volc --collection "demo_items" --query 'find().limit(10)'
bytedcli --json --site boe --vregion China-BOE bytedoc shell --service "example.bytedoc.demo_catalog" --backend volc --query 'db.getCollectionNames()'

# 文档操作（classic / cloud-native / Volc Mongo）
bytedcli --json --site cn --vregion China-North bytedoc document list --service "example.bytedoc.demo_catalog" --backend cloud-native --collection "demo_items" --filter-json '{"tenant":"demo"}' --limit 10
bytedcli --json --site cn --vregion China-North bytedoc document insert --service "example.bytedoc.demo_catalog" --backend cloud-native --collection "demo_items" --doc-json '{"tenant":"demo","value":1}'
bytedcli --json --site cn --vregion China-North bytedoc document update --service "example.bytedoc.demo_catalog" --backend cloud-native --collection "demo_items" --filter-json '{"tenant":"demo"}' --update-json '{"$set":{"value":2}}'
```

## 说明

### 后端分类

- `backend=classic`：传统 ByteDoc，Cloud Service Search 原始 `mode=classic`；Mongo 查询、允许的集合操作和文档操作走 DMS subscribe / evaluate，列表、关注、详情、慢查询走 classic ByteDoc API。
- `backend=cloud-native`：cloud-native ByteDoc，Cloud Service Search 原始 `mode=cloud-native`；Mongo 查询、允许的集合操作和文档操作走 DMS subscribe / evaluate，慢查询走 cloud-native slowquery API。
- `backend=volc`：DBW / Volc Mongo，Cloud Service Search 原始 `mode=volc`；实例信息来自 `instance_id`、`instance_type`、`region`、`vregion`，集合、查询和允许的文档/集合写操作走 DBW Mongo 执行链路。
- `mode` 是 Cloud Service Search 返回的原始字段；`backend` 是 CLI 对外的严格三态选择器；`deployMode` 是 legacy 平台路由字段，仅有 `classic|cloud-native` 两态。

### 命令链路

- ByteDoc 允许不同 backend 或不同 vregion 下存在同名数据库；除搜索/list 这类发现命令外，目标操作前必须明确 `backend` 和 `vregion`。
- `backend` 和 `vregion` 没有默认值：只能由用户明确提供，或由搜索/resolver 证明唯一。任一维度多值、或单条候选包含多个 vregion 时，Agent 必须展示候选让用户选择。
- `bytedoc search` 在支持 Cloud Service Search 的站点上默认全站搜索（不限 VRegion、不限 backend），返回 `backend=classic|cloud-native|volc` 和原始 `mode`；`--backend` 是可选的严格三态过滤项，`--deploy-mode` 仅作为 legacy 两态兼容过滤项。
- 海外控制面（例如 `--site i18n-tt`）如果没有 Cloud Service Search 聚合接口，会自动回退到 classic / cloud-native 平台搜索；此时没有独立 Volc 聚合来源，`--backend volc` 不返回结果。
- `bytedoc list` 默认展示关注列表；未显式传 `--deploy-mode` 时会合并 classic 和 cloud-native 的关注列表。
- `bytedoc list --all` 展示全量数据库；不要把 list 的兼容行为当成后续目标操作的 backend 默认值。
- `bytedoc list` / `bytedoc follow` / `bytedoc slow-query *` 仍只有 classic / cloud-native 平台接口，没有独立 Volc 后端；Volc 页面上的日志、备份、参数等能力属于 DBW / Volc 详情面。
- `bytedoc get` 可以传 `--db-name` 或 `--service`；classic / cloud-native 返回平台详情，Volc Mongo 只在解析到 DBW 元信息时返回 Cloud Service / DBW summary，且不返回 classic usage。
- `bytedoc collections` 不需要 `--deploy-mode`；后续命令应显式携带已确认的 `--backend classic|cloud-native|volc` 和全局 `--vregion`。`backend=volc` 时走 DBW `ListTables`，不是 classic IAM 权限接口。
- `bytedoc shell` 不需要 `--deploy-mode`；后续命令应显式携带已确认的 `--backend classic|cloud-native|volc` 和全局 `--vregion`；`--deploy-mode` 只能区分 legacy `classic|cloud-native` 两态。
- `bytedoc collection create` 与 `bytedoc document <list|insert|update>` 面向 classic / cloud-native / Volc Mongo；classic / cloud-native 使用 DMS，Volc 使用 DBW。
- 所有站点下，`bytedoc collection drop/rename`、`bytedoc document delete`、以及 `bytedoc shell/query` 中的 `drop`、`renameCollection`、`deleteOne`、`deleteMany`、`remove`、`findOneAndDelete`、drop index 和 `bulkWrite` 删除都会被本地拒绝，不会请求 DMS 或 DBW；BOE 也不是例外。收到 `BYTEDOC_UNSAFE_OPERATION_BLOCKED` 后，不得改用 BOE、DBW、DMS direct helper、query-file 或脚本绕过。
- `bytedoc document list` 是结构化文档查询入口；`bytedoc document find` / `query` 仍可作为兼容别名使用。
- `bytedoc access role list` 会读取数据库静态信息、IAM resource info 和角色绑定，输出可供 Agent 展示给用户选择的角色列表；支持 `--db-name`、`--service`、`--backend` 消歧。
- `bytedoc access permission get` 会校验当前用户是否具备发起授权流程的前置权限。
- `bytedoc access ticket create` 会读取已验证的 `apply_account` 表单 schema，JSON 输出在 `data.result` 下包含审批人和账号权限可选项、payload preview、`confirmation.requiredReview`、`confirmation.reviewTable`、`nextActions` 与 `agentProtocol`；默认 dry-run，不提交工单。
- `bytedoc access psm list` 用于主动查询 PSM 授权读取状态，并给出 `psm create` dry-run 的下一步动作。传 `--account <caller.psm>` 后读取 `accountCheck.status` 与 `accountCheck.roles`；Volc / DBW 的 PSM token 授权读取使用 ByteDoc multi-cloud `GetAccount`，`inspection.source=bytedoc_multicloud_get_account`。不要用 ByteCloud IAM ACL node 或 `role-bindings/list-by-resource` 判定 Volc PSM token 授权，也不要把 BPM ticket 历史当作当前权限。
- 如果当前授权真源接口未确认或读取失败，权限列表 / 巡检命令必须返回结构化 `inspection.status=unknown|unsupported`、明确原因和下一步动作；不得用 BPM 历史工单、Mongo 内部表或其他非实时来源伪装当前权限。
- `bytedoc access psm create --operation apply|modify|delete` 用于 PSM 授权新增、更新和删除；classic 使用 ByteDoc ticket `apply_account` / `modify_account` / `delete_account`，Volc / 多云使用 BPM workflow config `11975`，payload 内 `operation=1|2|3`、`auth_type=3`。
- `bytedoc access user grant` 给指定 principal 授权，`update` 修改有效期，`revoke` 回收权限，`apply` 给当前登录用户申请 IAM role。
- `bytedoc access user <grant|update|apply> --duration` 支持纯秒数、`h/H` 小时和 `d/D` 天，例如 `10800`、`3h`、`180d`；不支持 `m/M`，分钟级请换算成秒，按月有效期请换算成固定天数。
- Agent 必须先按 `data.result.nextActions` 把 dry-run 的可选项展示给用户并让用户选择，再用 `data.result.confirmation.reviewTable` 渲染表格给用户做最终确认，不能把字段挤成单行；用户确认页不要暴露隐藏执行材料。用户明确同意后，Agent 才可按 CLI 协议进入内部提交路径。
- `ticket create` 当前固定使用 ByteDoc 控制台已验证的 `user_type=PSM / User` 与 `mesh_method=Token + Mesh`；账号权限来自 schema 选项，例如 `read`、`readWrite`、`readWriteNoDrop`、`dbOwner`。
- 如果 `bytedoc shell`、`bytedoc collections`、`bytedoc document *` 返回 `BYTEDOC_ACCESS_REQUIRED`，Agent 应读取 `error.details.setup_commands` 并继续执行推荐的查询 / dry-run 命令，不需要让用户重新发现授权命令；Volc 场景的 setup commands 会先查询 `bytedoc access psm list`，再准备 `psm create` dry-run。
- classic / cloud-native 授权与权限管理按三步走：先 `role list` 展示可选项，再 `permission get` 检查前置权限，最后运行目标写命令 dry-run 展示完整工单或 IAM 字段；Volc PSM 授权必须走 `access psm list/create --backend volc`，不要使用 classic-only 的 `role list` / `permission get` 作为当前权限依据。只有用户确认 `data.result.confirmation.reviewTable` 后，Agent 才可按 CLI 协议进入内部提交路径。
- `bytedoc access` 后续命令必须保留当前全局 `--site` / `--vregion` 路由参数；用户确认页不要展示隐藏执行材料或底层 live 命令。
- `bytedoc slow-query subscribers` / `metrics` / `index-recommend` 在 classic 和 cloud-native 下都会统一走 slowquery 服务；执行前必须确认 backend/vregion，多值时让用户选择。
- `bytedoc slow-query detail` 在 classic 下既支持直接传 24 位 ObjectId，也支持传 `overview` 里的 fingerprint id；CLI 会自动尝试把 fingerprint 展开成 `_ids` 再查询 detail。
- `bytedoc shell` 的输入是 Mongo shell 风格 `--query` / `--query-file`；未以 `db.` 开头的查询需要 `--collection`，会自动改写成 `db.<collection>.<query>` 后再发给 DMS 或 DBW。以 `db.` 开头的库级只读命令（如 `db.getCollectionNames()`）不需要 `--collection`，但它走 ExecuteSQL 权限，不是 `collections` 主链路的等价替代。
- classic / cloud-native 不再走 legacy `web_query` 文本归一化链路；`bytedoc query` 和旧的 `bytedoc db query` 仍可作为 `shell` 别名使用。
- 复杂 aggregate / distinct / index 命令优先使用 `--query-file`。

## 选择器与权限枚举

- 定位阶段必须确认 `site`，并通过搜索/resolver 确认 `backend` 和 `vregion` 是否唯一；只有唯一候选才能进入后续目标操作。详见 `selection-guards.md`。
- 授权和写入阶段仍必须按对应 GUIDE 执行 role 枚举、dry-run/review 或用户确认；不要把“只需确认 site”理解成可以跳过授权和写入确认。
- PSM 授权 role 固定选项：`read`、`readWrite`、`readWriteNoDrop`、`dbOwner`、`dbOwnerNoDrop`、`enableShardingOnly`。
- IAM role-name 常用选项：`bytedoc.data_reader.cn`、`bytedoc.data_writer.cn`、`bytedoc.viewer.cn`、`bytedoc.operator.cn`、`bytedoc.owner.cn`、`owner`。
- `US-BOE` 是 `site=boe` 下的区域，不是 site 值。
