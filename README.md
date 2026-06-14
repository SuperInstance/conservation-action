# SuperInstance Conservation Action

> Conservation-law governance (γ + η ≤ C) as a GitHub Action for any repository.

[![GitHub Action](https://img.shields.io/badge/action-v1-blue?logo=github-actions)](https://github.com/SuperInstance/conservation-action)
[![License: MIT](https://img.shields.io/badge/license-MIT-green)](LICENSE)

Enforce **conservation invariants** in your CI/CD pipeline. The SuperInstance Conservation Action checks that coordination cost (γ) and entropy produced (η) remain within the conservation budget (C), blocking merges that violate system-level constraints.

---

## How It Works

The conservation law states:

```
γ + η ≤ C
```

Where:
- **γ (gamma)** — coordination cost: how much cross-team alignment a change requires
- **η (eta)** — entropy produced: disorder / unpredictability introduced into the system
- **C** — conservation budget (default: 1.0)

When γ + η exceeds C, the system has violated conservation and the CI check fails.

The action calls the `superinstance-mcp` tool to evaluate the invariant, then surfaces the result in your workflow.

---

## Quick Start

```yaml
- uses: SuperInstance/conservation-action@v1
  with:
    gamma: 0.8
    eta: 0.5
```

This will fail the job if `0.8 + 0.5 = 1.3 > 1.0`.

---

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `gamma` | Gamma (γ) value — coordination cost | ✅ | `0.5` |
| `eta` | Eta (η) value — entropy produced | ✅ | `0.5` |
| `fail-on-violation` | Fail the job if conservation is violated | ❌ | `true` |
| `fleet-api-url` | Fleet dashboard API URL (optional) | ❌ | `https://fleet-dashboard-api.casey-digennaro.workers.dev` |

### Outputs

| Output | Description |
|--------|-------------|
| `valid` | `true` if conservation holds, `false` if violated |

---

## Usage Examples

### 1. Basic Conservation Gate

Block PRs that violate the conservation budget:

```yaml
name: PR Checks

on:
  pull_request:
    branches: [main]

jobs:
  conservation:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Conservation gate
        uses: SuperInstance/conservation-action@v1
        with:
          gamma: 0.7
          eta: 0.4
          fail-on-violation: true
```

### 2. Dynamic γ/η from PR Diff Size

Compute conservation costs from the actual change size:

```yaml
name: Dynamic Conservation

on:
  pull_request:
    branches: [main]

jobs:
  conservation:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Estimate conservation cost
        id: estimate
        run: |
          LINES=$(git diff --numstat origin/main...HEAD | awk '{added+=$1; removed+=$2} END {print added+removed}')
          GAMMA=$(echo "scale=4; $LINES / 10000" | bc -l)
          # Clamp to [0, 1]
          if (( $(echo "$GAMMA > 1" | bc -l) )); then GAMMA=1; fi
          echo "gamma=$GAMMA" >> "$GITHUB_OUTPUT"
          echo "Computed γ=$GAMMA from $LINES changed lines"

      - name: Conservation check
        uses: SuperInstance/conservation-action@v1
        with:
          gamma: ${{ steps.estimate.outputs.gamma }}
          eta: 0.3
```

### 3. Matrix Builds with Per-Job Conservation

Apply different conservation budgets per job in a matrix:

```yaml
name: Matrix Conservation

on: [pull_request]

jobs:
  check:
    strategy:
      matrix:
        include:
          - component: frontend
            gamma: 0.3
            eta: 0.2
          - component: backend
            gamma: 0.6
            eta: 0.4
          - component: infra
            gamma: 0.8
            eta: 0.5
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Conservation check (${{ matrix.component }})
        uses: SuperInstance/conservation-action@v1
        with:
          gamma: ${{ matrix.gamma }}
          eta: ${{ matrix.eta }}
```

### 4. Combined with Other SuperInstance Tools

Chain conservation checks with fleet operations:

```yaml
name: Full Governance

on:
  pull_request:
    branches: [main]

jobs:
  governance:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # Step 1: Estimate coordination cost from diff
      - name: Estimate γ
        id: gamma
        run: |
          FILES=$(git diff --name-only origin/main...HEAD | wc -l)
          GAMMA=$(echo "scale=4; $FILES / 20" | bc -l)
          if (( $(echo "$GAMMA > 1" | bc -l) )); then GAMMA=1; fi
          echo "gamma=$GAMMA" >> "$GITHUB_OUTPUT"

      # Step 2: Conservation gate
      - name: Conservation check
        id: conservation
        uses: SuperInstance/conservation-action@v1
        with:
          gamma: ${{ steps.gamma.outputs.gamma }}
          eta: 0.4
          fail-on-violation: false

      # Step 3: Conditional fleet deployment
      - name: Deploy via Fleet
        if: steps.conservation.outputs.valid == 'true'
        run: |
          echo "✅ Conservation holds — deploying"
          # npx -y superinstance-mcp deploy ...
          echo "Deployed successfully"

      - name: Alert on violation
        if: steps.conservation.outputs.valid != 'true'
        run: |
          echo "::warning::Conservation violated — deployment skipped"
          # Optionally notify Slack, create issue, etc.
```

### 5. Soft Gate (Warn but Don't Fail)

Use `fail-on-violation: false` to surface violations without blocking:

```yaml
- uses: SuperInstance/conservation-action@v1
  with:
    gamma: 0.9
    eta: 0.8
    fail-on-violation: false
```

The check will still report the result via outputs, but won't fail the job.

---

## How γ and η Map to Real Metrics

| Metric | Symbol | Typical Source |
|--------|--------|----------------|
| Coordination cost | γ | PR diff size, files changed, teams touched |
| Entropy produced | η | Test flakiness, config churn, dependency changes |
| Conservation budget | C | Fixed at 1.0 (normalized) |

You decide how to compute γ and η for your project — the action just enforces the invariant.

### Common γ Estimators

```bash
# Lines changed → γ
LINES=$(git diff --numstat origin/main...HEAD | awk '{a+=$1; r+=$2} END {print a+r}')
GAMMA=$(echo "scale=4; $LINES / 10000" | bc -l)

# Files changed → γ
FILES=$(git diff --name-only origin/main...HEAD | wc -l)
GAMMA=$(echo "scale=4; $FILES / 20" | bc -l)

# Commits in PR → γ
COMMITS=$(git rev-list --count origin/main...HEAD)
GAMMA=$(echo "scale=4; $COMMITS / 10" | bc -l)
```

### Common η Estimators

```bash
# Config files changed → η
CONFIG_CHANGES=$(git diff --name-only origin/main...HEAD | grep -cE '\.(json|ya?ml|toml|env)$' || true)
ETA=$(echo "scale=4; $CONFIG_CHANGES / 5" | bc -l)

# Dependencies changed → η
DEP_CHANGES=$(git diff --name-only origin/main...HEAD | grep -cE '(package\.json|Cargo\.toml|go\.mod|requirements\.txt)' || true)
ETA=$(echo "scale=4; ($DEP_CHANGES * 0.3)" | bc -l)
```

---

## License

MIT
