# MongoDB Natural Language Querying Guidance for ByteDoc

Use this guide when the user describes data they want from a ByteDoc collection in natural language, for example "查询一条订单", "找最近失败的任务", or "帮我写一个筛选条件".

This guide adapts MongoDB official `mongodb-natural-language-querying` agent-skill guidance for ByteDoc. See `../references/mongodb-upstream-adaptation.md` for maintenance notes.

## Evidence First

- Identify the ByteDoc database with `bytedcli --json bytedoc search|get --service <psm>` or the command already returned by `sdk plan`.
- List collections before guessing names: `bytedcli --json bytedoc collections --service <psm>`.
- Inspect schema/sample data with safe read-only commands before generating filters, projections, or sort keys.
- Prefer `--json` output and preserve the raw evidence in your reasoning.

## Query Generation Rules

- Default to read-only queries: `findOne`, `find`, `countDocuments`, or equivalent ByteDoc safe query commands.
- Always add an explicit limit for `find`; use `limit 10` unless the user asks for a smaller number.
- Use selective filters derived from observed fields; avoid empty filters except for explicit sample exploration.
- Use projections to avoid returning large or sensitive fields when the user only needs a few fields.
- Ask a clarifying question when field names, tenant/account scope, time range, or collection name is ambiguous.

## ByteDoc CLI Workflow

```bash
bytedcli --json bytedoc collections --service "example.bytedoc.demo_orders"
bytedcli --json bytedoc document list --service "example.bytedoc.demo_orders" --collection "demo_items" --filter-json '{"tenant":"demo"}' --limit 10
bytedcli --json bytedoc shell --service "example.bytedoc.demo_orders" --collection "demo_items" --query 'find({"tenant":"demo"}).limit(10)'
```

## Safety Rules

- Do not run destructive commands such as `deleteMany`, `drop`, `renameCollection`, or index drops.
- Do not generate write queries unless the user explicitly requests a write and confirms the target service, collection, and filter.
- Do not expose sensitive sample document fields in the final answer; summarize shape and relevant field names instead.
- Do not present natural-language generated queries as guaranteed correct without showing the evidence used.

## Hand-Off To Code Generation

- For SDK code, run `bytedcli --json bytedoc sdk generate ...` after `sdk plan` is supported.
- Apply the filter/projection/sort from the evidence-based query to the generated code sample.
- Keep the driver code aligned with `mongodb-connection/GUIDE.md`.
