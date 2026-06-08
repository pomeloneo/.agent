---
name: bytedance-flink
description: "Read-only Flink REST access via bytedcli: cluster/job/taskmanager/vertex info, exceptions, checkpoints, metrics and logs from a Godel stream-applications web URL. Use when inspecting a Flink job's status, failures, or logs."
---

# bytedcli Flink (read-only)

## 如何调用 bytedcli

先选择一种调用方式。下面所有示例默认直接写 `bytedcli`。

```bash
# 方式 1：直接用 npx 运行最新版
NPM_CONFIG_REGISTRY=http://bnpm.byted.org npx -y @bytedance-dev/bytedcli@latest <command> [options]

# 方式 2：先全局安装，再直接调用 bytedcli
NPM_CONFIG_REGISTRY=http://bnpm.byted.org npm install -g @bytedance-dev/bytedcli@latest
bytedcli <command> [options]
```

## When to use
- Inspect a Flink job's state, exceptions, checkpoints, plan, or metrics.
- List/inspect TaskManagers and JobManager config/logs.
- You have a Flink dashboard (Godel stream-applications) web URL.

## Invocation
Commands are verb-last: `bytedcli flink <resource> [<subresource>] <verb> --url <flinkWebUrl> [options]`
Add `-j/--json` before the verb for JSON output.

## Quick start

```bash
bytedcli flink cluster overview get --url https://godel-stream-applications.byted.org/<cluster>/<appId>
bytedcli flink cluster config get   --url <flinkWebUrl>
bytedcli flink job list             --url <flinkWebUrl>
bytedcli flink job exception get    --url <flinkWebUrl>        # auto-detects single job
bytedcli flink taskmanager list     --url <flinkWebUrl>
bytedcli flink jobmanager log get   --url <flinkWebUrl> --tail 200
bytedcli flink raw get --url <flinkWebUrl> --path /jobs/overview
bytedcli flink raw get --url <flinkWebUrl> --path /jobmanager/log --text   # plain-text endpoints
```

## URL & network
- The passed URL is the REST base. Office and prod URLs are both accepted.
- `--network office|prod` rewrites the host between sibling Godel domains for cn/sg/va/gcp (cn is a no-op; gcp has office only).

## Office-network auth
Office-network entrypoints (`.tiktok-row.net`) require a Cookie.

The CLI auto-assembles it from your `bytedcli auth login` session: it navigates
the office Godel host with your SSO cookies (following bd_sso SSO redirects to
capture any `flink_session`), adds `titan_passport_id`, caches the result per
host, and attaches it as the `cookie` request header. Just run
`bytedcli auth login` once, then use office URLs directly.
This is best-effort: if assembly fails the request falls back to anonymous and a
401 prints guidance.

- `--refresh-cookie` — discard the cached office cookie and re-assemble from your SSO session (also auto-triggered on a 401)

Env knobs:
- `BYTEDCLI_FLINK_TITAN_SITE` — override the CloudSite used to issue titan/SSO JWT (else region map: cn→cn; sg/va/gcp→i18n-tt; else config site).
- `BYTEDCLI_FLINK_BOOTSTRAP_PATH` — path navigated to bootstrap the office session (default `/overview`).
- `BYTEDCLI_FLINK_SESSION_TTL_HOURS` — cached-cookie TTL in hours (default 2).

## Parameters (selected)
| Command | --url | --job-id | --tm-id | --vertex-id | other |
|---|:--:|:--:|:--:|:--:|---|
| cluster overview get / cluster config get | ✅ | | | | |
| job list | ✅ | | | | |
| job get / job config get / job exception get / job plan get / job accumulator get / job checkpoint list | ✅ | optional | | | |
| job metric list | ✅ | optional | | | `--names` |
| job checkpoint get | ✅ | optional | | | `--checkpoint-id` ✅ |
| job checkpoint-config get | ✅ | optional | | | |
| taskmanager list | ✅ | | | | |
| taskmanager get / taskmanager metric list / taskmanager log list / taskmanager log get | ✅ | | ✅ | | log get: `--name/--stdout/--tail/--full/--out` |
| jobmanager config get / jobmanager metric list / jobmanager log list / jobmanager log get | ✅ | | | | log get: `--name/--stdout/--tail/--full/--out` |
| vertex get / vertex watermark get / vertex backpressure get / vertex subtask-time list / vertex taskmanager list / vertex metric list | ✅ | optional | ✅ | | metric list: `--names` |
| raw get | ✅ | | | | `--path` ✅ |

## Notes
- Auth: best-effort ByteCloud JWT (`x-jwt-token`) plus, for office-network URLs, an auto-assembled office Cookie from your `bytedcli auth login` session. 401 ⇒ run `bytedcli auth login`, pass `--refresh-cookie`, or use a prod-net URL / `--network prod`.
- Logs default to the last 500 lines; use `--full` or `--out <path>`.
