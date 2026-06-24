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

## 🔴 Private repos + automerge — read before using the default preset

**TL;DR: if your repo is private and your org is not on a paid GitHub plan (Team or Enterprise), use the [conservative preset](#conservative-preset) instead.**

### Why automerge: true is unsafe without enforced branch protection

Renovate's automerge works by merging a PR as soon as it is eligible. "Eligible" means GitHub's branch merge conditions are satisfied. Those conditions come from branch protection rules — specifically: required status checks must pass, required reviewers must approve.

**GitHub's free plan does not enforce branch protection rules in private org repos.** Without enforcement, there are no conditions to satisfy. Renovate will merge the PR immediately after opening it — with failing CI, zero reviews, and no human ever seeing it.

This is not a Renovate bug. Renovate is correctly using the GitHub API. The GitHub API allows the merge because the branch protection rules aren't enforced.

### How to check

If your org is on a free plan and any of your repos are private:

1. Go to the repo → Settings → Branches → protection rule for `main`/`master`
2. If you can't enable "Require status checks" or "Require a pull request before merging" at all, or if they're greyed out — your plan doesn't enforce them
3. Use the conservative preset for that repo

### Conservative preset

Inherits all grouping rules, lockFileMaintenance, labels, and custom managers from the default preset. Disables all automerge. Safe for any repo regardless of branch protection setup.

```jsonc
{
    "$schema": "https://docs.renovatebot.com/renovate-schema.json",
    "extends": [
        "github>ArkanisCorporation/renovate-config:conservative"
    ],
    "reviewers": [
        "team:your-team-here"
    ]
}
```

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
- `conservative.json` — the preset loaded by `github>ArkanisCorporation/renovate-config:conservative`; same as default but all automerge disabled
- `example-consumer.jsonc` — copy this into a consumer repo as `renovate.jsonc` and fill in the team slug

> **Note:** `default.json` uses JSONC (JSON with comments). Renovate's config parser is JSONC-aware and handles this correctly. If a future Renovate version rejects inline comments in preset files, the same documentation lives in this README.
