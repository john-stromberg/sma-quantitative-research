# SMA Quantitative Research: Architecture & Best Practices

## Repository Overview

The SMA Quantitative Research hub contains a shared infrastructure library and research-grade projects:

### Shared Infrastructure: sma-quant-core

**Purpose**: Foundational library for all quantitative research

**Provides**:
- Data models (Asset, Portfolio, Constraint, TimeSeries, FactorModel, OptimizedPortfolio, BacktestResult)
- Interfaces (IOptimizer, IConstraint, ISolver, IBacktester, IMetrics)
- Metrics calculations (Sharpe, Sortino, Information Ratio, Max Drawdown, portfolio analytics)
- Event-driven backtester with performance attribution
- Decision memo templates and reporting utilities
- Sample data (versioned in repo)

**Location**: `C:\Users\johns\source\repos\sma-quant-core`

**Usage**:
```python
from sma_quant_core.models import Asset, PortfolioConstraints
from sma_quant_core.metrics import Metrics

assets = [Asset("VTSAX", "US Stock", "equity", 0.08, 0.15)]
weights = [1.0]
metrics = Metrics.portfolio_metrics(weights, assets)
```

### Research Project: multi-asset-optimization-lab

**Purpose**: Demonstrate portfolio optimization best practices

**Implements**: Mean-variance optimizer (IOptimizer interface)

**Features**:
- Constrained optimization (max Sharpe, min volatility, target return)
- Efficient frontier generation
- Automatic decision memo generation
- End-to-end workflow (optimize → validate → report)

**Locations**:
- Research: `research/` (exploratory notebooks)
- Production: `src/` (validated code)
- Tests: `tests/` (unit + integration + robustness)
- Outputs: `reports/` (decision memos, portfolios)

## Architecture Patterns

### 1. Methodology-First Design

**Define the research question before implementation**:
- What portfolio decision are we optimizing?
- What data/assumptions drive the decision?
- What metrics validate the approach?

**Example Decision Memo Structure**:
```
Question    → What is the optimal allocation?
Evidence    → Mean-variance optimization on {N} assets
Interpretation → Results show {X}% return, {Y}% volatility, {Z} Sharpe ratio
Recommendation → Allocate as follows: {asset_id: weight, ...}
Risks       → Model assumes {assumption}, backtested on {period}, does not include {caveat}
```

### 2. Research → Production Separation

**Three phases**:

**Phase 1: Exploration** (feature branches)
- Write exploratory notebooks
- Experiment with parameters and models
- Commit to branch (not main)

**Phase 2: Validation** (develop branch)
- Implement production code (src/)
- Write unit + integration tests
- Run robustness checks + backtests
- Get code review approval
- **Key gate**: All tests passing, coverage ≥ 80%, Sharpe ≥ 0.4

**Phase 3: Production** (main branch)
- Merge after validation gates pass
- Tag release
- Monitor metrics

**Branch strategy**:
```
main             [v0.1.0] ← validated, tested, deployed
  ↑
develop          ← staging area for validated research
  ↑
feature/xxx      ← experimental branches (exploratory)
```

### 3. Shared Interfaces & Contracts

Every optimizer must implement `sma_quant_core.interfaces.IOptimizer`:

```python
class IOptimizer(ABC):
	@abstractmethod
	def optimize(
		self,
		assets: List[Asset],
		constraints: PortfolioConstraints,
		correlation_matrix: Optional[np.ndarray] = None,
		risk_free_rate: float = 0.02,
		**kwargs
	) -> OptimizedPortfolio:
		"""Optimize portfolio weights."""
		pass
```

**Benefits**:
- Interchangeable implementations (fallback vs cvxpy vs Julia)
- Same API across all research projects
- Easy testing and mocking

### 4. Reproducibility & Versioning

**Data versioning**:
- Sample assets, constraints, correlations in `data/` (JSON)
- Pinned dependencies in `pyproject.toml`
- Tests use versioned data to ensure consistent results

**Code versioning**:
- Semantic versioning: v{major}.{minor}.{patch}
- Git tags for releases: `git tag v0.2.0`
- Changelog documenting breaking changes

**Docker containerization**:
- Dockerfile for reproducible environments
- Docker Compose for local dev/testing
- Enables "works on my machine" → "works everywhere"

## Development Workflow

### For New Researchers

1. **Install shared infrastructure**:
   ```bash
   pip install -e "path/to/sma-quant-core[dev]"
   ```

2. **Clone research project**:
   ```bash
   git clone https://github.com/john-stromberg/multi-asset-optimization-lab.git
   cd multi-asset-optimization-lab
   ```

3. **Explore research notebooks**:
   ```bash
   jupyter notebook research/01_optimization_exploration.ipynb
   ```

4. **Run tests**:
   ```bash
   pytest tests/ -v
   ```

5. **Create new feature branch**:
   ```bash
   git checkout -b feature/my-new-optimizer
   ```

### For Code Reviews

**Checklist**:
- ✓ Tests passing (automated via CI/CD)
- ✓ Coverage ≥ 80%
- ✓ Code style: black, flake8
- ✓ Type hints with mypy
- ✓ Docstrings on all public functions
- ✓ Assumptions documented in code comments
- ✓ Edge cases handled (empty portfolios, zero volatility, etc.)
- ✓ Decision memo clear and actionable

## Validation Framework

**Four-stage validation**:

1. **Unit Tests**
   - Test individual components
   - Run on every commit

2. **Integration Tests**
   - Test end-to-end workflows
   - Validate data flows

3. **Robustness Checks**
   - Sensitivity analysis (±10% return, ±15% volatility)
   - Edge cases (infeasible constraints, zero-vol assets)
   - Manual review on PRs to main

4. **Backtesting**
   - Historical performance validation
   - Realistic slippage/commissions
   - Success criteria: Sharpe ≥ 0.4, positive cumulative return
   - Manual review on PRs to main

**Gates to production**:
- ✓ All tests passing (automated)
- ✓ Coverage ≥ 80% (automated)
- ✓ Robustness stable under perturbations (manual)
- ✓ Backtest meets success metrics (manual)
- ✓ Code reviewed and approved (manual)

## Key Files & References

### Shared Core
- `sma-quant-core/sma_quant_core/models.py` - Data structures
- `sma-quant-core/sma_quant_core/interfaces.py` - Abstract base classes
- `sma-quant-core/sma_quant_core/metrics.py` - KPI calculations
- `sma-quant-core/sma_quant_core/backtester.py` - Event-driven simulator

### Research Project (multi-asset-optimization-lab)
- `src/mean_variance_optimizer.py` - Production optimizer
- `research/01_optimization_exploration.ipynb` - Research notebook
- `tests/test_mean_variance.py` - Unit tests
- `tests/test_integration.py` - End-to-end tests
- `tests/test_robustness.py` - Sensitivity + edge case tests
- `docs/WORKFLOW.md` - This workflow documentation
- `docs/VALIDATION_FRAMEWORK.md` - Validation gates
- `docs/DEPLOYMENT.md` - CI/CD and containerization

## Best Practices

### 1. Testing
- Write tests for new features before implementation (TDD)
- Aim for ≥ 80% code coverage
- Include edge cases and error conditions
- Use descriptive test names: `test_optimize_with_infeasible_constraints`

### 2. Documentation
- Docstrings on all public functions (Google style)
- Assumptions documented in code comments
- README with purpose, quick start, architecture
- Decision memo for significant changes

### 3. Code Quality
- Format code: `black src/`
- Check style: `flake8 src/`
- Type hints: `mypy src/`
- Run before commits: `pre-commit run --all-files`

### 4. Git Hygiene
- Small, focused commits
- Descriptive commit messages: `feat: add efficient frontier`, `fix: handle zero volatility`
- Pull requests before merging to main
- Squash trivial commits

### 5. Research Discipline
- State hypotheses clearly
- Document all assumptions
- Run sensitivity analysis
- Report limitations and caveats
- Avoid curve-fitting (correlations may not persist)

## Extending the Framework

### Adding a New Optimizer

1. **Create new file**: `src/my_optimizer.py`
2. **Implement IOptimizer interface**:
   ```python
   from sma_quant_core.interfaces import IOptimizer

   class MyOptimizer(IOptimizer):
	   def optimize(self, assets, constraints, correlation_matrix=None, risk_free_rate=0.02, **kwargs):
		   # Implementation
		   return OptimizedPortfolio(...)
   ```
3. **Write tests**: `tests/test_my_optimizer.py`
4. **Add to CLI**: Update `src/main.py` to accept your optimizer
5. **Document in README**

### Adding a New Research Project

**Use this template**:
```
new-research-project/
├── .github/workflows/tests.yml      # CI/CD
├── src/                             # Production code
│   └── my_algorithm.py
├── research/                        # Notebooks
│   └── 01_exploration.ipynb
├── tests/                           # Tests
│   ├── test_algorithm.py
│   └── test_integration.py
├── data/                            # Sample data
│   └── sample_inputs.json
├── docs/                            # Documentation
│   ├── WORKFLOW.md
│   └── VALIDATION_FRAMEWORK.md
├── pyproject.toml                   # Dependencies
├── Dockerfile                       # Containerization
└── README.md
```

## Q&A

**Q: When should I move research to production?**
- A: When all validation gates pass (tests ✓, robustness ✓, backtest ✓, review ✓)

**Q: How do I handle dependencies between research projects?**
- A: Via sma-quant-core shared library. Define interfaces in core; implement in projects.

**Q: What if a production optimizer breaks?**
- A: Rollback to previous version, investigate root cause, deploy fix to develop first.

**Q: How long does research → production typically take?**
- A: 1-2 weeks: exploration (3-5 days) + validation (2-3 days) + review (1-2 days)

## References

- Markowitz, H. (1952). "Portfolio Selection" - Foundational mean-variance paper
- Boyd, S., Vandenberghe, L. (2004). "Convex Optimization" - Optimization theory
- cvxpy Documentation: https://www.cvxpy.org/
- sma-quant-core: https://github.com/john-stromberg/sma-quant-core
