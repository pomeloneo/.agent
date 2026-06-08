---
name: bytedance-safe-puzzle
description: Feature platform (puzzle) operations via bytedcli safe domain. Query features, entities, datasources, tenants, packages, collections, scripts, and tickets.
---

# Safe Puzzle — Feature Platform

Query feature metadata, entities, datasources, tenants, packages, collections, scripts, and tickets on the puzzle feature platform.

## Commands

### Feature

```bash
# List features (paginated, default 20 per page)
bytedcli safe puzzle feature list [--page N] [--page-size N]
bytedcli safe puzzle feature list --keyword <name-or-code>
bytedcli safe puzzle feature list --entity-ids 123 456
bytedcli safe puzzle feature list --collection-code <collection-code>
bytedcli safe puzzle feature list --production-type stats model

# Get feature by ID
bytedcli safe puzzle feature get --id <feature-id>

# List feature dependencies
bytedcli safe puzzle feature list-dependencies --id <feature-id>

# Get feature rule configuration
bytedcli safe puzzle feature get-rule-conf --id <feature-id>

# Update feature rule configuration
bytedcli safe puzzle feature update-rule-conf --id <feature-id> --content data.json

# Create feature draft
bytedcli safe puzzle feature create-draft --type ds --content data.json
bytedcli safe puzzle feature create-draft --type script --content '{"key":"value"}'
bytedcli safe puzzle feature create-draft --type spi --content data.json
bytedcli safe puzzle feature create-draft --type spi --content '{"key":"value"}'

# Test feature calculation
bytedcli safe puzzle feature test --id <feature-id> --entity-params <json>

# Search similar features
bytedcli safe puzzle feature similar-search --keyword <meaning>
```

### Entity

```bash
# List all entities
bytedcli safe puzzle entity list

# Get entity detail
bytedcli safe puzzle entity get --id <entity-id>
```

### Datasource

```bash
# List datasources (default type: rpc)
bytedcli safe puzzle datasource list
bytedcli safe puzzle datasource list --type redis
bytedcli safe puzzle datasource list --keyword <search-text>

# Get datasource detail
bytedcli safe puzzle datasource get --id <datasource-id>

# Create a datasource
bytedcli safe puzzle datasource create --type rpc --psm <psm> --method <method> --name <name>
bytedcli safe puzzle datasource create --type spi --psm <psm> --name <name>
bytedcli safe puzzle datasource create --type redis --psm <psm> --name <name>
bytedcli safe puzzle datasource create --type abase --psm <psm> --name <name>

# Search similar data sources
bytedcli safe puzzle datasource similar-search --keyword <meaning>

# Get datasource schema
bytedcli safe puzzle datasource get-schema --psm <psm> --method <method>

# Get datasource schema with semantic meaning to recall related knowledge
bytedcli safe puzzle datasource get-schema --psm <psm> --method <method> --meaning "<feature semantics>"

# Short alias
bytedcli safe puzzle ds list
```

### Tenant

```bash
# List tenants
bytedcli safe puzzle tenant list

# Include test tenants
bytedcli safe puzzle tenant list --all
```

### Package

```bash
# List packages (paginated, default 10 per page)
bytedcli safe puzzle package list
bytedcli safe puzzle pkg list --keyword <name-or-code> --page 2
bytedcli safe puzzle pkg list --keyword my-pkg --exact-match

# Get package detail
bytedcli safe puzzle pkg get --id <package-id>

# List binded features of a package
bytedcli safe puzzle pkg list-features --id <package-id>

# Add features to a package (圈选特征)
bytedcli safe puzzle package add-features --id <package-id> --collection-id <collection-id> --feature <entity-inst-code>.<feature-code>
bytedcli safe puzzle pkg add-features --id <pkg-id> --collection-id <col-id> --feature <inst-code>.<feat-code> --tenant <tenant>
```

### Collection

```bash
# List feature collections (default: related to me)
bytedcli safe puzzle collection list
bytedcli safe puzzle collection list --keyword <name-or-code>
bytedcli safe puzzle collection list --related-to-me false

# Get collection detail
bytedcli safe puzzle collection get --id <collection-id>

# List collection features
bytedcli safe puzzle collection list-features --id <collection-id>

# Add a parameter to a collection
bytedcli safe puzzle collection add-param --id <collection-id> --key <key> --name <name> --value-type <type>
bytedcli safe puzzle collection add-param --id <collection-id> --key my_param --name 我的参数 --value-type string
bytedcli safe puzzle collection add-param --id <collection-id> --key my_param --name 我的参数 --value-type int --desc "my description"
```

### Script

```bash
# Generate script template
bytedcli safe puzzle script generate-template --type req --ds-type rpc

# Compile and analyze script content
bytedcli safe puzzle script compile --content ./script.go --entity-id 123
bytedcli safe puzzle script compile --content "package main\n..." --collection-id 456
```

### Ticket

```bash
# Create release ticket
bytedcli safe puzzle ticket create --id <feature-id>
bytedcli safe puzzle ticket create --id <feature-id> --type urgent

# List puzzle tickets
bytedcli safe puzzle ticket list
bytedcli safe puzzle ticket list --keyword demo --page 2
```

## Common Options

All puzzle sub-commands support `--tenant <tenant>` to specify the tenant for API requests. Priority: `--tenant` > `SAFE_TENANT` env > config (`bytedcli safe config set --key tenant --value <tenant>`) > default `ecology`.

## References

- [puzzle-api.md](references/puzzle-api.md) — API reference
- [puzzle-guide.md](references/puzzle-guide.md) — Usage guide
