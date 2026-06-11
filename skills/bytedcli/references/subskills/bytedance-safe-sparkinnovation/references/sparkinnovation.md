# SparkInnovation 命令面

## 发现类命令

```bash
# 业务线
bytedcli safe sparkinnovation biz-line list

# 部门
bytedcli safe sparkinnovation department list --biz-line 123

# 枚举值
bytedcli safe sparkinnovation change enum get
```

推荐顺序：

1. 不知道业务线时先 `biz-line list`
2. 不知道枚举值时先 `change enum get`
3. 创建小改变时，如果没有现成 `meego_link`，且用户没有明确要求跳过 Meego，先执行 `meego create`
4. 再执行 `list`、`create`、`update`、`claim`

## 查询类命令

```bash
# 列表
bytedcli safe sparkinnovation change list --biz-line 123

# 按标题和枚举条件过滤
bytedcli safe sparkinnovation change list \
  --biz-line 123 \
  --title "优化" \
  --category "技术优化" \
  --status "进行中" \
  --priority P2 \
  --start 2026-01-01T00:00:00Z \
  --end 2026-01-31T00:00:00Z

# 详情
bytedcli safe sparkinnovation change get --id 9527 --biz-line 123
```

说明：

- `change get` 必须同时提供 `--id` 和 `--biz-line`
- `change get` 会把后端富文本字段自动转换成 Markdown 展示
- `change list` 支持按 `title` 做标题文本过滤
- `change list` 支持的枚举过滤字段有 `category`、`status`、`priority`
- `change list` 的时间筛选 flag 使用 `--start` / `--end`，值仍然传 ISO 时间戳
- `change list` 默认按 `--page 1 --page-size 20` 分页
- 按 ID 获取单个 change 时用 `change get`，不要把 `change list` 当详情入口
- 文档中的 `123` 是占位值；真实 `biz_line` 先通过 `bytedcli safe sparkinnovation biz-line list` 获取

## 创建类命令

```bash
# 先创建 Meego，拿到返回的 URL
bytedcli meego create \
  --space demo-project \
  --title "优化 xxx 流程" \
  --description "这里可以复用小改变描述"

# 再创建小改变，并绑定 meego_link
bytedcli safe sparkinnovation change create \
  --title "优化 xxx 流程" \
  --category "技术优化" \
  --description "## 背景\n\n这里写 Markdown" \
  --solution "- step 1\n- step 2" \
  --cost-benefit "- 节省人力" \
  --meego-link "https://meego.larkoffice.com/demo-project/story/detail/123456" \
  --priority P2 \
  --biz-line 123
```

说明：

- `title`、`category`、`priority`、`biz_line` 是必填
- `description`、`solution`、`cost-benefit` 传 Markdown 即可
- CLI 会自动把 Markdown 转成后端需要的富文本 JSON
- 默认策略：创建小改变时，如果用户没给 `meego_link`，且没有明确要求跳过 Meego，先创建 Meego 再绑定
- 创建 Meego 优先复用现有 `bytedance-meego` skill 和 `bytedcli meego create`，不要在 `sparkinnovation` skill 里重复定义 Meego 创建协议
- 如果用户已经提供 `meego_link`，直接透传给 `change create`
- 如果用户明确说“不需要创建 Meego”或“不绑定 Meego”，则不要自动创建

## 自动创建 Meego 并绑定

推荐方案：

1. 先复用现有 `bytedance-meego` skill 的创建路径
2. 默认调用高层命令 `bytedcli meego create`，而不是直接拼更底层的 `meego workitem create`
3. 从返回结果中取 `url`
4. 把该 `url` 作为 `--meego-link` 传给 `safe sparkinnovation change create`

原因：

- `bytedance-meego` 已经封装了 Meego 登录、快速创建需求和结果输出
- `bytedcli meego create` 是面向“快速建需求”的高层命令，最适合这里的自动编排
- 这样 `sparkinnovation` skill 只负责工作流编排，不需要复制 Meego 领域细节

默认编排规则：

```text
if user provides meego_link:
  use it directly
else if user explicitly says no meego:
  create sparkinnovation change without meego_link
else:
  create meego from change title/description
  bind returned meego url to change create --meego-link
```

推荐命令序列：

```bash
# Step 1: 创建 Meego
bytedcli meego create \
  --space demo-project \
  --title "优化审核流转效率" \
  --description "当前人工流转成本较高，计划通过自动分发和流程收敛提升效率"

# Step 2: 创建小改变并绑定 Meego URL
bytedcli safe sparkinnovation change create \
  --title "优化审核流转效率" \
  --category "流程工具" \
  --description "## 背景\n\n当前人工流转成本较高" \
  --meego-link "https://meego.larkoffice.com/demo-project/story/detail/123456" \
  --priority P2 \
  --biz-line 123
```

参数映射建议：

- Meego `title`：优先复用小改变 `title`
- Meego `description`：优先复用小改变 `description` 的 Markdown 文本；必要时可以做轻量摘要，但不要编造额外背景
- 小改变 `meego_link`：使用 `meego create` 返回的 `url`

## 从创建到完成的完整流程

```bash
# 1. 创建 Meego
bytedcli meego create \
  --space demo-project \
  --title "优化审核流转效率" \
  --description "当前人工流转成本较高，计划通过自动分发和流程收敛提升效率"

# 2. 创建小改变：新建后通常是“待认领”
bytedcli safe sparkinnovation change create \
  --title "优化审核流转效率" \
  --category "流程工具" \
  --description "## 背景\n\n当前人工流转成本较高" \
  --meego-link "https://meego.larkoffice.com/demo-project/story/detail/123456" \
  --priority P2 \
  --biz-line 123

# 3. 认领：owner 变成当前用户，状态变成“进行中”
bytedcli safe sparkinnovation change claim --id 9527 --biz-line 123

# 4. 更新并完成：完成前必须保证 cost-benefit 非空
bytedcli safe sparkinnovation change update \
  --id 9527 \
  --biz-line 123 \
  --solution "## 实施方案\n\n- 接入自动分发\n- 缩短人工等待时间" \
  --cost-benefit "## 成本收益\n\n- 每日节省人工处理时长" \
  --completed
```

说明：

- 如果用户没有给 `meego_link`，默认应先创建 Meego 再创建小改变
- 创建后默认不是“进行中”，通常先进入“待认领”
- 认领后后端会把 `owner` 设为当前用户，并把状态推进到“进行中”
- 想把状态推进到“已完成”时，必须保证 `cost-benefit` 非空，否则后端会报错

## 更新类命令

```bash
bytedcli safe sparkinnovation change update \
  --id 9527 \
  --biz-line 123 \
  --title "更新后的标题" \
  --category "技术优化" \
  --priority P2 \
  --completed
```

说明：

- `id` 和 `biz_line` 必填
- 其他字段按需传；未传字段不会更新
- `--completed` 是 flag，带上时才会下发完成标记；不带时不会下发该字段
- `title` 未传不下发；传了但去空白后为空会报错
- 想完成 change 时，先确保 `cost-benefit` 不为空

## 认领类命令

```bash
bytedcli safe sparkinnovation change claim --id 9527 --biz-line 123
```

说明：

- 只需要 `id` 和 `biz_line`
- 适用于“认领”“参与”“加入处理该 change”等意图
- 缺少 `biz_line` 时，先运行 `bytedcli safe sparkinnovation biz-line list`

## 枚举值

- `category`
  - `业务创新`
  - `技术优化`
  - `流程工具`

- `priority`
  - `P0`
  - `P1`
  - `P2`

- `status`
  - `待认领`
  - `进行中`
  - `已完成`

## AI 使用建议

- 不确定枚举时先查 `change enum get`，不要猜
- 需要结构化结果时优先用 `bytedcli --json`
- 用户只说“认领这个 change”时，优先尝试 `change claim`
- 用户只说“改一下这个 change”时，优先用 `change update`
- 用户提到“按标题搜”时，优先用 `change list --title <text>`
- 用户要创建小改变但没给 `meego_link` 时，默认先走 `bytedcli meego create`，再把返回 URL 绑定到 `change create --meego-link`
