# CROSS-POLLINATION.md — conservation-action

> **Conservation Law Connection:** γ + η ≤ C enforcement as CI/CD

## Role in the Conservation Law

`conservation-action` IS the conservation law as a GitHub Action. It enforces
γ + η ≤ C on every push, PR, and deployment. This is the **governance layer**:
the mathematical law becomes executable policy.

The action measures:
- **γ metrics:** test pass rate, code coverage, build success (productive output)
- **η metrics:** build time, dependency count, CI minutes consumed (overhead)
- **C budget:** configurable per-repo, defaults to (1 − δ(team_size)) × max_γ

If γ + η > C: the action **fails the build**, preventing conservation law violations
from reaching production.

## delta-clt Verification Results

The delta-clt suite provides the theoretical basis for the action's thresholds:

- For a team of 5 developers: C budget should account for δ(5) = 0.334 overhead
- For a team of 50: δ(50) = 0.121 — much tighter enforcement possible
- Adversarial detection: if η > adversarial_threshold + 2×δ(n), flag as attack

The action should use `delta_theoretical(n)` from delta-clt to dynamically set
the C budget based on active contributor count.

## Cross-Repo Connections

### → ALL repos in the cluster
`conservation-action` should be installed on every repo. It is the universal
conservation law enforcer. Each repo's CROSS-POLLINATION.md defines its γ and η;
the action enforces the sum.

### → ternary-fleet-integration
Integration tests produce γ measurements; `conservation-action` consumes them.
Together: integration tests run, produce γ/η metrics, action enforces the bound.

**Shared:** Both are CI infrastructure. Both serve the conservation law.
**Different:** Integration tests measure quality; action enforces the budget.

### → superinstance-protocol
The protocol carries conservation law metadata (γ/η headers). `conservation-action`
can validate protocol compliance: do wire-format γ/η values match CI-computed values?

**Shared:** Both encode conservation law as operational data.
**Different:** `protocol` is runtime; `action` is build-time.

### → conservation-languages
`conservation-languages` benchmarks conservation law across languages. The action
can use these benchmarks to set language-specific η thresholds (Rust has lower η
than Python, etc.).

**Shared:** Both establish conservation law baselines.
**Different:** `languages` studies; `action` enforces.

## Fleet Position

```
┌───────────────────────────────────────────────────────┐
│  conservation-action — THE ENFORCER                    │
│                                                        │
│  ┌─────────┐    ┌──────────┐    ┌────────────────┐    │
│  │  Push    │───►│  Tests   │───►│ γ measurement  │    │
│  └─────────┘    └──────────┘    └────────────────┘    │
│                                       │                │
│  ┌─────────┐    ┌──────────┐    ┌────▼───────────┐    │
│  │  Build  │───►│ CI time  │───►│ η measurement  │    │
│  └─────────┘    └──────────┘    └────────────────┘    │
│                                       │                │
│                          ┌────────────▼────────────┐   │
│                          │  γ + η ≤ C?             │   │
│                          │  YES → ✅ merge allowed  │   │
│                          │  NO  → ❌ build fails    │   │
│                          └─────────────────────────┘   │
│                                                        │
│  C budget: (1 − δ(n)) × max_γ, from delta-clt          │
│  Installed on: ALL repos in cluster                    │
└───────────────────────────────────────────────────────┘
```

