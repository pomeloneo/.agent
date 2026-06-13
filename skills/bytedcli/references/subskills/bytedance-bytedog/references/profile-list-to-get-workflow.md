# ByteDog Profile List 到 Get 标准流程

本文用于 Agent 执行“从历史记录中找到 ByteDog profiling 任务，再拿结果文件”的标准流程。需要完整参数、输出字段或边界条件时，同时阅读 `profile-tool-command-reference.md`。

## 适用场景

用户已经创建过 profiling 任务，但没有提供 detail URL，或希望按目标信息搜索历史记录时使用本流程。

可查询的历史类型固定为：

- `oncpu`
- `sprofile`
- `offcpu`
- `pthread`
- `je-stats`
- `je-flamegraph`

## 标准步骤

1. 选择站点与历史类型。

   `--site` 是全局参数，写在 `bytedcli` 后、`bytedog` 前。历史类型要与用户要找的 profile 类型一致。

2. 选择过滤条件。

   普通历史命令至少提供一个过滤条件：`--ip`、`--pod`、`--psm`。`profile sprofile list` 只接受 `--ip`，不接受 `--pod` 或 `--psm`。

   `profile <type> list` 不做 IP 归一化，按传入文本过滤历史记录；如果用户给的是 hostname 或不确定的机器标识，优先让用户确认实际 IP 或直接使用能匹配历史记录的文本。

3. 查询历史记录。

   ```bash
   bytedcli --site cn bytedog profile oncpu list \
     --pod demo-pod \
     --psm demo.service \
     --page-size 10
   ```

   `sprofile` 示例：

   ```bash
   bytedcli --site cn bytedog profile sprofile list \
     --ip example-host \
     --page-size 10
   ```

   JSON 模式方便外层流程稳定读取 `detail_url`：

   ```bash
   bytedcli --json --site cn bytedog profile je-flamegraph list \
     --ip example-host \
     --page-size 10
   ```

4. 选择目标 detail URL。

   从列表结果中选择状态可用、时间最匹配、目标信息最匹配的记录，并记录 `detail_url`。如果列表结果太多，继续增加过滤条件或翻页；不要把当前页条数当作总数。

5. 用 detail URL 获取结果。

   ```bash
   bytedcli --site cn bytedog profile get \
     --url 'https://example.bytedog/profiling/on-cpu-profiling/detail?id=1001&from=tce' \
     --output-dir ./bytedog-output
   ```

   `profile get` 只在详情页任务状态为 `GOOD` 时获取结果文件。任务仍在运行时会提示稍后重试；任务失败时会输出任务错误信息和可用提示。

6. 批量获取多个历史结果。

   选择多条 `detail_url` 后，用英文逗号拼接：

   ```bash
   bytedcli --site cn bytedog profile get \
     --url 'https://example.bytedog/profiling/on-cpu-profiling/detail?id=1001&from=tce,https://example.bytedog/profiling/jemalloc-profiling/detail?id=1005&from=machine' \
     --output-dir ./bytedog-output
   ```

   获取完成后先读输出目录里的 `data-format.md`，再解析 `.collapse`、`.json` 等结果文件。

## Agent 输出要求

- 向用户汇报使用的过滤条件、选中的 detail URL、输出目录、`data-format.md` 路径和结果文件路径。
- 如果没有查到历史记录，说明实际使用的过滤条件，并建议改用更宽的 `--ip`、`--pod`、`--psm` 条件或确认 site。
- 如果查到多条候选记录，优先说明为什么选择目标记录；不能判断时列出候选 `detail_url` 和关键字段让用户选择。
