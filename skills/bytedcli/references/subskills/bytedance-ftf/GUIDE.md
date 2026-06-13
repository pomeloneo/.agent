---
name: bytedance-ftf
description: "FTF 流量回放平台 — diff 归因分析（相似 case 列表、inbound diff 详情）"
---

# bytedcli ftf

FTF（流量回放）平台的 CLI 接入，当前覆盖 diff 归因分析能力。

## When to use

- 用户提到"流量回放"、"FTF"、"diff 归因"
- 需要查询回放产生的相似 case、inbound diff 详情

## Quick start

```bash
# 列出相似 case（caseId 自动从 recordId 中提取）
bytedcli ftf diff-attribution inbound-similar-case list \
  --record-id "<record-id>" \
  --similar-case-id "<similar-case-id>" \
  --diff-id "<diff-id>"

# 获取 inbound diff 详情
bytedcli ftf diff-attribution inbound-diff-detail get \
  --record-id "<record-id>"
```

## 子命令组

| 命令 | 说明 |
|------|------|
| `ftf diff-attribution inbound-similar-case list` | 列出相似 case |
| `ftf diff-attribution inbound-diff-detail get` | 获取 inbound diff 详情 |

## 认证

使用 ByteCloud JWT（`X-Jwt-Token` header）。首次使用需 `bytedcli auth login`。
