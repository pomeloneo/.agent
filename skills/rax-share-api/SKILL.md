---
name: rax-share-api
version: 1.0.0
description: "RAX 分享接口（/share/create + /share/find_v2）。脚本/AI/MCP 工具读写分享数据的标准做法。涵盖 7 种已知 type（network_capture / browser_mock / event / router / bridge_call / bridge_event / ab_experiment），也支持任意自定义 type。Use when 用户想：保存/读取一个分享、把抓包/埋点/路由/Bridge 日志/AB 实验等数据存到 rax-api 后端、解出 share 链接里的数据、用脚本批量拉分享、给 MCP 工具加一个分享读写能力。"
metadata:
  endpoints:
    create: "POST /share/create"
    findV2: "POST /share/find_v2"
  baseUrl: "https://rax.tiktok-row.net/api"
---

# RAX Share API Skill

通过 rax-api 后端读写分享数据。**读用 `/share/find_v2`（gzip-base64 envelope，无 AES），写用 `/share/create`（普通 JSON）。**

## 何时用这个 skill

- 用户给一个 `shareId`，让你"拉一下分享的内容"
- 用户想把一段 JSON 数据（抓包记录、埋点参数、路由日志、Bridge 调用、AB 配置等）保存为一个可分享的链接
- 你在写 MCP 工具 / Python 脚本 / Bash 自动化，需要批量读写分享
- 用户提到 "share", "分享", "shareId", "share link"

## 基础信息

| 项 | 值 |
|---|---|
| Base URL | `https://rax.tiktok-row.net/api` |
| Auth | **无需** —— 全靠 shareId 的 UUID 熵防枚举 |
| 创建接口 | `POST /share/create` |
| 读取接口 | `POST /share/find_v2` |
| Content-Type | 都是 `application/json` |

## 已知 type 一览

| type | 含义 | 存储位置 |
|---|---|---|
| `network_capture` | 网络抓包记录 | 专表 `network_capture_share` |
| `browser_mock` | 浏览器扩展 mock 配置 | 专表 `rax_browser_mock_share` |
| `event` | 埋点事件 | 通用表 `rax_share` |
| `router` | 路由日志 | 通用表 `rax_share` |
| `bridge_call` | Bridge 调用日志 | 通用表 `rax_share` |
| `bridge_event` | Bridge 事件 | 通用表 `rax_share` |
| `ab_experiment` | AB 实验配置 | 通用表 `rax_share` |
| **任意 `[A-Za-z0-9_]{1,64}`** | 自定义 | 通用表 `rax_share` |

**任意非专表 type 都直接读写 `rax_share` 通用表**，不需要后端注册——你 `/share/create` 存什么 type，`/share/find_v2` 就能用什么 type 拉回来。

---

## 写：`POST /share/create`

### 请求

```jsonc
{
  "type": "event",                  // 必填：类型字符串，[A-Za-z0-9_]{1,64}
  "userId": "scieszka@bytedance.com", // 必填：作者，用于审计
  "data": { /* 任意 JSON */ }       // 必填：要分享的内容
}
```

### 响应

```jsonc
// 成功
{ "code": 0, "data": { "shareId": "new_abcdef1234567890..." } }

// 失败（缺字段、DB 错误）
{ "code": 403, "msg": "Missing required fields: type, userId, data" }
```

### curl 示例

**最小化**：

```bash
curl -sX POST 'https://rax.tiktok-row.net/api/share/create' \
  -H 'Content-Type: application/json' \
  -d '{
    "type": "event",
    "userId": "scieszka@bytedance.com",
    "data": { "eventName": "page_view", "params": { "page": "/home" } }
  }'
```

输出：
```json
{"code":0,"data":{"shareId":"new_2a8e3f1c45d24b9e9a7f8e6d5c4b3a2f"}}
```

**自定义 type**（不需要任何后端注册）：

```bash
curl -sX POST 'https://rax.tiktok-row.net/api/share/create' \
  -H 'Content-Type: application/json' \
  -d '{
    "type": "perf_trace",
    "userId": "you@bytedance.com",
    "data": { "fps": 58, "stage": "first_screen", "duration_ms": 320 }
  }'
```

**用 `jq` 提取 shareId**：

```bash
SHARE_ID=$(curl -sX POST 'https://rax.tiktok-row.net/api/share/create' \
  -H 'Content-Type: application/json' \
  -d '{"type":"router","userId":"you@x.com","data":{"from":"/a","to":"/b"}}' \
  | jq -r '.data.shareId')

echo "$SHARE_ID"
```

---

## 读：`POST /share/find_v2`

### 请求

```jsonc
{
  "shareId": "new_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",  // 必填
  "type":    "event"                                   // 必填，需跟 create 时一致
}
```

### 响应（**gzip-base64 envelope**，永远是这个结构）

```jsonc
{
  "compressed": true,
  "encoding":   "gzip-base64",
  "payload":    "<base64( gzip( JSON.stringify(InnerPayload) ) )>"
}
```

`InnerPayload` 解出来后：

```jsonc
// 成功
{
  "code": 200,
  "data": {
    "shareId":    "new_xxx",
    "type":       "event",                  // 跟请求 type 一致
    "userId":     "scieszka@bytedance.com", // 创建者
    "createdAt":  "2026-05-15T10:00:00Z",   // 只有 rax_share 通用表才有
    "sharedItem": { /* create 时塞的 data */ }
  }
}

// 错误
{ "code": 301, "errmsg": "missing shareId or type" }
{ "code": 400, "errmsg": "invalid type: xxx (must be [A-Za-z0-9_]{1,64})" }
{ "code": 404, "errmsg": "share not found" }
{ "code": 403, "errmsg": "db error: ..." }
```

### curl 示例 — 完整读取流程

**Python 3（推荐，单行）**：

```bash
SHARE_ID="new_2a8e3f1c45d24b9e9a7f8e6d5c4b3a2f"
TYPE="event"

curl -sX POST 'https://rax.tiktok-row.net/api/share/find_v2' \
  -H 'Content-Type: application/json' \
  -d "{\"shareId\":\"$SHARE_ID\",\"type\":\"$TYPE\"}" \
| python3 -c "
import sys, json, base64, gzip
env = json.load(sys.stdin)
print(json.dumps(json.loads(gzip.decompress(base64.b64decode(env['payload']))), indent=2, ensure_ascii=False))
"
```

**Node.js**（用 zlib，无第三方依赖）：

```bash
curl -sX POST 'https://rax.tiktok-row.net/api/share/find_v2' \
  -H 'Content-Type: application/json' \
  -d '{"shareId":"new_xxx","type":"event"}' \
| node -e "
const zlib=require('zlib');
let s='';process.stdin.on('data',c=>s+=c).on('end',()=>{
  const env=JSON.parse(s);
  const buf=Buffer.from(env.payload,'base64');
  console.log(zlib.gunzipSync(buf).toString('utf8'));
});
"
```

**纯 shell（jq + base64 + gunzip）**：

```bash
curl -sX POST 'https://rax.tiktok-row.net/api/share/find_v2' \
  -H 'Content-Type: application/json' \
  -d '{"shareId":"new_xxx","type":"router"}' \
| jq -r '.payload' \
| base64 -d \
| gunzip \
| jq .
```

### 抽出常用字段

```bash
read_share() {
  local id="$1" type="$2"
  curl -sX POST 'https://rax.tiktok-row.net/api/share/find_v2' \
    -H 'Content-Type: application/json' \
    -d "{\"shareId\":\"$id\",\"type\":\"$type\"}" \
  | jq -r '.payload' | base64 -d | gunzip
}

# 拿整个 sharedItem
read_share new_xxx event | jq '.data.sharedItem'

# 只看创建者
read_share new_xxx event | jq -r '.data.userId'

# 看 code（成功 / 失败）
read_share new_xxx event | jq -r '.code'
```

---

## 关键坑点

### 1. type 必须跟 create 时**完全一致**

```bash
# 创建时存 type=event
curl ... -d '{"type":"event", "userId":"x", "data":{...}}'
# 读的时候用 type=router → 404！
curl ... -d '{"shareId":"new_xxx", "type":"router"}'
```

`/share/find_v2` 在通用表查询时会用 `{shareId, type}` **双重匹配**。type 错了就返回 404，不会"自动猜"原始 type。

要"按 shareId 找 type 自己解决"，用老接口 `POST /share/find`（返回明文 `{code, data: {shareId, type, userId, data, createdAt}}`，但响应大时不压缩）。

### 2. 响应永远是 envelope，**错误也包**

哪怕 400/404/403 错误，外层也是：

```jsonc
{ "compressed": true, "encoding": "gzip-base64", "payload": "<base64 gzip JSON>" }
```

不要光看 HTTP status 200 就以为成功，必须解出 envelope 看 inner `code`。

### 3. 专表 type 没有 `createdAt`

`network_capture` / `browser_mock` 走专表，专表 schema 里没有 `createdAt`。返回的 `data` 里这个字段就是 undefined。`event`/`router`/...等通用表有。

### 4. shareId 是 capability —— 等于密码

接口**无鉴权**，任何人拿到 shareId 都能读。所以：
- 别把 shareId 放进 git / 公开日志 / URL query 里被爬
- 创建时 userId 仅做审计，不影响访问控制

### 5. 自定义 type 没有"注册"步骤

直接 `/share/create` 时用即可。后续 `/share/find_v2` 传同样的 type 就能查回。**不要**找开发同学加白名单。

---

## 通过 rax-cli 调用（封装好的，最省事）

如果你装了 rax-cli，等价的命令：

```bash
# 创建 — 暂未有 'share put' 简写命令，用 rax-cli 还得走老接口或自己 curl
# 读 — 一行搞定，自动解 envelope
rax share get new_xxxxxxxx event
rax share get new_xxxxxxxx network_capture
```

或者通过 daemon RPC（MCP / 脚本走这个）：

```bash
curl -sX POST 'http://127.0.0.1:2680/rpc' \
  -H 'Content-Type: application/json' \
  -d '{"method":"share_fetch_v2","args":{"shareId":"new_xxx","type":"event"}}'
```

---

## 完整 end-to-end demo

```bash
#!/usr/bin/env bash
set -euo pipefail

BASE='https://rax.tiktok-row.net/api'
USER='scieszka@bytedance.com'

# 1. 创建一个自定义 type 的分享
SHARE_ID=$(curl -sX POST "$BASE/share/create" \
  -H 'Content-Type: application/json' \
  -d "{
    \"type\": \"perf_trace\",
    \"userId\": \"$USER\",
    \"data\": { \"fps\": 58, \"stage\": \"first_screen\", \"duration_ms\": 320 }
  }" | jq -r '.data.shareId')

echo "created shareId: $SHARE_ID"

# 2. 立刻读回来
echo "--- read back ---"
curl -sX POST "$BASE/share/find_v2" \
  -H 'Content-Type: application/json' \
  -d "{\"shareId\":\"$SHARE_ID\",\"type\":\"perf_trace\"}" \
| jq -r '.payload' | base64 -d | gunzip | jq .

# 输出:
# {
#   "code": 200,
#   "data": {
#     "type": "perf_trace",
#     "shareId": "new_xxx",
#     "userId": "scieszka@bytedance.com",
#     "createdAt": "2026-05-15T...",
#     "sharedItem": { "fps": 58, "stage": "first_screen", "duration_ms": 320 }
  # }
# }
```

---

## 排错

| 现象 | 原因 | 解决 |
|---|---|---|
| `code: 301 missing shareId or type` | 请求体缺字段 | 检查 JSON 拼写 |
| `code: 400 invalid type` | type 含特殊字符或太长 | 只用 `[A-Za-z0-9_]`，不超过 64 字符 |
| `code: 404 share not found` | shareId 不存在，**或 type 跟 create 时不一致** | 用老 `/share/find` 看实际 type，或重新拉 type 列表 |
| `code: 403 db error` | 后端 DB 异常 | 联系后端 / 看 rax-api 日志 |
| `curl: (52) Empty reply` | 服务端没响应 / 网络问题 | 重试，检查公司 VPN |
| `gunzip: invalid magic` | envelope.payload 不是合法 base64 / 已经被解过一次 | 不要重复 decode |

---

## 设计依据（深入了解）

- **为什么用 envelope 而不是 HTTP `Content-Encoding: gzip`**：复用现有 `CompressedPayloadEnvelope` 工具，前后端共享 `compressPayload`/`decompressPayload`。代价是消费方多 3 行解包逻辑，收益是错误响应也走同样格式，调用方解码代码无分支。
- **为什么是 base64 不是裸 binary**：保持 `application/json` Content-Type，HTTP 中间设备（CDN/proxy）不会动 payload。
- **为什么没有 AES**：老接口 `/share/find` 出于历史原因响应数据加密，rax-app 桌面端有 CryptoJS 解。但 AI/MCP/Python 脚本难复用，所以这个 v2 接口去掉了 AES，只保留压缩。
- **为什么 type 要双重校验**：避免调用方传错 type 时拿到不匹配的数据。

源码位置：
- 后端实现：`apps/rax-api/controller/share.ts`、`apps/rax-api/service/unified-share.ts`
- SDK helper：`packages/rax-device-sdk/src/share/fetch-share.ts`（导出 `fetchShareById`）
- CLI 命令：`apps/rax-cli/src/handlers/share.ts`（`handleFetchShareV2`）
- spec / plan：`docs/superpowers/specs/2026-05-14-share-find-v2-design.md`
