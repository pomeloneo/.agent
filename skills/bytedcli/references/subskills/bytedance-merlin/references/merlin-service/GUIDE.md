---
name: merlin-service
description: 管理 Merlin 线上服务（Bernard）：查询服务配置、创建服务、部署模型、查看日志、调用 API。当用户说"部署模型/创建服务/Bernard/vLLM 部署/查看线上服务/deploy model/service 信息"时使用。
---

# 线上服务管理（Bernard）

管理 Merlin 线上服务（Bernard 平台）：创建服务、部署模型、查看容器日志、验证 API。

## 前置条件

- `bytedcli merlin` 可用
- 已登录：`bytedcli auth login`

```bash
bytedcli merlin --help &>/dev/null || \
  NPM_CONFIG_REGISTRY=http://bnpm.byted.org npx -y @bytedance-dev/bytedcli@latest merlin --help
```

---

## 查询服务

```bash
# 列出所有服务
bytedcli merlin deploy service list

# 查询指定服务详情
bytedcli merlin deploy service get --service-id <service-id> -j

# 查看可用 GPU 配额
bytedcli merlin deploy quota --service-id <service-id> -j
```

服务 URL 格式：`https://ml.bytedance.net/serviceList/<service_id>`

---

## 创建服务（custom-build + vLLM）

准备 JSON 配置文件，用 `--body-file` 创建：

```bash
bytedcli merlin deploy service create --body-file /tmp/service-config.json -j
```

JSON 配置关键字段：

```json
{
  "id": "<service-id>",
  "psm": "example.models.deploy",
  "cpus": 64,
  "mem": 481280,
  "gpus": 4,
  "container_image": "",
  "container_image_type": "custom-build",
  "model_location": "modelzoo://",
  "model_type": "tensorflow",
  "enable_ipv6": true,
  "num_ports": 8,
  "custom_build_meta": {
    "base_image": "seed.infer.debian12.python312.cuda124.torch29.base:latest",
    "runtime": 0,
    "load_path": "<scm-relative-path-to-start-script>",
    "image_namespace": "bernard",
    "scms": [
      {"name": "toutiao/runtime", "path": "/opt/tiger/toutiao/runtime", "version": "1.0.1.450"},
      {"name": "tce/tce_tools", "path": "/opt/tiger/tce/tce_tools", "version": "1.0.0.132"},
      {"name": "lab/bernard/load", "path": "/opt/tiger/toutiao/load", "version": "1.0.0.69"},
      {"name": "<your-scm>", "path": "/opt/tiger/<your-dir>", "version": "<version>"},
      {"name": "data/inf/hdfs_client", "path": "/opt/tiger/hdfs_client", "version": "1.9.28.81"}
    ],
    "packages": [
      {"name": "vllm==0.18.0 && pip --no-cache-dir install transformers==5.3.0", "package_type": "pip"}
    ]
  },
  "feature_gate": {
    "laplace_arch": "x86",
    "shm": {"enabled": true, "size": 100000}
  },
  "envs": [
    "BERNARD_MULTI_MODELS=<model-name>:<version>",
    "TCE_ENABLE_SIDECAR=100",
    "TCE_INSTALL_SIDECAR=True",
    "USE_BERNARD_STDOUT=1",
    "BGCP_LOGGING_STDOUT=1",
    "IGNORE_LIVENESS_CHECK=1",
    "MODEL_HDFS_PATH=hdfs://haruna/home/...",
    "TOOL_CALL_PARSER=qwen3_coder"
  ],
  "real_put_envs": true
}
```

### 关键参数说明

| 字段 | 说明 |
|------|------|
| `container_image` | custom-build 时必须为空字符串 `""` |
| `load_path` | 启动脚本路径，相对于 `/opt/tiger/`，只能是文件（不能是目录） |
| `lab/bernard/load` SCM | **必须包含**，负责执行 load_path 和注册服务 |
| `shm.size` | MB 单位，须 < `mem`，用于 /dev/shm 存放模型 |
| `packages.name` | 支持 `&&` 串联多条 pip 命令 |
| `real_put_envs` | 设为 `true` 确保 envs 被写入 |

### 必须的环境变量

| 变量 | 作用 |
|------|------|
| `BERNARD_MULTI_MODELS` | 触发 Bernard load 机制执行 load_path（值为 `<model>:<version>`，版本需通过 model store 注册） |
| `TCE_ENABLE_SIDECAR=100` | load 机制依赖 |
| `TCE_INSTALL_SIDECAR=True` | load 机制依赖 |
| `USE_BERNARD_STDOUT=1` | 日志输出到 /dev/bernard_stdout |
| `BGCP_LOGGING_STDOUT=1` | 日志捕获 |
| `IGNORE_LIVENESS_CHECK=1` | 跳过初始健康检查（模型加载耗时长） |

> **注意**：缺少以上环境变量会导致容器启动后只显示 "Hello from Bernard Container!" banner，不执行 start.sh。

---

## 等待构建

```bash
bytedcli merlin deploy service get --service-id <service-id> -j
# 检查 custom_build_meta.build_status: "building" → "ok" → 可部署
# 构建通常需 5-10 分钟（pip install vLLM 较慢）
```

---

## 创建部署

```bash
bytedcli merlin deploy create \
  --service-id <service-id> \
  --region lf-default \
  --gpu-type l40 \
  --instances 1
```

参数说明：
- **region**: `lf-default`, `hl-default`, `lq-default` 等，用 `deploy quota` 查看可用配额
- **gpu-type**: L40 (48GB), L20 (96GB), A100/A800 (80GB) 等
- 报 `"image is in status: building"` 说明构建未完成

---

## 监控部署

```bash
# 列出部署
bytedcli merlin deploy list --service-id <service-id> -j

# 容器详情（host、containerID、webshell URL）
bytedcli merlin deploy get --deployment-id <id> -j

# 容器日志
bytedcli merlin deploy logs \
  --service-id <service-id> \
  --host <ipv6-from-detail> \
  --container-id <containerID-from-detail>
```

日志正常流程：
1. "Hello from Bernard Container!" — load 初始化
2. HDFS download — 模型下载
3. vLLM serve started — 服务启动
4. "Application startup complete." — API 就绪
5. "service is ready!" — 健康检查通过（约 5-7 分钟）

---

## 验证 API

```bash
# 从 deploy get 获取 tasks[0].host 和 tasks[0].ports[0]

# 健康检查
curl -6 "http://[<ipv6>]:<port>/health"

# Chat Completion
curl -6 -X POST "http://[<ipv6>]:<port>/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -d '{"model": "<model-name>", "messages": [{"role": "user", "content": "Hello"}], "max_tokens": 100}'
```

---

## 删除部署

```bash
bytedcli merlin deploy delete --deployment-id <id>
```

---

## start.sh 模板

```bash
#!/usr/bin/env bash
exec > /dev/bernard_stdout 2>&1
set -exuo pipefail

if [ -z "${MODEL_HDFS_PATH:-}" ]; then
    echo "Error: MODEL_HDFS_PATH not set"; exit 1
fi

MODEL_DIR=$(basename "$MODEL_HDFS_PATH")
mkdir -p /dev/shm/model_download && cd /dev/shm/model_download

/opt/tiger/hdfs_client/bin/hdfs get -s -c 128 --ct 32 -t 8 "$MODEL_HDFS_PATH" .

vllm serve "$MODEL_DIR" \
    --tensor-parallel-size 2 \
    --pipeline-parallel-size 2 \
    --port $PORT --host "::" \
    --enable-auto-tool-choice --tool-call-parser $TOOL_CALL_PARSER
```

关键点：
- `exec > /dev/bernard_stdout 2>&1`：确保日志在 `deploy logs` 可见
- `$PORT`：Bernard 自动注入（`TCE_SERVICE_PORT`）
- TP × PP = gpus 总数（如 TP=2, PP=2 对应 4 GPU）

---

## Troubleshooting

| 现象 | 原因 | 解决 |
|------|------|------|
| 日志只有 banner | 缺 `BERNARD_MULTI_MODELS` 等 env | 补全环境变量，重建部署 |
| `image is in status: building` | 构建中 | 等 build_status=ok |
| `container_image is required` | custom-build 缺字段 | 设为 `""` |
| Build failed | load_path 是目录或绝对路径 | 改为相对文件路径 |
| `Not enough GPU quota` | 配额不足 | 换 region 或 gpu-type |
| 部署后一直 NotReady | 模型加载中 | 正常，等 5-7 分钟 |

---

## 关联技能

- `merlin-job-launch`：创建训练任务
- `merlin-recipe-eval-run`：运行评估
- `merlin-checkpoints`：Checkpoint 管理
