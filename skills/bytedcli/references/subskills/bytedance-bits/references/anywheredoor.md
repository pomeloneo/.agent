# Anywheredoor / 任意门

Use `bytedcli bits anywhere` for Anywheredoor proxy debugging. The original capture flow remains `listen` / `status` / `watch` / `get` / `stop`; newer commands add device selection, mock inspection/mutation, filters, and black paths.

## Safety

- Prefer `--json` for agent workflows.
- Use `device list` first and pass a real device id to state-changing commands. `did=0` is useful for some readonly list/config checks, but mock create/enable/delete requires a real device id.
- Treat these commands as state-changing: `listen`, `stop`, `mock create-local`, `mock enable`, `mock disable`, `mock delete`.
- Mock write commands require `--yes`. For probes, create a disabled local mock first, then delete it after validation.
- Do not put real app ids, device ids, mock ids, PSMs, or internal URLs in public examples. Use placeholders like `1234`, `1234567890123456`, and `/api/demo`.

## Device Selection

```bash
bytedcli --json bits anywhere device list --app-id 1234
```

Pick a real `did` from this list before starting capture or changing mocks.

## Original Capture Flow

```bash

bytedcli --json bits anywhere listen \
  --app-id 1234 --did 1234567890123456

bytedcli --json bits anywhere status \
  --app-id 1234 --did 1234567890123456

bytedcli bits anywhere watch \
  --app-id 1234 --did 1234567890123456 \
  --url-path /api/demo \
  --show-curl

bytedcli bits anywhere watch \
  --app-id 1234 --did 1234567890123456 \
  --window-sec 1800 \
  --include-snapshot \
  --url-path /api/demo

bytedcli bits anywhere get \
  --app-id 1234 --did 1234567890123456 \
  --history-id 1447858190 --env 8 --curl

bytedcli --json bits anywhere stop \
  --app-id 1234 --did 1234567890123456
```

`watch` defaults to HTTP polling mode and prints `id`, `path`, `log_id`, and `env`. Use `--window-sec` with `--include-snapshot` to dump recent captures; use `--show-curl` to render each matched capture as a curl command. `get --history-id <id> --env <env> --curl` converts one captured record into curl.

Advanced compatibility options from the original flow are still available:

- `--skip-listen`: watch without calling `listen` first.
- `--mode ws`: Arena WebSocket research escape hatch. It is usually silent for non-browser clients, so prefer default poll mode for real debugging.
- `--interval-ms`: polling interval, default `1500`.
- `--url-path`: client-side substring filter plus backend path hint.

The curl renderer removes headers that curl manages itself, such as `Host`, `Content-Length`, and `Accept-Encoding`.

## Mock

```bash
bytedcli --json bits anywhere mock list \
  --app-id 1234 --did 1234567890123456 \
  --type local --page-size 20

bytedcli --json bits anywhere mock get \
  --app-id 1234 --did 1234567890123456 \
  --mock-id 987654321 --type local

bytedcli --json bits anywhere mock create-local \
  --app-id 1234 --did 1234567890123456 \
  --name sample-local-mock \
  --method GET \
  --url-path /api/demo \
  --body '{"ok":true}' \
  --serializer json \
  --yes

bytedcli --json bits anywhere mock enable \
  --app-id 1234 --did 1234567890123456 \
  --mock-id 987654321 --yes

bytedcli --json bits anywhere mock disable \
  --app-id 1234 --did 1234567890123456 \
  --mock-id 987654321 --yes

bytedcli --json bits anywhere mock delete \
  --app-id 1234 --did 1234567890123456 \
  --mock-id 987654321 --yes
```

Supported mock type names: `local`, `remote`, `status-code`, `throttling`, `idl`, `rewrite`.

Supported serializer names for `create-local`: `json`, `text`, `html`, `javascript`, `protobuf`, `webcast-packer`, `webcast-im`, `stream-forecast`, `script`.

## Filter And Black Path

```bash
bytedcli --json bits anywhere filter get \
  --app-id 1234 --did 1234567890123456

bytedcli --json bits anywhere black-path get --app-id 1234
```

These are readonly inspection commands.
