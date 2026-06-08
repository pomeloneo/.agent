# Commands Quick Reference

本 skill 用到的 `bytedcli safe tcs` 子命令清单。完整签名、option 默认值、输出格式以 [`bytedance-safe-tcs`](../../bytedance-safe-tcs/SKILL.md) 为唯一来源，本文件只给最小用法行 + 跳转。

| 用途                                 | 最小命令                                                                                              | 详细说明                                                                                 |
| ------------------------------------ | ----------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| 查询单个队列基础信息（取名称）       | `bytedcli safe tcs project get --project-id <id>`                                                     | [bytedance-safe-tcs#project](../../bytedance-safe-tcs/SKILL.md)                          |
| 拉取父队列当前分流（superset 校验）  | `bytedcli safe tcs project get-related-project-list --shared-project-id <parent>`                     | [bytedance-safe-tcs#project](../../bytedance-safe-tcs/SKILL.md)                          |
| 设置/更新父队列分流配置（高危写）    | `bytedcli safe tcs project set-shared-project-split --shared-project-id <parent> --split-list <json>` | [bytedance-safe-tcs#project](../../bytedance-safe-tcs/SKILL.md)                          |
| 克隆为众包子队列（先向用户询问 `--group-name`） | `bytedcli safe tcs project clone --project-id <master> --is-hands-project --group-name "<用户回复>"` | [bytedance-safe-tcs#project](../../bytedance-safe-tcs/SKILL.md)                          |
| 克隆为父队列（盲审用）               | `bytedcli safe tcs project clone --project-id <master> --is-shared-project`                           | [bytedance-safe-tcs#project](../../bytedance-safe-tcs/SKILL.md)                          |
| 把队列 product_type 同步为另一个队列 | `bytedcli safe tcs project update-product-type --project-id <to-fix> --target-project-id <master>`    | [bytedance-safe-tcs#project](../../bytedance-safe-tcs/SKILL.md)                          |

## 共同约束

- 全部命令均使用 `--option` 形式参数，不使用位置参数。
- 写入类命令（`set-shared-project-split`、`clone`、`update-product-type`）必须严格遵循本 skill 中的阻塞点与确认护栏，禁止直接执行。
- JSON 模式（`-j` / `--json`）的语义与字段细节请参考主 skill；本 skill 在 agent 编排里默认使用 JSON 模式以便解析 `destProject.id`、当前分流列表等字段。
- `safe tcs project clone --is-hands-project` 调用前 Agent **必须主动**向用户索取 `--group-name`（取值参考飞书表格 https://bytedance.larkoffice.com/wiki/TZoBw7G72iaX7RkTtoXcxNvSnif?table=tblH8ScsAFYsdk02&view=vewdWVvBH5），拿到回复后再发起命令并把回复值作为 `--group-name "<用户回复>"` 透传。**禁止**仅依赖 CLI 缺省追问或直接用占位字符串调用；自动化 / `--json` / 非 TTY 场景同样适用，未携带时 CLI 抛 `SAFE_INPUT_ERROR`。

## 认证

本 skill 不重复维护登录步骤。若任意命令返回认证失败，按主 skill 提示先执行：

```bash
bytedcli auth login --session
bytedcli safe login
```

完成后重试当前步骤。
