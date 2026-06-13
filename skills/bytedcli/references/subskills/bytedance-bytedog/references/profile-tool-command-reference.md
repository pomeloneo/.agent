# `bytedcli bytedog` 命令接口说明

本文说明当前 `bytedcli bytedog` 下可执行命令的调用方式、参数、输出、限制与示例。示例中的参数均为占位值

## 通用参数与用法

下方参数适用于 `bytedcli bytedog` 末级命令。后续每个命令的参数表只列业务参数，不再重复列出这些通用参数。

### bytedcli 全局参数
文档：`references/invocation.md`

| 参数             | 默认值                        | 取值                                                  | 用法                                                                                 |
| ---------------- | ----------------------------- | ----------------------------------------------------- | ------------------------------------------------------------------------------------ |
| `--site <site>`  | `BYTEDCLI_CLOUD_SITE` 或 `cn` | `cn`、`boe`、`i18n-bd`、`i18n-tt`、`us-ttp`、`eu-ttp` | 选择 ByteDog 鉴权、任务、历史记录、工具接口和详情页 URL 站点。推荐写在 `bytedcli` 后、`bytedog` 前。 |
| `--json`         | `false`                       | 布尔开关                                              | 输出 bytedcli 标准 JSON envelope；业务对象放在 `data` 字段里，并附 `status` / `error` / `context`。 |

所有命令默认输出文本。需要机器可读输出时，在根命令使用全局 `--json`，例如：

```bash
bytedcli --json --site cn bytedog profile get \
  --url 'https://example.bytedog/profiling/on-cpu-profiling/detail?id=1001&from=tce' \
  --output-dir ./bytedog-output
```

### ByteDog 公共上下文参数

| 参数                | 默认值 | 用法                                                                 |
| ------------------- | ------ | -------------------------------------------------------------------- |
| `--question <text>` | 无     | 当前用户希望解决什么问题。提供该信息可以帮助当前工具更好地完成任务。 |
| `--reason <text>`   | 无     | 为何会选择使用当前工具。提供 WHY 上下文，帮助工具理解整体意图。      |

`--question` 与 `--reason` 支持 `bytedcli bytedog profile get`、`bytedcli bytedog profile <type> create/list` 与 `bytedcli bytedog tool process list`。

```bash
bytedcli --site cn bytedog profile oncpu create --pod demo-pod \
  --question "Why is CPU usage spiking after the latest deploy?" \
  --reason "Investigating an alert from APM about p99 latency"
```

```bash
bytedcli --json --site cn bytedog tool process list --pod demo-pod \
  --question "Find the candidate PID for jemalloc profiling" \
  --reason "Preparing a je-flamegraph run"
```

## Agent 快速选择

| 目标                                       | 推荐命令                                                                                                  | 关键输出                                  | 后续动作                                                                  |
| ------------------------------------------ | --------------------------------------------------------------------------------------------------------- | ----------------------------------------- | ------------------------------------------------------------------------- |
| 已有 ByteDog 详情页 URL，获取结果文件      | `bytedcli bytedog profile get --url <url>`                                                               | `data-format.md`、结果文件路径、任务目标  | 先读 `data-format.md`，再解析 `.collapse` / `.json` 结果文件              |
| 需要先找 PID 或确认进程命令                | `bytedcli bytedog tool process list`                                                                   | `PID`、`TID_NS`、`RSS_KB`、`CPU_%`、`CMD` | 把 `PID` 传给需要 `--pid` 的 create 命令                                  |
| 创建新的 on-cpu 任务                       | `bytedcli bytedog profile oncpu create`                                                                 | 详情页 URL                                | 等任务完成后执行 `bytedcli bytedog profile get --url <detail-url>`        |
| 创建新的 continuous profiling 任务         | `bytedcli bytedog profile sprofile create`                                                              | 详情页 URL                                | 等任务完成后执行 `bytedcli bytedog profile get --url <detail-url>`        |
| 创建新的 off-cpu / pthread / jemalloc 任务 | `bytedcli bytedog profile offcpu create` / `pthread create` / `je-stats create` / `je-flamegraph create` | 详情页 URL                                | 等任务完成后执行 `bytedcli bytedog profile get --url <detail-url>`        |
| 查历史任务并获取详情页 URL                 | `bytedcli bytedog profile <type> list`                                                                  | `detail_url`                              | 对目标 URL 执行 `bytedcli bytedog profile get`                            |

公开的 `profile <type> create` / `profile <type> list` 子命令固定为 `oncpu`、`sprofile`、`offcpu`、`pthread`、`je-stats`、`je-flamegraph`。不要使用内部类型名 `flamegraph`、`continuous-flamegraph`、`offcpu-flamegraph`、`memory-flamegraph` 作为 CLI 子命令。

## 执行约定

- `profile <type> create` 和 `tool process list` 的目标参数必须恰好提供一个：`--ip`、`--pod`、`--workspace-id`。其中 `sprofile` 只支持 `--ip`，`offcpu` / `pthread` / `je-stats` / `je-flamegraph` 不支持 `--workspace-id`。
- `profile <type> list` 的普通历史命令至少提供一个过滤条件：`--ip`、`--pod`、`--psm`。`profile sprofile list` 只接受 `--ip`，不接受 `--pod` / `--psm`。
- `profile <type> create` 和 `tool process list` 会尝试把 `--ip` 输入归一化为 ByteDog 可识别的目标 IP；`profile <type> list` 不做 IP 归一化，按传入文本过滤历史记录。
- `--idc` 与 `--container-type primary|sidecar` 都是 `--pod` 目标的可选消歧参数。默认先只传 `--pod`；只有 Pod 解析跨 IDC/容器有歧义，或明确要采 sidecar 时再补充。未传 `--container-type` 时默认为 `primary`。
- `--tob` 只用于非 TTP 站点的 `--ip` 目标；TTP 站点会忽略该开关。需要 ToB / mysql 机器模式时必须显式传入 `--tob`。
- `profile <type> create` 只提交异步任务并返回详情页 URL，不等待结果文件生成。完整结果统一用 `profile get --url <detail-url> --output-dir <dir>` 获取。
- `profile get` 只在详情页任务状态为 `GOOD` 时获取结果文件。任务仍在运行时会提示稍后重试；任务失败时会输出错误信息和可用提示。

## `bytedcli bytedog profile get`

获取一个或多个 ByteDog 详情页 URL 对应的结果文件，并在输出目录生成 `data-format.md`。该命令不创建任务，只读取已存在的详情页结果。

### 参数

| 参数                 | 必填 | 默认值                        | 取值                                                  | 说明                                                                                                                     |
| -------------------- | ---- | ----------------------------- | ----------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `--url <url>`        | 是   | 无                            | 单个 ByteDog detail URL，或英文逗号分隔的多个 URL     | 支持 on-cpu、off-cpu flamegraph、pthread lock、jemalloc stats、jemalloc memory flamegraph、continuous profiling 详情页。 |
| `--output-dir <dir>` | 否   | `.`                           | 本地目录路径                                          | 保存 `data-format.md` 与结果文件的目录。目录不存在时会创建。                                                             |

### 输出

文本模式输出获取提示、任务目标和文件列表：

```text
- HINT
成功获取数据，请优先阅读 ./bytedog-output/data-format.md 获取**采集结果数据介绍和使用须知**
- 任务目标
1001: psm=demo.service ip=- podname=demo-pod
- 数据列表
./bytedog-output/data-format.md
./bytedog-output/1001-perf_stack_url.collapse
```

JSON 模式输出 bytedcli 标准 envelope，业务字段在 `data` 内：

```json
{
  "status": "success",
  "data": {
    "hint": "成功获取数据，请优先阅读 ./bytedog-output/data-format.md 获取采集结果数据介绍和使用须知",
    "targets": [
      {
        "id": 1001,
        "psm": "demo.service",
        "ip": null,
        "podname": "demo-pod"
      }
    ],
    "files": ["./bytedog-output/data-format.md", "./bytedog-output/1001-perf_stack_url.collapse"]
  },
  "error": null,
  "context": {
    "execution_time_ms": 123,
    "timestamp": "2026-06-08T10:00:00+08:00",
    "api_endpoint": "ByteDog Profile Get"
  }
}
```

`data-format.md` 会解释每个结果文件的格式和使用注意事项。结果文件可能包括 `.collapse`、`.json` 等格式，具体取决于详情页的 profile 类型和任务结果。

多个 URL 用英文逗号分隔时，命令会把所有结果写入同一个输出目录，并生成一份合并后的 `data-format.md`。`data.targets` 与 `data.files` 会包含所有任务和文件。

当详情页任务状态不是 `GOOD` 时，命令不会获取结果文件。任务仍在运行时会提示稍后重试；任务失败时会输出任务错误信息和可用提示。

### Example

```bash
bytedcli --site cn bytedog profile get \
  --url 'https://example.bytedog/profiling/on-cpu-profiling/detail?id=1001&from=tce' \
  --output-dir ./bytedog-output
```

```bash
bytedcli --site cn bytedog profile get \
  --url 'https://example.bytedog/profiling/on-cpu-profiling/detail?id=1001&from=tce,https://example.bytedog/profiling/jemalloc-profiling/stats?id=1004&from=machine' \
  --output-dir ./bytedog-output
```

```bash
bytedcli --json --site cn bytedog profile get \
  --url 'https://example.bytedog/profiling/continuous-profiling/detail?id=1006&time=long' \
  --output-dir ./bytedog-output
```

## `bytedcli bytedog profile oncpu create`

创建 on-cpu flamegraph 异步任务。

### 参数

| 参数                                  | 必填     | 默认值                        | 取值                                                                                                                                                                         | 说明                                                                                                                                    |
| ------------------------------------- | -------- | ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| `--ip <ip>`                           | 条件必填 | 无                            | 主机 IP 或 hostname                                                                                                                                                          | 目标参数三选一。用于对机器发起采样。                                                                                                    |
| `--pod <podname>`                     | 条件必填 | 无                            | TCE Pod 名称                                                                                                                                                                 | 目标参数三选一。用于对 TCE Pod 发起采样。                                                                                               |
| `--workspace-id <id>`                 | 条件必填 | 无                            | Cloud IDE workspace ID                                                                                                                                                       | 目标参数三选一。用于对 Cloud IDE workspace 发起采样。                                                                                   |
| `--idc <idc>`                         | 否       | 无                            | IDC 标识                                                                                                                                                                     | 只在 `--pod` 目标下可选使用，用于 Pod 歧义消解；默认不需要传。                                                                          |
| `--container-type <primary\|sidecar>` | 否       | `primary`                     | `primary`、`sidecar`                                                                                                                                                         | 只在 `--pod` 目标下可选使用，用于容器歧义消解。                                                                                          |
| `--pid <pid>`                         | 否       | 无                            | 单个正整数 PID                                                                                                                                                               | 限定单进程采样。只允许一个 PID，不支持逗号或空格分隔。                                                                                  |
| `--duration <seconds>`                | 否       | `30`                          | `1` 到 `300` 的整数秒；`python --tools-type ebpf_profiler` 最少 `10` 秒                                                                                                      | 采样时长。超过 `300` 秒会自动按 `300` 秒提交；启用 `--inline` 且 `--pid` 存在、工具为 `perf` 或 `bytekd` 时，实际采样时长最多 `5` 秒。   |
| `--type <type>`                       | 否       | `cpp`                         | `cpp`、`java`、`python`、`go`、`rust`                                                                                                                                        | 采样语言类型。                                                                                                                          |
| `--tools-type <type>`                 | 否       | 按语言决定                    | `perf`、`bcc`、`bytekd`、`ebpf_profiler`、`pyspy`                                                                                                                            | `cpp/go/rust` 默认 `bytekd`，允许 `perf/bcc/bytekd`；`python` 默认 `ebpf_profiler`，TTP 站点只允许并默认 `pyspy`；`java` 会忽略该参数。 |
| `--callgraph-type <type>`             | 否       | `fp`                          | `fp`、`lbr`、`dwarf`                                                                                                                                                         | 仅在 `--tools-type perf` 时生效。                                                                                                       |
| `--interval <ns>`                     | 否       | `10000000`                    | 正整数纳秒                                                                                                                                                                   | 仅在 `--type java` 时生效，表示 Java 采样间隔。                                                                                         |
| `--perf-event <event[,event...]>`     | 否       | `cpu-cycles`                  | `cpu-cycles`、`cpu-clock`、`branch-misses`、`L1-icache-load-misses`、`L1-dcache-load-misses`、`LLC-load-misses`、`iTLB-load-misses`、`dTLB-load-misses`、`dTLB-store-misses` | 仅在 `--tools-type perf` 时生效。多个值用英文逗号分隔。                                                                                 |
| `--inline`                            | 否       | `false`                       | 布尔开关                                                                                                                                                                     | 在 `--pid` 存在且工具为 `perf` 或 `bytekd` 时解析 inline frame，并把实际采样时长限制在最多 `5` 秒。                                     |
| `--line-info`                         | 否       | `false`                       | 布尔开关                                                                                                                                                                     | 在 `--inline`、`--pid`、`--tools-type perf` 同时满足时解析源码行信息。                                                                  |
| `--python-subprocess`                 | 否       | `false`                       | 布尔开关                                                                                                                                                                     | 仅在 `--type python` 时包含 Python 子进程。                                                                                             |
| `--include-idle`                      | 否       | `false`                       | 布尔开关                                                                                                                                                                     | 仅在 `--type python --tools-type pyspy` 时包含 idle 样本；与其他 Python 工具组合会报错。                                                |
| `--include-native`                    | 否       | `true`                        | 布尔开关                                                                                                                                                                     | 仅在 `--type python` 时包含 native Python stacks；当前 CLI 不提供关闭开关。                                                            |
| `--tob`                               | 否       | `false`                       | 布尔开关                                                                                                                                                                     | 非 TTP 站点仅支持 `--ip` 目标。用于 ToB 或 mysql 机器模式；TTP 站点会忽略该参数。                                                       |

`--ip`、`--pod`、`--workspace-id` 必须恰好提供一个。多传或全不传都会报输入错误。

### 输出

创建成功后输出详情页 URL 和后续获取命令提示。命令只提交异步任务，不等待结果文件生成。

```json
{
  "status": "success",
  "data": {
    "url": "https://example.bytedog/profiling/on-cpu-profiling/detail?id=1001&from=machine",
    "hint": "任务创建成功，执行 `bytedcli bytedog profile get --url 'https://example.bytedog/profiling/on-cpu-profiling/detail?id=1001&from=machine' --output-dir ./bytedog-output` 查看任务状态以及获取结果数据。"
  },
  "error": null,
  "context": {
    "execution_time_ms": 123,
    "timestamp": "2026-06-08T10:00:00+08:00",
    "api_endpoint": "ByteDog Profile Create"
  }
}
```

### Example

```bash
bytedcli --site cn bytedog profile oncpu create \
  --ip example-host \
  --type cpp \
  --tools-type bytekd \
  --duration 30
```

```bash
bytedcli --site cn bytedog profile oncpu create \
  --pod demo-pod \
  --pid 12345 \
  --type go \
  --tools-type perf \
  --perf-event cpu-clock,branch-misses \
  --inline
```

```bash
bytedcli --json --site cn bytedog profile oncpu create \
  --workspace-id sample-workspace \
  --type python \
  --tools-type pyspy \
  --include-idle
```

## `bytedcli bytedog profile sprofile create`

创建 continuous flamegraph 异步任务。

### 参数

| 参数                               | 必填 | 默认值                        | 取值                                                  | 说明                                                                                                                    |
| ---------------------------------- | ---- | ----------------------------- | ----------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| `--ip <ip>`                        | 是   | 无                            | 主机 IP 或 hostname                                   | 只支持机器目标。                                                                                                        |
| `--duration <seconds>`             | 否   | `1800`                        | `1` 到 `3600` 的整数秒                                | 采集时长。超过 `3600` 秒会自动按 `3600` 秒提交。                                                                         |
| `--start <unix_seconds>`           | 否   | 当前时间减去 `duration`       | Unix 秒级时间戳                                       | 采集开始时间。                                                                                                          |
| `--display-core`                   | 否   | `false`                       | 布尔开关                                              | 展示 core 信息。                                                                                                        |
| `--tob`                            | 否   | `false`                       | 布尔开关                                              | 非 TTP 站点仅支持 `--ip` 目标。显式传入时使用 ToB 或 mysql 机器模式；TTP 站点会忽略该参数。 |

该命令不支持 `--pod` 和 `--workspace-id`。默认使用内场机器模式；非 TTP 站点显式传入 `--tob` 时使用 ToB 或 mysql 机器模式。

### 输出

创建成功后输出详情页 URL 和后续获取命令提示。命令只提交异步任务，不等待结果文件生成。

```json
{
  "status": "success",
  "data": {
    "url": "https://example.bytedog/profiling/continuous-profiling/detail?id=1003&time=long",
    "hint": "任务创建成功，执行 `bytedcli bytedog profile get --url 'https://example.bytedog/profiling/continuous-profiling/detail?id=1003&time=long' --output-dir ./bytedog-output` 查看任务状态以及获取结果数据。"
  },
  "error": null,
  "context": {
    "execution_time_ms": 123,
    "timestamp": "2026-06-08T10:00:00+08:00",
    "api_endpoint": "ByteDog Profile Create"
  }
}
```

### Example

```bash
bytedcli --site cn bytedog profile sprofile create \
  --ip example-host \
  --duration 1800
```

```bash
bytedcli --site cn bytedog profile sprofile create \
  --ip example-host \
  --duration 900 \
  --start 1700000000 \
  --display-core
```

```bash
bytedcli --json --site cn bytedog profile sprofile create \
  --ip example-host
```

## `bytedcli bytedog profile offcpu create`

创建 off-cpu flamegraph 异步任务。

### 参数

| 参数                                  | 必填     | 默认值                        | 取值                                                  | 说明                                                                              |
| ------------------------------------- | -------- | ----------------------------- | ----------------------------------------------------- | --------------------------------------------------------------------------------- |
| `--ip <ip>`                           | 条件必填 | 无                            | 主机 IP 或 hostname                                   | 目标参数二选一。用于对机器发起采样。                                              |
| `--pod <podname>`                     | 条件必填 | 无                            | TCE Pod 名称                                          | 目标参数二选一。用于对 TCE Pod 发起采样。                                         |
| `--idc <idc>`                         | 否       | 无                            | IDC 标识                                              | 只在 `--pod` 目标下可选使用，用于 Pod 歧义消解；默认不需要传。                    |
| `--container-type <primary\|sidecar>` | 否       | `primary`                     | `primary`、`sidecar`                                  | 只在 `--pod` 目标下可选使用，用于容器歧义消解。                                  |
| `--pid <pid>`                         | 是       | 无                            | 单个正整数 PID                                        | 采集进程。只允许一个 PID，不支持逗号或空格分隔。                                  |
| `--duration <seconds>`                | 否       | `30`                          | `1` 到 `300` 的整数秒                                 | 采样时长。超过 `300` 秒会自动按 `300` 秒提交。                                  |
| `--tools-type <type>`                 | 否       | `bytekd`                      | `bcc`、`bytekd`                                       | off-cpu 采样工具类型。                                                            |
| `--enhance`                           | 否       | `true`                        | 布尔开关                                              | 启用增强栈解析。                                                                  |
| `--tob`                               | 否       | `false`                       | 布尔开关                                              | 非 TTP 站点仅支持 `--ip` 目标。用于 ToB 或 mysql 机器模式；TTP 站点会忽略该参数。 |

`--ip` 与 `--pod` 必须恰好提供一个。

### 输出

创建成功后输出详情页 URL 和后续获取命令提示。命令只提交异步任务，不等待结果文件生成。

### Example

```bash
bytedcli --site cn bytedog profile offcpu create \
  --pod demo-pod \
  --pid 12345 \
  --duration 30
```

```bash
bytedcli --site cn bytedog profile offcpu create \
  --ip example-host \
  --pid 12345 \
  --tools-type bcc
```

```bash
bytedcli --json --site cn bytedog profile offcpu create \
  --ip example-host \
  --pid 12345
```

## `bytedcli bytedog profile pthread create`

创建 pthread lock profiling 异步任务。

### 参数

| 参数                                  | 必填     | 默认值                        | 取值                                                  | 说明                                                                              |
| ------------------------------------- | -------- | ----------------------------- | ----------------------------------------------------- | --------------------------------------------------------------------------------- |
| `--ip <ip>`                           | 条件必填 | 无                            | 主机 IP 或 hostname                                   | 目标参数二选一。用于对机器发起采样。                                              |
| `--pod <podname>`                     | 条件必填 | 无                            | TCE Pod 名称                                          | 目标参数二选一。用于对 TCE Pod 发起采样。                                         |
| `--idc <idc>`                         | 否       | 无                            | IDC 标识                                              | 只在 `--pod` 目标下可选使用，用于 Pod 歧义消解；默认不需要传。                    |
| `--container-type <primary\|sidecar>` | 否       | `primary`                     | `primary`、`sidecar`                                  | 只在 `--pod` 目标下可选使用，用于容器歧义消解。                                  |
| `--pid <pid>`                         | 是       | 无                            | 单个正整数 PID                                        | 采集进程。只允许一个 PID，不支持逗号或空格分隔。                                  |
| `--duration <seconds>`                | 否       | `30`                          | `1` 到 `300` 的整数秒                                 | 采样时长。超过 `300` 秒会自动按 `300` 秒提交。                                  |
| `--enhance`                           | 否       | 非 TTP: `true`；TTP: `false`  | 布尔开关                                              | 启用增强栈解析。                                                                  |
| `--tob`                               | 否       | `false`                       | 布尔开关                                              | 非 TTP 站点仅支持 `--ip` 目标。用于 ToB 或 mysql 机器模式；TTP 站点会忽略该参数。 |

`--ip` 与 `--pod` 必须恰好提供一个。

### 输出

创建成功后输出详情页 URL 和后续获取命令提示。命令只提交异步任务，不等待结果文件生成。

### Example

```bash
bytedcli --site cn bytedog profile pthread create \
  --pod demo-pod \
  --pid 12345 \
  --duration 120
```

```bash
bytedcli --json --site cn bytedog profile pthread create \
  --ip example-host \
  --pid 12345
```

## `bytedcli bytedog profile je-stats create`

创建 jemalloc stats 异步任务。

### 参数

| 参数                                  | 必填     | 默认值                        | 取值                                                  | 说明                                                                              |
| ------------------------------------- | -------- | ----------------------------- | ----------------------------------------------------- | --------------------------------------------------------------------------------- |
| `--ip <ip>`                           | 条件必填 | 无                            | 主机 IP 或 hostname                                   | 目标参数二选一。用于对机器发起采样。                                              |
| `--pod <podname>`                     | 条件必填 | 无                            | TCE Pod 名称                                          | 目标参数二选一。用于对 TCE Pod 发起采样。                                         |
| `--idc <idc>`                         | 否       | 无                            | IDC 标识                                              | 只在 `--pod` 目标下可选使用，用于 Pod 歧义消解；默认不需要传。                    |
| `--container-type <primary\|sidecar>` | 否       | `primary`                     | `primary`、`sidecar`                                  | 只在 `--pod` 目标下可选使用，用于容器歧义消解。                                  |
| `--pid <pid>`                         | 是       | 无                            | 单个正整数 PID                                        | 采集进程。只允许一个 PID，不支持逗号或空格分隔。                                  |
| `--tob`                               | 否       | `false`                       | 布尔开关                                              | 非 TTP 站点仅支持 `--ip` 目标。用于 ToB 或 mysql 机器模式；TTP 站点会忽略该参数。 |

`--ip` 与 `--pod` 必须恰好提供一个。

### 输出

创建成功后输出详情页 URL 和后续获取命令提示。命令只提交异步任务，不等待结果文件生成。

### Example

```bash
bytedcli --site cn bytedog profile je-stats create \
  --ip example-host \
  --pid 12345
```

```bash
bytedcli --json --site cn bytedog profile je-stats create \
  --pod demo-pod \
  --pid 12345
```

## `bytedcli bytedog profile je-flamegraph create`

创建 jemalloc memory flamegraph 异步任务。

### 参数

| 参数                                  | 必填     | 默认值                        | 取值                                                  | 说明                                                                                 |
| ------------------------------------- | -------- | ----------------------------- | ----------------------------------------------------- | ------------------------------------------------------------------------------------ |
| `--ip <ip>`                           | 条件必填 | 无                            | 主机 IP 或 hostname                                   | 目标参数二选一。用于对机器发起采样。                                                 |
| `--pod <podname>`                     | 条件必填 | 无                            | TCE Pod 名称                                          | 目标参数二选一。用于对 TCE Pod 发起采样。                                            |
| `--idc <idc>`                         | 否       | 无                            | IDC 标识                                              | 只在 `--pod` 目标下可选使用，用于 Pod 歧义消解；默认不需要传。                       |
| `--container-type <primary\|sidecar>` | 否       | `primary`                     | `primary`、`sidecar`                                  | 只在 `--pod` 目标下可选使用，用于容器歧义消解。                                      |
| `--pid <pid>`                         | 是       | 无                            | 单个正整数 PID                                        | 采集进程。只允许一个 PID，不支持逗号或空格分隔。                                     |
| `--type <stock\|increment>`           | 否       | `increment`                   | `increment`、`stock`                                  | `increment` 表示增量采集；`stock` 表示全量采集。主机目标 `--ip` 仅支持 `increment`。 |
| `--process-name <cmd>`                | 否       | 无                            | 进程命令字符串                                        | 指定进程命令。建议先用 `bytedog tool process list` 确认。                          |
| `--je-version <version>`              | 否       | 自动选择                      | jemalloc 版本字符串                                   | 仅在 `--type stock` 时使用。显式传入时优先级最高。                                   |
| `--tob`                               | 否       | `false`                       | 布尔开关                                              | 非 TTP 站点仅支持 `--ip` 目标。用于 ToB 或 mysql 机器模式；TTP 站点会忽略该参数。    |

`--ip` 与 `--pod` 必须恰好提供一个。创建前会校验 jemalloc 环境状态；环境不可采集时不会创建任务。主机目标 `--ip` 只支持 `--type increment`；TCE Pod 目标可使用 `--type stock`。若当前实例不支持增量采集，命令会提示改用全量采集。

### 输出

创建成功后输出详情页 URL 和后续获取命令提示。命令只提交异步任务，不等待结果文件生成。

### Example

```bash
bytedcli --site cn bytedog profile je-flamegraph create \
  --pod demo-pod \
  --pid 12345 \
  --process-name /opt/demo/bin/server
```

```bash
bytedcli --site cn bytedog profile je-flamegraph create \
  --pod demo-pod \
  --pid 12345 \
  --type stock \
  --je-version 5.2.1.sample
```

```bash
bytedcli --json --site cn bytedog profile je-flamegraph create \
  --ip example-host \
  --pid 12345
```

## `bytedcli bytedog profile oncpu list`

查询 on-cpu flamegraph 历史记录，并输出可交给 `profile get` 的详情页 URL。

### 参数

| 参数                   | 必填     | 默认值                        | 取值                                                  | 说明                                               |
| ---------------------- | -------- | ----------------------------- | ----------------------------------------------------- | -------------------------------------------------- |
| `--ip <ip>`            | 条件必填 | 无                            | 目标 IP 或 hostname 文本                              | 目标过滤参数之一。不会做 IP 归一化。               |
| `--pod <podname>`      | 条件必填 | 无                            | TCE Pod 名称                                          | 目标过滤参数之一。                                 |
| `--psm <psm>`          | 条件必填 | 无                            | PSM 名称                                              | 目标过滤参数之一。                                 |
| `--status <statuses>`  | 否       | 无                            | `INIT`、`RUNNING`、`GOOD`、`BAD`，可用英文逗号分隔    | 按任务状态过滤。                                   |
| `--creator <creators>` | 否       | 无                            | 创建人标识，可用英文逗号分隔                          | 按任务创建人过滤。                                 |
| `--page <n>`           | 否       | `1`                           | 正整数                                                | 页码，从 `1` 开始。                                |
| `--page-size <n>`      | 否       | `20`                          | 正整数                                                | 每页条数。                                         |
| `--url-only`           | 否       | `false`                       | 布尔开关                                              | 文本模式只逐行输出详情页 URL。                     |

`--ip`、`--pod`、`--psm` 至少提供一个。多个过滤参数可以同时提供，后端按组合条件查询。

### 输出

文本模式输出表格列：`ID`、`STATUS`、`TYPE`、`TARGET`、`CREATOR`、`START_AT`、`URL`，并输出 `Current Count` 与 `Has More`。

JSON 模式输出 bytedcli 标准 envelope，业务对象在 `data` 字段里：

```json
{
  "status": "success",
  "data": {
    "items": [
      {
        "id": 1001,
        "profile_type": "oncpu",
        "raw_type": "CPU_CPP_FLAMEGRAPH_ON_TCE",
        "status": "GOOD",
        "target": {
          "ip": null,
          "pod": "demo-pod",
          "psm": "demo.service"
        },
        "creator": "demo-user",
        "description": "sample task",
        "start_at": "2026-06-05T02:00:00.000Z",
        "end_at": "2026-06-05T02:00:30.000Z",
        "detail_url": "https://example.bytedog/profiling/on-cpu-profiling/detail?id=1001&from=tce"
      }
    ],
    "page": 1,
    "page_size": 20,
    "current_count": 1,
    "has_more": false
  },
  "error": null,
  "context": {
    "execution_time_ms": 123,
    "timestamp": "2026-06-08T10:00:00+08:00",
    "api_endpoint": "ByteDog Profile List"
  }
}
```

`current_count` 表示当前页返回条数，不代表历史总数。`has_more=true` 表示可能还有下一页。后面 list 子命令的 JSON envelope 形态相同，仅 `data` 内容不同。

### Example

```bash
bytedcli --site cn bytedog profile oncpu list \
  --ip example-host \
  --status GOOD,RUNNING \
  --page 1 \
  --page-size 20
```

```bash
bytedcli --site cn bytedog profile oncpu list \
  --pod demo-pod \
  --psm demo.service \
  --url-only
```

```bash
bytedcli --json --site cn bytedog profile oncpu list \
  --ip example-host
```

## `bytedcli bytedog profile sprofile list`

查询 continuous flamegraph 历史记录，并输出可交给 `profile get` 的详情页 URL。

### 参数

| 参数                   | 必填 | 默认值                        | 取值                                                  | 说明                                               |
| ---------------------- | ---- | ----------------------------- | ----------------------------------------------------- | -------------------------------------------------- |
| `--ip <ip>`            | 是   | 无                            | 目标 IP 或 hostname 文本                              | 按机器目标过滤。不会做 IP 归一化。                 |
| `--status <statuses>`  | 否   | 无                            | `INIT`、`RUNNING`、`GOOD`、`BAD`，可用英文逗号分隔    | 按任务状态过滤。                                   |
| `--creator <creators>` | 否   | 无                            | 创建人标识，可用英文逗号分隔                          | 按任务创建人过滤。                                 |
| `--page <n>`           | 否   | `1`                           | 正整数                                                | 页码，从 `1` 开始。                                |
| `--page-size <n>`      | 否   | `20`                          | 正整数                                                | 每页条数。                                         |
| `--url-only`           | 否   | `false`                       | 布尔开关                                              | 文本模式只逐行输出详情页 URL。                     |

该命令不接受 `--pod` 和 `--psm`。

### 输出

文本模式输出表格列：`ID`、`STATUS`、`TYPE`、`TARGET`、`CREATOR`、`START_AT`、`URL`，并输出 `Current Count` 与 `Has More`。

JSON envelope 形态同 `profile oncpu list`；`data` 示例：

```json
{
  "items": [
    {
      "id": 1007,
      "profile_type": "sprofile",
      "raw_type": "STEBPF-SPLIT",
      "status": "GOOD",
      "target": {
        "ip": "example-host",
        "pod": null,
        "psm": null
      },
      "creator": "demo-user",
      "description": null,
      "start_at": "2026-06-05T02:00:00.000Z",
      "end_at": "2026-06-05T02:30:00.000Z",
      "detail_url": "https://example.bytedog/profiling/continuous-profiling/detail?id=1007&time=long"
    }
  ],
  "page": 1,
  "page_size": 20,
  "current_count": 1,
  "has_more": false
}
```

`current_count` 表示当前页返回条数，不代表历史总数。`has_more=true` 表示可能还有下一页。

### Example

```bash
bytedcli --site cn bytedog profile sprofile list \
  --ip example-host \
  --status GOOD
```

```bash
bytedcli --site cn bytedog profile sprofile list \
  --ip example-host \
  --url-only
```

```bash
bytedcli --json --site cn bytedog profile sprofile list \
  --ip example-host
```

## `bytedcli bytedog profile offcpu list`

查询 off-cpu flamegraph 历史记录，并输出可交给 `profile get` 的详情页 URL。

### 参数

| 参数                   | 必填     | 默认值                        | 取值                                                  | 说明                                               |
| ---------------------- | -------- | ----------------------------- | ----------------------------------------------------- | -------------------------------------------------- |
| `--ip <ip>`            | 条件必填 | 无                            | 目标 IP 或 hostname 文本                              | 目标过滤参数之一。不会做 IP 归一化。               |
| `--pod <podname>`      | 条件必填 | 无                            | TCE Pod 名称                                          | 目标过滤参数之一。                                 |
| `--psm <psm>`          | 条件必填 | 无                            | PSM 名称                                              | 目标过滤参数之一。                                 |
| `--status <statuses>`  | 否       | 无                            | `INIT`、`RUNNING`、`GOOD`、`BAD`，可用英文逗号分隔    | 按任务状态过滤。                                   |
| `--creator <creators>` | 否       | 无                            | 创建人标识，可用英文逗号分隔                          | 按任务创建人过滤。                                 |
| `--page <n>`           | 否       | `1`                           | 正整数                                                | 页码，从 `1` 开始。                                |
| `--page-size <n>`      | 否       | `20`                          | 正整数                                                | 每页条数。                                         |
| `--url-only`           | 否       | `false`                       | 布尔开关                                              | 文本模式只逐行输出详情页 URL。                     |

`--ip`、`--pod`、`--psm` 至少提供一个。多个过滤参数可以同时提供，后端按组合条件查询。

### 输出

文本模式输出表格列：`ID`、`STATUS`、`TYPE`、`TARGET`、`CREATOR`、`START_AT`、`URL`，并输出 `Current Count` 与 `Has More`。

JSON envelope 形态同 `profile oncpu list`；`data` 示例：

```json
{
  "items": [
    {
      "id": 1003,
      "profile_type": "offcpu",
      "raw_type": "OFFCPU_FLAMEGRAPH_ON_TCE",
      "status": "GOOD",
      "target": {
        "ip": null,
        "pod": "demo-pod",
        "psm": "demo.service"
      },
      "creator": "demo-user",
      "description": null,
      "start_at": null,
      "end_at": null,
      "detail_url": "https://example.bytedog/profiling/off-cpu-profiling/flamegraph/detail?id=1003"
    }
  ],
  "page": 1,
  "page_size": 20,
  "current_count": 1,
  "has_more": false
}
```

### Example

```bash
bytedcli --site cn bytedog profile offcpu list \
  --pod demo-pod \
  --status GOOD
```

```bash
bytedcli --site cn bytedog profile offcpu list \
  --psm demo.service \
  --url-only
```

```bash
bytedcli --json --site cn bytedog profile offcpu list \
  --ip example-host
```

## `bytedcli bytedog profile pthread list`

查询 pthread lock profiling 历史记录，并输出可交给 `profile get` 的详情页 URL。

### 参数

| 参数                   | 必填     | 默认值                        | 取值                                                  | 说明                                               |
| ---------------------- | -------- | ----------------------------- | ----------------------------------------------------- | -------------------------------------------------- |
| `--ip <ip>`            | 条件必填 | 无                            | 目标 IP 或 hostname 文本                              | 目标过滤参数之一。不会做 IP 归一化。               |
| `--pod <podname>`      | 条件必填 | 无                            | TCE Pod 名称                                          | 目标过滤参数之一。                                 |
| `--psm <psm>`          | 条件必填 | 无                            | PSM 名称                                              | 目标过滤参数之一。                                 |
| `--status <statuses>`  | 否       | 无                            | `INIT`、`RUNNING`、`GOOD`、`BAD`，可用英文逗号分隔    | 按任务状态过滤。                                   |
| `--creator <creators>` | 否       | 无                            | 创建人标识，可用英文逗号分隔                          | 按任务创建人过滤。                                 |
| `--page <n>`           | 否       | `1`                           | 正整数                                                | 页码，从 `1` 开始。                                |
| `--page-size <n>`      | 否       | `20`                          | 正整数                                                | 每页条数。                                         |
| `--url-only`           | 否       | `false`                       | 布尔开关                                              | 文本模式只逐行输出详情页 URL。                     |

`--ip`、`--pod`、`--psm` 至少提供一个。多个过滤参数可以同时提供，后端按组合条件查询。

### 输出

文本模式输出表格列：`ID`、`STATUS`、`TYPE`、`TARGET`、`CREATOR`、`START_AT`、`URL`，并输出 `Current Count` 与 `Has More`。

JSON envelope 形态同 `profile oncpu list`；`data` 示例：

```json
{
  "items": [
    {
      "id": 1004,
      "profile_type": "pthread",
      "raw_type": "USER_LOCK_STAT_ON_TCE",
      "status": "GOOD",
      "target": {
        "ip": null,
        "pod": "demo-pod",
        "psm": "demo.service"
      },
      "creator": "demo-user",
      "description": null,
      "start_at": null,
      "end_at": null,
      "detail_url": "https://example.bytedog/profiling/off-cpu-profiling/lock/detail?id=1004"
    }
  ],
  "page": 1,
  "page_size": 20,
  "current_count": 1,
  "has_more": false
}
```

### Example

```bash
bytedcli --site cn bytedog profile pthread list \
  --psm demo.service \
  --status GOOD
```

```bash
bytedcli --site cn bytedog profile pthread list \
  --pod demo-pod \
  --url-only
```

```bash
bytedcli --json --site cn bytedog profile pthread list \
  --ip example-host
```

## `bytedcli bytedog profile je-stats list`

查询 jemalloc stats 历史记录，并输出可交给 `profile get` 的详情页 URL。

### 参数

| 参数                   | 必填     | 默认值                        | 取值                                                  | 说明                                               |
| ---------------------- | -------- | ----------------------------- | ----------------------------------------------------- | -------------------------------------------------- |
| `--ip <ip>`            | 条件必填 | 无                            | 目标 IP 或 hostname 文本                              | 目标过滤参数之一。不会做 IP 归一化。               |
| `--pod <podname>`      | 条件必填 | 无                            | TCE Pod 名称                                          | 目标过滤参数之一。                                 |
| `--psm <psm>`          | 条件必填 | 无                            | PSM 名称                                              | 目标过滤参数之一。                                 |
| `--status <statuses>`  | 否       | 无                            | `INIT`、`RUNNING`、`GOOD`、`BAD`，可用英文逗号分隔    | 按任务状态过滤。                                   |
| `--creator <creators>` | 否       | 无                            | 创建人标识，可用英文逗号分隔                          | 按任务创建人过滤。                                 |
| `--page <n>`           | 否       | `1`                           | 正整数                                                | 页码，从 `1` 开始。                                |
| `--page-size <n>`      | 否       | `20`                          | 正整数                                                | 每页条数。                                         |
| `--url-only`           | 否       | `false`                       | 布尔开关                                              | 文本模式只逐行输出详情页 URL。                     |

`--ip`、`--pod`、`--psm` 至少提供一个。多个过滤参数可以同时提供，后端按组合条件查询。

### 输出

文本模式输出表格列：`ID`、`STATUS`、`TYPE`、`TARGET`、`CREATOR`、`START_AT`、`URL`，并输出 `Current Count` 与 `Has More`。

JSON envelope 形态同 `profile oncpu list`；`data` 示例：

```json
{
  "items": [
    {
      "id": 1005,
      "profile_type": "je-stats",
      "raw_type": "JEMALLOC_STATS_ON_MACHINE",
      "status": "GOOD",
      "target": {
        "ip": "example-host",
        "pod": null,
        "psm": null
      },
      "creator": "demo-user",
      "description": null,
      "start_at": null,
      "end_at": null,
      "detail_url": "https://example.bytedog/profiling/jemalloc-profiling/stats?id=1005&from=machine"
    }
  ],
  "page": 1,
  "page_size": 20,
  "current_count": 1,
  "has_more": false
}
```

### Example

```bash
bytedcli --site cn bytedog profile je-stats list \
  --ip example-host \
  --status GOOD
```

```bash
bytedcli --site cn bytedog profile je-stats list \
  --pod demo-pod \
  --url-only
```

```bash
bytedcli --json --site cn bytedog profile je-stats list \
  --ip example-host
```

## `bytedcli bytedog profile je-flamegraph list`

查询 jemalloc memory flamegraph 历史记录，并输出可交给 `profile get` 的详情页 URL。

### 参数

| 参数                   | 必填     | 默认值                        | 取值                                                  | 说明                                               |
| ---------------------- | -------- | ----------------------------- | ----------------------------------------------------- | -------------------------------------------------- |
| `--ip <ip>`            | 条件必填 | 无                            | 目标 IP 或 hostname 文本                              | 目标过滤参数之一。不会做 IP 归一化。               |
| `--pod <podname>`      | 条件必填 | 无                            | TCE Pod 名称                                          | 目标过滤参数之一。                                 |
| `--psm <psm>`          | 条件必填 | 无                            | PSM 名称                                              | 目标过滤参数之一。                                 |
| `--status <statuses>`  | 否       | 无                            | `INIT`、`RUNNING`、`GOOD`、`BAD`，可用英文逗号分隔    | 按任务状态过滤。                                   |
| `--creator <creators>` | 否       | 无                            | 创建人标识，可用英文逗号分隔                          | 按任务创建人过滤。                                 |
| `--page <n>`           | 否       | `1`                           | 正整数                                                | 页码，从 `1` 开始。                                |
| `--page-size <n>`      | 否       | `20`                          | 正整数                                                | 每页条数。                                         |
| `--url-only`           | 否       | `false`                       | 布尔开关                                              | 文本模式只逐行输出详情页 URL。                     |

`--ip`、`--pod`、`--psm` 至少提供一个。多个过滤参数可以同时提供，后端按组合条件查询。

### 输出

文本模式输出表格列：`ID`、`STATUS`、`TYPE`、`TARGET`、`CREATOR`、`START_AT`、`URL`，并输出 `Current Count` 与 `Has More`。

JSON envelope 形态同 `profile oncpu list`；`data` 示例：

```json
{
  "items": [
    {
      "id": 1010,
      "profile_type": "je-flamegraph",
      "raw_type": "JEMALLOC_FLAMEGRAPH_ON_TCE_INCREMENT_PROFILE",
      "status": "GOOD",
      "target": {
        "ip": null,
        "pod": "demo-pod",
        "psm": "demo.service"
      },
      "creator": "demo-user",
      "description": null,
      "start_at": null,
      "end_at": null,
      "detail_url": "https://example.bytedog/profiling/jemalloc-profiling/detail?id=1010&from=tce"
    }
  ],
  "page": 1,
  "page_size": 20,
  "current_count": 1,
  "has_more": false
}
```

### Example

```bash
bytedcli --site cn bytedog profile je-flamegraph list \
  --pod demo-pod \
  --status GOOD
```

```bash
bytedcli --site cn bytedog profile je-flamegraph list \
  --ip example-host \
  --url-only
```

```bash
bytedcli --json --site cn bytedog profile je-flamegraph list \
  --ip example-host
```

## `bytedcli bytedog tool process list`

列出目标上的进程，用于确认 `profile <type> create` 需要的 PID、容器内 namespace PID、进程命令、RSS 和 CPU 信息。

### 参数

| 参数                                  | 必填     | 默认值                        | 取值                                                  | 说明                                                                              |
| ------------------------------------- | -------- | ----------------------------- | ----------------------------------------------------- | --------------------------------------------------------------------------------- |
| `--pod <podname>`                     | 条件必填 | 无                            | TCE Pod 名称                                          | 目标参数三选一。                                                                  |
| `--ip <ip>`                           | 条件必填 | 无                            | 主机 IP 或 hostname                                   | 目标参数三选一。                                                                  |
| `--workspace-id <workspace_id>`       | 条件必填 | 无                            | Cloud IDE workspace ID                                | 目标参数三选一。                                                                  |
| `--idc <idc>`                         | 否       | 无                            | IDC 标识                                              | 只在 `--pod` 目标下可选使用，用于 Pod 歧义消解；默认不需要传。                    |
| `--container-type <primary\|sidecar>` | 否       | `primary`                     | `primary`、`sidecar`                                  | 只在 `--pod` 目标下可选使用，用于容器歧义消解。                                  |
| `--tob`                               | 否       | `false`                       | 布尔开关                                              | 非 TTP 站点仅支持 `--ip` 目标。用于 ToB 或 mysql 机器模式；TTP 站点会忽略该参数。 |

`--pod`、`--ip`、`--workspace-id` 必须恰好提供一个。

### 输出

文本模式输出表格列：`PID`、`TID_NS`、`RSS_KB`、`CPU_%`、`CMD`，并输出 `Current Count`。

JSON 模式输出 bytedcli 标准 envelope，业务对象在 `data` 字段里：

```json
{
  "status": "success",
  "data": {
    "target_type": "pod",
    "target": "demo-pod",
    "processes": [
      {
        "pid": 1001,
        "tid_ns": 11,
        "cmd": "/opt/demo/bin/server --flag",
        "rss": 2048,
        "cpu": 1.25
      }
    ],
    "current_count": 1
  },
  "error": null,
  "context": {
    "execution_time_ms": 123,
    "timestamp": "2026-06-08T10:00:00+08:00",
    "api_endpoint": "ByteDog Tool ListProcesses"
  }
}
```

### Example

```bash
bytedcli --site cn bytedog tool process list \
  --pod demo-pod
```

```bash
bytedcli --site cn bytedog tool process list \
  --ip example-host \
  --tob
```

```bash
bytedcli --json --site cn bytedog tool process list \
  --workspace-id sample-workspace
```
