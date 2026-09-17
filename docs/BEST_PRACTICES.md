# SMA Quantitative Research: Best Practices Guide

## Table of Contents

1. [Methodology & Research Discipline](#methodology--research-discipline)
2. [Shared Infrastructure Architecture](#shared-infrastructure-architecture)
3. [Research → Production Workflow](#research--production-workflow)
4. [Testing & Validation Strategy](#testing--validation-strategy)
5. [Code Quality Standards](#code-quality-standards)
6. [Data Management & Reproducibility](#data-management--reproducibility)
7. [Documentation Standards](#documentation-standards)
8. [Team Roles & Responsibilities](#team-roles--responsibilities)
9. [Troubleshooting & Support](#troubleshooting--support)

---

## Methodology & Research Discipline

### Starting with a Research Question

All research projects begin with a clear hypothesis and research question:

**Good**:
- "What allocation maximizes risk-adjusted returns subject to sector constraints?"
- "Does mean-variance optimization outperform risk parity on real data?"
- "How stable is efficient frontier under different correlation assumptions?"

**Avoid**:
- "Let's try some optimization" (unclear objective)
- "See what the data says" (no control)
- "Implement this paper" (without understanding limitations)

### Documenting Assumptions

Maintain a "Assumptions Register" in your research notebook:

```markdown
### Key Assumptions
- Price returns are normally distributed (violated in practice)
- Correlations are stable over time (test in robustness checks)
- No transaction costs (add to backtest)
- Risk-free rate = 2% annually (calibrate to real rates)
- Portfolio rebalances quarterly (test monthly/daily)
```

### Testing Hypotheses Rigorously

Use the scientific method:

1. **Hypothesis**: "Mean-variance beats risk parity"
2. **Prediction**: "Sharpe ratio will be ≥ 0.5"
3. **Experiment**: Historical backtest on 10 years
4. **Results**: Sharpe = 0.48, p-value = 0.15 (not significant)
5. **Conclusion**: Insufficient evidence; hypothesis rejected

**Key principle**: Confirm hypotheses with statistical significance, not anecdotal evidence.

---

## Shared Infrastructure Architecture

### Data Model Layer

All research uses shared dataclasses from `sma_quant_core.models`:

```
┌──────────────────────────────────────────┐
│        sma_quant_core.models             │
├──────────────────────────────────────────┤
│ Asset (ticker, return, volatility, ...)  │
│ PortfolioConstraints (weight bounds, ...) │
│ OptimizedPortfolio (weights, metrics)    │
│ TimeSeries (prices, returns over time)   │
│ FactorModel (factor loadings, betas)     │
│ BacktestResult (NAV, returns, metrics)   │
└──────────────────────────────────────────┘
					↓
			(used by all layers)
					↓
┌──────────────────────────────────────────┐
│   Downstream Research Algorithms          │
│ (Optimizer, Factor Model, Risk System)    │
└──────────────────────────────────────────┘
```

**Benefits**:
- Consistent data encoding across projects
- Type safety (Python dataclasses with validation)
- Easy serialization (to/from JSON)
- Standardized schemas for APIs

### Interface Layer

Each capability is defined as an abstract interface:

```python
class IOptimizer(ABC):
	"""Portfolio optimizer interface."""
	@abstractmethod
	def optimize(self, assets, constraints, ...) -> OptimizedPortfolio:
		pass

class IBacktester(ABC):
	"""Strategy backtester interface."""
	@abstractmethod
	def run(self, strategy, historical_prices, ...) -> BacktestResult:
		pass

class IMetrics(ABC):
	"""Performance metrics interface."""
	@abstractmethod
	def sharpe_ratio(self, returns, risk_free_rate) -> float:
		pass
```

**Benefits**:
- Implementations can vary (cvxpy, scipy, custom)
- Easy testing (mock interfaces)
- Contract-driven development

### Metrics & Analytics Layer

Reusable calculation utilities:

```python
from sma_quant_core.metrics import Metrics

metrics = Metrics.portfolio_metrics(
	weights=[0.4, 0.3, 0.2, 0.1],
	assets=[VTSAX, VTIAX, BND, VGSLX],
	correlation_matrix=corr_matrix
)
print(f"Sharpe: {metrics['sharpe_ratio']}")
print(f"Max Drawdown: {metrics['max_drawdown']}")
```

---

## Research → Production Workflow

### Phase 1: Exploration (Research Branch)

**Timeline**: 3-5 days

**Deliverables**:
- Jupyter notebook with exploratory analysis
- Preliminary results and visualizations
- Key findings and open questions

**Git Branch**: `feature/my-research`

**Example Notebook Structure**:
```python
# Cell 1: Imports & setup
# Cell 2: Load sample data
# Cell 3: Define research question
# Cell 4: Basic algorithm / analysis
# Cell 5: Compete vs benchmarks
# Cell 6: Sensitivity analysis
# Cell 7: Key Findings & Next Steps
```

**Review Criteria**:
- ✓ Code is clear and reproducible
- ✓ Findings are grounded in evidence
- ✓ Limitations are acknowledged

### Phase 2: Validation (Develop Branch)

**Timeline**: 2-3 weeks

**Activities**:
1. Extract code from notebook → production module
2. Write unit tests (target ≥80% coverage)
3. Write integration tests (end-to-end workflows)
4. Run robustness checks (sensitivity analysis)
5. Backtest on historical data
6. Code review by peer

**Validation Gates**:
| Gate | Criteria | Owner |
|------|----------|-------|
| Unit Tests | All pass, coverage ≥80% | Automated (CI/CD) |
| Integration | All workflows pass | Automated (CI/CD) |
| Robustness | Stable under perturbations | Manual (reviewer) |
| Backtest | Sharpe ≥0.4, positive return | Manual (reviewer) |
| Code Review | Approved by peer | Manual (reviewer) |

**Example Validation Memo**:
```markdown
## Validation Report: Mean-Variance Optimizer

✓ Unit Tests: 5/5 passing, 85% coverage
✓ Integration: 4/4 workflows passing
✓ Robustness: stable under ±10% return perturbations
✓ Backtest: Sharpe 0.59, cumulative return 15.3%
✓ Review: Approved by [Reviewer]

**Recommendation**: Ready for production merge
**Release Date**: 2026-09-20
```

### Phase 3: Production (Main Branch)

**Timeline**: 1 week

**Activities**:
1. Merge to main
2. Tag release (v0.2.0)
3. Build Docker image
4. Deploy to production
5. Monitor metrics

**Monitoring Dashboard**:
- Build success rate
- Test pass rate
- Code coverage
- Deployment frequency
- Optimization runtime
- Sharpe ratio trend
- Allocation stability

---

## Testing & Validation Strategy

### Test Pyramid

```
		/\
	   /  \  Integration Tests (4)
	  /____\
	 /      \
	/  Unit  \  Unit Tests (5)
   /  Tests   \
  /____________\

Base: 1 line of code = 1+ test lines
```

### Unit Test Examples

```python
def test_optimizer_initialization():
	"""Test optimizer can be instantiated."""
	optimizer = MeanVarianceOptimizer()
	assert optimizer is not None

def test_sharpe_objective():
	"""Test max-Sharpe optimization."""
	portfolio = optimizer.optimize(assets, constraints, objective="max_sharpe")
	assert portfolio.sharpe_ratio >= 0.3

def test_weight_constraints():
	"""Test weight bounds are respected."""
	portfolio = optimizer.optimize(assets, constraints)
	assert all(w >= 0 for w in portfolio.weights.values())
	assert all(w <= 1 for w in portfolio.weights.values())
	assert abs(sum(portfolio.weights.values()) - 1.0) < 1e-6
```

### Integration Test Examples

```python
def test_full_workflow():
	"""Test complete optimization workflow."""
	# Load data
	assets = load_sample_assets()
	constraints = load_sample_constraints()

	# Optimize
	optimizer = MeanVarianceOptimizer()
	portfolio = optimizer.optimize(assets, constraints)

	# Validate
	assert portfolio.sharpe_ratio > 0.4
	assert portfolio.expected_return > 0.05

	# Report
	memo = portfolio.generate_decision_memo()
	assert "Recommendation" in memo

def test_sensitivity_analysis():
	"""Test portfolio stability under return perturbations."""
	base_return = 0.08
	portfolio1 = optimize_with_return(base_return)
	portfolio2 = optimize_with_return(base_return * 0.95)

	# Allocations should not change drastically
	allocation_diff = sum(abs(w1 - w2) for w1, w2 in zip(portfolio1, portfolio2))
	assert allocation_diff < 0.1  # Less than 10% total reallocation
```

### Robustness Checks

Run before merging to main:

```python
def test_return_perturbations():
	"""Portfolio should be stable under ±10% return changes."""
	base = optimize_with_returns([0.08, 0.08, 0.04, 0.07, 0.05])
	perturbed_low = optimize_with_returns([0.072, 0.072, 0.036, 0.063, 0.045])
	perturbed_high = optimize_with_returns([0.088, 0.088, 0.044, 0.077, 0.055])

	# Allocations should not change by more than 5% per asset
	for base_w, low_w, high_w in zip(base, perturbed_low, perturbed_high):
		assert abs(base_w - low_w) < 0.05
		assert abs(base_w - high_w) < 0.05

def test_edge_cases():
	"""Test handling of edge cases."""
	# Empty portfolio
	with pytest.raises(ValueError):
		optimizer.optimize([])

	# Infeasible constraints
	constraints = PortfolioConstraints()
	constraints.add_weight_constraint("VTSAX", 1.0, 1.0)
	constraints.add_weight_constraint("VTIAX", 1.0, 1.0)
	portfolio = optimizer.optimize(assets, constraints)
	assert portfolio is not None  # Should return something valid
```

---

## Code Quality Standards

### Code Style

**Format with black**:
```bash
black src/ tests/
```

Configuration in `pyproject.toml`:
```toml
[tool.black]
line-length = 100
target-version = ["py310"]
```

**Lint with flake8**:
```bash
flake8 src/ tests/ --max-line-length=100
```

### Type Hints

All public functions must have type hints:

```python
# Good
def optimize(
	self,
	assets: List[Asset],
	constraints: PortfolioConstraints,
	correlation_matrix: Optional[np.ndarray] = None,
	risk_free_rate: float = 0.02,
) -> OptimizedPortfolio:
	"""Optimize portfolio weights.

	Args:
		assets: List of assets to include
		constraints: Weight and portfolio constraints
		correlation_matrix: Optional correlation matrix (default: identity)
		risk_free_rate: Risk-free rate for Sharpe calculation (default: 0.02)

	Returns:
		OptimizedPortfolio with weights and metrics
	"""
	...

# Bad
def optimize(assets, constraints):
	...
```

**Type check with mypy**:
```bash
mypy src/ --strict
```

### Docstrings

Google-style docstrings for all public functions and classes:

```python
def sharpe_ratio(returns: np.ndarray, risk_free_rate: float = 0.02) -> float:
	"""Calculate annualized Sharpe ratio.

	Args:
		returns: Array of period returns (e.g., monthly)
		risk_free_rate: Annual risk-free rate (default: 0.02)

	Returns:
		Annualized Sharpe ratio

	Raises:
		ValueError: If returns array is empty

	Example:
		>>> returns = np.array([0.01, 0.02, -0.01, 0.03])
		>>> sharpe = sharpe_ratio(returns)
		>>> print(f"Sharpe: {sharpe:.2f}")
	"""
	...
```

### Error Handling

Meaningful error messages:

```python
# Good
if len(assets) == 0:
	raise ValueError("Assets list cannot be empty; need at least 1 asset")

if not isinstance(correlation_matrix, np.ndarray):
	raise TypeError(f"correlation_matrix must be np.ndarray, got {type(correlation_matrix)}")

# Bad
if len(assets) == 0:
	raise ValueError("error")
```

---

## Data Management & Reproducibility

### Version Control

**Code versioning**:
- Semantic versioning: v{major}.{minor}.{patch}
- Tag releases: `git tag v0.2.0 -m "Release: Mean-variance optimizer"`

**Data versioning**:
- Store sample data in repo (JSON, CSV)
- Document data source and date
- Include version in file names: `sample_assets_v1.0.json`

**Dependency versioning**:
- `pyproject.toml` with pinned versions
- `requirements-dev.txt` for reproducible dev environment

### Reproducibility Checklist

- [ ] Data is versioned and tracked in repo
- [ ] Random seeds are set for reproducible results
- [ ] Tests use deterministic data (no live market data)
- [ ] Environment is containerized (Dockerfile)
- [ ] README includes exact commands to reproduce results
- [ ] All assumptions (returns, volatilities, correlations) are documented
- [ ] Results can be generated by running `python src/main.py`

---

## Documentation Standards

### README for Research Projects

Every research project needs a README:

```markdown
# My Research Project

## Overview
Brief description of the problem, approach, key findings.

## Architecture
How the algorithm works, key components, assumptions.

## Quick Start
Installation, running tests, generating outputs.

## Results
Key metrics, backtesting performance, comparison to benchmarks.

## References
Academic papers, related work, data sources.
```

### Decision Memos for Portfolio Managers

Every optimization run produces a decision memo:

```markdown
# Decision Memo: Portfolio Optimization Results

**Date**: September 16, 2026
**Analyst**: Quantitative Research Team

## Question
What is the optimal asset allocation under current constraints?

## Evidence
- Mean-variance optimization on 5-asset universe
- 3-year backtest with quarterly rebalancing
- Historical correlation matrix (5-year lookback)

## Interpretation
Optimal allocation achieves:
- Expected Return: 5.6%
- Expected Volatility: 6.1%
- Sharpe Ratio: 0.59

Allocation:
- VTSAX: 20%
- VTIAX: 20%
- BND: 20%
- VGSLX: 20%
- GLD: 20%

## Recommendation
Implement this allocation. Rebalance quarterly or when any position drifts >5%.

## Risks
- Model assumes correlations will persist (backtest on live data)
- Equal-weight allocation suggests cvxpy not installed (install for true optimization)
- Does not include dynamic hedging (future enhancement)
```

### Notebook Documentation

Research notebooks should be self-contained and reproducible:

1. **Introduction**: Problem statement and hypothesis
2. **Data**: Load and describe data, assumptions
3. **Methodology**: Algorithm design, parameters
4. **Analysis**: Experimental results
5. **Findings**: Key takeaways, limitations
6. **Next Steps**: Open questions for production implementation

---

## Team Roles & Responsibilities

### Quantitative Researcher
- ✓ Formulate research hypotheses
- ✓ Develop algorithms in notebooks
- ✓ Document assumptions
- ✓ Check statistical significance
- ✓ Create decision memos

### Software Engineer
- ✓ Refactor research code to production quality
- ✓ Write unit/integration tests
- ✓ Set up CI/CD pipelines
- ✓ Containerize applications
- ✓ Monitor production metrics

### Portfolio Manager
- ✓ Define optimization objectives and constraints
- ✓ Review decision memos
- ✓ Validate against real-world performance
- ✓ Provide feedback on results

### DevOps Engineer
- ✓ Manage infrastructure and deployments
- ✓ Monitor system health
- ✓ Configure alerts
- ✓ Support production troubleshooting

---

## Troubleshooting & Support

### Common Issues

**Q: Tests pass locally but fail in CI/CD**
- Check Python version (CI uses 3.10/3.11)
- Verify dependencies in pyproject.toml
- Look for hardcoded paths (use relative paths)
- Check environment variables

**Q: Optimization is too slow**
- Profile code: `python -m cProfile -s cumtime src/main.py`
- Install cvxpy: `pip install cvxpy`
- Consider approximate algorithms (random search, genetic algorithms)

**Q: Backtest results are unrealistic**
- Add transaction costs (0.05% slippage, 0.1% commission)
- Use realistic slippage models (impact on large trades)
- Check for look-ahead bias (can't use future data)
- Validate against paper trading results

**Q: Coverage is below 80%**
- Use `pytest --cov=src --cov-report=html` to identify gaps
- Prioritize testing critical algorithms first
- Add edge case tests (empty inputs, boundary conditions)

### Support Resources

- **Documentation**: `docs/ARCHITECTURE.md`, `docs/WORKFLOW.md`
- **Code Examples**: `multi-asset-optimization-lab/research/01_optimization_exploration.ipynb`
- **Template**: `sma-quantitative-research/TEMPLATE.md`
- **Core Library**: `sma-quant-core/README.md`
- **Issues**: Create GitHub issue in the relevant repository

---

## Appendix: Checklist for Production Readiness

Before merging to main:

- [ ] **Code**
  - [ ] Passes linting (flake8, black)
  - [ ] Passes type checking (mypy)
  - [ ] No hardcoded values or debug prints

- [ ] **Testing**
  - [ ] Unit tests: ≥80% coverage
  - [ ] Integration tests: all workflows pass
  - [ ] Robustness: stable under perturbations
  - [ ] Backtest: Sharpe ≥0.4, positive return

- [ ] **Documentation**
  - [ ] README is complete and accurate
  - [ ] Docstrings on all public functions
  - [ ] Assumptions documented
  - [ ] Decision memo is comprehensive

- [ ] **Reproducibility**
  - [ ] All dependencies pinned
  - [ ] Random seeds set for reproducibility
  - [ ] Can run `python src/main.py` successfully
  - [ ] Docker image builds without errors

- [ ] **Review**
  - [ ] Peer code review approved
  - [ ] PM has reviewed decision memo
  - [ ] No outstanding feedback

✓ **Ready for production!**

---

**Document Version**: 1.0
**Last Updated**: September 16, 2026
**Maintained By**: Quantitative Research Team
