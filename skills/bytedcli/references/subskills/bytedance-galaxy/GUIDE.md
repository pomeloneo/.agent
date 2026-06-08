---
name: bytedance-galaxy
description: "Query Galaxy asset management and host information via bytedcli: list hosts under a PSM, inspect capacity summaries, and understand host distribution across control planes."
---

# bytedcli Galaxy

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

- 按 PSM 查询该服务下挂的所有机器列表
- 查看机器容量统计（CPU 核数、内存、磁盘、SSD）和机型分布
- 按 PSM 反查 Galaxy node 路径，自动推断 control plane

## 前置条件

- 使用通用调用方式：`../../invocation.md`
- Galaxy 请求需要 ByteCloud JWT，首次调用前先登录：`bytedcli auth login`
- 国际站（i18n-bd / i18n-tt）请求需要对应站点的 JWT

> 执行前缀见 `../../invocation.md`；下面示例直接写 `bytedcli`。

## Quick start

```bash
# 按 PSM 列出所有主机（默认分页：page 1, page-size 20）
bytedcli galaxy host list --psm "example.service.api"

# 指定 control plane（自动推断可省略）
bytedcli galaxy host list --psm "example.service.api" --control-plane i18n-bd

# 拉取全部主机（受 MAX_PAGES=50 安全上限保护）
bytedcli galaxy host list --psm "example.service.api" --all

# 自定义分页
bytedcli galaxy host list --psm "example.service.api" --page 2 --page-size 50

# 机器可读输出
bytedcli --json galaxy host list --psm "example.service.api"
```

## Notes

- `--json` 是全局参数，放在 `galaxy` 前面，例如 `bytedcli --json galaxy host list ...`
- `searchNodes`（PSM 反查）固定走 CN plane 的 `galaxy-api.bytedance.net`，不随 `--control-plane` 变化
- hosts search 按 control plane 路由到不同 host（cn / i18n-bd / i18n-tt）
- control plane 默认按 node path 自动推断：`path` 含 `i18n-bd` → i18n-bd，`i18n-tt` → i18n-tt，其余 → cn
- `--all` 模式会逐页拉取直到全部获取或达到 `MAX_PAGES = 50` 上限；超限时抛 `GALAXY_LIMIT_EXCEEDED` 错误并给出分页提示
- 分页默认值：`page=1`，`page-size=20`；`page-size` 会被 clamp 到 `[1, 100]`
- 鉴权依赖 ByteCloud JWT，请优先用 `auth login` 获取

## References

- `references/galaxy.md`
