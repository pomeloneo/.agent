---
name: bytedance-lagrange
description: "Use when tasks involve Lagrange Admin Torch SCM releases, ICM image builds, build-history payloads, cpu/cuda/mlu Torch version selection, use-cache, skip-arm, ICM rebuild, or custom image options through bytedcli lagrange."
---

# Lagrange

Use `bytedcli lagrange` for Lagrange Admin operations. The current command surface covers the lg-admin Torch release page.

## Torch Releases

SCM release, matching the build history form:

```bash
bytedcli lagrange torch release scm build \
  --branch demo-branch \
  --platform cpu=2.7 \
  --platform cuda=2.10 \
  --platform mlu=2.10 \
  --comment dev
```

Submit with `--yes`; without it the command fetches meta and permission, then renders a dry-run summary.

Useful SCM release flags:

- `--release-type DEV|LTS`
- `--pub-base branch|tag|commit`, plus `--branch`, `--git-tag`, or `--git-commit`
- `--regions cn,va`
- `--platform cpu=2.10`, repeatable for `cpu`, `cuda`, `mlu`, or other supported platforms
- `--use-cache` / `--no-use-cache`
- `--skip-arm`, `--skip-arch <arch>`
- `--icm-mode default|rebuild|custom|custom-cmd`
- `--image platform=namespace/image:tag` for `--icm-mode custom`
- `--cmd-option platform=cmd` for `--icm-mode custom-cmd`
- `--skip-permission`

ICM release, matching the ICM build history form:

```bash
bytedcli lagrange torch release icm build \
  --branch demo-branch \
  --compute-platforms cpu,cuda,mlu \
  --regions cn \
  --comment dev
```

Query an ICM release task:

```bash
bytedcli lagrange torch release icm get --task-id 821
```

Useful ICM release flags:

- `--release-type DEV|LTS`
- `--pub-base branch|tag|commit`, plus `--branch`, `--git-tag`, or `--git-commit`
- `--regions cn,va`
- `--compute-platforms cpu,cuda,mlu`
- `--build-describe <text>`
- `--yes`

## Auth And Headers

The native commands use the ByteCloud personal JWT flow and send lg-admin headers:

- `X-Jwt-Token`
- `X-Lgx-Admin-Control-Plane`, default `cn`
- `X-Lgx-Admin-Region`, default `cn`
- `X-Lgx-Admin-Domain-Id`, default `online`

Override API or header values with `--host`, `--control-plane`, `--admin-region`, and `--domain-id`.
