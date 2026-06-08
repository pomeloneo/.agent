---
name: bytedance-bytedoc
description: "Operate ByteDoc via bytedcli: search/list/get databases, plan SDK access paths, inspect access authorization roles, preflight database access apply flows, manage PSM authorization tickets and IAM user permissions, list collections, run safe Mongo shell style queries, perform allowed document list/insert/update operations for classic/cloud-native/Volc Mongo, and inspect slow-query data. Use when tasks mention ByteDoc, bytedoc, ByteDoc SDK access, Go SDK generation, ByteDoc database access authorization, PSM authorization management, user permission grant/update/revoke/apply, Mongo collections/documents in ByteDoc, ByteDoc slow queries, or when a ByteDoc/Mongo operation fails with not authorized, access denied, forbidden, no permission, or BYTEDOC_ACCESS_REQUIRED."
---

# bytedcli ByteDoc

## 如何调用 bytedcli

```bash
NPM_CONFIG_REGISTRY=http://bnpm.byted.org npx -y @bytedance-dev/bytedcli@latest <command> [options]
NPM_CONFIG_REGISTRY=http://bnpm.byted.org npm install -g @bytedance-dev/bytedcli@latest
bytedcli <command> [options]
```

下面示例默认直接写 `bytedcli`；使用 `npx` 时把 `bytedcli` 替换成上面的 npx 前缀。

## When to use

- 搜索、列出或查看 ByteDoc 数据库。
- 查看集合，执行 Mongo shell 风格命令。
- 对 classic / cloud-native / Volc Mongo ByteDoc 做集合创建；集合删除 / 重命名在所有站点均已由 bytedcli 禁用。
- 对 classic / cloud-native / Volc Mongo ByteDoc 做文档 list / insert / update；文档删除在所有站点均已由 bytedcli 禁用。
- 查看 ByteDoc 慢查询 overview、detail、scope、subscriber、metrics、index recommend。
- 用户只给数据库 PSM，希望获得 ByteDoc 接入指导、场景判断、本地 / TCE / FaaS 前置条件或 Go SDK 代码生成素材。
- 检查 ByteDoc 数据库访问授权选项、角色绑定与提单前置条件。
- 管理 ByteDoc 权限：PSM 授权新增 / 更新 / 删除，IAM 用户权限授权 / 改期 / 回收 / 自助申请。
- 当 ByteDoc / Mongo 访问失败并返回 `not authorized`、`access denied`、`forbidden`、`no permission`、`BYTEDOC_ACCESS_REQUIRED` 时，按错误里的 `setup_commands` 继续处理授权。

## 前置条件

- 需要鉴权时先登录：`bytedcli auth login`
- BOE ByteDoc / Volc Mongo 场景通常使用全局 `--site boe`。
- 海外 ByteDoc 控制面使用全局站点和 vregion，例如 `--site i18n-tt --vregion Singapore-Central`；`--site i18ntt` 可作为 `i18n-tt` 的别名。
- 更完整的调用、站点和 JSON 输出约定见 `../../invocation.md`。
- `bytedoc access` 跟随全局 `--site` / `--vregion` 路由，支持 CN、BOE、i18n 与 limited gateway 站点；支持 `--db-name`、`--service`、`--backend` 消歧。访问授权和权限管理写操作默认 dry-run，真实提交必须显式确认。

## Routing

- 主入口是 `bytedoc <action>`；旧的 `bytedoc db <action>` 作为兼容入口保留，不作为新示例首选。
- `bytedoc search` 在支持 Cloud Service Search 的站点上返回 `backend=classic|cloud-native|volc` 和原始 `mode`；不支持该聚合接口的海外控制面会自动回退到 classic / cloud-native 平台搜索。
- `volc` 是独立 backend，表示 Cloud Service Search 返回的 DBW / Volc Mongo 后端；不要把它归到 cloud-native backend。
- 海外控制面回退到平台搜索时没有独立 Volc 聚合来源；`--backend volc` 不会返回结果。
- 读集合和 Mongo shell 风格命令优先用 `bytedoc collections` / `bytedoc shell`；用户只给数据库 PSM 时直接传 `--service <psm>`，CLI 会自动解析 dbName、classic / cloud-native / Volc backend 与 DMS / DBW 数据面路由；不要先要求用户补 `--db-name`、`--deploy-mode`、`--site` 或 `--vregion`，除非 CLI 返回明确 ambiguity / routing unresolved。
- classic / cloud-native 的查询、允许的集合操作和文档操作统一走 DMS；Volc Mongo 走 DBW。
- DMS 查询必须使用当前全局 `--site` / `--vregion` 的 JWT、origin、referer 和 endpoint；海外 classic 查询不能回退到 CN JWT / origin / `dc=cn`。`i18n-bd` 的 DMS API host 是 `https://fedms-api.byteintl.net`，origin 是 `https://dms.byteintl.net`；`i18n-tt` 的 DMS API host 是 `https://fedms-i18n-api.byteintl.net`，origin 是 `https://dms-i18n.byteintl.net`；都不是 CN 的 `https://fedms-api.bytedance.net`。若错误出现 `ERR_CRYPTO_INVALID_STATE: Not initialised`，优先检查是否把 i18n JWT 发到了错误 DMS host。若 `subscribe/from_db` 没返回 `db.region` 且 CLI 返回 `BYTEDOC_ROUTING_UNRESOLVED`，应按错误里的 site / vregion / service 说明数据面 region 不可安全判定，不要盲试 CN。
- Resolver 不应把 dotted PSM 自动猜成 cloud-native；自动模式先查 Cloud Service Search、cloud-native 平台和 classic 平台，只有查到唯一匹配才执行。`BYTEDOC_NOT_FOUND` / `AUTH_REQUIRED` / `BYTEDOC_AMBIGUOUS` 应被视为解析结论或需要用户消歧的信号，不要让 Agent 自行补猜 backend。
- 结构化 Mongo 操作使用 `bytedoc collection ...` 和 `bytedoc document ...`。
- `bytedoc shell` / `query` 和结构化 Mongo 操作会在所有站点本地屏蔽破坏性删除操作，包括 `drop`、`renameCollection`、`deleteOne`、`deleteMany`、`remove`、`findOneAndDelete`、drop index 和 `bulkWrite` 删除；BOE 也不是例外。收到 `BYTEDOC_UNSAFE_OPERATION_BLOCKED` 后，不得改用 BOE、DBW、DMS direct helper、query-file 或脚本绕过。
- 访问授权主动入口使用 `bytedoc access role list`、`bytedoc access permission get`、`bytedoc access ticket create`、`bytedoc access psm list|create` 和 `bytedoc access user <grant|update|revoke|apply>`；Agent 应先展示角色、审批人、账号权限、principal（grant/update/revoke）、申请用户（apply）、有效期等可选项，再让用户选择目标值。
- `bytedoc access *` 写操作默认只做 dry-run，JSON 输出在 `data.result` 下包含 payload preview、`confirmation.requiredReview`、`confirmation.reviewTable`、`nextActions` 和 `confirmation.internalCommand`；`internalCommand` 仅供 Agent 在用户确认后内部执行，不能展示给用户。Agent 必须先按 `data.result.nextActions` 展示可选项并让用户选择，再用 `data.result.confirmation.reviewTable` 渲染表格给用户确认，不能把字段挤成单行。
- 被动授权入口来自结构化错误：如果查询/集合/文档操作返回 `BYTEDOC_ACCESS_REQUIRED`，优先执行 `error.details.setup_commands`，不要要求用户自己重新发现 access 命令。
- `list` / `follow` / `slow-query` 仍只有 classic / cloud-native 平台接口，没有独立 Volc 后端；Volc 页面上的日志、备份、参数等能力属于 DBW / Volc 详情面，不等同于这些平台接口。
- 慢查询使用 `bytedoc slow-query ...`；classic 的 detail 支持 fingerprint id 自动展开。
- ByteDoc SDK 无痛接入先用 `bytedoc sdk plan --service <psm> --access-env <env>`；TCE/FaaS 运行网络优先使用用户显式输入。若用户给了调用方 TCE PSM 但未说明 BOE/CN，直接传 `--caller-psm <caller-psm>` 让 CLI 自动识别；不要先要求用户自己判断。若 CLI 返回多环境证据，会按目标库网络选择可行路径并在 `runtimeNetworkResolution` / `warnings` 中说明假设。
- `sdk plan` 是通用接入指导，不绑定 Go；Go SDK 生成只是后续可选 language adapter。复杂接入矩阵见 `sdk-access/GUIDE.md`。
- 若目标 PSM 可能同时存在 classic、cloud-native、Volc 后端，或 CLI 返回 backend ambiguity，`sdk plan` / `sdk generate` 必须传 `--backend classic|cloud-native|volc` 消歧。
- `sdk plan` 默认使用固定权威 references；需要补充字节云动态搜索 Top3 时再传 `--reference-mode hybrid`。动态搜索结果只作为补充，不能覆盖固定接入矩阵。
- 只有 `sdk plan` 返回 `supportStatus=supported` 且 `codeGeneration.supported=true` 时，才调用 `bytedoc sdk generate --service <psm> --access-env <env> --runtime-network <boe|cn> --language go`；生成结果是给 coding agent 使用的代码素材，不代表 bytedcli 会修改业务仓库。
- 用户明确要求“生成代码”“SDK 代码”“查询一条数据的代码素材”时，最终答复必须直接展开 `sdk generate` 返回的 `codeSamples[].content`，不能只总结 `codeSamples` 数量、文件名、依赖或环境变量；这些摘要只能放在代码之后作为补充说明。
- SDK 接入类最终答复必须稳定包含“接入指导”：优先复述 `sdk generate` 的 `integrationGuidance[]`，再说明目标库、backend、access method、运行网络假设、collection 占位/真实值、验证方式和下一步建议；不能只输出代码、命令、权限提示或 doctor 诊断。若 collection 仍是占位值，用自然语言提示“如果不知道 collection 名，可以让我继续帮你查询”。
- SDK 接入流程中的 PSM 授权检查不能阻塞接入判断和代码生成：当用户同时提供调用方 PSM 与目标库 PSM 时，先完成 `sdk plan` / `sdk generate` 并输出代码，再在末尾说明当前授权状态；若已授权，说明已有权限并给验证命令；若未授权，询问用户是否需要继续辅助申请权限。
- Go SDK 生成必须使用 ByteDoc 官方内部 driver `code.byted.org/bytedoc/mongo-go-driver`；MongoDB 官方 driver 文档和 skill 只作为 API 形态与质量规则参考。
- `sdk generate` 返回的 `relatedGuides[]` 指向 ByteDoc 内部适配过的 MongoDB 官方 skill 分组；coding agent 应按这些 guide 补强连接、查询、优化和 schema 设计，不要求用户另装 MongoDB upstream skill。
- 当用户提供 ByteDoc SDK 连接失败、鉴权失败、超时、Consul / Mesh / DNS 报错或日志片段时，先调用 `bytedoc sdk doctor --error-text <text>`；若已知 PSM、backend、访问环境和运行网络，同时传 `--service`、`--backend`、`--access-env` 与 `--runtime-network`。

## MongoDB Quality Guides

这些子 GUIDE 保留 MongoDB 官方 agent-skills 的分组名称，但内容已按 ByteDoc 的 PSM、权限、Consul / Mesh、站点 / vRegion 和安全边界重写。维护来源、未引入的 upstream skill 与排除原因见 `references/mongodb-upstream-adaptation.md`。

| 场景                                                       | 使用指南                                     |
| ---------------------------------------------------------- | -------------------------------------------- |
| SDK 连接、连接失败、token / 临时账密、client 生命周期      | `mongodb-connection/GUIDE.md`                |
| 用户用自然语言描述查询需求，需要生成安全读查询             | `mongodb-natural-language-querying/GUIDE.md` |
| 查询慢、超时、慢查询、索引和 explain / query shape 分析    | `mongodb-query-optimizer/GUIDE.md`           |
| 集合结构、embed/reference、数组增长、TTL、schema trade-off | `mongodb-schema-design/GUIDE.md`             |

## 数据库访问与权限管理流程

Agent 处理 ByteDoc 授权和权限管理时必须按以下顺序执行，不要跳过 dry-run 或最终确认：

1. 先执行 `bytedoc access role list`，展示角色、当前绑定、数据库信息和可选项；如果用户只给 `db_name`，需要让 CLI 通过 `--db-name` 解析，必要时用 `--service` / `--backend` 消歧。
2. 再执行 `bytedoc access permission get`，确认当前用户是否具备发起授权流程的前置权限；若返回阻断原因，先向用户解释，不要直接提单。
3. 数据库接入授权使用不带 `--execute` 的 `bytedoc access ticket create` dry-run，展示 `data.result.options`、`data.result.payload`、`data.result.confirmation.reviewTable` 和 `data.result.nextActions`。
4. PSM 授权管理先用 `bytedoc access psm list` 检查当前读取状态；若要判断某个调用方 PSM 是否已有权限，传 `--account <caller.psm>` 并读取 `accountCheck.status` 与 `accountCheck.roles`。Volc / DBW 的 PSM token 授权读取使用 ByteDoc multi-cloud `GetAccount`，`inspection.source=bytedoc_multicloud_get_account`；不能用 ByteCloud IAM ACL node role bindings 判定。随后使用 `bytedoc access psm create --operation apply|modify|delete` dry-run；classic 走 ByteDoc ticket，Volc / 多云走 BPM workflow `11975`，确认后才提交。
5. 用户权限管理使用 `bytedoc access user grant|update|revoke|apply` dry-run；`grant` 用于给他人授权，`update` 改有效期，`revoke` 回收权限，`apply` 给当前登录用户申请角色。`--duration` 支持纯秒数、`h/H` 小时和 `d/D` 天，例如 `10800`、`3h`、`180d`；不要使用 `m/M`，分钟级请换算成秒，按月有效期请换算成固定天数。

关键约束：

- `bytedoc access` 跟随全局 `--site` / `--vregion`；如果 dry-run 输出后续可执行命令，这些命令必须保留当前路由参数，避免确认前后落到不同控制面。
- 用户确认页只展示操作类型、角色、principal / 申请用户、审批人、账号权限、有效期、原因、region / vRegion、数据库 PSM 等完整工单字段，不展示 `confirmation.internalCommand` 或带 `--execute` 的底层命令；用户确认后由 Agent 内部执行 `internalCommand`，不要要求用户复制命令。
- 真实提交必须同时具备用户明确确认、`--execute` 和 `--yes-i-know-this-is-live`；否则只允许停留在 dry-run。
- 如果查询/集合/文档操作返回 `BYTEDOC_ACCESS_REQUIRED`，优先读取 `error.details.setup_commands` 并进入上述流程，不要让用户重新发现 access 命令。
- Volc 未授权错误的 `setup_commands` 应先运行 `access psm list --backend volc` 查询当前读取状态，再进入 `access psm create --backend volc` dry-run 预览；不要先走 classic 专用的 `access role list` / `permission get` 巡检链路。主动查询某个 PSM 是否已有权限时，使用 `access psm list --backend volc --account <caller.psm>`；若返回 `accountCheck.status=authorized`，直接使用 `accountCheck.roles` 判断已有权限。不要把 BPM ticket 历史或 IAM role bindings 当作当前 PSM 授权。

## Quick start

```bash
# 搜索 / 列表 / 详情
bytedcli bytedoc search --keyword "demo_orders"
bytedcli bytedoc search --keyword "demo_orders" --backend volc
bytedcli --site i18n-tt --vregion Singapore-Central bytedoc search --keyword "demo_orders"
bytedcli --json bytedoc list
bytedcli --json bytedoc list --all --deploy-mode cloud-native
bytedcli --json bytedoc get --service "demo_orders" --deploy-mode classic

# 集合与 Mongo shell 风格命令
bytedcli --json bytedoc collections --service "demo_orders"
bytedcli --json bytedoc shell --service "demo_orders" --collection "demo_records" --query 'find().limit(10)'
bytedcli --site i18n-bd --vregion Asia-SouthEastBD --json bytedoc shell --service "bytedance.bytedoc.seedreplay" --collection "snapshots_v2" --query 'find().limit(1)'
bytedcli --json bytedoc shell --service "example.bytedoc.demo_catalog" --backend volc --collection "demo_items" --query 'find().limit(10)'

# classic / cloud-native / Volc Mongo 结构化操作
bytedcli --json bytedoc collection create --service "example.bytedoc.demo_catalog" --collection "demo_items"
bytedcli --json bytedoc document list --service "example.bytedoc.demo_catalog" --collection "demo_items" --filter-json '{"tenant":"demo"}' --limit 10
bytedcli --json bytedoc document insert --service "example.bytedoc.demo_catalog" --collection "demo_items" --doc-json '{"tenant":"demo","value":1}'

# 数据库访问授权角色与权限检查
bytedcli --json bytedoc access role list --service "example.bytedoc.demo_orders" --backend classic
bytedcli --json bytedoc access permission get --db-name "demo_orders" --role-name "bytedoc.data_reader.cn"
bytedcli --site i18n-tt --vregion Singapore-Central --json bytedoc access role list --db-name "demo_orders"
bytedcli --json bytedoc access ticket create --db-name "demo_orders" --account "example.caller.psm"
bytedcli --json bytedoc access ticket create --db-name "demo_orders" --account "example.caller.psm" --reviewers "alice" --roles "read" --reason "Need read access for service integration"
bytedcli --json bytedoc access ticket create --db-name "demo_orders" --account "example.caller.psm" --reviewers "alice" --roles "read" --reason "Need read access for service integration" --execute --yes-i-know-this-is-live
bytedcli --site boe --json bytedoc access psm list --service "example.bytedoc.demo_volc" --backend volc
bytedcli --site boe --json bytedoc access psm list --service "example.bytedoc.demo_volc" --backend volc --account "example.caller.psm"
bytedcli --json bytedoc access psm create --db-name "demo_orders" --operation apply --account "example.biz.psm" --reviewers "alice" --roles "read" --reason "Need PSM read authorization"
bytedcli --site boe --json bytedoc access psm create --service "example.bytedoc.demo_volc" --backend volc --operation modify --account "example.biz.psm" --roles "readWrite" --reason "Update Volc PSM authorization"
bytedcli --json bytedoc access user grant --db-name "demo_orders" --role-name "bytedoc.viewer.cn" --principal "demo.user" --principal-type user --duration 30d --reason "Need temporary view access"
bytedcli --json bytedoc access user update --db-name "demo_orders" --role-name "bytedoc.viewer.cn" --principal "demo.user" --duration 7d --reason "Shorten authorization"
bytedcli --json bytedoc access user revoke --db-name "demo_orders" --role-name "bytedoc.viewer.cn" --principal "demo.user" --reason "Access no longer needed"
bytedcli --json bytedoc access user apply --db-name "demo_orders" --role-name "bytedoc.viewer.cn" --duration 180d --reason "Need self-service view access"

# 慢查询
bytedcli --json bytedoc slow-query overview --service "demo_orders" --deploy-mode classic --millis 100

# SDK 接入计划
bytedcli --json bytedoc sdk plan --service "example.bytedoc.demo_orders" --access-env local-mac --language go
bytedcli --json bytedoc sdk plan --service "example.bytedoc.demo_orders" --access-env tce --runtime-network cn --reference-mode hybrid --language go
bytedcli --json bytedoc sdk generate --service "example.bytedoc.demo_orders" --access-env tce --runtime-network cn --language go --collection "demo_items" --operation find-one
bytedcli --json bytedoc sdk doctor --service "example.bytedoc.demo_orders" --access-env tce --runtime-network cn --error-text "not authorized on demo_orders"
```

## Notes

- 需要结构化输出加 `--json`，并放在 `bytedoc` 前面。
- 完整 collection / document 命令矩阵、`--query-file`、JSON 文件入参和慢查询命令矩阵见 `references/bytedoc.md`。
- 复杂 aggregate / distinct / index 命令优先使用 `--query-file`，减少 shell quoting 干扰。
- 对现有业务集合执行写入前，先确认目标 service、db、collection 和 filter；破坏性删除、drop 和 rename 操作在所有站点都会被 bytedcli 本地拒绝。
- `bytedoc access ticket create` 用于创建 PSM/User 接入授权工单；`bytedoc access psm create` 用于 PSM 授权新增 / 更新 / 删除；`bytedoc access user grant|update|revoke|apply` 用于 IAM 用户权限管理。
- 所有 `bytedoc access` 写操作第一次运行不要带 `--execute`，先展示 options / payload / `confirmation.reviewTable` / `nextActions` 给用户确认；不要把底层执行命令展示给用户。
- 所有 `bytedoc access` 写操作带 `--execute` 时必须同时带 `--yes-i-know-this-is-live`；缺少确认标记时 CLI 会拒绝提交并返回确认命令。
- JSON 错误中若出现 `error.details.setup_commands`，Agent 应直接执行或引导用户执行这些命令完成授权检查。
- 当前授权真源接口未确认时，ByteDoc 权限列表 / 巡检命令必须返回结构化 `inspection.status=unknown|unsupported`、明确原因和下一步动作；不得用 BPM 历史工单、Mongo 内部表或其他非实时来源伪装当前权限。

## References

- `references/bytedoc.md`
- `../../invocation.md`
- `../../troubleshooting.md`
- `sdk-access/GUIDE.md`
