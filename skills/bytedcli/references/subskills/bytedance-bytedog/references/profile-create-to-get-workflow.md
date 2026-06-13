# ByteDog Profile Create 到 Get 标准流程

本文用于 Agent 执行“创建新的 ByteDog profiling 任务，再拿结果文件”的标准流程。需要完整参数、输出字段或边界条件时，同时阅读 `profile-tool-command-reference.md`。

## 适用场景

用户需要新采集一次 profiling 数据时使用本流程，例如：

- CPU 忙、CPU 使用率高、需要看热点函数：`profile oncpu create`
- 需要机器维度长时间 CPU 画像或历史时间窗口：`profile sprofile create`
- RT/p99 高但 CPU 不高，怀疑 IO wait、sleep、futex、调度等待：`profile offcpu create`
- 怀疑 pthread mutex/rwlock 锁竞争或临界区争用：`profile pthread create`
- 需要快速查看 jemalloc allocator stats：`profile je-stats create`
- 怀疑 jemalloc 内存泄漏、RSS 持续增长、需要看内存分配调用栈：`profile je-flamegraph create`

## 标准步骤

1. 选择站点与 profile 类型。

   `--site` 是全局参数，写在 `bytedcli` 后、`bytedog` 前。若用户没有指定站点，按当前环境默认值执行；需要跨站点排查时让用户给出目标 site 或目标 detail URL。

2. 确认目标与 PID。

   `oncpu` 支持 `--ip`、`--pod`、`--workspace-id` 三选一；`sprofile` 只支持 `--ip`；`offcpu`、`pthread`、`je-stats`、`je-flamegraph` 支持 `--ip` 或 `--pod` 二选一且需要 `--pid`。

   需要 PID、进程命令、RSS 或 CPU 信息时先执行：

   ```bash
   bytedcli --site cn bytedog tool process list \
     --pod demo-pod
   ```

3. 创建 profiling 任务。

   `profile <type> create` 只提交异步任务，不等待结果文件生成。创建成功后记录输出里的 detail URL。

   ```bash
   bytedcli --site cn bytedog profile oncpu create \
     --pod demo-pod \
     --type go \
     --duration 30
   ```

   对需要 PID 的任务：

   ```bash
   bytedcli --site cn bytedog profile offcpu create \
     --pod demo-pod \
     --pid 12345 \
     --duration 30
   ```

   JSON 模式方便外层流程稳定提取 URL：

   ```bash
   bytedcli --json --site cn bytedog profile je-flamegraph create \
     --ip example-host \
     --pid 12345
   ```

4. 等待任务完成。

   创建命令返回 detail URL 后，等待采样时长加处理时间。`profile get` 只在任务状态为 `GOOD` 时获取结果文件；如果任务仍在运行，按提示稍后重试同一条 `profile get` 命令。

5. 用 detail URL 获取结果。

   ```bash
   bytedcli --site cn bytedog profile get \
     --url 'https://example.bytedog/profiling/on-cpu-profiling/detail?id=1001&from=tce' \
     --output-dir ./bytedog-output
   ```

   `profile get` 会在输出目录生成 `data-format.md` 和结果文件。拿到文件后先读 `data-format.md`，再解析 `.collapse`、`.json` 等结果文件。

6. 批量获取多个结果。

   多个 detail URL 用英文逗号分隔，并写入同一个输出目录：

   ```bash
   bytedcli --site cn bytedog profile get \
     --url 'https://example.bytedog/profiling/on-cpu-profiling/detail?id=1001&from=tce,https://example.bytedog/profiling/jemalloc-profiling/stats?id=1004&from=machine' \
     --output-dir ./bytedog-output
   ```

## Agent 输出要求

- 向用户汇报创建出的 detail URL、输出目录、`data-format.md` 路径和结果文件路径。
- 如果 `profile get` 提示任务未完成，说明任务仍在运行，并给出可直接重试的 `profile get --url ... --output-dir ...` 命令。
- 如果任务失败，保留错误信息和 detail URL，建议用户根据错误提示修正目标、PID、采样类型或 jemalloc 环境后重新创建。
