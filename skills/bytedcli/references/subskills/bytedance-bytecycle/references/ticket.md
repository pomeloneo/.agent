# Ticket commands

Top-level: `bytedcli bytecycle ticket ...`

All write subcommands print a dry-run payload first; pass `--yes` to execute.

## Read

```bash
bytedcli bytecycle ticket get --ticket-id <id>
bytedcli bytecycle ticket list \
  --workspace-id <id> \
  --train-id <id> \
  --status wip \
  --creator <username>
bytedcli bytecycle ticket pipeline list --ticket-id <id> --region cn
```

`ticket list` filters: `--train-id`, `--feature-id`, `--workspace-id`, `--status`, `--creator`, `--page`, `--page-size`.

## Create / apply

Ticket apply/create payloads are project-specific (`workspace_id`, `feature_info`, `change_items`, `ticket_vars`, `ticket_template_vars` all vary by team). The CLI does not ship a built-in profile builder for these yet.

Do not expose private apply request details to agents. Add a wrapped command with a typed payload builder before automating this flow.

## Edit ticket

```bash
bytedcli bytecycle ticket update --ticket-id <id> \
  --title "new title" \
  --description "..." \
  --branch feat-x \
  --template-var-key context.params.example_key \
  --template-var-value new-value
  # add --yes to execute
```

Or supply a full payload:

```bash
bytedcli bytecycle ticket update --payload-file ticket-edit.json --yes
```

The handler fetches ticket detail first, normalizes the editable payload, and patches the requested fields. `--template-var-key` and `--template-var-value` must be passed together; they replace `ticket_template_vars[0]`.

## Boarding flow

```bash
bytedcli bytecycle ticket wip board --wip-ticket-id <id> --train-id <id>
bytedcli bytecycle ticket approve   --ticket-id <id>
bytedcli bytecycle ticket reject --ticket-id <id> --note "..."
```

## Edit ticket variables

```bash
bytedcli bytecycle ticket vars update \
  --ticket-id <id> \
  --payload-file ticket-vars.json
```

The payload file must contain the full `ticket_vars` object.

## Boarding warning

```bash
bytedcli bytecycle ticket warning get --ticket-id <id>
```

Read-only. Returns the same warnings the page would show before letting the user click "board".

## Off-board: revert + approve/reject

```bash
bytedcli bytecycle ticket revert request --ticket-id <a> --ticket-id <b>
bytedcli bytecycle ticket revert approve --ticket-id <id>
bytedcli bytecycle ticket revert reject  --ticket-id <id>
```

All three dry-run by default; pass `--yes` to execute.

## Re-merge & pipeline create-and-build

```bash
bytedcli bytecycle ticket remerge \
  --train-id <id> \
  --ticket-id <a> --ticket-id <b>

bytedcli bytecycle ticket pipeline create-and-build \
  --ticket-id <id> \
  --region cn
```

When you need to trigger an already-created ticket pipeline build, use `bytedcli bytecycle pipeline trigger-ticket ...` -- see `pipeline.md`.
