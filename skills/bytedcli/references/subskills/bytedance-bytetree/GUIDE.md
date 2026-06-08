---
name: bytedance-bytetree
description: "Query ByteTree service tree nodes, node resources, child nodes, parent chains, and owner role members via bytedcli. Invoke when tasks mention 服务树、ByteTree、节点层级、服务归属、资源挂载、负责人、Owner 或父子链路查询."
---

# bytedcli ByteTree

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

- 搜索服务树节点
- 查询单个节点详情
- 查看某个节点下挂载的资源列表
- 查看某个节点的直接子节点
- 查询某个节点到根节点的父链
- 根据服务树节点定位服务、资源、PSM、文件夹或负责人信息
- 查看或调整 ByteTree 节点 IAM Owner 角色成员

## Quick start

```bash
# 模糊搜索
bytedcli bytetree search --query "demo-service-tree"

# 查看节点详情
bytedcli bytetree get --node-id 1234567

# 查看节点资源
bytedcli bytetree resources --node-id 1234567 --provider codebase

# 查看子节点
bytedcli bytetree children --node-id 1234567

# 查看父链
bytedcli bytetree parents --node-id 1234567

# 查看 Owner 角色成员
bytedcli bytetree owner list --node-id 1234567

# 预览 Owner 变更，默认只 dry-run
bytedcli bytetree owner add --node-id 1234567 --user demo.user
```

## Notes

- ByteTree API 通过 `x-jwt-token` 鉴权，bytedcli 会自动复用当前登录态获取 token，无需手动复制浏览器请求头。
- 服务树本身是全球一棵树；`--site` 主要影响走哪个控制面鉴权与入口 host。省略时默认走 `cn`。
- 使用全局 `--site` 切换控制面 host，例如 `--site cn`、`--site i18n-bd`、`--site i18n-tt`。
- `resources` 走 `/nodes/{id}/resources_v2`，支持 `--provider`、`--offset`、`--page-size`；适合查询 `codebase` 这类不会直接出现在 `children/get` 返回里的挂载资源。
- `children` 默认查询 `service,resource,psm,employee,top-node,folder` 六类子节点；可以通过重复传 `--type` 或逗号分隔值缩小范围。
- `get` 返回值里的 `resources` 很关键，常见字段包括 `provider`、`resource_type`、`resource_id`、`partition`、`env`、`region`、`link.view`，可用于提前判断节点背后挂了哪些 TCE/TCC/RDS 等资源。
- `owner list/add/delete/set` 走 IAM Owner 角色接口。默认角色按站点推断：`cn/boe/eu-ttp` 用 `owner`，`i18n-tt/i18n-bd` 用 `owner.i18n`，`us-ttp` 用 `owner.tx`。
- `owner add/delete/set` 默认本地 dry-run；确认 JSON payload 后再加 `--yes` 执行真实变更。`set` 只替换当前 `--user-type`，并保留另一类账号成员。
- IAM 写接口如果返回 403，通常需要走 ByteCloud IAM UI 或申请 `/api/v2/acl/node/role` allowlist。

## References

- [bytetree.md](./references/bytetree.md)
- [invocation.md](./../../invocation.md)
- [troubleshooting.md](./../../troubleshooting.md)
