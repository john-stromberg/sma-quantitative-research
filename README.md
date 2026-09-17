# SMA Quantitative Research

**Portfolio index** for quantitative SMA research projects across **equities, bonds, alternatives, ETFs, and mutual funds**.

This repository serves as an organizational hub and landing page. Each child project is maintained as an independent repository.

## 📚 Documentation & Guides

**Quick Links**:
- 🏗️ **[Architecture Guide](docs/ARCHITECTURE.md)** - System design, patterns, interfaces
- 📋 **[Best Practices](docs/BEST_PRACTICES.md)** - Research discipline, testing, code quality
- 📋 **[Repository Template](TEMPLATE.md)** - Structure & setup for new projects
- ✅ **[Implementation Complete](IMPLEMENTATION_COMPLETE.md)** - Deliverables summary & status

**For New Researchers**: Start with [TEMPLATE.md](TEMPLATE.md) for project structure and [BEST_PRACTICES.md](docs/BEST_PRACTICES.md) for workflow.

**For Portfolio Managers**: See example [Decision Memo](multi-asset-optimization-lab/reports/) and [Initial Outputs Summary](multi-asset-optimization-lab/reports/INITIAL_OUTPUTS_SUMMARY.md).

**For DevOps**: Check [CI/CD Pipeline](multi-asset-optimization-lab/.github/workflows/tests.yml) and [Deployment Guide](multi-asset-optimization-lab/docs/DEPLOYMENT.md).

## Shared Infrastructure

### sma-quant-core

**Foundational library** for all quantitative research projects.

Provides:
- **Data Models**: Asset, Portfolio, Constraints, TimeSeries, FactorModel, OptimizedPortfolio, BacktestResult
- **Interfaces**: IOptimizer, IConstraint, ISolver, IBacktester, IMetrics (language-neutral contracts)
- **Metrics**: Sharpe Ratio, Sortino Ratio, Information Ratio, Max Drawdown, portfolio analytics
- **Backtesting**: Event-driven simulator with performance attribution, slippage, and commissions
- **Reporting**: Decision memo templates, efficient frontier reports, backtest summaries
- **Sample Data**: 5-asset portfolio (equities, bonds, alternatives) with constraints and correlations

**Installation**:
```bash
pip install -e "path/to/sma-quant-core[dev]"
```

**Quick Start**:
```python
from sma_quant_core.models import Asset
from sma_quant_core.metrics import Metrics

assets = [Asset("VTSAX", "US Stock", "equity", 0.08, 0.15)]
weights = [1.0]
metrics = Metrics.portfolio_metrics(weights, assets)
```

Repository: `sma-quant-core/` (local)

---

## Projects

### Portfolio Optimization & Construction

- **[multi-asset-optimization-lab](https://github.com/john-stromberg/multi-asset-optimization-lab)**  
  Portfolio optimization under constraints and multi-asset frameworks. Outputs: efficient frontiers, optimized weights, sensitivity tables.

- **[portfolio-construction-case-study-suite](https://github.com/john-stromberg/portfolio-construction-case-study-suite)**  
  End-to-end applied quantitative case studies. Outputs: decision memos, case outputs, comparative studies.

### Risk & Exposure Analysis

- **[portfolio-risk-exposure-decomposition](https://github.com/john-stromberg/portfolio-risk-exposure-decomposition)**  
  Factor and asset-class exposure decomposition. Outputs: exposure heatmaps, contribution tables.

- **[risk-attribution-engine](https://github.com/john-stromberg/risk-attribution-engine)**  
  Risk contribution decomposition and analysis. Outputs: marginal and component risk contribution outputs.

- **[exposure-aware-rebalancing-optimizer](https://github.com/john-stromberg/exposure-aware-rebalancing-optimizer)**  
  Rebalancing with exposure and risk constraints. Outputs: risk budget vs turnover trade-off analysis.

### Hedging & Overlay

- **[beta-hedging-engine](https://github.com/john-stromberg/beta-hedging-engine)**  
  Beta overlay and hedge-ratio frameworks. Outputs: hedge ratios, residual beta diagnostics.

- **[fx-hedging-and-overlay-lab](https://github.com/john-stromberg/fx-hedging-and-overlay-lab)**  
  Currency exposure and hedge policy analysis. Outputs: hedge policy scenarios, net FX risk views.

### Performance Analysis

- **[performance-attribution-lab](https://github.com/john-stromberg/performance-attribution-lab)**  
  Return attribution analytics and decomposition. Outputs: Brinson/factor attribution summaries.

## Research Principles

- **Methodology-first**: Define the research problem and approach before implementation.
- **Research → Production separation**: Use git branches (develop=research, main=production) with validation gates.
- **Reproducibility**: Version-controlled data, pinned dependencies, containerization, comprehensive documentation.
- **Methodological clarity**: Define assumptions, model choices, and robustness checks.
- **Explainable outputs**: Connect quant results to portfolio-management implications.
- **Applied focus**: Research artifacts inform actionable portfolio decisions.

## Repository Structure

Each research repository follows this pattern:

```
project/
├── .github/workflows/      # CI/CD pipelines
├── src/                    # Production-ready code
├── research/               # Exploratory notebooks & scripts
├── tests/                  # Unit & integration tests
├── data/                   # Sample & versioned data
├── reports/                # Generated decision memos
├── setup.py                # Python packaging
├── requirements.txt        # Pinned dependencies
└── README.md
```

## Development Workflow

1. **Create feature branch** from develop
2. **Implement & test** with pytest
3. **Validate** against sma-quant-core interfaces
4. **Merge to develop** (research staging)
5. **PR to main** with methodology memo + test results
6. **Merge after review** (production)

## Getting Started

To use the research infrastructure:

```bash
# Install shared core (from sma-quant-core directory)
pip install -e ".[dev]"

# Run tests
pytest tests/ -v

# Install a research project
pip install -e "path/to/research-project[dev]"

# Explore research notebooks
jupyter notebook research/
```

## Best Practices

1. **Define the question**: What portfolio decision are you optimizing?
2. **Document assumptions**: Asset models, constraints, correlations, risk-free rate
3. **Backtest rigorously**: Use realistic slippage, commissions, market conditions
4. **Test edge cases**: Extreme scenarios, constraint conflicts, data gaps
5. **Report robustness**: Sensitivity analysis, parameter sweeps, confidence intervals
6. **Decision memos**: Distill research into actionable portfolio recommendations
7. **Code review**: Peer review before promoting to production



