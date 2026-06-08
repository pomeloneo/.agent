# ByteDoc SDK Access Guide

Use this guide when the user wants to access a ByteDoc database from code, asks how to connect after authorization, or provides only a ByteDoc PSM and asks for SDK integration help.

## Core Workflow

- For Go SDK generation requests, first call `bytedcli --json bytedoc sdk plan --service <psm> --access-env <env> --language go`; add `--backend classic|cloud-native|volc` when the target may be ambiguous. Omit `--language go` only for pure access-plan questions that do not ask for code.
- For `tce` and `faas`, pass `--runtime-network boe|cn` when the user already knows the business runtime network.
- If the user did not specify the access environment, use `--access-env auto`.
- If `tce/faas` is known but the runtime network is not known, do not ask the user first when a caller TCE PSM is available; pass `--caller-psm <caller-psm>` so the CLI can inspect TCE/Keel evidence and fill `runtimeNetwork`.
- Map clear BOE service evidence to `--runtime-network boe`, and clear CN / online_cn / prod CN evidence to `--runtime-network cn`; if the service appears in multiple environments and the target DB has a known network, accept the CLI's `target-network-fallback` result and explain the assumption from `warnings[]`.
- Only omit `--runtime-network` and ask the returned `questions[]` when both user input and TCE/ENV lookup cannot identify the runtime network.
- Use `--reference-mode fixed` by default. Use `--reference-mode hybrid` when the user explicitly wants supplemental ByteCloud document search results or when stale fixed links are a concern.
- Treat `sdk plan` as the source of truth for ByteDoc access method, prerequisites, warnings, and next actions.
- If `sdk plan` returns `references[]`, show each reference title, URL, and reason to the user; use fixed references as evidence for unsupported or ambiguous conclusions.
- If `referenceSearch.status=success`, present dynamic Top3 references as supplemental material only; do not let dynamic results override the deterministic access matrix.
- If `referenceSearch.status=error`, continue with the fixed references and mention that supplemental search was unavailable.
- Do not generate SDK code when `supportStatus` is `unsupported` or `needs_input`.
- Treat bytedcli as the ByteDoc expert tool, not the coding agent that edits business projects.
- Do not scan or modify the user's project from bytedcli; coding agents apply the returned plan and code material to the project after user confirmation.

## Access Env Values

- `local-mac`: local Mac development.
- `devbox`: DevBox development.
- `tce`: TCE runtime.
- `faas`: ByteFaaS runtime.
- `visual`: visualization-only access.
- `auto`: unknown access environment; use this only when the user has not provided enough context.

## Current Matrix

- Keep `accessEnv` and `runtimeNetwork` separate: `accessEnv` describes local Mac, DevBox, TCE, FaaS, or visual access; `runtimeNetwork` describes whether TCE/FaaS runs in BOE or CN.
- CN classic from CN TCE/FaaS uses `consul-token`.
- CN classic from BOE local development or BOE TCE/FaaS is unsupported because the network is not reachable.
- BOE Volc MongoDB 4.0 from BOE local development or BOE TCE/FaaS uses `consul-token`.
- BOE Volc MongoDB 8.0 from local development uses `consul-temporary-credential`; credentials are for development only.
- BOE Volc MongoDB 8.0 from BOE TCE/FaaS uses `mesh-token` and requires ByteDoc Service Mesh egress.
- BOE Volc from CN TCE/FaaS is unsupported because the network is not reachable.
- Visualization-only access uses `visual-query` and must not trigger SDK code generation.

## Go SDK Material

- Use `bytedcli --json bytedoc sdk generate --service <psm> --access-env <env> --runtime-network <boe|cn> --language go --collection <collection> --operation find-one` only after `sdk plan` is supported; pass the same `--backend` used in `sdk plan` if backend disambiguation was required.
- Omit `--runtime-network` for `local-mac`, `devbox`, and `visual`; include it for `tce` and `faas`.
- Treat `codeSamples[]` as material for the coding agent to adapt into the business repository.
- When the user asks for SDK code, Go code, or code material, include the full `codeSamples[].content` in the final answer before summarizing dependencies, env vars, or verification commands.
- Always include an "接入指导" section after the code. Use `integrationGuidance[]` as the stable source, then summarize the selected access method, runtime network assumptions, collection placeholder/real value, verification path, and next human-friendly action.
- Do not omit "接入指导" even when the answer also includes permission status, authorization dry-run output, warnings, or SDK doctor diagnosis.
- If `BYTEDOC_COLLECTION` is still a placeholder, do not dump only CLI commands; say "如果不知道 collection 名，可以让我继续帮你查询一下".
- Generated Go code must use ByteDoc official driver `code.byted.org/bytedoc/mongo-go-driver`, not the upstream `go.mongodb.org/mongo-driver` module directly.
- Read and apply `relatedGuides[]` from `sdk generate`; these point to ByteDoc-adapted MongoDB official skill guidance.
- Do not write generated code into the user's repository from bytedcli.
- Preserve the quality guidance: reuse `mongo.Client`, use `context.WithTimeout`, avoid logging tokens or full URIs, and avoid write operations unless the user explicitly requests them.

## Authorization Check Placement

- Treat PSM authorization as a runtime readiness check, not a blocker for SDK code material generation.
- If the user provides a caller PSM and target ByteDoc PSM, complete `sdk plan` and `sdk generate` first whenever the access matrix is supported.
- After code material, run or recommend the `bytedoc access` flow to inspect authorization: `access role list`, then `access permission get`, then `access psm create` dry-run only if authorization is missing and the user wants help applying.
- If authorization already exists, state the current permission and continue with verification commands.
- If authorization is missing, do not replace the generated code with an authorization warning; append a final "权限状态" section that asks whether to proceed with assisted authorization.
- Dry-run authorization may be run proactively, but live submission requires explicit user confirmation. After confirmation, execute the internal command yourself; do not tell the user to copy `--execute --yes-i-know-this-is-live` commands.
- For `backend=volc`, do not use classic-only `access role list` / `access permission get` as proof of existing authorization. Use `BYTEDOC_ACCESS_REQUIRED.details.setup_commands` or `access psm create --backend volc` dry-run to prepare the authorization preview, and state clearly if existing Volc permissions cannot yet be read from CLI.

## SDK Doctor

- Use `bytedcli --json bytedoc sdk doctor --error-text "<error text>"` when the user reports SDK connection, authentication, timeout, Consul, Mesh, DNS, or permission failures.
- Include `--service <psm>`, `--backend classic|cloud-native|volc`, `--access-env <env>`, and `--runtime-network <boe|cn>` when the user has provided them; omit only when unknown.
- Treat the returned `category`, `severity`, `evidence`, `likelyCauses`, `nextActions`, and `verificationCommands` as diagnosis material for the coding agent and user.
- If the category is `permission`, follow the returned `bytedoc access ...` dry-run workflow before creating tickets.
- If the category is `network` or `credential`, run `sdk plan` and compare the actual code/runtime environment with the returned access method.
- Do not scan or modify business code from bytedcli while diagnosing; coding agents may inspect the user project after presenting the CLI diagnosis.

## Adapted MongoDB Guides

- `mongodb-connection/GUIDE.md`: use for client lifecycle, context timeouts, token / temporary credential handling, and connection troubleshooting.
- `mongodb-natural-language-querying/GUIDE.md`: use when adapting user query intent into bounded read filters, projections, and sample queries.
- `mongodb-query-optimizer/GUIDE.md`: use for slow-query, index, explain, sort, and query-shape analysis.
- `mongodb-schema-design/GUIDE.md`: use when generated SDK structs or query requirements reveal schema design trade-offs.
- The upstream source and excluded MongoDB official skills are documented in `references/mongodb-upstream-adaptation.md`.
- ByteDoc SDK package choices are based on ByteDoc official docs: `https://cloud.bytedance.net/docs/bytedoc/docs/63d767b87df7d2021dfbee21/64aebcd25310ab022710a9d5?x-resource-account=public&x-bc-region-id=bytedance`.

## Agent Rules

- Always explain unsupported results instead of trying to work around them with guessed connection strings.
- For ambiguous results, ask the `questions[]` returned by CLI.
- Keep ByteDoc-specific decisions above generic MongoDB best practices; Mesh, Consul, Token, temporary credentials, PSM authorization, and BOE/CN network boundaries are decided by ByteDoc.
- MongoDB official agent-skill guidance is already adapted into ByteDoc rules; do not require users to install upstream MongoDB skills.

## Backend-Aware Resolution

- ByteDoc has three backends (`classic`, `cloud-native`, `volc`) with different metadata sources; `sdk plan` must reuse the resolver-decided source instead of unconditionally calling the legacy classic / cloud-native detail API.
- For `backend=volc` (DBW / Volc Mongo), use the Cloud Service Search summary returned by the resolver; do not call `getDatabaseDetail` against the cloud-native `/api/service/:service` endpoint, which returns `mongo: no documents in result` for Volc databases.
- When adding new ByteDoc service flows that need database overview, prefer `fetchDatabaseOverviewForQuery` (or an equivalent backend-aware helper) so DBW / Volc paths bypass the classic detail API.
- Tests must cover the path where Cloud Service Search resolves a database but the legacy detail API does not support it (typical Volc case).
