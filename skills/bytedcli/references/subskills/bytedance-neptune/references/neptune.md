# Neptune

Neptune 用于服务治理配置查询（安全/稳定性/限流/调度）和严格授权申请，支持跨站点排查差异。

## 环境与站点

Use global `--site` to select the ByteCloud deployment. Per-service `--neptune-site` is a hidden alias for backward compatibility.

- CN: `--site cn`（默认）
- BOE: `--site boe`
- ByteIntl: `--site byteintl`
- TikTok ROW: `--site i18n-tt`（aliases: `i18ntt|row|tiktok|tiktok-row`；`sg` 保持兼容 ByteIntl）

站点差异（bytedcli 内部已处理）：

- CN/BOE：API host 在 ByteCloud 控制台域名下（`cloud.bytedance.net` / `cloud-boe.bytedance.net`），请求需要 `x-bcgw-tenant-id: bytedance`
- ByteIntl：API host 为 `cloud.byteintl.net`，请求不需要 `x-bcgw-tenant-id`
- TikTok ROW：API host 为 `cloud.tiktok-row.net`，请求不需要 `x-bcgw-tenant-id`

## 站点/VRegion 自动发现（best-effort）

命令：`neptune list-sites`

bytedcli 会调用平台 meta 接口 `list_platform_vregions?platform=neptune`（并缓存 1 天）来尽量列出支持的站点与 VRegion。

## zones/vregions 列表（best-effort）

命令：`bytedcli --site <site> neptune list-cp-regions`

用于查询 Neptune 当前站点支持的 `zones` 与 `vregions` 列表，便于为后续配置查询选择正确的 `--zone`。

## 命令映射

- `neptune security`：安全配置（支持 `--method`，默认 `*`；支持 `--direction ingress|egress`）
- `neptune stability`：稳定性配置（支持 `--direction ingress|egress`）
- `neptune rate-limit`：限流配置（支持 `--direction ingress|egress`；`--v2` 仅对 ingress 生效）
- `neptune dispatch`：调度配置（支持 `--direction ingress|egress`）
- `neptune acl-status`：zone 级 ACL 总开关查询（`acl/get_status`），返回 `mode: offline|online`
- `neptune strict-auth-status`：服务级严格授权状态（`acl/strict_authorization/status`），返回 `mode: deleted|open`
- `neptune acl-check`：综合 ACL 三层（acl-status + strict-auth-status + security/ingress），按 caller/callee/zone 输出 `verdict: allow|allow-explicit|deny-strict|deny-explicit|indeterminate`。判断"能不能调通"优先用这个，单跑 `neptune security` 容易误判（exist=false 在 ACL 未启用的 zone 不等于"被拒"）
- `neptune strict-auth apply`：严格授权申请（支持结构化参数，或 `--payload-json` / `--payload-file` 传完整 payload）

## 严格授权申请

命令：`bytedcli --site <site> neptune strict-auth apply`

结构化参数模式：

```bash
bytedcli --site i18n-tt neptune strict-auth apply \
  --caller-psm example.caller.service \
  --caller-cluster default \
  --callee-psm example.callee.service \
  --callee-cluster default \
  --method GetProductByID \
  --method MGetProductsByIds \
  --zone SGALI \
  --reason "Need access for demo workflow"
```

完整 payload 模式：

```bash
bytedcli --site i18n-tt neptune strict-auth apply --payload-file ./sample-neptune-strict-auth.json
```

- `--method` 和 `--zone` 支持重复传入或逗号分隔。
- 未传 `--zone` 时默认：`CN(cn/byteintl)`、`BOE(boe)`、`SG(i18n-tt)`。
- 平台新增字段尚未映射为 CLI flag 时，优先使用 `--payload-json` 或 `--payload-file`；raw payload 模式不要混用结构化 flags。
