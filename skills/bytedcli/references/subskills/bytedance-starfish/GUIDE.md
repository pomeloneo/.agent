---
name: bytedance-starfish
description: "Query Starfish moderation traces, manage review task type catalog (list/max/save), create or update review task types, and inspect/create play events (玩法注册管理) on webcast.bytedance.net via bytedcli. Use when tasks mention Starfish, review trace, moderation流水, object_id, span timeline, trace search, webcast audit trace, task type, task_type list, review task type, QueryReviewTaskTypeList, SaveReviewTaskType, play event, 玩法注册, query_event, create_event, query_event_config_detail, or 新建审核任务类型/玩法."
---

# bytedcli Starfish

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

## When to use

- 查询某个 `object_id` 的 Starfish trace 流水
- 通过 `room_id`、`task_type`、时间范围收窄 trace 结果
- 让 agent 根据 trace span 时间线总结直播审核流水
- 查询 Starfish 审核任务类型（Review Task Type）目录，例如某个 `task_type` 数字对应什么名称、是否仍在生效
- 创建或更新 Starfish 审核任务类型（`SaveReviewTaskType`）
- 查询、检索或创建 Starfish「玩法注册管理」事件（play event），覆盖线上 / BOE 两套环境

## Do not use

- 不要把它当成通用的 `safe` 域 skill；Starfish 是独立顶层 domain
- 不要先手写 `webcast.bytedance.net` 请求；优先直接调用 `bytedcli starfish ...`
- 当前不要用它查询未接入的 Starfish 其他能力；本阶段支持 `trace`、`task-type list` / `max` / `save`、`play-event list` / `get` / `config` / `create` / `operators`

## 前置条件

先完成 Starfish 认证，三选一：

```bash
bytedcli starfish login
bytedcli starfish auth status
bytedcli starfish login --cookie "session=xxx"
export STARFISH_COOKIE="your_webcast_cookie"
```

> 执行前缀见 `../../invocation.md`；下面示例直接写 `bytedcli`。

## Quick start

```bash
bytedcli starfish login
bytedcli starfish auth status
bytedcli starfish trace --object-id hotsoon_live_audioslice@7634173330450402086_1777478421667
bytedcli starfish trace --object-id example_obj@1 --room-id 7634173330450402086 --task-type 203
bytedcli starfish trace --object-id example_obj@1 --start "2h ago" --end now
bytedcli --json starfish trace --object-id example_obj@1 --size 5
bytedcli starfish task-type list
bytedcli starfish task-type list --status 1 --page-limit 50
bytedcli starfish task-type list --task-type 203
bytedcli starfish task-type max
bytedcli starfish task-type save --task-type 99995 --name test_save --position test_save_99995
bytedcli --json starfish task-type list --offset 20 --page-limit 100
bytedcli starfish play-event list
bytedcli starfish play-event list --page 1 --page-size 50 --creator demo-user
bytedcli starfish play-event list --event demo_event --keyword demo
bytedcli starfish play-event get --event-id 7000000000000000000
bytedcli starfish play-event config --event-id 7000000000000000000
bytedcli starfish play-event list-operators
bytedcli starfish play-event create --name demo --event demo_event --desc 'demo desc' --callback-psm demo.demo.demo --yes
bytedcli starfish play-event list --env boe --cookie "boe_sid_starfish=...; sid_starfish=..."
```

## Workflow

1. 先执行 `bytedcli starfish login`，必要时用 `bytedcli starfish auth status` 检查当前 session 是否已经授权成功。
2. 用 `starfish trace --object-id <id>` 查询指定对象的流水。
3. 需要缩小结果范围时，追加 `--room-id`、`--task-type`、`--start`、`--end`。
4. 需要查询/翻阅审核任务类型目录时，使用 `starfish task-type list`，按需追加 `--status`、`--source`、`--task-type`、`--page-limit`、`--offset` 翻页过滤；查单条任务类型可直接 `--task-type 203`；想知道当前最大的 `task_type` 数字时使用 `starfish task-type max`（按需配合 `--status` / `--source`）。
5. 需要新建或更新审核任务类型时，使用 `starfish task-type save --task-type <n> --name <name> --position <position>`；`Operator/Priority/TaskCategory/Source/Status` 默认按线上常用值（`agent-op / 0 / 20 / 1 / 1`）填入，必要时再单独覆盖。
6. 需要查询玩法注册（play-event）时，使用 `starfish play-event list`，按需追加 `--page` / `--page-size`（默认 1/20）、`--event` / `--keyword` / `--creator` 过滤；单条详情用 `play-event get --event-id <id>`；玩法配置详情用 `play-event config --event-id <id>`（默认输出 pretty JSON，便于 `jq` 后处理）。创建玩法用 `starfish play-event create --name --event --desc --callback-psm`，默认会有 y/N 二次确认，`--yes` 跳过。
7. 通过 `--env online|boe` 切换线上 / BOE host：线上是 `webcast.bytedance.net`（`x-businessid=3 / x-businesskey=review`），BOE 是 `testboe-webcast.bytedance.net`（`x-businessid=11 / x-businesskey=test_001`）。BOE 还需要额外的 `boe_sid_starfish` cookie，目前仅通过 `--cookie` 或 `STARFISH_COOKIE` 透传，`starfish login` 不会自动颁发。
8. 需要 machine-readable 输出时，把 `--json` 放在 `starfish` 前面。
9. 让 agent 总结时，优先让它读取 span 时间线、标题、tag 和 show_fields 摘要，而不是直接消费长 `origin_value`。

## Natural language examples

- 查询 `hotsoon_live_audioslice@7634173330450402086_1777478421667` 的 Starfish 流水
- 用 Starfish 查这个 `object_id` 的审核 span 时间线，并总结关键节点
- 查 `object_id=example_obj@1` 最近 2 小时的 trace，按时间顺序说明发生了什么
- 列出当前 Starfish 所有处于 active 状态的审核任务类型
- 查 `task_type=203` 对应的审核任务名称是什么
- 当前 Starfish 在用的 task_type 最大值是多少？对应的任务类型名是什么
- 帮我新建一个 task_type=99995、name=test_save、position=test_save_99995 的审核任务类型
- 列出我创建的所有 Starfish 玩法（play event）
- 查 `event_id=7000000000000000000` 这个玩法的基础信息和配置详情
- 在 BOE 环境创建一个 event=demo_event、name=demo、callback_psm=demo.demo.demo 的玩法

## Notes

- 当前 Starfish CLI 已接入 `trace` 查询、`task-type list` / `max` / `save`、以及 `play-event list` / `get` / `config` / `create` / `operators`（玩法注册管理）能力，后续可以继续扩展更多子能力
- 认证优先级是 `--cookie` > `STARFISH_COOKIE` > `starfish login` 生成的 session > `starfish login --cookie` 保存的本地 auth
- `starfish auth status` 会同时展示当前 cookie 来源、关键 cookie 名称，以及 `portal_api/user_info` 是否已授权，可用来判断 `trace` 是否应当可用
- 文本模式会输出紧凑的 span 摘要表；`--json` 保留完整结构化结果
- `task-type list` 走 BFF (`/starfish/api/bff/webcast/review/config/QueryReviewTaskTypeList`)，与 `trace` 共用同一份 webcast cookie；`--status 1` 表示只列在用任务类型，`--status 2` 表示已废弃；`--task-type <n>` 作为后端过滤提示传入，但服务端实际过滤行为未做完整线上验证，必要时以 list 输出再做一次确认
- `task-type max` 内部基于 `QueryReviewTaskTypeList` 默认按 `TaskType` 升序的事实，先用 `pageLimit=1` 拿 `TotalCount`，再用 `offset=TotalCount-1, pageLimit=1` 取尾页一条，所以总共只发两次请求
- `task-type save` 走 BFF (`/starfish/api/bff/webcast/review/config/SaveReviewTaskType`)，是 **写操作**，会直接更新线上 Starfish 任务类型目录；`Operator` 默认值 `agent-op` 仅是占位，若后端做了账号合法性校验请通过 `--operator` 显式传入；其他固定值默认填 `Priority=0 / TaskCategory=20 / Source=1 / Status=1`，调用前先用 `task-type list --task-type <n>` 自检是否会覆盖已存在的任务类型
- `play-event` 走 `/starfish/api/review/general_manage/*`，与 `trace`、`task-type` 共用 webcast cookie，但走独立的 fixed headers；`--env online|boe` 决定 host 与 `x-businessid` / `x-businesskey`，BOE 还需额外提供 `boe_sid_starfish` cookie。`play-event create` 是 **写操作**，默认有 y/N 二次确认，`--yes` 跳过；`-j/--json` 模式下也会自动跳过确认（agent 场景视为无 TTY）
- `play-event list` 的 `total` 字段是后端过滤前的全量计数；空结果配合非零 `total` 通常意味着过滤条件命中 0 条，并非鉴权失败
- `play-event list-operators` 走 `/starfish/api/review/general_manage/query_all_operators`，返回算子目录（operator_id ↔ 中文名、param 字典）；`play-event config` 默认会并行拉一次 operator catalog，把 `operator_id` / `operator_param_id` 反查成可读的 `operator_name` / `param_name` / `param_desc` 后再渲染。catalog 拉取失败时会优雅降级为原始 ID 文本
- BFF 接口返回的 `Id` 可能是 19 位整数；CLI 在解析阶段会把 16+ 位的整数 token 自动转成字符串，确保 JSON / 文本输出里 `id` 字段保持完整精度

## References

- `references/starfish.md`
- `../../invocation.md`
- `../../troubleshooting.md`
