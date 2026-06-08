---
name: bytedance-luban
description: "Operate Luban package and PyPI artifact workflows via bytedcli. Use when tasks mention Luban, Bytedance or TTP npm packages, bnpm package lookup, checking whether a package exists in Luban, searching package versions by prefix, or querying/publishing Luban PyPI artifact versions."
---

# bytedcli Luban

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

- 需要确认某个 Bytedance 或 TTP npm 包是否已进入 Luban 组件库
- 需要在默认 Bytedance Luban 与 TTP Luban 两个环境之间切换查询
- 需要按版本前缀筛某个 bnpm 包的记录
- 需要把 `@scope/name` 形式的包名映射成 Luban 查询里的 `group` 和 `name`
- 需要查询 Luban PyPI 制品仓库、版本列表/详情，或创建 PyPI 版本发布任务

## Prerequisites

- 先完成 `bytedcli auth login`
- 默认 Luban 查询会复用 ByteCloud 凭据，并从 `https://cloud.bytedance.net` 获取 JWT
- 传入 `--site us-ttp` 时，会改为查询 TTP Luban，并从 `https://cloud-ttp-us.bytedance.net` 获取 TTP JWT
- `luban pypi` 走 Luban 发布页使用的 SCM API，并返回 Luban 版本页 URL

## Quick start

```bash
# 查询某个包的全部结果
bytedcli luban search --npm @demo/uploader

# 查询某个包的指定版本前缀
bytedcli luban search --npm @demo/uploader -v 2.1.5

# 查询 TTP Luban 环境
bytedcli luban search --npm @demo/uploader --site us-ttp

# 使用长参数写法
bytedcli luban search --npm @demo/uploader --package-version 2.1.5

# 需要机器可读结果时，优先加 --json
bytedcli --json luban search --npm @demo/uploader --package-version 2.1.5

# 查询 PyPI 制品仓库和版本
bytedcli --json luban pypi repo get --repo-id 12345
bytedcli --json luban pypi version list --repo-id 12345 --page 1 --page-size 20
bytedcli --json luban pypi version get --version-id 67890

# PyPI 发布 dry-run；会先检查同版本是否已有成功发布
bytedcli --json luban pypi version publish \
  --repo-id 12345 \
  --version 1.2.3 \
  --desc "demo release" \
  --branch master \
  --dry-run

# 基于指定 commit 发布，真实写操作必须显式确认
bytedcli --json luban pypi version publish \
  --repo-id 12345 \
  --version 1.2.3 \
  --desc "demo release" \
  --pub-base commit \
  --commit abcdef123 \
  --yes
```

## Notes

- `--npm` 需要传 `@scope/name` 形式的 npm 包名。
- `-v, --package-version` 为可选参数；传入时会映射为 Luban 请求体里的 `version_prefix`，不传时会搜索该包名的所有结果。
- `--site us-ttp` 为可选参数；传入时切换到 TTP Luban API、JWT host 与 Web origin，不传时走默认 Bytedance Luban 环境。
- 文本模式优先输出紧凑表格；`--json` 模式返回原始 API 响应。
- `luban pypi version publish` 默认等待并轮询发布状态；`--no-wait` 只创建任务并返回。
- `--dry-run` 与真实发布都会先检查同版本成功记录；命中时返回 `LUBAN_PYPI_VERSION_EXISTS`，不会继续创建发布任务。
- `--pub-base` 支持 `branch` / `commit` / `tag`；commit 模式必须传 `--commit`，tag 模式必须传 `--tag`。
- 轮询时会在 stderr 打印 URL 与进度动画，不影响 `--json` stdout 的最终 JSON。
