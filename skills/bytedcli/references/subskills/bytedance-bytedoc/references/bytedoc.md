# ByteDoc

ByteDoc 数据库搜索、列表、详情、关注、集合查看、允许的文档读写、安全 Mongo shell 风格命令与慢查询分析统一收敛在 `bytedoc <action>`。

旧的 `bytedoc db <action>` 兼容入口仍可使用；新示例和新脚本优先使用 flat 命令。

## 常见命令

```bash
# 搜索数据库
bytedcli bytedoc search --keyword "demo_orders"
bytedcli bytedoc search --keyword "bytedoc.demo_catalog" --backend cloud-native
bytedcli bytedoc search --keyword "bytedoc.demo_catalog" --backend volc
bytedcli --site i18n-tt --vregion Singapore-Central bytedoc search --keyword "demo_orders"

# 查看我关注的库 / 数据库详情
bytedcli --json bytedoc list
bytedcli --json bytedoc list --all --deploy-mode cloud-native
bytedcli --json bytedoc get --service "demo_orders" --deploy-mode classic

# 数据库访问授权角色 / 权限检查（跟随全局 --site / --vregion）
bytedcli --json bytedoc access role list --service "example.bytedoc.demo_orders" --backend classic
bytedcli --json bytedoc access permission get --db-name "demo_orders" --role-name "bytedoc.data_reader.cn"
bytedcli --site i18n-tt --vregion Singapore-Central --json bytedoc access role list --db-name "demo_orders"

# PSM/User 接入授权工单（先 dry-run 展示可选项和 payload，用户确认后再 execute）
bytedcli --json bytedoc access ticket create --db-name "demo_orders" --account "example.caller.psm"
bytedcli --json bytedoc access ticket create --db-name "demo_orders" --account "example.caller.psm" --reviewers "alice" --roles "read" --reason "Need read access for service integration"
bytedcli --json bytedoc access ticket create --db-name "demo_orders" --account "example.caller.psm" --reviewers "alice" --roles "read" --reason "Need read access for service integration" --execute --yes-i-know-this-is-live

# PSM 授权管理：classic 走 ByteDoc ticket，Volc / 多云走 BPM workflow 11975
bytedcli --site boe --json bytedoc access psm list --service "example.bytedoc.demo_volc" --backend volc
bytedcli --json bytedoc access psm create --db-name "demo_orders" --operation apply --account "example.biz.psm" --reviewers "alice" --roles "read" --reason "Need PSM read authorization"
bytedcli --json bytedoc access psm create --db-name "demo_orders" --operation modify --account "example.biz.psm" --reviewers "alice" --roles "readWrite" --reason "Update PSM authorization"
bytedcli --json bytedoc access psm create --db-name "demo_orders" --operation delete --account "example.biz.psm" --reviewers "alice" --roles "read" --reason "Remove PSM authorization"
bytedcli --site boe --json bytedoc access psm create --service "example.bytedoc.demo_volc" --backend volc --operation apply --account "example.biz.psm" --roles "read" --reason "Need Volc PSM authorization"

# IAM 用户权限管理：授权、改期、回收、自助申请
bytedcli --json bytedoc access user grant --db-name "demo_orders" --role-name "bytedoc.viewer.cn" --principal "demo.user" --principal-type user --duration 30d --reason "Need temporary view access"
bytedcli --json bytedoc access user update --db-name "demo_orders" --role-name "bytedoc.viewer.cn" --principal "demo.user" --duration 7d --reason "Shorten authorization"
bytedcli --json bytedoc access user revoke --db-name "demo_orders" --role-name "bytedoc.viewer.cn" --principal "demo.user" --reason "Access no longer needed"
bytedcli --json bytedoc access user apply --db-name "demo_orders" --role-name "bytedoc.viewer.cn" --duration 180d --reason "Need self-service view access"

# 集合与慢查询
bytedcli --json bytedoc collections --service "demo_orders"
bytedcli --json bytedoc collections --db-name demo_catalog
bytedcli --json bytedoc collection create --service "example.bytedoc.demo_catalog" --collection "demo_items"
bytedcli --json bytedoc slow-query overview --service "demo_orders" --deploy-mode classic --millis 100
bytedcli --json bytedoc slow-query metrics --service "demo_orders" --deploy-mode classic --interval 5m

# Mongo shell 风格命令（默认自动识别 classic / cloud-native / volc；同名歧义时优先用 --backend）
bytedcli --json bytedoc shell --service "demo_orders" --collection "demo_records" --query 'find().limit(10)'
bytedcli --json bytedoc shell --service "demo_orders" --collection "demo_records" --query-file ./query.mongo
bytedcli --json bytedoc shell --db-name demo_catalog --collection "demo_items" --query 'find().limit(10)'
bytedcli --json bytedoc shell --db-name demo_catalog --backend cloud-native --collection "demo_items" --query 'find().limit(10)'
bytedcli --json bytedoc shell --service "example.bytedoc.demo_catalog" --backend volc --collection "demo_items" --query 'find().limit(10)'

# 文档操作（classic / cloud-native / Volc Mongo）
bytedcli --json bytedoc document list --service "example.bytedoc.demo_catalog" --collection "demo_items" --filter-json '{"tenant":"demo"}' --limit 10
bytedcli --json bytedoc document insert --service "example.bytedoc.demo_catalog" --collection "demo_items" --doc-json '{"tenant":"demo","value":1}'
bytedcli --json bytedoc document update --service "example.bytedoc.demo_catalog" --collection "demo_items" --filter-json '{"tenant":"demo"}' --update-json '{"$set":{"value":2}}'
```

## 说明

### 后端分类

- `backend=classic`：传统 ByteDoc，Cloud Service Search 原始 `mode=classic`；Mongo 查询、允许的集合操作和文档操作走 DMS subscribe / evaluate，列表、关注、详情、慢查询走 classic ByteDoc API。
- `backend=cloud-native`：cloud-native ByteDoc，Cloud Service Search 原始 `mode=cloud-native`；Mongo 查询、允许的集合操作和文档操作走 DMS subscribe / evaluate，慢查询走 cloud-native slowquery API。
- `backend=volc`：DBW / Volc Mongo，Cloud Service Search 原始 `mode=volc`；实例信息来自 `instance_id`、`instance_type`、`region`、`vregion`，集合、查询和允许的文档/集合写操作走 DBW Mongo 执行链路。
- `mode` 是 Cloud Service Search 返回的原始字段；`backend` 是 CLI 对外的严格三态选择器；`deployMode` 是 legacy 平台路由字段，仅有 `classic|cloud-native` 两态。

### 命令链路

- `bytedoc search` 在支持 Cloud Service Search 的站点上默认返回 `backend=classic|cloud-native|volc` 和原始 `mode`；`--backend` 是严格三态过滤项，`--deploy-mode` 仅作为 legacy 两态兼容过滤项。
- 海外控制面（例如 `--site i18n-tt --vregion Singapore-Central`）如果没有 Cloud Service Search 聚合接口，会自动回退到 classic / cloud-native 平台搜索；此时没有独立 Volc 聚合来源，`--backend volc` 不返回结果。
- `bytedoc list` 默认展示关注列表；未显式传 `--deploy-mode` 时会合并 classic 和 cloud-native 的关注列表。
- `bytedoc list --all` 展示全量数据库；未显式传 `--deploy-mode` 时默认查 `classic`。
- `bytedoc list` / `bytedoc follow` / `bytedoc slow-query *` 仍只有 classic / cloud-native 平台接口，没有独立 Volc 后端；Volc 页面上的日志、备份、参数等能力属于 DBW / Volc 详情面。
- `bytedoc get` 可以传 `--db-name` 或 `--service`；classic / cloud-native 返回平台详情，Volc Mongo 只在解析到 DBW 元信息时返回 Cloud Service / DBW summary，且不返回 classic usage。
- `bytedoc collections` 不再需要 `--deploy-mode`；CLI 会自动识别 classic、cloud-native 或 Volc Mongo 数据库，也可用 `--backend classic|cloud-native|volc` 消歧。
- `bytedoc shell` 默认不需要 `--deploy-mode`；CLI 会自动识别 classic、cloud-native 或 Volc Mongo 数据库，并在对应场景下自动选择查询链路。同一个 `service` / `dbName` 存在多个 backend 时，显式传 `--backend classic|cloud-native|volc` 消歧；`--deploy-mode` 只能区分 legacy `classic|cloud-native` 两态。
- `bytedoc collection create` 与 `bytedoc document <list|insert|update>` 面向 classic / cloud-native / Volc Mongo；classic / cloud-native 使用 DMS，Volc 使用 DBW。
- 所有站点下，`bytedoc collection drop/rename`、`bytedoc document delete`、以及 `bytedoc shell/query` 中的 `drop`、`renameCollection`、`deleteOne`、`deleteMany`、`remove`、`findOneAndDelete`、drop index 和 `bulkWrite` 删除都会被本地拒绝，不会请求 DMS 或 DBW；BOE 也不是例外。收到 `BYTEDOC_UNSAFE_OPERATION_BLOCKED` 后，不得改用 BOE、DBW、DMS direct helper、query-file 或脚本绕过。
- `bytedoc document list` 是结构化文档查询入口；`bytedoc document find` / `query` 仍可作为兼容别名使用。
- `bytedoc access role list` 会读取数据库静态信息、IAM resource info 和角色绑定，输出可供 Agent 展示给用户选择的角色列表；支持 `--db-name`、`--service`、`--backend` 消歧。
- `bytedoc access permission get` 会校验当前用户是否具备发起授权流程的前置权限。
- `bytedoc access ticket create` 会读取已验证的 `apply_account` 表单 schema，JSON 输出在 `data.result` 下包含审批人和账号权限可选项、payload preview、`confirmation.requiredReview`、`confirmation.reviewTable`、`nextActions` 与 `confirmation.internalCommand`；默认 dry-run，不提交工单。
- `bytedoc access psm list` 用于主动查询 PSM 授权读取状态，并给出 `psm create` dry-run 的下一步动作。传 `--account <caller.psm>` 后读取 `accountCheck.status` 与 `accountCheck.roles`；Volc / DBW 的 PSM token 授权读取使用 ByteDoc multi-cloud `GetAccount`，`inspection.source=bytedoc_multicloud_get_account`。不要用 ByteCloud IAM ACL node 或 `role-bindings/list-by-resource` 判定 Volc PSM token 授权，也不要把 BPM ticket 历史当作当前权限。
- 如果当前授权真源接口未确认或读取失败，权限列表 / 巡检命令必须返回结构化 `inspection.status=unknown|unsupported`、明确原因和下一步动作；不得用 BPM 历史工单、Mongo 内部表或其他非实时来源伪装当前权限。
- `bytedoc access psm create --operation apply|modify|delete` 用于 PSM 授权新增、更新和删除；classic 使用 ByteDoc ticket `apply_account` / `modify_account` / `delete_account`，Volc / 多云使用 BPM workflow config `11975`，payload 内 `operation=1|2|3`、`auth_type=3`。
- `bytedoc access user grant` 给指定 principal 授权，`update` 修改有效期，`revoke` 回收权限，`apply` 给当前登录用户申请 IAM role。
- `bytedoc access user <grant|update|apply> --duration` 支持纯秒数、`h/H` 小时和 `d/D` 天，例如 `10800`、`3h`、`180d`；不支持 `m/M`，分钟级请换算成秒，按月有效期请换算成固定天数。
- Agent 必须先按 `data.result.nextActions` 把 dry-run 的可选项展示给用户并让用户选择，再用 `data.result.confirmation.reviewTable` 渲染表格给用户做最终确认，不能把字段挤成单行；用户确认页不要暴露底层执行命令。用户明确同意后，Agent 才可内部执行 `data.result.confirmation.internalCommand`。
- `ticket create` 当前固定使用 ByteDoc 控制台已验证的 `user_type=PSM / User` 与 `mesh_method=Token + Mesh`；账号权限来自 schema 选项，例如 `read`、`readWrite`、`readWriteNoDrop`、`dbOwner`。
- 如果 `bytedoc shell`、`bytedoc collections`、`bytedoc document *` 返回 `BYTEDOC_ACCESS_REQUIRED`，Agent 应读取 `error.details.setup_commands` 并继续执行推荐的查询 / dry-run 命令，不需要让用户重新发现授权命令；Volc 场景的 setup commands 会先查询 `bytedoc access psm list`，再准备 `psm create` dry-run。
- 授权与权限管理必须按三步走：先 `role list` 展示可选项，再 `permission get` 检查前置权限，最后运行目标写命令 dry-run 展示完整工单或 IAM 字段；只有用户确认 `data.result.confirmation.reviewTable` 后，Agent 才可内部执行 `data.result.confirmation.internalCommand`。
- `bytedoc access` 后续命令必须保留当前全局 `--site` / `--vregion` 路由参数；用户确认页不要展示 `confirmation.internalCommand` 或带 `--execute` 的底层命令。
- `bytedoc slow-query subscribers` / `metrics` / `index-recommend` 在 classic 和 cloud-native 下都会统一走 slowquery 服务；只给 classic `--db-name` 时，CLI 会先解析 service。
- `bytedoc slow-query detail` 在 classic 下既支持直接传 24 位 ObjectId，也支持传 `overview` 里的 fingerprint id；CLI 会自动尝试把 fingerprint 展开成 `_ids` 再查询 detail。
- `bytedoc shell` 的输入仍是 `--collection` 和 Mongo shell 风格 `--query` / `--query-file`；未以 `db.` 开头的查询会自动改写成 `db.<collection>.<query>` 后再发给 DMS 或 DBW。
- classic / cloud-native 不再走 legacy `web_query` 文本归一化链路；`bytedoc query` 和旧的 `bytedoc db query` 仍可作为 `shell` 别名使用。
- 复杂 aggregate / distinct / index 命令优先使用 `--query-file`。
