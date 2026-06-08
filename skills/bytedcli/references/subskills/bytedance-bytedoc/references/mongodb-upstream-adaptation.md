# MongoDB Upstream Skill Adaptation

ByteDoc reuses MongoDB driver and query concepts, but ByteDoc access is governed by PSM authorization, Consul / Mesh routing, ByteDoc control-plane metadata, and BOE / CN / vRegion network boundaries. The guides below adapt selected MongoDB official agent-skills into the `bytedance-bytedoc` skill so users do not need to install upstream MongoDB skills separately.

Go SDK code generation must follow ByteDoc official SDK docs and use `code.byted.org/bytedoc/mongo-go-driver`. Upstream MongoDB skills and docs are used for API-shape and quality guidance only, not as dependency-selection authority.

## Upstream Source

- Repository: `https://github.com/mongodb/agent-skills/tree/main/skills`
- ByteDoc SDK docs: `https://cloud.bytedance.net/docs/bytedoc/docs/63d767b87df7d2021dfbee21/64aebcd25310ab022710a9d5?x-resource-account=public&x-bc-region-id=bytedance`
- Adaptation rule: keep upstream skill boundary names when useful, but rewrite examples and decision rules for ByteDoc.
- Update workflow: when upstream changes, compare each corresponding upstream `SKILL.md` with the ByteDoc child `GUIDE.md`, keep generally applicable MongoDB guidance, and reject Atlas-only or MongoDB-MCP-only workflows unless ByteDoc has an equivalent bytedcli capability.

## Included Skills

- `mongodb-connection` -> `../mongodb-connection/GUIDE.md`
  - Included for client lifecycle, timeout, credential handling, pool guidance, and connection troubleshooting.
  - Adapted to use `bytedoc sdk plan` / `sdk generate`, ByteDoc PSM, token, temporary credential, Consul, Mesh, and network-boundary evidence.
- `mongodb-natural-language-querying` -> `../mongodb-natural-language-querying/GUIDE.md`
  - Included for transforming user intent into safe read queries.
  - Adapted to require ByteDoc schema / sample / index evidence before generating filters or projections.
- `mongodb-query-optimizer` -> `../mongodb-query-optimizer/GUIDE.md`
  - Included for slow-query, explain, filter-shape, sort, and index analysis.
  - Adapted to use ByteDoc slow-query and collection/index metadata instead of Atlas Performance Advisor.
- `mongodb-schema-design` -> `../mongodb-schema-design/GUIDE.md`
  - Included for data-model trade-offs such as embed vs reference, unbounded arrays, TTL, and validation.
  - Adapted to advisory-only output; bytedcli must not modify business schemas automatically.

## Excluded Skills

- `mongodb-atlas-stream-processing`
  - Excluded because Atlas Stream Processing is not a ByteDoc access path.
  - Revisit only if ByteDoc exposes an equivalent streaming or change-stream product surface through bytedcli.
- `mongodb-search-and-ai`
  - Excluded because Atlas Search, Vector Search, and AI search setup are not universally available in ByteDoc.
  - Revisit only after ByteDoc documents supported search/vector capabilities and safe CLI inspection commands.
- `mongodb-mcp-setup`
  - Excluded because users should not need MongoDB MCP setup for ByteDoc workflows.
  - ByteDoc workflows must prefer `bytedcli bytedoc ...` commands and the ByteDoc MCP/skill surfaces already bundled with bytedcli.

## Maintenance Checklist

- Preserve upstream skill names in child guide directory names when a guide maps to an upstream skill.
- Keep ByteDoc-specific constraints first: authorization, site/vRegion routing, network boundaries, and write-operation safety override generic MongoDB advice.
- Do not add Atlas-only product steps unless there is a ByteDoc equivalent and a safe bytedcli command.
- Use example PSMs such as `example.bytedoc.demo_orders`; never hard-code real production PSMs in guide examples.
- Run `npm run validate:skills` after editing these guides.
