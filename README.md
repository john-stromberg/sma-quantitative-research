# SMA Quantitative Research

Portfolio hub for quantitative SMA research projects across **equities, bonds, alternatives, ETFs, and mutual funds**.

## Objective

This parent repository organizes research-focused projects that demonstrate:
- portfolio construction and optimization depth,
- rigorous treatment of risk and exposure concepts,
- practical application of beta/FX hedging and attribution frameworks.

## Research principles

- **Methodological clarity:** define assumptions, model choices, and robustness checks.
- **Explainable outputs:** connect quant results to portfolio-management implications.
- **Reproducibility:** code, data assumptions, and evaluation steps should be repeatable.
- **Applied focus:** research artifacts should inform actionable portfolio decisions.

## Repository map

| Repository | Primary focus | Typical outputs |
|---|---|---|
| `multi-asset-optimization-lab` | Portfolio optimization under constraints | Efficient frontiers, optimized weights, sensitivity tables |
| `portfolio-risk-exposure-decomposition` | Factor and asset-class exposure decomposition | Exposure heatmaps, contribution tables |
| `beta-hedging-engine` | Beta overlay and hedge-ratio frameworks | Hedge ratios, residual beta diagnostics |
| `fx-hedging-and-overlay-lab` | Currency exposure and hedge policy analysis | Hedge policy scenarios, net FX risk views |
| `risk-attribution-engine` | Risk contribution decomposition | Marginal/component risk contribution outputs |
| `performance-attribution-lab` | Return attribution analytics | Brinson/factor attribution summaries |
| `exposure-aware-rebalancing-optimizer` | Rebalancing with exposure/risk constraints | Trade-offs between risk budget and turnover |
| `portfolio-construction-case-study-suite` | End-to-end applied quant case studies | Decision memos, case outputs, comparative studies |

## Standard research workflow

1. Frame hypothesis and portfolio objective.
2. Specify model and constraints.
3. Run baseline and robustness variants.
4. Translate findings into implementation guidance.

## Decision memo format

- **Question:** Which portfolio choice is being tested?
- **Evidence:** Which model outputs are decisive?
- **Interpretation:** What is robust vs sample-specific?
- **Action:** Adopt, calibrate, or reject approach.

## Submodule usage

After cloning this parent repository:

```bash
git submodule update --init --recursive
```

