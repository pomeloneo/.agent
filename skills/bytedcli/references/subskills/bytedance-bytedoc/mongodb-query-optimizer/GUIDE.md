# MongoDB Query Optimizer Guidance for ByteDoc

Use this guide when the user says a ByteDoc / Mongo query is slow, times out, scans too much data, sorts slowly, or asks for index/query optimization.

This guide adapts MongoDB official `mongodb-query-optimizer` agent-skill guidance for ByteDoc. See `../references/mongodb-upstream-adaptation.md` for upstream mapping and exclusions.

## Evidence To Collect

- ByteDoc database identity: service PSM, backend, deploy mode, site, vRegion, and collection.
- Query shape: filter, sort, projection, limit, and operation type.
- Slow-query evidence: `bytedcli --json bytedoc slow-query overview ...` and detail/fingerprint data when available.
- Collection evidence: schema/sample fields, existing indexes, data volume if available, and recent error messages.
- Runtime evidence: whether the caller is local/devbox/TCE/FaaS and whether latency is connection setup or query execution.

## Analysis Flow

1. Confirm the access path with `bytedoc sdk plan`; connection-path issues are not query optimizer issues.
2. Normalize the query shape and remove user-specific literal values when describing it.
3. Check whether predicates and sort keys can use an existing index.
4. Look for unbounded scans, missing tenant/account/time filters, regex without anchors, large `$in`, and in-memory sort symptoms.
5. Prefer query-shape changes before index requests when the query is overly broad.
6. If recommending an index, explain the exact query shape it supports and the write/storage trade-off.

## ByteDoc Commands

```bash
bytedcli --json bytedoc slow-query overview --service "example.bytedoc.demo_orders" --deploy-mode classic --millis 100
bytedcli --json bytedoc slow-query detail --service "example.bytedoc.demo_orders" --fingerprint-id "<fingerprint>"
bytedcli --json bytedoc collections --service "example.bytedoc.demo_orders"
bytedcli --json bytedoc shell --service "example.bytedoc.demo_orders" --collection "demo_items" --query 'find({"tenant":"demo"}).limit(10)'
```

## Optimization Rules

- Apply ESR thinking where appropriate: equality predicates, sort fields, then range predicates.
- Do not recommend blind indexes for every field; each index must map to a repeated query shape.
- Do not tune pool size or timeout values until query execution evidence rules out query shape and index issues.
- Do not rely on Atlas Performance Advisor; use ByteDoc slow-query and metadata surfaces instead.
- If explain is unavailable or unsafe, state that the recommendation is based on slow-query/schema/index evidence rather than an execution plan.

## Output Format

- Summarize the current query shape.
- List evidence gathered and gaps.
- Give safe query rewrites first.
- Give index recommendations only with trade-offs.
- Provide verification commands the user or agent can run after changes.
