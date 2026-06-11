---
name: bytedance-ai-dev-pro
description: "获取 ai-dev-pro.bytedance.net 平台提供的 afs (Agent File System) 知识库查询能力，可获取代码/接口/PSM 知识、调用图等研发流程中的知识，覆盖 生活服务、电商、广告 业务域。"
---

# bytedcli AI Dev Pro AFS

## 如何调用 bytedcli

先选择一种调用方式。下面所有示例默认直接写 `bytedcli`。

```bash
# 方式 1：直接用 npx 运行最新版
NPM_CONFIG_REGISTRY=http://bnpm.byted.org npx -y @bytedance-dev/bytedcli@latest <command> [options]

# 方式 2：先全局安装，再直接调用 bytedcli
NPM_CONFIG_REGISTRY=http://bnpm.byted.org npm install -g @bytedance-dev/bytedcli@latest
bytedcli <command> [options]
```

- 使用 `npx` 时，把后文示例里的 `bytedcli` 替换成 `NPM_CONFIG_REGISTRY=http://bnpm.byted.org npx -y @bytedance-dev/bytedcli@latest`
- 已全局安装时，直接按后文示例执行 `bytedcli ...`

## 适用场景

- 查询代码原文类知识：语义检索代码、正则检索代码、读取代码实体、读取文件内容、查询文件内方法列表、查询 MR 变更内容。
- 查询服务和接口知识：PSM 检索、接口检索、接口出入参、接口上游调用方、接口下游调用、IDL、README、AGENTS.md、服务摘要、TCC/Dolphin 配置列表、DB 依赖。
- 查询关系类知识：代码调用图、接口上下游关系、DB 表反向调用方、FE wiki。

## 核心规则

- 所有 AFS 查询都通过 `bytedcli ai-dev-pro afs ...` 执行；具体命令和参数以对应命令的 `--help` 为准。
- 命令、必填参数、option 名称、枚举值和示例都以对应命令的 `--help` 为事实源。
- 不要自行添加 help 中不存在的参数，也不要根据历史协议或记忆拼接参数。
- 不要猜数字枚举值。部分 AFS option 会透传服务端定义的数字枚举，必须读取叶子命令的 `--help`，按 help 中的映射传参。
- 不要在最终回答、日志、示例或生成文件中打印、保存或回显服务账号密钥和已签发的 JWT。

## 鉴权

AFS 服务端当前要求服务账号 JWT。调用 `bytedcli ai-dev-pro afs ...` 前，先确认当前 shell 中已经设置环境变量 `BYTEDCLI_SERVICE_ACCOUNT_JWT`。

使用 IAM 平台申请到的服务账号密钥，向 ByteCloud JWT 接口签发临时服务账号 JWT：

```bash
curl -I -X GET 'https://cloud.bytedance.net/auth/api/v1/jwt' \
  -H 'Authorization: Bearer <service-account-secret>'
```

从响应头中读取签发后的 JWT，然后写入当前 shell 的临时环境变量：

```bash
export BYTEDCLI_SERVICE_ACCOUNT_JWT='<service-account-jwt>'
```

JWT 有过期时间。如果 AFS 命令返回 `miss token`、token 过期或其他服务账号 JWT 鉴权失败信息，需要重新签发 JWT，并更新环境变量 `BYTEDCLI_SERVICE_ACCOUNT_JWT` 后再重试。 如仍遇到“服务账号未在 AFS 配置中”等其他鉴权错误时，去 ../../troubleshooting.md 按步骤排查。

## 命令发现

先用父级 help 选择命令组，再用叶子命令 help 查看准确的参数：

```bash
bytedcli ai-dev-pro afs --help
bytedcli ai-dev-pro afs code --help
bytedcli ai-dev-pro afs interface --help
bytedcli ai-dev-pro afs psm --help
bytedcli ai-dev-pro afs callgraph --help
```

生成具体命令前，优先查看叶子命令 help：

```bash
bytedcli ai-dev-pro afs code search --help
bytedcli ai-dev-pro afs interface callee list --help
bytedcli ai-dev-pro afs psm agent-md get --help
```

## 命令选择

| 需求                                            | 优先查看                                                                                                                                                                                           |
| ----------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 根据自然语言检索可能相关的代码                  | `bytedcli ai-dev-pro afs code search --help`                                                                                                                                                       |
| 在单个 PSM 内按正则/模式搜索代码                | `bytedcli ai-dev-pro afs code glob --help`                                                                                                                                                         |
| 根据代码实体 ID、PSM + 方法名等方式读取代码实体 | `bytedcli ai-dev-pro afs code get --help`                                                                                                                                                          |
| 查询 PSM/repository 包路径下的文件列表          | `bytedcli ai-dev-pro afs code file list --help`                                                                                                                                                    |
| 读取指定文件或文件行号范围                      | `bytedcli ai-dev-pro afs code file get --help`                                                                                                                                                     |
| 查询文件中包含的方法列表                        | `bytedcli ai-dev-pro afs code file method list --help`                                                                                                                                             |
| 查询 MR 代码变更内容                            | `bytedcli ai-dev-pro afs code mr get --help`                                                                                                                                                       |
| 根据自然语言检索可能相关的接口                  | `bytedcli ai-dev-pro afs interface search --help`                                                                                                                                                  |
| 根据 PSM + 方法名或 method ID 查询接口详情      | `bytedcli ai-dev-pro afs interface list --help`                                                                                                                                                    |
| 查询接口出入参                                  | `bytedcli ai-dev-pro afs interface param get --help`                                                                                                                                               |
| 查询接口下游调用                                | `bytedcli ai-dev-pro afs interface callee list --help`                                                                                                                                             |
| 查询接口上游调用方                              | `bytedcli ai-dev-pro afs interface caller list --help`                                                                                                                                             |
| 查询接口使用的 DB 表                            | `bytedcli ai-dev-pro afs interface db list --help`                                                                                                                                                 |
| 根据 query 发现 PSM                             | `bytedcli ai-dev-pro afs psm search --help`                                                                                                                                                        |
| 查询 PSM 的 IDL、摘要、README 或 AGENTS.md      | `bytedcli ai-dev-pro afs psm idl get --help`、`bytedcli ai-dev-pro afs psm summary get --help`、`bytedcli ai-dev-pro afs psm readme get --help`、`bytedcli ai-dev-pro afs psm agent-md get --help` |
| 查询 PSM 依赖的 TCC、Dolphin 或 DB              | `bytedcli ai-dev-pro afs psm tcc list --help`、`bytedcli ai-dev-pro afs psm dolphin list --help`、`bytedcli ai-dev-pro afs psm db list --help`                                                     |
| 查询代码调用图                                  | `bytedcli ai-dev-pro afs callgraph code-entity get --help` 或 `bytedcli ai-dev-pro afs callgraph method get --help`                                                                                |
| 反查 DB 表被哪些服务/接口调用                   | `bytedcli ai-dev-pro afs db caller list --help`                                                                                                                                                    |
| 根据前端仓库和路径查询 FE wiki                  | `bytedcli ai-dev-pro afs fe-wiki get --help`                                                                                                                                                       |

## 查询策略

- 面对模糊的产品或实现问题，先用 `psm search`、`interface search`、`code search` 等发现类命令收集候选 PSM、方法名或 method ID。
- 已知 PSM 和方法名时，优先使用 `interface list`、`interface param get`、`code get`、`psm idl get`、`psm readme get` 等精确读取命令。
- 已知 method ID 或代码实体 ID 时，直接使用 ID 查询类命令，不要重复做语义检索。
- 需要查看实现细节时，先定位代码实体或文件，再使用 `code get` 或 `code file get` 读取源码内容。
- 需要分析影响面或依赖关系时，先定位服务、接口或代码实体，再使用 `interface caller list`、`interface callee list`、`interface db list`、`psm db list`、`db caller list` 或 `callgraph ... get`。
- 如果结果为空，先扩大搜索 query、先检索 PSM/接口候选，或查看叶子命令 help 中是否存在其他查询组合；不要直接下结论说数据不存在。

## 输出模式

AFS 服务端结果通常已经是 JSON-like 数据。日常排查和人工阅读可以直接使用默认文本输出。

当结果需要被脚本、Agent workflow 或后续结构化步骤继续解析时，再使用 `--json`：

```bash
bytedcli --json ai-dev-pro afs code search --q "create order" --psm example.service --limit 3
```

不要因为没有使用 `--json` 就认为结果不可解析；是否使用 `--json` 取决于外层流程是否需要 bytedcli 的 JSON envelope。

## 服务端定义的数字枚举

AFS 部分 option 会直接透传服务端定义的数字枚举。这是 `ai-dev-pro afs` 这个薄包装 domain 的例外设计。

使用 `--call-types`、`--entity-types`、`--entry-types`、`--source`、`--type`、`--wiki-type`、`--return-strategy` 等数字枚举参数时，必须读取叶子命令 help，并传入 help 中列出的数字值。不要凭记忆复制枚举映射。

不要把这种数字枚举透传风格扩散到其他 bytedcli domain；大多数 domain 应该暴露语义值，并在内部映射到后端数字编码。

## 示例

```bash
bytedcli ai-dev-pro afs code search --q "create order" --psm example.service --limit 3
bytedcli ai-dev-pro afs code glob --psm example.service --pattern "func" --entity-types 1,2 --entry-types 1,3
bytedcli ai-dev-pro afs interface callee list --psm example.service --method-name CreateOrder --call-types 0,1,2
bytedcli ai-dev-pro afs psm summary get --psm example.service --type 2
bytedcli ai-dev-pro afs psm agent-md get --psm example.service --source 1,2
bytedcli ai-dev-pro afs callgraph code-entity get --psm example.service --method-id 2438590 --up-depth 2 --down-depth 2 --return-strategy 3
bytedcli ai-dev-pro afs fe-wiki get --repo example/frontend-repo --name /layout --app-path apps/demo --wiki-type 1
```
