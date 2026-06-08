---
name: bytedance-safe-octp
description: OCTP entity trace event queries via bytedcli safe domain. Use when the user wants to inspect content-safety events for an entity ID such as video, account, audio, or other moderated entity across review submission, machine review, disposal, communication, and report links; also use for 中文意图：内容安全链路、关键链路、上报事件信息、实体ID查询、视频/账号/音频事件、送审、机审、处置、沟通、举报、OCTP、千寻/qianxun.
---

# Safe OCTP

Query OCTP trace events for one Safe entity.

## Command

```bash
bytedcli safe octp event list --entity-id <entity-id> [--conditions <json>]
```

## Workflow

1. Ensure Safe authentication exists: `bytedcli safe login`
2. Run `bytedcli safe octp event list --entity-id <entity-id>`
3. Use text output for a compact event timeline
4. Use `--json` when another tool needs the full `info_list`, `tags`, `assembly_events`, or `conclusion` payload

## Examples

```bash
bytedcli safe octp event list --entity-id <entity-id>

bytedcli --json safe octp event list --entity-id <entity-id>

bytedcli safe octp event list --entity-id <entity-id> --conditions "[]"
```

## Output

Text mode shows a compact table with:

- event time
- event title
- reported event key
- path ID
- PSM
- log ID

JSON mode returns the complete OCTP aggregation:

- `entity_info`
- `event_count`
- `events[*].info_list`
- `events[*].tags`
- `events[*].assembly_events`
- `events[*].conclusion`

## References

- [invocation.md](../../invocation.md)
- [troubleshooting.md](../../troubleshooting.md)
