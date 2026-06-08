# Decision Tree — 输入场景判定矩阵

按用户输入信号选择正确的执行路径。命中第一条匹配条件即进入对应分支，不再向下匹配。

> **全局硬约束**：调用 `set-shared-project-split` 时，`--shared-project-id`（父队列）**绝不能**作为条目出现在 `--split-list` 中。下表与示例已按此规则书写。
>
> **进入判定前**：必须先调用 `bytedcli --json safe tcs project get --project-id <master_project_id>` 解析主队列角色：
>
> - `project_verify_type === 1` → `master_role = parent`；
> - `project_verify_type === 0` 且响应中包含非空的 `shared_project_id` / `parent_project_id` 字段 → `master_role = child`，并取该字段值作为 `master_parent_id`；
> - 其它情况 → `master_role = normal`。
>
> 解析失败 → 停止并向用户回显原始响应，禁止编造。

## 主队列角色 × 是否盲审 矩阵

| #   | `master_role` | `is_blind` | 最终 `parent_project_id` 来源       | `splitList` 条目                              | 分支                                                             |
| --- | ------------- | ---------- | ----------------------------------- | --------------------------------------------- | ---------------------------------------------------------------- |
| A-1 | `normal`      | 否         | `master_project_id`（主审作父队列） | 仅众包                                        | 分支 1（缺众包先走分支 2）                                       |
| A-2 | `normal`      | 是         | 用户给的父队列 / 分支 3a 克隆       | 主审 + 众包                                   | 分支 3 → 分支 1                                                  |
| B-1 | `parent`      | 否         | `master_project_id`（主队列已是父） | 仅众包                                        | 分支 1（缺众包先走分支 2）                                       |
| B-2 | `parent`      | 是         | 分支 5 克隆得到的新父队列           | 主审 + 众包；每条带 `projectVerifyType`，取自 `safe tcs project get` | 分支 5 → 一次 `set-shared-project-split` 同时完成分流 + 主审降级 |
| C-1 | `child`       | 否         | `master_parent_id`（沿用现有父）    | 仅众包                                        | 分支 1                                                           |
| C-2 | `child`       | 是         | `master_parent_id`（已是盲审拓扑）  | 主审 + 众包                                   | 分支 1                                                           |

## 输入信息组合判定

> 主队列角色已按上表确定后，再根据用户提供的信息组合决定如何补齐缺口。

| #   | 用户输入特征                                                                              | 是否盲审 | 父队列已给 | 主审已给 | 众包已给 | 百分比已给 | 最终 `parent_project_id`        | 子队列条目（splitList）                                                              | 分支                             |
| --- | ----------------------------------------------------------------------------------------- | -------- | ---------- | -------- | -------- | ---------- | ------------------------------- | ------------------------------------------------------------------------------------ | -------------------------------- |
| 1   | "把 example-master-id 切到盲审，主审 70 / 众包 30"                                        | 是       | 否         | 是       | 否       | 是         | 克隆得到的新父队列              | 主审 + 众包                                                                          | 3a → 3b → 1（克隆父+众包+分流）  |
| 2   | "盲审：父队列 example-parent-id 分流到主审 example-master-id 与众包 example-hands-id 7:3" | 是       | 是         | 是       | 是       | 是         | 用户给的 `example-parent-id`    | 主审 + 众包                                                                          | 3b → 1（直接父队列分流）         |
| 3   | "把 example-master-id 切到盲审众包模式"（仅给主审）                                       | 是       | 否         | 是       | 否       | 否         | 克隆得到的新父队列              | 主审 + 众包                                                                          | 3a → 询问百分比 → 3b → 1         |
| 4   | "队列 example-master-id 绑定众包子队列 example-hands-id，主审 70% / 众包 30%"             | 否       | n/a        | 是       | 是       | 是         | `example-master-id`（主审作父） | **仅众包**（主审作父，不能再作子队列）；如需 7:3 主审/众包同时作子队列，请按盲审处理 | 1（直分流）                      |
| 5   | "把 example-master-id 改成众包审核"（仅给主审）                                           | 否       | n/a        | 是       | 否       | 否         | `example-master-id`（主审作父） | **仅众包**                                                                           | 2（克隆众包）→ 询问 → 1          |
| 6   | "把 example-master-id 改成众包审核，主审 70 / 众包 30"（仅给主审 + 百分比）               | 否       | n/a        | 是       | 否       | 是         | `example-master-id`（主审作父） | **仅众包**（同上：主审作父则只剩众包）                                               | 2（克隆众包）→ 1                 |
| 7   | "切换队列审核模式"（信息不足）                                                            | ?        | ?          | 否       | ?        | ?          | -                               | -                                                                                    | 阻塞：先追问 `master_project_id` |
| 8   | "把 example-master-id 切到盲审"，且主队列本身是父队列（`master_role = parent`）           | 是       | 否         | 是       | 否       | ?          | 分支 5 克隆得到的新父队列       | 主审 + 众包；每条带 `projectVerifyType`，取自 `safe tcs project get`                 | 5 → 2（如缺众包）→ 逐条 project get → 1（一次性分流 + 主审降级） |
| 9   | "把 example-master-id 改成众包审核"，且主队列已经挂在父队列下（`master_role = child`）    | 否       | n/a        | 是       | 否       | ?          | `master_parent_id`              | **仅众包**（挂到现有父队列）                                                         | 2（如缺众包）→ 1                 |

> 同时给齐主审 + 众包 + 百分比但**也提到了盲审**时，盲审优先：必须有 `parent_project_id` 才能继续，否则按场景 3 / 1 处理。
>
> 当用户希望主审 / 众包按比例切分（如 7:3、6:4）时，必须走盲审路径：先克隆或提供独立父队列，再把**主审队列 + 众包队列**同时作为子队列挂到该父队列下；其中父队列为克隆得到的父队列（或用户显式提供的独立父队列），`splitList` 只包含主审与众包，不包含父队列自身。

## 各场景最简编排示例

> 以下示例统一使用占位 id：`example-master-id`（主审）、`example-hands-id`（众包）、`example-parent-id`（父队列），分流比例 0.7 / 0.3。所有 `--split-list` 示例都**不**包含父队列条目。
>
> 每条子队列条目可选 `projectVerifyType`，值取自子队列 `bytedcli --json safe tcs project get --project-id <id>` 返回的 `project_verify_type`；下面示例为可读性省略，实际编排时如已查询到子队列详情，建议在条目中显式带上该字段（缺失时不要填写）。
>
> 所有 `safe tcs project clone` 调用都可选追加 `--title "<用户给出的名称>"`：仅当用户在需求中明确给出「克隆后队列名称」时才透传，未提及则不写该参数，沿用后端默认命名。下方示例为简洁起见均省略 `--title`。
>
> 所有 `safe tcs project clone --is-hands-project` 调用前，Agent **必须主动**先向用户询问 `--group-name`（取值参考飞书表格 https://bytedance.larkoffice.com/wiki/TZoBw7G72iaX7RkTtoXcxNvSnif?table=tblH8ScsAFYsdk02&view=vewdWVvBH5）。下方示例中的 `--group-name "<用户回复>"` 仅作占位示意；实际编排时务必先拿到用户回复，再把回复值替换到该位置。**禁止**直接用 `demo-group` 等占位字符串调用，自动化 / `--json` / 非 TTY 场景同样适用。

### 场景 1：盲审 + 仅给主审 + 百分比

```bash
# Step 1: 克隆父队列
bytedcli --json safe tcs project clone \
  --project-id example-master-id \
  --is-shared-project
# → 解析 destProject.id 为 example-parent-id

# Step 2: 克隆众包子队列
bytedcli --json safe tcs project clone \
  --project-id example-master-id \
  --is-hands-project \
  --group-name "demo-group"
# → 解析 destProject.id 为 example-hands-id

# Step 3: 父队列分流（superset 校验 + CLI 自动 preview + 确认后再执行；自动化场景需追加 `--yes`）
bytedcli safe tcs project get-related-project-list \
  --shared-project-id example-parent-id
bytedcli safe tcs project get --project-id example-parent-id
bytedcli safe tcs project get --project-id example-master-id
bytedcli safe tcs project get --project-id example-hands-id

# splitList 同时含主审与众包，不含父队列
bytedcli safe tcs project set-shared-project-split \
  --shared-project-id example-parent-id \
  --split-list '[{"projectId":"example-master-id","sharedTaskPercent":0.7},{"projectId":"example-hands-id","sharedTaskPercent":0.3}]'
```

### 场景 2：盲审 + 父队列已给

```bash
# Step 1: 父队列分流（superset 校验 + CLI 自动 preview + 确认后再执行；自动化场景需追加 `--yes`）
bytedcli safe tcs project get-related-project-list \
  --shared-project-id example-parent-id
# 名称解析略

# splitList 同时含主审与众包，不含 example-parent-id
bytedcli safe tcs project set-shared-project-split \
  --shared-project-id example-parent-id \
  --split-list '[{"projectId":"example-master-id","sharedTaskPercent":0.7},{"projectId":"example-hands-id","sharedTaskPercent":0.3}]'
```

### 场景 4：直分流（主审 + 众包 + 百分比都给齐，主审作父队列）

```bash
# 主审队列同时作为父队列分流，子队列只能是众包队列；
# 因此在该编排下众包占 100%（hands_percent = 1）。如果业务真的要主审 / 众包按 7:3 分流，
# 应改走盲审：克隆独立父队列后再把主审与众包同时挂到父队列下。
bytedcli safe tcs project get-related-project-list \
  --shared-project-id example-master-id
bytedcli safe tcs project get --project-id example-master-id
bytedcli safe tcs project get --project-id example-hands-id

# splitList 仅含众包，不含父队列（即主审 example-master-id）自身
bytedcli safe tcs project set-shared-project-split \
  --shared-project-id example-master-id \
  --split-list '[{"projectId":"example-hands-id","sharedTaskPercent":1}]'
```

### 场景 5/6：仅给主审（可选百分比）→ 克隆众包 → 主审作父队列直分流

```bash
# Step 1: 克隆众包子队列
bytedcli --json safe tcs project clone \
  --project-id example-master-id \
  --is-hands-project \
  --group-name "demo-group"
# → 解析 destProject.id 为 example-hands-id

# Step 2: 询问 / 确认；主审作父队列时子队列仅为众包，hands_percent = 1。
#         若用户坚持主审 70 / 众包 30，澄清并改走盲审场景（场景 1）。

# Step 3: 直分流（superset 校验 + CLI 自动 preview + 确认后再执行；自动化场景需追加 `--yes`）
# splitList 仅含众包，不含父队列（即主审 example-master-id）自身
bytedcli safe tcs project set-shared-project-split \
  --shared-project-id example-master-id \
  --split-list '[{"projectId":"example-hands-id","sharedTaskPercent":1}]'
```

### 场景 8（B-2）：主队列原本是父队列 + 要求盲审

```bash
# Step 0: 角色预检（确认 master_role = parent）
bytedcli --json safe tcs project get --project-id example-master-id

# Step 1: 克隆新父队列（盲审拓扑下的真正父队列）
bytedcli --json safe tcs project clone \
  --project-id example-master-id \
  --is-shared-project
# → 解析 destProject.id 为 example-new-parent-id

# Step 2: 如缺众包，先克隆（同分支 2）
bytedcli --json safe tcs project clone \
  --project-id example-master-id \
  --is-hands-project \
  --group-name "demo-group"
# → 解析 destProject.id 为 example-hands-id

# Step 3: 逐条调用 `safe tcs project get` 拿子队列 `project_verify_type`
#         （Agent 不得自行赋值；查询失败 / 字段缺失 → 该条目不写 projectVerifyType 并回显警告）
bytedcli --json safe tcs project get --project-id example-master-id
# → 假设响应中 project_verify_type = 1，记为 master_verify_type
bytedcli --json safe tcs project get --project-id example-hands-id
# → 假设响应中 project_verify_type = 0，记为 hands_verify_type

# Step 4: 一次 set-shared-project-split 同时完成分流 + 主审降级（superset 校验 + CLI 自动 preview + 确认后再执行；自动化场景需追加 `--yes`）
# splitList 同时含主审与众包，每条 projectVerifyType 透传自 Step 3 的查询结果，不含 example-new-parent-id
bytedcli safe tcs project set-shared-project-split \
  --shared-project-id example-new-parent-id \
  --split-list '[{"projectId":"example-master-id","sharedTaskPercent":0.7,"projectVerifyType":1},{"projectId":"example-hands-id","sharedTaskPercent":0.3,"projectVerifyType":0}]'
```

### 场景 9（C-1）：主队列已经是子队列 + 直分流（挂到现有父队列）

```bash
# Step 0: 角色预检（确认 master_role = child，取 master_parent_id）
bytedcli --json safe tcs project get --project-id example-master-id
# → 假设响应中 shared_project_id = example-existing-parent-id

# Step 1: 克隆众包子队列（先向用户索取 group-name，再调用本命令）
bytedcli --json safe tcs project clone \
  --project-id example-master-id \
  --is-hands-project \
  --group-name "<用户回复>"
# → 解析 destProject.id 为 example-hands-id

# Step 2: 把众包挂到现有父队列下（superset 校验 + CLI 自动 preview + 确认后再执行；自动化场景需追加 `--yes`）
# splitList 仅含众包；父队列沿用 master_parent_id
bytedcli safe tcs project set-shared-project-split \
  --shared-project-id example-existing-parent-id \
  --split-list '[{"projectId":"example-hands-id","sharedTaskPercent":1}]'
```

## ProductType 修复（任意分支失败后触发）

仅当上一步 `set-shared-project-split` 因 `product_type` 不一致失败。`update-product-type` 由 CLI handler 自动 preview + 等用户确认；自动化场景请追加 `--yes`：

```bash
# 交互式终端：CLI 自动 preview，等待用户输入 y/yes/确认
bytedcli safe tcs project update-product-type \
  --project-id example-parent-id \
  --target-project-id example-master-id
# 若众包队列也不一致，再补一次：
bytedcli safe tcs project update-product-type \
  --project-id example-hands-id \
  --target-project-id example-master-id

# 自动化 / Skill 链式调用：必须显式 --yes，否则抛 SAFE_INPUT_ERROR
bytedcli --json safe tcs project update-product-type \
  --project-id example-parent-id \
  --target-project-id example-master-id \
  --yes

# 修复完成后重试分流；splitList 仍然不含父队列；CLI 自动 preview，自动化场景同样需要 `--yes`
bytedcli safe tcs project set-shared-project-split \
  --shared-project-id example-parent-id \
  --split-list '[{"projectId":"example-master-id","sharedTaskPercent":0.7},{"projectId":"example-hands-id","sharedTaskPercent":0.3}]'
```
