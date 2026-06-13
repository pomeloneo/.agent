# FTF Diff Attribution

## 接口来源

FTF 平台的 Diff 归因模块，域名 `assert.byted.org`，用于查询回放 diff 中的相似 Case 分析。

## 认证

使用 ByteCloud JWT（`X-Jwt-Token` header），与 Tesla RM token 无关。

## 命令

### ftf diff-attribution inbound-similar-case list

列出指定 record 下的相似用例。caseId 自动从 recordId 的第一个 `_` 之前截取。

```bash
bytedcli ftf diff-attribution inbound-similar-case list \
  --record-id "<record-id>" \
  --similar-case-id "<similar-case-id>" \
  --diff-id "<diff-id>"

# JSON 输出
bytedcli --json ftf diff-attribution inbound-similar-case list \
  --record-id "<record-id>" \
  --similar-case-id "<similar-case-id>" \
  --diff-id "<diff-id>"

# 获取全部页
bytedcli ftf diff-attribution inbound-similar-case list \
  --record-id "<record-id>" \
  --similar-case-id "<similar-case-id>" \
  --diff-id "<diff-id>" \
  --all
```

### 参数

| 参数 | 必填 | 说明 |
|------|------|------|
| `--record-id` | 是 | Record ID（复合查询参数，caseId 自动提取） |
| `--similar-case-id` | 是 | 相似 Case 标识 |
| `--diff-id` | 是 | Diff 标识 |
| `--page` | 否 | 页码，默认 1 |
| `--page-size` | 否 | 每页条数，默认 20 |
| `--all` | 否 | 自动翻页获取全部结果 |
| `--business` | 否 | 业务标识 |

### ftf diff-attribution inbound-diff-detail get

获取指定 record 的 inbound diff 详情（字段路径 + 标注状态）。caseId 自动从 recordId 的第一个 `_` 之前截取。

```bash
bytedcli ftf diff-attribution inbound-diff-detail get \
  --record-id "<record-id>"

# 指定业务
bytedcli ftf diff-attribution inbound-diff-detail get \
  --record-id "<record-id>" \
  --business <business>
```

### 参数

| 参数 | 必填 | 说明 |
|------|------|------|
| `--record-id` | 是 | Record ID（复合查询参数，caseId 自动提取） |
| `--business` | 否 | 业务标识 |

## API

- `GET https://assert.byted.org/nova/valuediff/getSimilarCaseList/{caseId}?recordId=xxx&similarCaseId=xxx&diffId=xxx&page=1&size=20`
- `GET https://assert.byted.org/nova/valuediff/getValueDiffDetail/{caseId}?recordId=xxx&business=xxx`
