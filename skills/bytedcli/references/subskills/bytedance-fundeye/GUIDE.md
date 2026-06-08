---
name: bytedance-fundeye
description: "Use bytedcli for FundEye / Fullink / TCheck workflows: get rule details, list or create rules, inspect diffs and alarms, and resolve business ownership paths to IDs. Trigger this skill whenever the user mentions FundEye, Fullink, TCheck, reconciliation rules, business ownership, biz path, rule_id, diff_id, alarm_order_id, or asks to investigate discrepancies, alarms, or rule configuration in these systems."
---

# bytedcli FundEye

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

- 查询 FundEye / Fullink 核对规则详情
- 按产品类型分页查询规则列表
- 发布 TCheck 规则（publish）
- 按业务归属名称路径查询 `business_ownership` ID
- 查询某条 diff 的明细
- 按规则分页查询 diff 列表
- 标记/处理 diff（update）
- 查询告警列表
- 用户提到 `rule_id`、`diff_id`、`alarm_order_id`、`business_ownership`、业务归属路径、核对规则、规则列表、差异详情、差异列表、FundEye、Fullink、TCheck

## 能力范围

当前 skill 覆盖以下命令：

- 规则详情：`fundeye rule get`
- 规则列表：`fundeye rule list`
- 规则创建：`fundeye rule create`
- 规则发布：`fundeye rule deploy`
- 业务归属查询：`fundeye biz get`
- 差异详情：`fundeye diff get`
- 差异列表：`fundeye diff list`
- 差异处理：`fundeye diff update`
- 差异重试：`fundeye diff retry`
- 告警列表：`fundeye alarm list`

## 前置条件

- 使用通用调用方式：`../../invocation.md`
- 首次使用前先确保 `bytedcli auth login` 已完成
- FundEye 请求依赖当前登录态自动补 `x-jwt-token` 和 `UserName`
- 默认站点是 `cn`；如需海外机房可显式传 `--sitename sg`，火山云继续使用 `--sitename volc/火山云`
- `fundeye rule create` 目前仅支持 `cn`；只要显式传入非 `cn` 的 `--sitename`，CLI 会直接报不支持
- `fundeye biz get` 使用 `-` 连接层级路径；如果节点名有重名，必须传完整路径避免歧义
- `fundeye diff list` 需要提供 `--rule-id`；排查 `fullink` 告警或按 `--alarm-order-id` 过滤时，推荐显式传 `--start`、`--end`

> 执行前缀见 `../../invocation.md`；下面示例直接写 `bytedcli`。

## 工作流约定

1. 需要机器可读输出时默认加 `--json`，并把它放在 `fundeye` 前面。
2. 规则详情优先用 `fundeye rule get --rule-id <id>`。
3. 业务归属筛选优先走两步：先用 `fundeye biz get --path '<层级1-层级2>'` 拿到 `value`，再把该值传给 `fundeye rule list --business-ownership <value>`。
4. 规则列表优先用 `fundeye rule list --product-type fullink|tcheck`；默认查询 `fullink`，需要时再补 `--name`、`--owner`、`--status`、`--business-ownership`，`tcheck` 还支持 `--period`。
5. 新建规则优先用 `fundeye rule create --product-type fullink|tcheck --rule-owner <owner> --params '<json>'`；该能力目前仅支持 `cn` 机房，`--params` 只传业务参数对象，CLI 会自动包装成 `source + agent_task_list` 请求体；需要等最终落规则时再加 `--poll`。
6. 需要发布 `tcheck` 规则时，用 `fundeye rule deploy --product-type tcheck --rule-id <id>`；火山云暂不支持。
7. diff 明细优先用 `fundeye diff get --diff-id <id> --rule-id <id>`。
8. diff 列表必须提供 `--rule-id`，默认查询 `fullink`；查 `tcheck` 时加 `--product-type tcheck`，其中 `--rule-version` 可省略且默认按 `0` 请求；排查 `fullink` 告警、尤其按 `--alarm-order-id` 缩小时，优先补 `--start`、`--end` 时间窗，再视情况加 `--rule-version`。
9. 需要标记/处理 diff 时，用 `fundeye diff update --reason <parent||child> --diff-list '<json>'`；其中 `diff_id`/`diff_version` 从 `fundeye diff list --json` 返回的 `diffs[].diff_id`/`diffs[].diff_version` 获取；status 固定为 `PROCESS_RESULT`。
10. 需要重试 diff 时，用 `fundeye diff retry --try-list '<json>'`；其中 `try_list` 是 diff_id 的 JSON 数组字符串，可从 `fundeye diff list --json` 的 `diffs[].diff_id` 获取。
11. 如果服务端返回 500，优先保留 `request_id` 给后端排查；`fullink diff list` 还要先确认是否遗漏了时间窗。

## Quick start

```bash
# 规则详情
bytedcli --json fundeye rule get --rule-id 2604202570843580

# 规则列表
bytedcli --json fundeye rule list \
  --product-type fullink \
  --status RUNNING \
  --owner demo-owner \
  --page 1 \
  --page-size 10

# 先用业务归属路径换取 business_ownership ID
bytedcli --json fundeye biz get --path '财经-数据平台'

# 再按 business_ownership 过滤规则
bytedcli --json fundeye rule list \
  --product-type fullink \
  --business-ownership 7000000000000000000 \
  --page 1 \
  --page-size 10

# 使用 fullink 参数格式创建规则
bytedcli --json fundeye rule create \
  --product-type fullink \
  --rule-owner demo-owner \
  --params '{"owner":"demo-owner","rule_type":"double_check","data_sources":[{"vertex":"up","db_name":"sample_upstream_db","tb_name":"sample_upstream_table","filter_logic":"status == 98","is_trigger":true},{"vertex":"down","db_name":"sample_downstream_db","tb_name":"sample_downstream_table","filter_logic":"pay_status == \"SUCCESS\"","is_trigger":true}],"join":[{"from_vertex":"up","to_vertex":"down","join_info":"[{\"upstream\":\"order_id\",\"downstream\":\"out_order_no\"}]"}],"check_logic":"[up.total_amount] == [down.total_amount]"}' \
  --poll \
  --max-retries 30 \
  --interval 3

# 使用 tcheck 参数格式创建规则并轮询到 rule_link
bytedcli --json fundeye rule create \
  --product-type tcheck \
  --rule-owner demo-owner \
  --params '{"data_source_type":"krypton","check_tables":["sample_db.sample_table_a","sample_db.sample_table_b"],"user_check_requirement":"关联键: sample_key_a 和 sample_key_b; 核对规则: 筛选上游有记录但下游无匹配记录的异常数据; 输出字段: sample_field_a、sample_field_b"}' \
  --poll \
  --max-retries 30 \
  --interval 3

# 发布 tcheck 规则
bytedcli --json fundeye rule deploy \
  --product-type tcheck \
  --rule-id 20260601_1234567890000

# diff 明细
bytedcli --json fundeye diff get \
  --diff-id "DOUBLE_DS_CHECK#^#0#^#demo-diff" \
  --rule-id 2601142357560097

# diff 列表
bytedcli --json fundeye diff list \
  --rule-id 2601142357560097 \
  --product-type fullink \
  --rule-version 11 \
  --start "2026-04-21 00:00:00" \
  --end "2026-04-21 23:59:59" \
  --page 1 \
  --page-size 20

# diff 处理（update）
bytedcli --json fundeye diff update \
  --reason "BIZ_ISSUES||NEW_BIZ_DEPLOY" \
  --remark "demo remark" \
  --diff-list '[{"diff_id":"DOUBLE_DS_CHECK#^#0#^#demo","diff_version":1}]'

# diff 重试（retry）
bytedcli --json fundeye diff retry \
  --try-list '["DOUBLE_DS_CHECK#^#0#^#demo"]'

# 告警列表
bytedcli --json fundeye alarm list --page 1 --page-size 20
```

## 常见工作流

### 1. 查看规则

- 使用 `fundeye rule get --rule-id <id>`
- 优先关注 `baseInfo` 和 `graphData`

### 2. 按产品类型列规则

- 使用 `fundeye rule list --product-type fullink|tcheck`
- 默认查询 `fullink`
- `fullink` 支持 `--name`、`--owner`、重复 `--status`、`--business-ownership`
- `tcheck` 额外支持 `--period`

### 3. 创建规则

- CLI 的 `--params` 只传业务参数对象，不需要手动包 `source`、`agent_task_list`
- 规则创建目前仅支持 `cn`；显式传 `--sitename sg`、`--sitename volc` 或其他非 `cn` 机房都会直接报不支持
- `fullink` 的 `--params` 至少包含：`owner`、`rule_type`、`data_sources`、`join`、`check_logic`
- `fullink` 的 `join[].join_info` 需要传字符串，字符串内容通常仍是 JSON 数组
- `tcheck` 的 `--params` 至少包含：`data_source_type`、`check_tables`、`user_check_requirement`
- 需要自定义幂等单号时加 `--out-biz-no`；需要轮询最终 `rule_link` 时加 `--poll`

#### Agent Guidance: `data_source_type` 推断规则

`data_source_type` 合法值为 `"krypton"` 或 `"hive"`。若用户显式指定，以用户为准；未指定时，根据 `check_tables` 中的库名/表名自动推断，按以下优先级判断：

1. **Krypton DB 前缀白名单（最强信号，零误判）**：库名前缀命中以下任一，判定为 `"krypton"`：
   `noveldb`、`novel_op`、`novel_original`、`novel_bookdb`、`novelsale_distributordb`、`parallel_commerce`、`lvideo_compass`、`pgcincome`、`ocean_story`、`ocean_story_split`、`dpa_data`

2. **Hive 表名分层关键字（100% Hive）**：表名含以下任一分层前缀，判定为 `"hive"`：
   `ods_`、`ods_mysql_`、`dwd_`、`dim_`、`dm_`、`ads_`、`dwm_`、`dwa_`

3. **Hive 高频 DB 前缀**：库名前缀命中以下，判定为 `"hive"`：
   `webcast`、`open`、`aweme`、`pgc`、`ies_wallet`、`ad_star`、`dm_ttgame`、`dm_effect_platform`、`toutiao_dw`、`caijing_dw`

4. **无法判断时**：主动询问用户，不要猜测。提示："请确认数据源类型：krypton（业务在线库/MySQL 镜像）还是 hive（离线数仓）？"

### 4. 先查业务归属，再筛规则

- 业务归属名称路径已知时，先执行 `fundeye biz get --path '<层级1-层级2>'`
- 推荐传完整层级路径，例如 `财经-数据平台-会员`
- 成功后取返回中的 `value`
- 再执行 `fundeye rule list --business-ownership <value>`

### 5. 查看某条 diff

- 已知 `diff_id` 且知道所属规则时，用 `fundeye diff get`
- 如果只有告警单和规则信息，先用 `fundeye diff list` 缩小范围，再取具体 `diff_id`

### 6. 按规则排查 diff

- 使用 `fundeye diff list --rule-id <id>`
- 默认查询 `fullink`，查 `tcheck` 时加 `--product-type tcheck`
- `tcheck` 的 `--rule-version` 可省略；未传时默认按 `0` 请求上游
- 排查 `fullink` 时优先补 `--start`、`--end` 时间窗
- 需要进一步缩小范围时，加 `--rule-version` 或 `--alarm-order-id`
- 对结果中的 `diff_id` 再调用 `fundeye diff get`

### 7. 先看告警，再查 diff

- 先执行 `fundeye alarm list`
- 取返回里的 `alarmOrderId`、`ruleId`
- 再执行 `fundeye diff list --rule-id ... --alarm-order-id ... --start ... --end ...`

## Notes
- `--json` 是全局参数，必须放在 `fundeye` 前面，例如 `bytedcli --json fundeye rule get --rule-id ...`
- `fundeye rule create` 发送给服务端的请求体会自动包装成 `{"source":"platform_api","agent_task_list":[...]}`；CLI 侧 `--params` 只需要传单条任务里的业务参数对象
- `fundeye diff` 现在是分组命令；详情请用 `fundeye diff get`，列表请用 `fundeye diff list`
- `fundeye biz get` 返回的是 `{ path, value }`；其中 `value` 就是后续传给 `--business-ownership` 的 ID
- 业务归属树里可能有重名节点；只给单层标题时可能报歧义，必须传完整路径
- `fundeye diff list` 缺少必填参数时会返回结构化 help JSON
- `fundeye diff list --json` 的 `diffs[]` 字段名为 snake_case（例如 `diff_id`、`rule_id`），不再输出 camelCase 版本（例如 `diffId`、`ruleId`）
- `fullink diff list` 排查告警时，优先显式传 `--start`、`--end`，再观察是否仍返回 `HTTP 500`
- 如果 diff/list 仍返回 `HTTP 500`，优先记录 `request_id`

## References

- `references/fundeye.md`
- `../../invocation.md`
- `../../troubleshooting.md`
