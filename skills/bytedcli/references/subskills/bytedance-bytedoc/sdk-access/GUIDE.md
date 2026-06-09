# ByteDoc SDK 接入指南

当用户希望从代码访问 ByteDoc 数据库、询问授权后如何连接，或只提供 ByteDoc PSM 并希望获得 SDK 接入帮助时，使用本指南。

## 首屏规则

- 先确认 `site`；再确认目标数据库的 `backend` 和 `vregion` 是否唯一。缺少 `site` 时按 `../references/selection-guards.md` 询问。
- `backend` 和 `vregion` 没有默认值：只能由用户明确提供，或由搜索/resolver 证明唯一；任一维度多值时必须让用户选择。
- `BYTEDOC_AMBIGUOUS` / `agent_protocol.next_state=ASK_USER` 是硬边界：展示候选表，不要自行选择后继续 `sdk plan` / `sdk generate`。
- 先运行 `sdk plan` 判断接入方式、前置条件、warning 和 next actions；`supportStatus=unsupported|needs_input` 时停止，不生成 SDK 代码。
- 对 `unsupported` 结果要解释原因，不要用猜测连接串绕过；对 `ambiguous` 结果，询问 CLI 返回的 `questions[]`。
- `accessMethod.kind=visual-query` 只说明用户要可视化查询，不需要 SDK 代码；不要把 `sdk plan` 的可视化建议当成 `collections` / `shell` / `document` 查询命令直接执行。
- 如果用户在 `visual-query` 后继续要求查看集合、抽样数据或执行查询，切换到 `mongo-ops` 或 `mongodb-natural-language-querying` 流程。
- ByteDoc 专属决策优先于通用 MongoDB 最佳实践；Mesh、Consul、Token、临时凭证、PSM 授权和 BOE/CN 网络边界由 ByteDoc 决定。
- 将 bytedcli 视为 ByteDoc 专家工具，不要从 bytedcli 扫描或修改用户项目；coding agent 在用户确认后再把返回的方案和代码素材应用到项目中。

## 核心流程

- Go SDK 生成请求：确认 site 后，先用 `bytedoc search` 或 `sdk plan` resolver 确认 backend/vregion 唯一；唯一后后续命令显式携带已确认的 `--backend` 和必要的全局 `--vregion`。只有纯接入方案问题且不需要代码时，才可以省略 `--language go`。
- 对 `tce` 和 `faas`，如果用户已知业务运行网络，传 `--runtime-network boe|cn`；如果用户没有指定接入环境，使用 `--access-env auto`。
- 如果已知是 `tce/faas` 但运行网络未知，且有调用方 TCE PSM，不要先问用户；传 `--caller-psm <caller-psm>`，让 CLI 检查 TCE / Keel 证据并填充 `runtimeNetwork`。
- 明确的 BOE 服务证据映射到 `--runtime-network boe`，明确的 CN / online_cn / prod CN 证据映射到 `--runtime-network cn`；如果服务出现在多个环境且目标库网络已知，只把 CLI 的 `target-network-fallback` 作为证据展示。若仍需要用户决策，必须停止并询问，不把 fallback 当成可继续执行的猜测依据。
- 只有当用户输入和 TCE/ENV 查询都无法识别运行网络时，才省略 `--runtime-network` 并询问 CLI 返回的 `questions[]`。
- 默认使用 `--reference-mode fixed`。只有用户明确希望补充字节云文档搜索结果，或担心固定链接过期时，才使用 `--reference-mode hybrid`。
- 如果 `sdk plan` 返回 `references[]`，向用户展示每个 reference 的标题、URL 和原因；固定 reference 可作为 unsupported 或 ambiguous 结论的证据。
- 如果 `referenceSearch.status=success`，只把动态 Top3 reference 当作补充材料；不要让动态结果覆盖确定性接入矩阵。若 `referenceSearch.status=error`，继续使用固定 reference，并说明补充搜索不可用。

## DO / DON'T

| 场景 | DO | DON'T |
| --- | --- | --- |
| 用户说"生成 SDK 代码" | 先 `sdk plan` 确认支持，再 `sdk generate` | 不要跳过 plan 直接 generate |
| `supportStatus=unsupported` | 展示原因和 references，停止生成代码 | 不要猜一个连接串给用户 |
| `accessMethod.kind=visual-query` | 口语化说明可用控制台可视化查看；继续查询时切到查询流程 | 不要直接执行 `collections` / `shell` / `document` |
| 用户没说 collection | 先询问或列集合 | 不要用 `BYTEDOC_COLLECTION` 占位值当真实代码 |
| 用户有 caller PSM | 传 `--caller-psm` 让 CLI 检查 TCE 证据 | 不要直接问用户 runtime-network |
| `access-env` 未知 | 使用 `--access-env auto` | 不要猜 `tce` 或 `local-mac` |
| Volc 后端 | 按 resolver 返回的结果走，用 `access psm list --backend volc` | 不要用 classic 的 `role list` 验证 Volc 权限 |

## Access Env 取值

- `local-mac`：本地 Mac 开发。
- `devbox`：DevBox 开发。
- `tce`：TCE 运行时。
- `faas`：ByteFaaS 运行时。
- `visual`：仅可视化查询。
- `auto`：接入环境未知；仅在用户没有提供足够上下文时使用。

## 当前矩阵

- 区分 `accessEnv` 和 `runtimeNetwork`：`accessEnv` 描述 local Mac、DevBox、TCE、FaaS 或 visual access；`runtimeNetwork` 描述 TCE/FaaS 运行在 BOE 还是 CN。
- CN classic 从 CN TCE/FaaS 访问使用 `consul-token`。
- CN classic 从 BOE 本地开发或 BOE TCE/FaaS 访问不支持，因为网络不可达。
- BOE Volc MongoDB 4.0 从 BOE 本地开发或 BOE TCE/FaaS 访问使用 `consul-token`。
- BOE Volc MongoDB 8.0 从本地开发访问使用 `consul-temporary-credential`；临时凭证仅用于开发。
- BOE Volc MongoDB 8.0 从 BOE TCE/FaaS 访问使用 `mesh-token`，且需要 ByteDoc Service Mesh egress。
- BOE Volc 从 CN TCE/FaaS 访问不支持，因为网络不可达。
- 仅可视化访问使用 `visual-query`，不得触发 SDK 代码生成。

## Go SDK 素材

- 仅当 `sdk plan` 支持且 backend/vregion 已确认后，才调用 `bytedcli --json --site <site> --vregion <confirmed-vregion> bytedoc sdk generate --service <psm> --backend <confirmed-backend> --access-env <env> --runtime-network <boe|cn> --language go --collection <collection> --operation find-one`。
- `local-mac`、`devbox` 和 `visual` 省略 `--runtime-network`；`tce` 和 `faas` 需要携带。
- 将 `codeSamples[]` 视为给 coding agent 适配到业务仓库的素材。
- 当用户要求 SDK 代码、Go 代码或代码素材时，最终答复先完整展开 `codeSamples[].content`，再总结依赖、环境变量或验证命令。
- 代码后必须包含“接入指导”段落。优先使用 `integrationGuidance[]`，再总结选定接入方式、运行网络假设、collection 占位/真实值、验证路径和下一步友好动作。
- 即使答复还包含权限状态、授权 dry-run 输出、warning 或 SDK doctor 诊断，也不要省略“接入指导”。
- 如果 `BYTEDOC_COLLECTION` 仍是占位值，不要只输出 CLI 命令；需要说明“如果不知道 collection 名，可以让我继续帮你查询一下”。
- 如果用户要求真实可运行代码但未提供 collection，先询问 collection，不能猜业务集合名。
- 生成的 Go 代码必须使用 ByteDoc 官方 driver `code.byted.org/bytedoc/mongo-go-driver`，不要直接使用 upstream `go.mongodb.org/mongo-driver` module。
- 阅读并应用 `sdk generate` 返回的 `relatedGuides[]`；这些 guide 指向 ByteDoc 适配过的 MongoDB 官方 skill 指南。
- 不要从 bytedcli 把生成代码写入用户仓库。
- 保留质量要求：复用 `mongo.Client`，使用 `context.WithTimeout`，不要记录 token 或完整 URI，除非用户明确要求，否则避免写操作。

## 授权检查位置

- 将 PSM 授权视为运行时就绪检查，而不是生成 SDK 代码素材的阻断条件。
- 如果用户同时提供调用方 PSM 和目标 ByteDoc PSM，且接入矩阵支持，先完成 `sdk plan` 和 `sdk generate`。
- 代码素材之后，再执行或推荐 `bytedoc access` 流程检查授权：先 `access role list`，再 `access permission get`，只有授权缺失且用户希望继续协助申请时，才进入 `access psm create` dry-run。
- 如果已经授权，说明当前权限并继续给验证命令。
- 如果授权缺失，不要用授权 warning 替代生成代码；在末尾追加“权限状态”段落，询问是否继续协助申请授权。
- 授权 dry-run 可以主动执行，但真实提交必须获得用户明确确认。确认后由 Agent 自己遵循 CLI 确认协议；不要让用户复制隐藏 live-submit 命令。
- 对 `backend=volc`，不要用 classic 专属的 `access role list` / `access permission get` 证明已有授权。使用 `access psm list --backend volc --account <caller.psm>` 或 `BYTEDOC_ACCESS_REQUIRED.details.setup_commands`；当返回 `inspection.source=bytedoc_multicloud_get_account` 和 `accountCheck.status` 时信任它们，只有授权缺失时才准备 `access psm create --backend volc` dry-run。

## SDK 诊断

- 当用户报告 SDK 连接、鉴权、超时、Consul、Mesh、DNS 或权限失败时，使用 `bytedcli --json bytedoc sdk doctor --error-text "<error text>"`。
- 如果用户已提供相关信息，携带 `--service <psm>`、`--access-env <env>` 和 `--runtime-network <boe|cn>`；site 未知时先询问。
- 将返回的 `category`、`severity`、`evidence`、`likelyCauses`、`nextActions` 和 `verificationCommands` 作为给 coding agent 和用户的诊断材料。
- 如果 category 是 `permission`，先遵循返回的 `bytedoc access ...` dry-run 流程，再创建工单。
- 如果 category 是 `network` 或 `credential`，运行 `sdk plan`，并将实际代码/运行环境与返回的接入方式对齐。
- 诊断时不要从 bytedcli 扫描或修改业务代码；coding agent 可以在展示 CLI 诊断后再检查用户项目。

## 已适配的 MongoDB 指南

- `mongodb-connection/GUIDE.md`：client 生命周期、context timeout、token / 临时凭证处理和连接故障排查。
- `mongodb-natural-language-querying/GUIDE.md`：将用户查询意图适配为有边界的 read filter、projection 和 sample query。
- `mongodb-query-optimizer/GUIDE.md`：慢查询、索引、explain、sort 和 query shape 分析。
- `mongodb-schema-design/GUIDE.md`：生成 SDK struct 或查询需求暴露 schema 设计取舍时使用。
- upstream 来源和排除的 MongoDB 官方 skill 记录在 `../references/mongodb-upstream-adaptation.md`。
- MongoDB 官方 agent-skill 指南已经适配进 ByteDoc 规则；不要要求用户另装 upstream MongoDB skill。
- ByteDoc SDK 包选择基于 ByteDoc 官方文档；具体链接以 `sdk plan` / `sdk generate` 返回的 `references[]` 为准，不在 Skill 中固化具体文档 token。

## 端到端示例：SDK 接入完整流程

```bash
# 场景：用户说"帮我生成 example_db 的 Go 接入代码，我在 TCE 环境"

# Step 1: 确认 site 后解析目标数据库
bytedcli --json --site cn bytedoc search --keyword "example.bytedoc.example_db"
# → 确认唯一候选：backend=classic, vregion=China-North

# Step 2: 执行 plan（显式带入已确认 backend/vregion）
bytedcli --json --site cn --vregion China-North bytedoc sdk plan --service "example.bytedoc.example_db" --backend classic --access-env tce --language go
# → 返回 supportStatus=supported, accessMethod=consul-token, runtimeNetwork=cn

# Step 3: 生成代码
bytedcli --json --site cn --vregion China-North bytedoc sdk generate --service "example.bytedoc.example_db" --backend classic --access-env tce --runtime-network cn --language go --collection "orders" --operation find-one
# → 返回 codeSamples[] + integrationGuidance[]

# Step 4: 展示代码和接入指导
# Agent 完整展示 codeSamples[].content，再附接入指导段落

# Step 5: 检查权限（后置，不阻断代码生成）
bytedcli --json --site cn --vregion China-North bytedoc access psm list --service "example.bytedoc.example_db" --backend classic --account "example.caller.psm"
# → 如果缺少权限，追加"权限状态"段落询问是否协助申请
```

## Backend 感知解析

- ByteDoc 有三个 backend（`classic`、`cloud-native`、`volc`），元数据来源不同；`sdk plan` 必须复用 resolver 确认的来源，不要无条件调用 legacy classic / cloud-native detail API。
- 对 `backend=volc`（DBW / Volc Mongo），使用 resolver 返回的 Cloud Service Search summary；不要调用 cloud-native `/api/service/:service` 的 `getDatabaseDetail`，该接口对 Volc 数据库会返回 `mongo: no documents in result`。
- 新增需要数据库 overview 的 ByteDoc service flow 时，优先使用 `fetchDatabaseOverviewForQuery` 或等价的 backend-aware helper，让 DBW / Volc 路径绕过 classic detail API。
- 测试必须覆盖 Cloud Service Search 能解析数据库、但 legacy detail API 不支持该数据库的路径（典型 Volc 场景）。
