# Blade Notes

## Command surface

当前 Blade domain 只开放一个命令：

```bash
bytedcli --site i18n-bd blade task get --id sample-task-id
bytedcli blade task get --region mycis --id sample-task-id
```

## URL to CLI

Blade 控制台 URL 常见格式：

```text
https://blade.byteintl.net/resource/task/<taskId>
```

CLI 参数映射：

- 路径里的 `<taskId>` 对应 `--id`
- `mycis` 任务可直接传 `--region mycis`，CLI 会自动映射到 `i18n-bd`
- 未显式传 region 时，站点默认建议 `--site i18n-bd`

示例：

```bash
bytedcli --site i18n-bd blade task get --id sample-task-id
bytedcli blade task get --region mycis --id sample-task-id
```

## Authentication summary

- 首选：`blade.byteintl.net` 站点 cookie + fresh `ByteCloud JWT`
- `--region mycis` 会自动走 `i18n-bd` 站点获取鉴权材料
- 兜底：Titan Passport cookie
- 无 cookie 也可直接请求，但前提是 JWT 仍然有效
- 如果命中 `code=82000` 或 `redirect_url=/auth/api/v1/jwt`，优先判断 JWT 是否过期

建议命令：

```bash
bytedcli --site i18n-bd auth login --session
bytedcli --site i18n-bd blade task get --id sample-task-id
```

纯 JWT 自动化场景：

```bash
export BYTEDCLI_USER_CLOUD_JWT="sample-fresh-jwt"
bytedcli --site i18n-bd blade task get --id sample-task-id
```

## Normalized fields

文本模式和 JSON 顶层会优先暴露这些字段：

- `id`
- `name`
- `status`
- `taskType`
- `projectId`
- `owner`
- `region`
- `sourceRegion`
- `targetRegion`
- `sourceDb`
- `sourceTable`
- `targetDb`
- `targetTable`
- `consoleUrl`

完整后端响应仍保留在 `raw` 里。
