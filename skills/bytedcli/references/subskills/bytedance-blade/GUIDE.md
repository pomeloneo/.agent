---
name: bytedance-blade
description: "Inspect Blade data sync tasks via bytedcli: get task detail from a task ID, confirm source/target database and table metadata, read project/owner/region fields, and troubleshoot Blade auth on `blade.byteintl.net`. Use when tasks mention Blade, `blade.byteintl.net`, data sync tasks, `des_controller`, or Blade console URLs such as `/resource/task/<id>`."
---

# bytedcli Blade

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

- 按 Blade task ID 查看单个 data sync task 详情
- 从 `blade.byteintl.net/resource/task/<id>` 页面回到 CLI
- 确认任务的 `projectId`、owner、源/目标 region、db/table
- 排查 Blade 鉴权问题，尤其是 JWT 过期或页面态未准备好的场景

## 前置条件

- 使用通用调用方式：`../../invocation.md`
- Blade 国际站默认建议显式带 `--site i18n-bd`
- 如果已经知道任务在 `mycis`，可直接传 `--region mycis`，CLI 会自动映射到 `i18n-bd` 鉴权站点
- 认证优先使用 fresh `ByteCloud JWT`
- 如果需要先准备浏览器态，再执行：`bytedcli --site i18n-bd auth login --session`
- 如果只想走纯 JWT 模式，也可以直接提供 `BYTEDCLI_USER_CLOUD_JWT`

> 执行前缀见 `../../invocation.md`；下面示例直接写 `bytedcli`。

## Quick start

当前只接入 `blade task get`。

```bash
# 直接按 task ID 查询
bytedcli --site i18n-bd blade task get --id sample-task-id

# 直接用 region 推导站点
bytedcli blade task get --region mycis --id sample-task-id

# 需要完整 raw payload 时用 --json
bytedcli --site i18n-bd --json blade task get --id sample-task-id

# 纯 JWT 模式：显式提供 fresh ByteCloud JWT
BYTEDCLI_USER_CLOUD_JWT="sample-fresh-jwt" \
  bytedcli --site i18n-bd blade task get --id sample-task-id
```

如果用户给的是 Blade 控制台 URL：

```text
https://blade.byteintl.net/resource/task/sample-task-id
```

则直接把路径里的 task ID 提出来即可：

```bash
bytedcli blade task get --region mycis --id sample-task-id
```

## Authentication

- CLI 会优先复用 `blade.byteintl.net` 的站点 cookie，并同时携带 fresh `X-Jwt-Token`
- `--region mycis` 会自动选择 `i18n-bd` 站点来换取 JWT 与 Titan cookie
- 如果站点 cookie 不可用，会继续尝试 Titan Passport cookie
- 即使 cookie 都不可用，只要当前 `ByteCloud JWT` 仍然有效，也允许直接请求 Blade 详情接口
- 运行期已验证：**过期 JWT** 常见返回是 `code=82000`，并带 `redirect_url=/auth/api/v1/jwt`
- 因此 Blade 鉴权排查时，先看 JWT 是否 fresh，再看站点页面态是否准备好

推荐顺序：

```bash
# 推荐：先刷新页面态 + JWT
bytedcli --site i18n-bd auth login --session

# 然后查询任务
bytedcli --site i18n-bd blade task get --id sample-task-id

# 或者直接按 region 查询
bytedcli blade task get --region mycis --id sample-task-id
```

如果调用方只维护自动化 JWT，不希望依赖 session：

```bash
export BYTEDCLI_USER_CLOUD_JWT="sample-fresh-jwt"
bytedcli --site i18n-bd blade task get --id sample-task-id
```

## Output

文本模式会优先展示这些归一化字段：

- `Task ID`
- `Name`
- `Status`
- `Type`
- `Project ID`
- `Owner`
- `Region`
- `Source Region` / `Target Region`
- `Source DB` / `Source Table`
- `Target DB` / `Target Table`
- `Console URL`

`--json` 模式会保留完整 `raw` payload，适合继续补字段映射或和浏览器抓包对齐。

## Notes

- `--json` 是全局参数，必须放在 `blade` 前面，例如 `bytedcli --site i18n-bd --json blade task get --id sample-task-id`
- 当前内置 region 只支持 `mycis`
- 当前只支持**只读详情查询**，还没有 list / search / create / update 能力
- parser 会优先读取 `data_sync_task.basic_info`、`source_resource_config`、`target_resource_config` 的显式路径字段；如果某个任务仍有字段遗漏，优先查看 `raw`

## References

- `references/blade.md`
- `../../invocation.md`
