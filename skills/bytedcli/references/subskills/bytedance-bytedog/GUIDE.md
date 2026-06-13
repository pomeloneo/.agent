---
name: bytedance-bytedog
description: "Use whenever users mention ByteDog, bytedcli bytedog, flamegraphs, CPU profiling, continuous profiling, jemalloc, off-CPU, pthread or lock contention, PID lookup, profile result download, or performance diagnostics"
---

# ByteDog (`bytedcli bytedog`) — Performance Profiling & Diagnostics

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

## 前置条件

- 使用通用调用方式：`../../invocation.md`
- ByteDog 需要 ByteCloud JWT 认证：`references/auth.md`
- `bytedcli bytedog` 命令参数: `references/profile-tool-command-reference.md`

## When to use

- 获取或解析已有 ByteDog detail URL 的 profiling 结果，使用 `bytedcli bytedog profile get`。
- 创建 profiling 任务前需要确认目标进程、PID、RSS、CPU、命令行时，先用 `bytedcli bytedog tool process list`。
- `bytedcli bytedog profile oncpu create`：用于 CPU 忙、CPU 使用率高、需要定位 C++/Go/Rust/Java/Python 热点函数或调用栈时；如果只想采某个进程，先拿 PID 再传 `--pid`。
- `bytedcli bytedog profile sprofile create`：用于需要机器维度的 continuous profiling、长时间 CPU 画像、历史时间窗口或难以稳定复现的 CPU 问题；该命令只支持 `--ip` 目标。
- `bytedcli bytedog profile offcpu create`：用于 p99/RT 飙升但 CPU 未饱和、怀疑线程卡在 IO wait、sleep、futex、调度等待或阻塞调用时；需要明确 PID。
- `bytedcli bytedog profile pthread create`：用于怀疑 pthread mutex/rwlock 等用户态锁竞争、锁等待或临界区争用导致延迟时；需要明确 PID。
- `bytedcli bytedog profile je-stats create`：用于快速查看 jemalloc allocator stats，判断 RSS、arena/bin/tcache、碎片或分配状态是否异常；需要明确 PID。
- `bytedcli bytedog profile je-flamegraph create`：用于怀疑 jemalloc 内存泄漏、RSS 持续增长、分配热点不清楚，或需要按调用栈定位内存分配来源时；需要明确 PID，必要时先用 `bytedcli bytedog tool process list` 确认 `--process-name`。
- 需要确认 `bytedcli bytedog` 命令参数、输出、限制或示例时，读 `references/profile-tool-command-reference.md`。

## 常用流程

执行 ByteDog profile 任务时，常见的使用场景对应的流程文档：

| Workflow | Use for |
| -------- | ------- |
| `references/tce-on-cpu-profile-workflow.md` | 根据 TCE PSM + podname 定位实例，创建 on-cpu 火焰图并获取结果。 |
| `references/profile-create-to-get-workflow.md` | 创建新的 profile 任务，等待完成并获取结果文件。 |
| `references/profile-list-to-get-workflow.md` | 查询历史 profile 任务，选择 `detail_url` 并获取结果文件。 |

## CLI Reference

Before executing or recommending `bytedcli bytedog`, read `references/profile-tool-command-reference.md` for the full command matrix, common options, output shapes, limitations, and examples.

| Command | Use for |
| ------- | ------- |
| `bytedcli bytedog profile get` | Get profile results from existing ByteDog detail URLs and generate `data-format.md`. |
| `bytedcli bytedog profile <oncpu/sprofile/offcpu/pthread/je-stats/je-flamegraph> create` | Submit new profiling tasks and return detail URLs. |
| `bytedcli bytedog profile <oncpu/sprofile/offcpu/pthread/je-stats/je-flamegraph> list` | Search historical profiling tasks and detail URLs. |
| `bytedcli bytedog tool process list` | List target processes before creating PID-scoped profiling tasks. |
