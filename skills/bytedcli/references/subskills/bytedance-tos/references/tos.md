# TOS 命令参考

> 统一使用内部源 npx 调用：`references/invocation.md`

## 环境与站点

- `--site cn|boe|i18n|i18n-tt|eu-ttp`：切换 ByteCloud 站点（影响请求 host + `x-bcgw-vregion`）
- `tos list-sites`：从平台 meta API 拉取站点/VRegion 列表（best-effort，缓存 1 天）

## 命令

### 1) tos list-sites

列出 TOS 平台支持的站点与 VRegion（用于 UI/平台维度的环境发现）。

```bash
bytedcli tos list-sites
```

### 2) tos user-info

获取当前用户信息。

```bash
bytedcli tos user-info
```

### 3) tos list-starred-buckets

列出收藏（favorited）的 buckets。

```bash
bytedcli tos list-starred-buckets --page 1 --size 5
```

### 4) tos list-viewable-buckets

列出当前用户可访问的 buckets。

```bash
bytedcli tos list-viewable-buckets --page 1 --size 5
```

### 5) tos list-objects

列出指定 bucket 下的对象。

- `--bucket-id`：目标 bucket id
- `--prefix`：对象 key 前缀；不传则列 bucket 根范围
- `--limit`：本次最多返回对象数，默认 `100`
- `--last-key`：翻页游标；当上一页 `has_more=true` 时，传上一页最后一个对象 key

```bash
# 查看 bucket 根范围对象
bytedcli tos list-objects --bucket-id 123 --limit 100

# 按前缀过滤对象
bytedcli tos list-objects --bucket-id 123 --prefix datasets --limit 100

# 继续翻页
bytedcli tos list-objects --bucket-id 123 --prefix datasets --last-key datasets/sample.txt
```

### 6) tos get-object-url

获取指定对象的下载地址。

- `--bucket-id`：目标 bucket id
- `--key`：完整对象 key
- `--type`：地址类型，支持 `internal`、`office`、`both`，默认 `internal`

```bash
# 获取内网下载地址
bytedcli tos get-object-url --bucket-id 123 --key datasets/sample.txt

# 获取办公网下载地址
bytedcli tos get-object-url --bucket-id 123 --key datasets/sample.txt --type office

# 同时获取两种地址
bytedcli tos get-object-url --bucket-id 123 --key datasets/sample.txt --type both
```

### 7) tos delete-object

删除指定对象。

- `--bucket-id`：目标 bucket id
- `--key`：完整对象 key
- `--yes`：确认删除；必须传入

```bash
bytedcli tos delete-object --bucket-id 123 --key datasets/sample.txt --yes
```

### 8) tos upload-object

上传本地文件到指定 bucket。

- `--bucket-id`：目标 bucket id
- `--file`：本地文件路径
- `--key`：完整对象 key；不传时默认使用本地文件名
- `--prefix`：对象名前缀；仅在未传 `--key` 时使用

`--key` 与 `--prefix` 不能同时使用。同名对象会被覆盖。

```bash
# 上传到 bucket 根目录，object key 为 sample.txt
bytedcli tos upload-object --bucket-id 123 --file ./sample.txt

# 上传到指定 object key
bytedcli tos upload-object --bucket-id 123 --file ./sample.txt --key datasets/sample.txt

# 上传到指定前缀，object key 为 datasets/sample.txt
bytedcli tos upload-object --bucket-id 123 --file ./sample.txt --prefix datasets
```

### 9) tos list-records

列出用户记录：

- `--status apply`：我的申请记录（支持 `--record-type`，默认 1）
- `--status toaudit`：待我审批记录

```bash
bytedcli tos list-records --status apply --record-type 1 --page 1 --size 5
bytedcli tos list-records --status toaudit --page 1 --size 5
```
