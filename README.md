# ArkanisCorporation/renovate-config

Shared [Renovate](https://docs.renovatebot.com/) preset for all Arkanis Corporation repositories.

## Usage

Add one line to your repo's `renovate.jsonc`:

```jsonc
{
    "$schema": "https://docs.renovatebot.com/renovate-schema.json",
    "extends": [
        "github>ArkanisCorporation/renovate-config"
    ],
    "reviewers": [
        "team:your-team-here"
    ]
}
```

That's it. All grouping rules, automerge policy, lockFileMaintenance, and custom managers are inherited. Updates to this preset roll out automatically on the next Renovate run — no PR required in consumer repos.

---

## ⚠️ Behavior change from old per-repo configs

If your repo previously had its own `renovate.json` with automerge enabled for minor/patch updates, **two things now work differently**:

### 1. NuGet (.NET) updates: grouped PR, manual review required

**Before:** each package update got its own PR and automerged on minor/patch.

**After:** all NuGet minor/patch updates are batched into a single **".NET dependencies and tools"** PR. This PR requires manual approval — it does **not** automerge.

**Why:** a grouped PR lets you assess the full picture before merging. The volume is manageable in one review, and it's worth a human glance before dependencies shift.

**To restore automerge** (the group remains; the PR just merges automatically once all checks pass), add to your `renovate.jsonc`:

```jsonc
"packageRules": [
    { "matchManagers": ["nuget"], "automerge": true },
]
```

### 2. GitHub Actions updates: grouped PR, manual review required

Same deal. All GitHub Actions minor/patch/digest updates batch into a single **"GitHub Actions"** PR requiring approval.

**To restore automerge:**

```jsonc
"packageRules": [
    { "matchManagers": ["github-actions"], "automerge": true },
]
```

> Both opt-out lines are already present as commented-out blocks in `example-consumer.jsonc` — copy that file as your starting point so you have them right where you need them.

---

## What the preset includes

| Rule | Behavior |
|------|----------|
| `extends: config:recommended` | Renovate defaults + dependency dashboard |
| `labels: ["automated"]` | tag on all Renovate PRs |
| `lockFileMaintenance` | weekly lock-file refresh PR |
| minor/patch/pin/digest | automerge (overridden for groups below) |
| **GitHub Actions** | grouped, manual review |
| **.NET / NuGet** | grouped, manual review |
| semantic-release toolchain | grouped, manual review |
| Kubernetes tools (kubectl, helm) | grouped, manual review |
| major | never automerge |
| custom managers | tracks `semantic_version`, `kubectl_version`, `helm_version` pins in workflow YAML |

---

## Per-repo overrides

All `extends`-based rules can be overridden locally. Examples:

**Add extra label:**
```jsonc
"labels": ["automated", "build/publish"]
```

**Re-enable NuGet automerge:**
```jsonc
"packageRules": [
    { "matchManagers": ["nuget"], "automerge": true },
]
```

**Add a repo-specific package grouping:**
```jsonc
"packageRules": [
    { "matchDepNames": ["MyCompany.Sdk.*"], "groupName": "MyCompany SDK", "automerge": false }
]
```

Per-repo `packageRules` entries are **appended** to the preset's rules. Later rules override earlier ones, so per-repo entries take priority.

---

## Files

- `default.json` — the preset loaded by `github>ArkanisCorporation/renovate-config`
- `example-consumer.jsonc` — copy this into a consumer repo as `renovate.jsonc` and fill in the team slug

> **Note:** `default.json` uses JSONC (JSON with comments). Renovate's config parser is JSONC-aware and handles this correctly. If a future Renovate version rejects inline comments in preset files, the same documentation lives in this README.
