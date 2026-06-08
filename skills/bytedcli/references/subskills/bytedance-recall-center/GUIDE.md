---
name: bytedance-recall-center
description: "Operate Recall Center via bytedcli recall-center. Use for recallcli migration workflows, Recall Center authentication issues, recall version create/copy/list/get, config tree/get/create/update/delete, compile/compile-sync/log/result/status, debug mock/drainage/sandbox, publish, resource quota, traffic pool, and alert receiver tasks."
---

# bytedcli Recall Center

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
- 需要机器可读结果时使用 `--json`，并把它放在 `recall-center` 前面。

## When to use

- Recall Center 链路/版本查询、创建或复制。
- Recall Center 配置树、配置文件读取、新建、更新或删除。
- DSL 编译、同步编译、编译日志、编译产物、workflow 状态查询。
- sandbox debug env、mock debug、drainage debug、drainage 状态查询。
- 发布前置检查、发布提交、资源池/quota、流量池、告警接收人。
- 用户提到 `recallcli` 能力迁移、Recall Center OpenAPI、CN/BOE 调 i18n Recall Center 控制面，或需要复刻 recallcli 的端到端工作流。

## Core rules

- 统一用 `bytedcli --json recall-center --region <region> ...`；`--region` 是 Recall Center 控制面 region，常见值是 `cn`、`i18n`、`us`、`eu`、`nontt`、`boe`。
- region 选择优先级：用户显式指定 > 当前任务上下文 > `BYTERECLI_REGION` > 追问用户。CN/BOE 机器要调用 i18n 控制面时，显式传 `--region i18n`。
- 认证使用 bytedcli 现有 ByteCloud Auth。未登录或 token 过期时，运行 `bytedcli --site <site> auth login`，不要引入 Recall Center 专用登录流程。
- `recall-center` 命令只走 `/openapi/...`。平台 `/api/...` 是 SSO session 面，不支持 JWT 鉴权，不要把 OpenAPI 失败 fallback 到 `/api`。
- 业务命令在 `/openapi` 返回 `AUTH_REQUIRED`、HTML 登录页或 invalid JSON 时，先运行对应 site 的 `bytedcli --site <site> auth login`。如果响应里出现 `not login for /api/...`，说明打到了不支持 JWT 的 session API 面，要改回 OpenAPI 或反馈后端未开放 OpenAPI。
- 不要编造 recall id、config id、workflow id、traffic id、product、team、namespace、regions、quota 等关键参数。
- 写操作前确认用户意图：`recall create/copy`、`config create/update/delete`、`publish submit`、`resource scale`、`traffic-pool create`、`alert set-receiver` 都会改变服务端状态。
- 对写操作做闭环校验：`config update` 后再 `config get`；`config delete` 后再 `config tree`；drainage 提交后查 `debug drainage-status`；发布提交后返回可追踪的 workflow/BPM/工单标识。
- 做 DSL smoke 或 drainage 测试时，默认复用已有 specific traffic pool。先 `traffic-pool list --product <product> --status available --page 1 --page-size 20`，用户说“默认第一个/随便选一个流量池”就选第一页第一条可用池。不要主动创建流量池；只有列表为空或用户明确要求创建时才走 `traffic-pool create`。
- 本 guide 只沉淀稳定工作流规则。运行中的 id、当前发布包/本地 checkout 差异、一次性路径、临时 auth 状态等易过期事实，写到最终报告或记忆，不写进 skill。

## Authentication

```bash
# i18n Recall Center 常用认证
bytedcli --site i18n auth login
```

Recall Center OpenAPI 复用 bytedcli 现有 ByteCloud Auth。

## Lifecycle workflow

1. 认证失败时运行 `bytedcli --site <site> auth login`。
2. 定位版本：`recall get` 或 `recall list`；新建用 `recall create`，复制旧版本用 `recall copy`。
3. 配置编辑：先 `config tree` 找文件，再 `config get`、`config create`、`config update` 或 `config delete`。更新/删除后必须回读确认。
4. 编译：默认 `workflow compile-sync`。失败时继续 `workflow compile-log`，不要只返回“编译失败”。只有明确需要异步任务标识时才用 `workflow compile` + `workflow status`。
5. 调试：需要 sandbox 时先 `debug env`；mock 用 `debug mock`；引流用 `debug drainage`，然后 `debug drainage-status`。
6. 发布：先 `workflow compile-sync` 确保最新配置可编译，再 `publish check`。如果涉及资源池/quota，先查 `resource list-group` 和 `resource quota`，让用户确认资源池、quota、description 与告警接收人后再 `publish submit`。
7. 维护：资源扩容用 `resource scale-check` 预检查，再确认执行 `resource scale`；流量池用 `traffic-pool`；告警接收人用 `alert get-receiver` / `alert set-receiver`。

## DSL smoke workflow

用户要求“生成 DSL、上传平台、保证编译和引流通过”时，用这个固定顺序闭环：

1. 全流程显式固定站点和控制面 region，例如 i18n 用 `bytedcli --site i18n --json recall-center --region i18n ...`。认证失败时运行 `bytedcli --site i18n auth login`。
2. 创建或定位 recall version 后先 `config tree`。中断恢复时，先 `recall list --mine` 或按关键词查最近草稿；如果 name、owner、product 和配置内容都匹配任务，复用最新草稿继续闭环，不重复创建。新建版本通常已有默认配置文件：`serving/desc.py`、`root_dir/main.py`、`training/fermat.json`、`training/filter.lua`。不要对根路径执行 `config create --path main.py`，这类路径可能被后端当成非法目录；要按 tree 返回的 `conf_id` 用 `config update`。
3. DSL 主逻辑放 `serving/desc.py`，`root_dir/main.py` 保持平台入口包装，通常调用 `DSLKernel(script_path="./serving/desc.py")`。更新后分别 `config get` 回读确认。
4. 正式编译用 `workflow compile-sync --recall <id>`，不要只跑 `--dsl-file` dry-run。失败时继续拉 `workflow compile-log`；成功后按需查 `workflow compile-result`。
5. 引流前先 `debug env --recall <id>`，再用 `workflow status --type debug_deploy --only-brief` 轮询到 `wf_status=succ` 且 `sandbox_status=running`。
6. 流量池默认复用现有池：先 `traffic-pool list --product <product> --status available --page 1 --page-size 20`，选第一条可用 specific pool 的 `traffic_id`。只有没有可用池或用户明确要求创建时，才 `list-parent`、`create`、`wait`。
7. `debug drainage` 的输入至少给 `{"runtime_params":{}}`；`traffic_id`、`drainage_count` 由 CLI/后端参数补齐。提交后用 `debug drainage-status` 查到 completed、`status=SUCCESS`、`failure_percent=0`、success/completed/test case 计数一致；需要逐 case 再查 `debug drainage-case-status`。

## Evidence checklist

完成 DSL smoke 时，最终报告至少覆盖：

- Recall：version id、recall id、name、version、product、team/business line、namespace、regions。
- Config：`serving/desc.py` conf id、`root_dir/main.py` conf id，以及更新后的 `config get` 回读确认。
- Compile：workflow id、`status=SUCCESS`、`diagnostics.errors=[]`；如有 warnings，补 `workflow compile-log` 中的 `COMPILED SUCCESSFUL` 或 `VALIDATION PASSED` 作为判定证据。
- Debug：debug workflow id、`wf_status=succ`、`sandbox_status=running`。
- Drainage：traffic id/name、suite id、target/debug region、`status=SUCCESS`、success/completed/test case 数、failure percent。

## Quick start

```bash
# 列出 recall 版本
bytedcli --json recall-center --region i18n recall list --page 1 --page-size 20

# 按 id 或 name+version 查版本
bytedcli --json recall-center --region i18n recall get --id 12345
bytedcli --json recall-center --region i18n recall get --name my_recall --version 3

# 新建 / 复制 recall 版本
bytedcli --json recall-center --region i18n recall create \
  --name my_recall --product tiktok --team rec --namespace my_namespace
bytedcli --json recall-center --region i18n recall copy 12345 --name copied_recall

# 配置树、读取、创建、更新、删除
bytedcli --json recall-center --region i18n config tree --id 12345
bytedcli --json recall-center --region i18n config get --id 678
bytedcli --json recall-center --region i18n config create --id 12345 --path training/fermi.py --file ./fermi.py
bytedcli --json recall-center --region i18n config update --recall 12345 --id 678 --file ./fermi.py
bytedcli --json recall-center --region i18n config delete --recall 12345 --id 678

# 同步编译；本地 DSL 文件 dry-run；失败后查日志
bytedcli --json recall-center --region i18n workflow compile-sync --recall 12345
bytedcli --json recall-center --region i18n workflow compile-sync --recall 12345 --dsl-file ./fermi.py
bytedcli --json recall-center --region i18n workflow compile-log --recall 12345 --workflow <workflow_id>

# 异步编译与产物查询
bytedcli --json recall-center --region i18n workflow compile --recall 12345
bytedcli --json recall-center --region i18n workflow status --recall 12345 --workflow <workflow_id> --type compile
bytedcli --json recall-center --region i18n workflow compile-result --workflow <workflow_id>
```

## Debug and drainage

```bash
# 启动 sandbox debug environment
bytedcli --json recall-center --region i18n debug env --recall 12345

# mock debug：input 可直接传 JSON，也可从文件读取
bytedcli --json recall-center --region i18n debug mock \
  --recall 12345 \
  --target debug \
  --debug-region sg1 \
  --input-file ./input.json

# drainage debug：需要 traffic pool id
bytedcli --json recall-center --region i18n debug drainage \
  --recall 12345 \
  --traffic-id 98765 \
  --target debug \
  --debug-region sg1 \
  --input-file ./input.json

# drainage 状态
bytedcli --json recall-center --region i18n debug drainage-status --recall 12345 --suite-id <suite_id>
bytedcli --json recall-center --region i18n debug drainage-case-status --recall 12345 --suite-id <suite_id> --case-id <case_id>

# 辅助查询
bytedcli --json recall-center --region i18n debug ab-template --recall 12345 --target debug
bytedcli --json recall-center --region i18n debug list-regions --recall 12345 --target debug
```

Debug 失败时先分辨是 CLI 参数错误、OpenAPI 未开放、鉴权错误，还是后端环境错误。比如 `hosts is empty`、`service discovery failed` 一类错误通常说明 sandbox 还没 ready、site/region 不对，或后端服务发现不可用，不是 CLI 参数拼错；先确认全程显式 `--site i18n --region i18n`、必要时运行 `bytedcli --site i18n auth login`，再看 `debug_deploy` workflow 和 `sandbox_status`。`debug ab-template`、`debug list-regions`、`debug drainage-case-status` 在部分 region 可能返回 HTML 登录页或 OpenAPI 未暴露错误；这时把原始 `/openapi` endpoint、HTTP/body 摘要一起反馈，不要尝试改调 `/api`。

## Publish and resources

```bash
# 发布前检查
bytedcli --json recall-center --region i18n publish check --recall 12345

# 查询资源池与已申请 quota
bytedcli --json recall-center --region i18n resource list-group \
  --namespace my_namespace --product tiktok --team rec --resource-type viking --order-by update_time --desc
bytedcli --json recall-center --region i18n resource quota --recall 12345

# 扩容预检查与提交
bytedcli --json recall-center --region i18n resource scale-check --recall 12345 --quota-scale-infos '[...]'
bytedcli --json recall-center --region i18n resource scale --recall 12345 --quota-scale-infos '[...]' --description 'scale for test'

# 告警接收人
bytedcli --json recall-center --region i18n alert get-receiver --recall 12345
bytedcli --json recall-center --region i18n alert set-receiver --recall 12345 \
  --alert-need enabled --alert-staffs zhangsan,lisi --alert-chats <chat_id>

# 发布提交
bytedcli --json recall-center --region i18n publish submit --recall 12345
bytedcli --json recall-center --region i18n publish submit --recall 12345 \
  --submit-by-resource-group \
  --resource-group-id 1001 \
  --quota-need '[...]' \
  --description 'release with resource group'
```

发布工作流要先解释 `publish check` 结论；如果需要资源池或告警注入，先让用户确认资源池、quota、description、staffs/chats/oncall 参数。提交成功后优先返回 Recall Center 自身发布工单链接，再返回 BPM/workflow 等追踪信息。

## Traffic pool

做测试时默认不要创建流量池。先列 existing specific pools，用户没有指定名字时就选第一条可用池：

```bash
bytedcli --json recall-center --region i18n traffic-pool list --product tiktok --status available --page 1 --page-size 20
bytedcli --json recall-center --region i18n traffic-pool list-parent --product tiktok
bytedcli --json recall-center --region i18n traffic-pool get --traffic-id 12345
bytedcli --json recall-center --region i18n traffic-pool get --product tiktok --name my_pool
bytedcli --json recall-center --region i18n traffic-pool create \
  --name my_specific_pool \
  --traffic-pool-uri <parent_uri> \
  --sql 'select * from sample_table limit 100'
bytedcli --json recall-center --region i18n traffic-pool wait --traffic-id 12345 --timeout 120
```

只有在没有可用 specific pool，或用户明确说要创建新流量池时，才用 `list-parent` 选 parent，再 `create` + `wait`。

## Notes

- `workflow compile-sync --dsl-file` 是本地文件 dry-run，不会把 DSL 保存成线上配置，也不等价于正式编译记录。dry-run 通过后，如果用户目标是落库/发布，必须先 `config update`，再执行不带 `--dsl-file` 的正式编译。
- 新建 recall 后优先查看平台默认配置树并更新已有文件。`serving/desc.py` 是 DSL 主逻辑常用位置，`root_dir/main.py` 是入口包装；不要因为本地文件名叫 `main.py` 就在根目录新建 `main.py`。
- i18n 任务要全程显式 `--site i18n --region i18n`，避免拿到 BOE/CN token 后调 i18n 控制面导致后端服务发现或鉴权异常。
- `workflow compile-log` 需要 `--workflow` 或 `--record` 二选一；`workflow compile-result` 需要 `--workflow`，或 `--name` + `--version`。
- `debug mock` 和 `debug drainage` 的 `--target` 通常是 `online` 或 `debug`；debug 目标常配合 `--debug-region` 或 `--ip-port`。
- 对认证问题，优先确认 `--site` 与 `--region` 是否匹配；未登录或 token 过期时运行对应 site 的 `bytedcli --site <site> auth login`。不要把 `/api` 的 SSO session 登录当作 OpenAPI JWT 登录。
