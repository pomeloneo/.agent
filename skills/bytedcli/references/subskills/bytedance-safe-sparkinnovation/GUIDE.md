---
name: bytedance-safe-sparkinnovation
description: "Operate SparkInnovation micro-change workflows via bytedcli safe domain: list/get/create/update/claim changes, list biz lines/departments, inspect enum values, and auto-create/bind Meego when creating a change. Use when tasks mention sparkinnovation, 小改变, or safe sparkinnovation changes."
---

# Safe SparkInnovation

SparkInnovation 小改变工作流 skill，覆盖小改变列表、详情、创建、更新、认领，以及业务线、部门和枚举值查询。

## Prerequisites

- Requires unified bytedcli BDSSO authentication. If not already logged in:

```bash
bytedcli auth login --session
```

- 通用调用方式见 `../../invocation.md`
- 如果要在创建小改变时自动创建 Meego，还需要先准备 Meego 登录态：

```bash
bytedcli meego login
```

## When to use

- 用户要查询 SparkInnovation 小改变列表
- 用户要查看某个 change 的详情
- 用户要创建、更新或认领小改变
- 用户要创建小改变，并希望自动补建 Meego 需求后绑定到 `meego_link`
- 用户不知道 `category`、`priority`、`status` 的可选值
- 用户只给了业务语义，需要你自动选择 `safe sparkinnovation` 相关命令

## Quick start

```bash
# 列业务线
bytedcli safe sparkinnovation biz-line list

# 列部门
bytedcli safe sparkinnovation department list --biz-line 123

# 查询小改变列表
bytedcli safe sparkinnovation change list --biz-line 123

# 查看详情
bytedcli safe sparkinnovation change get --id 9527 --biz-line 123

# 查看枚举
bytedcli safe sparkinnovation change enum get

# 先创建 Meego，再把返回 URL 绑定到小改变
bytedcli meego create \
  --space demo-project \
  --title "优化 xxx 流程" \
  --description "这里可以复用小改变描述"

# 创建（创建后初始状态通常是“待认领”）
bytedcli safe sparkinnovation change create \
  --title "优化 xxx 流程" \
  --category "技术优化" \
  --description "## 背景\n\n这里写 Markdown" \
  --meego-link "https://meego.larkoffice.com/demo-project/story/detail/123456" \
  --priority P2 \
  --biz-line 123

# 认领（认领后 owner 变成当前用户，状态变成“进行中”）
bytedcli safe sparkinnovation change claim --id 9527 --biz-line 123

# 更新并完成（完成前必须保证 cost-benefit 不为空）
bytedcli safe sparkinnovation change update \
  --id 9527 \
  --biz-line 123 \
  --solution "## 实施方案\n\n- step 1\n- step 2" \
  --cost-benefit "## 成本收益\n\n- 节省人工处理成本" \
  --completed
```

## Recommended workflow

1. 如果用户没有给 `biz_line`，先执行 `biz-line list`
2. 如果用户没有给枚举值，先执行 `change enum get`，不要猜
3. 查看单个 change 时优先用 `change get --id ... --biz-line ...`，不要把 `change list` 当详情入口
4. 创建小改变时，如果用户没有给 `meego_link`，且没有明确说“不需要 Meego”，优先先创建 Meego 再绑定
5. 创建和更新富文本字段时，直接传 Markdown 给 CLI
6. 需要机器可读输出时，优先使用 `bytedcli --json ...`
7. `change list` 的分页参数默认是 `--page 1 --page-size 20`
8. 如果目标是“从创建推进到完成”，推荐顺序是 `create -> claim -> update --completed`

## Key rules

- 小改变创建默认要关联 Meego：如果用户没给 `meego_link`，且没有明确说不需要 Meego，先调用 `bytedance-meego` / `bytedcli meego create` 创建需求，再把返回 URL 填给 `change create --meego-link`
- 创建 Meego 时优先复用用户给的小改变标题和描述；若描述较长，可提炼成适合 Meego 的简版标题与描述，但不要编造额外需求信息
- 只有当用户已经提供 `meego_link`，或明确表示“不需要创建 Meego / 不要绑定 Meego”时，才跳过自动建 Meego
- `change create` 的 `description`、`solution`、`cost-benefit` 接收 Markdown，CLI 会自动转成富文本 JSON
- `change get` 会把后端富文本字段自动转成 Markdown 展示
- `change list` 用于列表筛选；按 ID 看详情时用 `change get`
- `change list` 的时间筛选 flag 使用 `--start` / `--end`，值仍然传 ISO 时间戳
- `change update` 未传的字段不会更新
- `change update` 中 `title` 未传则不下发；传了但 `trim` 后为空会报错
- `change update --completed` 之前必须保证 `cost-benefit` 非空，否则后端会拒绝完成
- `--completed` 是布尔 flag；只有显式带上时才标记完成，不带时不会下发该字段
- `change claim` 只需要 `id` 和 `biz_line`
- 如果缺少 `biz_line`，先运行 `bytedcli safe sparkinnovation biz-line list`
- 文档里的 `123` 只是占位值；真实 `biz_line` 请先通过 `bytedcli safe sparkinnovation biz-line list` 获取
- `--json` 是全局参数，必须放在命令前

## References

- `../../invocation.md`
- `references/sparkinnovation.md`
- `../../troubleshooting.md`
- `../bytedance-meego/SKILL.md`
