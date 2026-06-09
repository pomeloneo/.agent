---
name: bytedcli
description: "Unified skill for the bytedcli command surface. Use when tasks involve ByteDance internal R&D platforms and the agent should prefer bytedcli through CLI, MCP, or bundled references instead of opening web pages or hand-writing internal API calls. Covers auth/tokens; insearch internal knowledge and authenticated internal GET; Feishu/Lark, Jinshu, Cloud Docs/Ticket/Kani, Lark Oncall; FundEye/Fullink/TCheck; Codebase, BAM, BITS, Devflow, SCM, AGW, Janus/Janus Mini, Luban npm search and PyPI artifact publishing, Lynx, Overpass, Goofy; Fornax, Helix, Meego, AIME, Tika, Starling, TikTok Gecko, Holmes, Byterec; Live Trace; TCE/TCC/Spark Platform/ByteFlow/ENV/ByteCopy/TOS/FaaS/TAE/Volcano/ByteCloud/Bytetree/Netlink/Neptune/Settings/Kross; BMT, ABase, RDS, ByteHouse, ByteDoc, Merlin, Hive, Dorado, Blade, Oceanus, Aeolus, DataQ, TQS, Forge, ES, Cache, BMQ/RMQ/EventBus; Cronjob, Log, APM, Slardar, Codecov, Archer; DKMS, KMS v2, IAM; ByteStable WCC; TestIDE/SmartQ; MCP startup and update flows."
---

# bytedcli

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

这是 bytedcli 的统一入口说明。任务涉及字节内部研发平台时，先用它判断该走哪个命令域，再按需进入对应领域的详细说明。

## How to use

1. 先判断任务属于哪个平台或对象类型。
2. 先选对命令域，再调用对应的 bytedcli 命令。
3. 需要稳定机器可读输出时，默认加 `--json`，并把它放在 `<domain>` 前面。
4. 不确定命令名或参数时，先看 `--help`，不要猜。
5. 命令支持控制台 URL、文档 URL、仓库上下文自动推断时，优先直接使用这些输入。

## Quick start

```bash
bytedcli --help
bytedcli --json auth status
bytedcli <domain> --help
bytedcli self tracking enable
bytedcli self plugin install --repo demo-tools/plugin-demo
bytedcli self plugin list
bytedcli self plugin doctor --name demo-tools
```

## 生产网 prod

VA / Maliva / 生产开发机上，调 i18n-tt / i18n-bd / sg 命令前先 `export BYTEDCLI_NETWORK_PROFILE=prod`。办公网跳过。

## Route by task

- 登录、状态、JWT、用户信息：`auth`
  - 例子：“帮我登录”“看当前账号”“拿 codebase jwt”
- 仓库、MR、Issue、Review、CI、文件、跨仓搜索：`codebase`
  - 例子：“看这个 MR”“给这个 MR 回复评论”“查这个仓库的 CI”
- 研发任务管理（创建发布任务、查看任务详情与部署状态、搜索任务列表、查询项目与需求）：`devflow`
  - 例子：”创建一个发布任务””查这个 devflow 任务详情””搜我的发布任务””查这个空间有哪些项目””查项目下有哪些需求”
- ByteFlow 工作流引擎、应用/状态机/revision 查询、workflow JSON/ASL 校验与安全写操作辅助：`bytedance-byteflow`
  - 例子：“查 ByteFlow app/状态机”“校验这个 workflow JSON”“创建或更新 ByteFlow revision”
- Feishu / Lark 文档、评论、日历、待办、消息、表格、多维表格：`feishu`
  - 例子：“读这个 Lark 文档”“在文档里追加一段”“给某人发飞书消息”
- People 自助请假记录查询与半天假申请：`people`
  - 例子：“查我今天请了几天假”“帮我再请半天病假”“给今天下午补提半天假”
- Jinshu / 云锦书消息预览与发送：`jinshu`
  - 例子：“预览这段锦书消息”“发送这段锦书消息”
- Lark Oncall 工单、打标/归因元数据、oncall 群消息和值班信息：`lark-oncall` / `bytedance-lark-oncall`
  - 例子：“查某业务线过去 7 天的 oncall 工单”“拿某个 ticket 的 oncall 群聊天记录”“列 oncall tag/归因选项”
- VOC（抖音 CEM 用户反馈，cem.bytedance.net）feedback 详情查询：`voc` / `bytedance-voc`
  - 例子：“查这条 voc feedback 详情”“按 voc 控制台 URL 拉反馈”“按 voc 分享链接拉反馈列表”“看本地 voc 鉴权 cookie 是否还在”
- FundEye / Fullink 核对规则、diff、告警：`fundeye`
  - 例子：“查这个规则详情”“按 rule_id 看最近一天的 diff”“查某个 alarm_order_id 对应的 diff”
- Starling 文案平台、项目、空间、文案搜索：`starling`
  - 例子：“查这个 Starling 项目”“创建一个 Starling space”“搜索某个文案 key”
- Luban npm 包查询、PyPI 制品仓库/版本查询与发布：`luban`
  - 例子：“查某个 TTP-US npm 包在 Luban 里有没有”“看某个包的 2.1.x 版本”“查 Luban PyPI 仓库版本列表”“基于 master 分支发布 PyPI 版本，先 dry-run，再确认发布并轮询结果”
- Lynx 工具集合、LynxExample 产物元信息与下载：`lynx`
  - 例子：“查 LynxExample 产物”“下载 LynxExample Android 包”
- Mango / 芒果任务管理：`bytedance-mango`
  - 例子：“登录 Mango”“列出 Mango 空间”“查询 Mango 任务”
- Gecko CN 控制台只读资源查询（App、Project、Deployment、Channel、离线规则树与资源包）：`gecko`
  - 例子：“查 Gecko CN 某个 channel 的详情”“列某个 channel 下的离线资源包”“查看资源包处理日志”
- TikTok Gecko 控制台只读资源查询（工作台、App、Channel、Ticket、Host App、Deployment）：`tiktok-gecko`
  - 例子：“查 Gecko 某个 channel 的详情”“列出 deployment 下所有 channel”“筛选某个 creator 的 Gecko 工单”
- 技术文章、知识问答、AI 对话 / 附件 prompt：、`bitsai`、`tika`、`aime`
- AI Dev Pro AFS / Agent File System 代码、接口、PSM、调用图、DB caller、FE wiki 等研发流程内的知识查询：`ai-dev-pro afs`
  - 例子：“查这个 PSM 的接口参数”“搜某个代码实体”“查接口下游”“查 某个psm的AGENTS.md 知识”
- 内部搜索（飞书文档、ByteCloud 文档、内网、ByteTech 文章、BitsAI 问答）与内部 URL 只读 GET：`insearch`
  - 例子：“搜索 kitex ppe 环境”、“查飞书文档里有没有 BMQ 接入指南”、“用 BitsAI 问一下 TCC 怎么配置”、“用当前登录态 GET 这个内部接口”
  - 读取内部 HTTP(S) URL 用 `insearch get <url>`；无法结构化解析的允许 URL 会自动走只读 GET fallback
- SQL、数据库平台、数据资产、报表、离线任务：`rds`（子组：`db`/`slow`/`alert`/`ops`/`bpm`）、`bytedoc`、`hive`、`dorado`（子组：`project`/`task`/`instance`/`adhoc`）、`blade`（子组：`task`，当前支持 `task get` 和 `--region mycis`；鉴权优先复用 `blade.byteintl.net` 站点 cookie 与 fresh ByteCloud JWT）、`oceanus`（子组：`project`/`tree-node`/`node`/`task`）、`aeolus`、`dataq`、`tqs`、`forge`
  - 例子：”查我关注的 ByteDoc 数据库””看这个库的慢查询””执行一段 SQL””查这个 dashboard 对应的数据集”
  - 在 hive-sql 上进行语法检查、提交 sql 任务、获取状态和结果：`tqs`
- Eventbus event 详情查询：`eventbus`
  - 例子：“查这个 event 的详情”“确认 event 绑定了哪些 topic”“验证 eventbus OpenAPI 是否可用”
- BMT 多租户服务、标签、资源、隔离集与 user role 查询：`bmt`
  - 例子：“按 PSM 反查 BMT service”“看这个 PSM 绑定了哪些 RDS 或 RocketMQ 资源”“查某个 BMT resource code 对应的连接信息”
- 查询 Forge 任务日志：`forge logs`
- Chronos 调度平台：`chronos`
  - 例子：“分页列出命名空间”“按 namespace 看任务列表”“按 task_id 看 HTTP 调度链接和报警接收人”
- OneService query 元信息、版本详情、SQL 提取：`oneservice`
  - 例子：“查这个 OneService query 的元信息”“按 queryId 拿当前 ONLINE 版本的 SQL”“按 versionId 看 query detail”
- Holmes TBase 产品、字段、字段新增提审与 row-key 查询：`holmes tbase` / `bytedance-holmes-tbase`
  - 例子：“查这个 TBase product 的配置”“列某个产品的字段”“给 TBase 新增字段提审”“按 row key 查多字段或 all-fields”
- Byterec index service 产品信息、配置、模型配置列表、Viking DB 调试/资源查询，以及同组件在 Holmes 平台下的 proto / record 调试：`byterec indexservice` / `byterec model list` / `byterec viking` / `holmes indexservice` / `bytedance-byterec-indexservice`
  - 例子：“查这个 Byterec indexservice 的 product 信息”“按 PSM 看 Byterec config”“列 Byterec 某个 namespace 下的模型列表/版本”“判断模型是否绑定 Viking serving”“执行 Viking DB DSL recall”“查看 Viking DB database/model”“列 Holmes IndexService proto”“按显式参数读一条 IndexService record”
- Recall Center recall 版本、配置、编译、debug/mock/drainage、发布、资源池、流量池和告警接收人：`recall-center` / `recallcenter` / `bytedance-recall-center`
  - 例子：“查这个 recall 版本”“更新 Recall Center 配置并编译”“跑 mock debug / drainage debug”“发布前检查”“查资源池 quota”
- Merlin job 提交和从中抽取 YAML 描述，Merlin job run 列表与 job->trial 解析，Merlin trial diagnose/local-log，Merlin job/trial 的 stdout/stderr 日志查询，Merlin tracking project、run、metrics 和 job 链接读取，`merlin` 计算资源 quota 的 group、cluster 只读查询：`merlin`
  - 例子：“提取这个 Merlin job 的 YAML”“把这份 `trial.yaml` 重提到 `seed-cn`”“看这个 Merlin tracking run 的 config/summary”“根据 job id 找 tracking 链接”“拉这个 Merlin trial 的 stdout/stderr”“查这个 trial 为什么还在排队”
- Helix 模型与 AI 任务生命周期，当前覆盖 Video AIPF 数据准备、训练/评估提交、状态查询、停止和记录查询：`helix`
  - 例子：“提交一个 Video AIPF 数据准备任务”“提交一个 Video AIPF 训练任务”“查询 Helix 评估状态”“停止这个 Video AIPF Ray 评估任务”“查最近的 Helix 训练记录”
- Fornax prompt workspace、prompt 草稿与版本、prompt 发布，以及 experiment 创建与结果查询：`fornax`
  - 例子：“列出这个 Fornax workspace 下的 prompts”“读取 prompt 个人草稿”“发布这个 prompt 到 ppe”“查询 experiment 第一页结果”“配置 Fornax experiment 的 AK/SK 或 JWT”
- BES 元信息修改工单：`bes`
  - 例子：`bytedcli bes metadata update --config '{"title":"demo-ticket"}'`
- Kani 权限审批：`kani`
  - 例子：“查 Kani request 页工单”“看 reviewer 视角的已完结审批”
- 报销单查询与 AI 订阅报销草稿：`reimbursement`
  - 例子：“查最近报销单”“检测这个报销单”“用这张订阅收据创建 AI 学习研究报销草稿”“关闭/删除这个草稿”“确认后提交这个报销单”
- 配置中心、配置查询、新建、更新、发布与权限申请：`tcc`（子组：`namespace`/`config`/`deployment`/`env`/`site`/`permission`）、`bytestable`（子组：`wcc`、`qcss`）
  - 例子：”查 TCC namespace 下的配置””更新一个 TCC 配置并发布””申请某个 TCC namespace 的读写权限””在 WCC 里新建配置””更新 WCC 配置值””发起 WCC 配置工单””通过 QCSS 检查项”
- 部署、环境、ByteCopy、服务树、域名治理、对象存储、云函数资源与 MOSS 测试物料管理平台查询：`tce`、`env`（子组：`site`/`service`/`bytecopy`/`device`/`ticket`）、`bytetree`、`goofy`、`netlink`、`neptune`、`tos`、`faas`、`volcano`、`bytecloud`、`moss`
  - 例子：”查服务实例””看发布单””更新配置””做一个 Goofy preview”
- TAE / AI PaaS（MCP Server/Tool 优先用 `bytedcli tae mcp ...`；Agent、Sandbox、Memory、Skill 等未覆盖能力走内部 API 指南）：`bytedance-tae`
  - 例子：“在 TAE MCP Server 批量录入 RPC tools”“修复 MCP Input Schema”“把 HTTP tools 改成 RPC tools”“发布 MCP server revision”“调研 TAE Agent/Sandbox/Memory API”
- Janus / Janus Mini 泳道、group、配置、IDL、endpoint、backend 与发布 workflow：`janus` / `janus-mini` / `bytedance-janus` / `bytedance-janus-mini`
  - 例子：“创建 Janus 泳道”“查询 Janus Mini group service_type”“创建 endpoint/backend”“创建 publish workflow 并查询状态”
- Spark Platform 空间与链路资源：`spark-platform`
  - 例子：“列出 Spark space”“按业务线 bid 列 link”“某个 space 下的 link”“拿某个 link 的完整 raw（含所有 version 与原始 deployConfig）用 `link get`”“要最新已发布 version + 解析后的 schema（含 schemaUrl / bundle / bundlePath）用 `link summary`”“列某个 link 的 env 配置”“给 link 设置 PPE env，先 `--dry-run` 看 payload 再真实执行”“删除某个 env”“指定非默认 `--app-id`”
- Kross 多平台容器环境（workload）列表、创建、容器内远程执行与文件传输：`kross`
  - 例子：“先用 `kross workspace list` 看我有哪些 workspace，再用 `kross template list` 查这个 workspace 在当前 cluster 下可用模板，然后用 `kross workload list` 看这个 workspace 下已有 workload，最后直接创建 job workload；quick create 默认会带 1000m CPU / 2048 MB memory / 300 秒 timeout / 自动删除”“删除某个 workload”“通过 webshell 在 workload 容器里执行命令”“上传或下载 workload 容器文件”
- 日志、监控、告警、Dashboard、App 异常趋势、App/OS symbol、Redis / ABase / Kafka / RocketMQ / EventBus：`log`、`apm`（子组：`service`/`redis`）、`slardar`（子组：`web`/`app`/`os`）、`cache`、`abase`、`bmq`、`rmq`、`eventbus`
  - 例子：”查这个 logid””先看某个接口的总体瓶颈””按 logid 看链路各节点延迟””看 Redis 大 key””分析这个告警页””根据 Slardar dashboard URL 看看板配置或改标题””用 Slardar App issue URL retrace native 栈””用 Slardar OS issue URL 解析主线程 native 栈””搜索 RocketMQ topic””查看 RocketMQ consumer group 列表”
- Libra / DataTester A/B 实验、指标组、指标组模版：`libra`
  - 例子：“看这个实验详情”“查这个 flight 的报告”“根据 template 页面 URL 查看 metric-group template”
- Tesla RM 自动化测试平台：测试任务触发/查询/run 等待、列表、失败归因，测试计划 CRUD 与统计：`tesla`
  - 例子：“按计划 507863 触发一个 Tesla 任务并等结果”“查这个 Tesla 任务的状态和失败用例”“列出这个计划最近 7 天的任务”“看这个测试计划的执行统计”
- TestIDE / SmartQ 小 Q UI 自动化：读取用例集、自动化步骤、片段引用，创建/更新用例集和步骤，创建测试任务并查询进度/结果：`smartq`
  - 例子：“读取这个 TestIDE 用例集的自动化步骤和引用片段”“创建一个小 Q UI 测试任务”“查询 TestIDE 任务结果”
- OneService 查询：`oneservice`
  - 例子：“查这个 query 的 meta”“查这个 query version detail”“取当前 ONLINE 版本 SQL”
- Life 生活服务生财有数平台：`life live-screen`
  - 例子：“根据直播间 ID 看直播数据工作台核心指标”“按主播昵称 / 主播 ID / 抖音号 / 直播间 ID 获取用户信息”
- Live Trace / 消息链路排查：`live trace`
  - 例子：“发起 ack_trace”“查这个 task_id 的明细”“解析这段 pb_payload”
- ByteIO 埋点、需求、点位、测试用例与广告元数据查询：`bytedance-byteio`
  - 例子：“查这个 app_id 下某个 event_name 是否存在”“校验这个埋点参数”“查询 ByteIO 需求详情 / BTM 点位 / 测试用例 / 广告 tag”
- 跨区域数据交换：`decc`
  - 例子："创建 DECC channel""注册 DECC data""申请 channel 权限"
- 安全与权限：`dkms`、`kmsv2`、`iam`
  - 例子："查 data key 权限""给 key 加 ACL""搜一个员工"
- 混沌工程、故障注入、演练方案创建：`chaos`
  - 例子："创建一个演练方案，为 xxx 注入 cpu 受限 80% 的故障，持续时间 300 s，使用新建隔离环境的策略，选择最新的 rhino 流量"
- 将 bytedcli 暴露给宿主、升级本地安装或管理受信任 CLI 插件：`mcp`、`self`

## Common inputs

- 如果用户给的是 MR / issue / 文档 / 配置 / 告警控制台 URL，优先直接用 URL，不要先手拆 ID。
- 如果任务是 Meego，优先直接使用工作项 / 视图 URL；很多命令支持 `--url` 自动回填 `project_key`、`work_item_id`、`view_id` 等标识。
- 如果用户给的是仓库目录上下文，优先让 Codebase 自动从当前 `origin` 推断仓库；当前支持 `code.byted.org` 和 `code-tx.byted.org` remote。如果推断失败，CLI 会继续说明是非 Git 仓库、缺少 `origin`、host 不支持，还是 remote 无法解析。
- 如果任务跨站点，先确认 `--site` 或 `BYTEDCLI_CLOUD_SITE`。
- 如果命令失败，优先看 `error.hint`、`error.auth_command`，或参考排障说明。

## Domain guides

任务已经收敛到某个具体领域时，继续看对应领域说明：

- Codebase: [references/subskills/bytedance-codebase/GUIDE.md](references/subskills/bytedance-codebase/GUIDE.md)
- Feishu: [references/subskills/bytedance-feishu/GUIDE.md](references/subskills/bytedance-feishu/GUIDE.md)
- Jinshu: [references/subskills/bytedance-jinshu/GUIDE.md](references/subskills/bytedance-jinshu/GUIDE.md)
- Lark Oncall: [references/subskills/bytedance-lark-oncall/GUIDE.md](references/subskills/bytedance-lark-oncall/GUIDE.md)
- VOC: [references/subskills/bytedance-voc/GUIDE.md](references/subskills/bytedance-voc/GUIDE.md)
- FundEye: [references/subskills/bytedance-fundeye/GUIDE.md](references/subskills/bytedance-fundeye/GUIDE.md)
- Starling: [references/subskills/bytedance-starling/GUIDE.md](references/subskills/bytedance-starling/GUIDE.md)
- Luban: [references/subskills/bytedance-luban/GUIDE.md](references/subskills/bytedance-luban/GUIDE.md)
- Lynx: [references/subskills/bytedance-lynx/GUIDE.md](references/subskills/bytedance-lynx/GUIDE.md)
- TCC: [references/subskills/bytedance-tcc/GUIDE.md](references/subskills/bytedance-tcc/GUIDE.md)
- Chronos: [references/subskills/bytedance-chronos/GUIDE.md](references/subskills/bytedance-chronos/GUIDE.md)
- BES: [references/subskills/bytedance-bes/GUIDE.md](references/subskills/bytedance-bes/GUIDE.md)
- WCC / QCSS: [references/subskills/bytedance-bytestable-wcc/GUIDE.md](references/subskills/bytedance-bytestable-wcc/GUIDE.md)
- TCE: [references/subskills/bytedance-tce/GUIDE.md](references/subskills/bytedance-tce/GUIDE.md)
- ENV / ByteCopy: [references/subskills/bytedance-env/GUIDE.md](references/subskills/bytedance-env/GUIDE.md)
- ByteSD: [references/subskills/bytedance-sd/GUIDE.md](references/subskills/bytedance-sd/GUIDE.md)
- TQS: [references/subskills/bytedance-tqs/GUIDE.md](references/subskills/bytedance-tqs/GUIDE.md)
- Kross: [references/subskills/bytedance-kross/GUIDE.md](references/subskills/bytedance-kross/GUIDE.md)
- Bytetree: [references/subskills/bytedance-bytetree/GUIDE.md](references/subskills/bytedance-bytetree/GUIDE.md)
- BMT: [references/subskills/bytedance-bmt/GUIDE.md](references/subskills/bytedance-bmt/GUIDE.md)
- RDS: [references/subskills/bytedance-rds/GUIDE.md](references/subskills/bytedance-rds/GUIDE.md)
- ByteHouse: [references/subskills/bytedance-bytehouse/GUIDE.md](references/subskills/bytedance-bytehouse/GUIDE.md)
- ByteFlow: [references/subskills/bytedance-byteflow/GUIDE.md](references/subskills/bytedance-byteflow/GUIDE.md)
- Blade: [references/subskills/bytedance-blade/GUIDE.md](references/subskills/bytedance-blade/GUIDE.md)
- OneService: [references/subskills/bytedance-oneservice/GUIDE.md](references/subskills/bytedance-oneservice/GUIDE.md)
- Merlin: [references/subskills/bytedance-merlin/GUIDE.md](references/subskills/bytedance-merlin/GUIDE.md)
- MOSS: [references/subskills/bytedance-moss/GUIDE.md](references/subskills/bytedance-moss/GUIDE.md)
- Helix: [references/subskills/bytedance-helix/GUIDE.md](references/subskills/bytedance-helix/GUIDE.md)
- Holmes TBase: [references/subskills/bytedance-holmes-tbase/GUIDE.md](references/subskills/bytedance-holmes-tbase/GUIDE.md)
- Byterec Indexservice: [references/subskills/bytedance-byterec-indexservice/GUIDE.md](references/subskills/bytedance-byterec-indexservice/GUIDE.md)
- Recall Center: [references/subskills/bytedance-recall-center/GUIDE.md](references/subskills/bytedance-recall-center/GUIDE.md)
- AI Dev Pro AFS: [references/subskills/bytedance-ai-dev-pro/GUIDE.md](references/subskills/bytedance-ai-dev-pro/GUIDE.md)
- Fornax: [references/subskills/bytedance-fornax/GUIDE.md](references/subskills/bytedance-fornax/GUIDE.md)
- Meego: [references/subskills/bytedance-meego/GUIDE.md](references/subskills/bytedance-meego/GUIDE.md)
- Libra: [references/subskills/bytedance-libra/references/libra.md](references/subskills/bytedance-libra/references/libra.md)
- Tesla: [references/subskills/bytedance-tesla/GUIDE.md](references/subskills/bytedance-tesla/GUIDE.md)
- SmartQ / TestIDE: [references/subskills/bytedance-smartq/GUIDE.md](references/subskills/bytedance-smartq/GUIDE.md)
- FaaS: [references/subskills/bytedance-faas/GUIDE.md](references/subskills/bytedance-faas/GUIDE.md)
- TAE / AI PaaS: [references/subskills/bytedance-tae/GUIDE.md](references/subskills/bytedance-tae/GUIDE.md)
- Log: [references/subskills/bytedance-log/GUIDE.md](references/subskills/bytedance-log/GUIDE.md)
- Archer: [references/subskills/bytedance-archer/GUIDE.md](references/subskills/bytedance-archer/GUIDE.md)
- Slardar: [references/subskills/bytedance-slardar/GUIDE.md](references/subskills/bytedance-slardar/GUIDE.md)
- Devflow: [references/subskills/bytedance-devflow/GUIDE.md](references/subskills/bytedance-devflow/GUIDE.md)
- Safe: [references/subskills/bytedance-safe/GUIDE.md](references/subskills/bytedance-safe/GUIDE.md)
- Life: [references/subskills/bytedance-data-life-live/GUIDE.md](references/subskills/bytedance-data-life-live/GUIDE.md)
- Live Trace: [references/subskills/bytedance-live/GUIDE.md](references/subskills/bytedance-live/GUIDE.md)
- ByteIO: [references/subskills/bytedance-byteio/GUIDE.md](references/subskills/bytedance-byteio/GUIDE.md)
- Janus: [references/subskills/bytedance-janus/GUIDE.md](references/subskills/bytedance-janus/GUIDE.md)
- Janus Mini: [references/subskills/bytedance-janus-mini/GUIDE.md](references/subskills/bytedance-janus-mini/GUIDE.md)
- Search: [references/subskills/bytedance-insearch/GUIDE.md](references/subskills/bytedance-insearch/GUIDE.md)
- People: [references/subskills/bytedance-people/GUIDE.md](references/subskills/bytedance-people/GUIDE.md)
- Reimbursement: [references/subskills/bytedance-reimbursement/GUIDE.md](references/subskills/bytedance-reimbursement/GUIDE.md)
- 其他领域路径索引: [references/subskills-index.md](references/subskills-index.md)

## References

- `references/command-surface.md`
- `references/invocation.md`
- `references/troubleshooting.md`
