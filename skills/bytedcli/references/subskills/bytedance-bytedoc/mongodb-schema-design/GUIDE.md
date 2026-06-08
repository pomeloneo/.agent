# MongoDB Schema Design Guidance for ByteDoc

Use this guide when reviewing ByteDoc collection shape, deciding whether to embed or reference data, diagnosing document growth, or giving schema/index design advice for generated SDK code.

This guide adapts MongoDB official `mongodb-schema-design` agent-skill guidance for ByteDoc. See `../references/mongodb-upstream-adaptation.md` for update rules.

## ByteDoc Boundaries

- bytedcli gives schema and design advice; it must not automatically modify business schemas.
- Do not create collections, indexes, validators, or migration scripts unless the user explicitly requests that operation and confirms the target environment.
- ByteDoc access authorization, site/vRegion routing, and network boundaries remain prerequisites for any schema investigation.

## Evidence To Gather

- Collection names and representative sample documents.
- Field cardinality, array growth risk, and typical query filters/sorts.
- Existing indexes and slow-query fingerprints when performance is part of the question.
- Data lifecycle requirements such as TTL, archive, retention, and compliance constraints.

## Design Rules

- Model around access patterns, not around relational normalization by default.
- Embed data that is read together and has bounded growth.
- Reference data when embedded arrays can grow without bound, when update frequency differs greatly, or when ownership boundaries differ.
- Avoid unbounded arrays in hot documents; use child collections or bucketing patterns when growth is user/event driven.
- Keep documents under MongoDB document size limits and avoid returning large subtrees by default.
- Add TTL or archival design only when retention semantics are clear and supported by the target ByteDoc backend.

## ByteDoc-Specific Guidance

- Use `mongodb-natural-language-querying/GUIDE.md` to derive real query shapes before recommending schema changes.
- Use `mongodb-query-optimizer/GUIDE.md` when the schema question is triggered by slow queries.
- For generated Go SDK code, keep structs minimal and aligned with the fields actually read or written.
- Prefer additive migration guidance and validation steps; do not suggest destructive schema rewrites as a first step.

## Output Format

- State the observed access patterns and schema evidence.
- Identify document growth, cardinality, and index risks.
- Recommend one primary design and one alternative when trade-offs are meaningful.
- Include migration/verification steps, but keep execution as a separate user-confirmed task.
