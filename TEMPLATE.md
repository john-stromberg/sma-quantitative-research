# SMA Quantitative Research: Repository Template

This template demonstrates industry best practices for quantitative research projects.

## Quick Start (for new research projects)

1. **Copy this structure**:
   ```bash
   cp -r template/ my-research-project/
   cd my-research-project
   ```

2. **Install dependencies**:
   ```bash
   python -m venv .venv
   .venv\Scripts\Activate.ps1
   pip install -e "path/to/sma-quant-core[dev]"
   pip install -e ".[dev]"
   ```

3. **Run tests**:
   ```bash
   pytest tests/ -v
   ```

4. **Start exploring**:
   ```bash
   jupyter notebook research/01_exploration.ipynb
   ```

## Directory Structure

```
my-research-project/
├── .github/
│   └── workflows/
│       └── tests.yml                # CI/CD: auto-run tests on push/PR
├── src/
│   ├── __init__.py
│   ├── my_algorithm.py              # Main production code
│   └── main.py                      # CLI entry point
├── research/
│   └── 01_exploration.ipynb         # Exploratory notebook template
├── tests/
│   ├── __init__.py
│   ├── test_algorithm.py            # Unit tests
│   ├── test_integration.py          # End-to-end workflow tests
│   └── test_robustness.py           # Sensitivity & edge case tests
├── data/
│   ├── sample_inputs.json           # Versioned sample data
│   └── sample_outputs.json
├── reports/                         # Generated outputs
│   ├── decision_memos/
│   ├── backtests/
│   └── .gitkeep
├── docs/
│   ├── WORKFLOW.md                  # Research → Production workflow
│   ├── VALIDATION_FRAMEWORK.md      # Testing & validation strategy
│   └── DEPLOYMENT.md                # CI/CD & containerization
├── .github/
│   └── workflows/
│       └── tests.yml                # Automated testing
├── .gitignore
├── pyproject.toml                   # Project metadata & dependencies
├── Dockerfile                       # Container definition
├── docker-compose.yml               # Local dev & testing
├── README.md                        # Project overview
└── LICENSE
```

## File Templates

### pyproject.toml

```toml
[build-system]
requires = ["setuptools>=65.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "my-research-project"
version = "0.1.0"
description = "Description of your research"
authors = [{name = "your-name", email = "your-email"}]
requires-python = ">=3.10"
dependencies = [
	"numpy>=1.24.0",
	"pandas>=2.0.0",
	"scipy>=1.10.0",
	"sma-quant-core>=0.1.0",
]

[project.optional-dependencies]
# For algorithms requiring constrained optimization
optimization = ["cvxpy>=1.3.0"]
# For development (testing, linting, formatting)
dev = [
	"pytest>=7.4.0",
	"pytest-cov>=4.1.0",
	"black>=23.0.0",
	"flake8>=6.0.0",
	"mypy>=1.5.0",
	"jupyter>=1.0.0",
]

[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = "test_*.py"

[tool.black]
line-length = 100
target-version = ["py310"]

[tool.mypy]
python_version = "3.10"
warn_return_any = true
warn_unused_ignores = true
```

### README.md

```markdown
# My Research Project

## Overview

Brief description of your research, the problem you're solving, and the approach.

## Quick Start

```bash
pip install -e ".[dev]"
pytest tests/ -v
python src/main.py --help
```

## Architecture

Describe your algorithm, approach, key assumptions.

## Key Results

Early findings, metrics, performance vs benchmarks.

## References

Academic papers, related work, data sources.
```

### tests/test_algorithm.py

```python
"""Unit tests for the main algorithm."""

import pytest
import numpy as np
from src.my_algorithm import MyAlgorithm
from sma_quant_core.models import Asset

def test_algorithm_with_valid_inputs():
	"""Test algorithm returns valid output for valid inputs."""
	algorithm = MyAlgorithm()
	# Setup test data
	# Execute algorithm
	# Verify results
	assert output is not None

def test_algorithm_with_empty_inputs():
	"""Test algorithm gracefully handles empty inputs."""
	algorithm = MyAlgorithm()
	with pytest.raises(ValueError):
		algorithm.run([])
```

### research/01_exploration.ipynb

```python
# Cell 1: Imports
from src.my_algorithm import MyAlgorithm
from sma_quant_core.models import Asset
import numpy as np
import pandas as pd

# Cell 2: Load sample data
with open("data/sample_inputs.json") as f:
	data = json.load(f)

# Cell 3: Experiment with algorithm
algorithm = MyAlgorithm()
results = algorithm.run(data)
print(f"Results: {results}")

# Cell 4: Visualize findings
# Plots, tables, metrics

# Cell 5: Document findings
"""
Key Findings:
- Finding 1
- Finding 2

Next Steps:
- Investigation 1
- Investigation 2
"""
```

## Workflow Checklist

### Before Pushing (Local)

- [ ] Tests pass: `pytest tests/ -v`
- [ ] Coverage ≥ 80%: `pytest --cov=src`
- [ ] Code formatted: `black src/ tests/`
- [ ] Linting passes: `flake8 src/ tests/`
- [ ] Type hints valid: `mypy src/`
- [ ] No debug prints or TODOs

### Creating a PR

- [ ] Branch name is descriptive: `feature/new-algorithm`, `fix/edge-case`
- [ ] Commit messages are clear: `feat: add efficient frontier generation`
- [ ] PR description explains **what** and **why**
- [ ] Linked to any relevant issues

### Code Review Checklist

- [ ] Tests all passing (automated)
- [ ] Coverage ≥ 80%
- [ ] Code is readable and maintainable
- [ ] Assumptions documented
- [ ] Edge cases handled
- [ ] Docstrings present
- [ ] Decision memo clear and actionable

### Before Merging to Main

- [ ] All gates passed (tests ✓, review ✓)
- [ ] Robustness checks confirm stable behavior
- [ ] Backtest meets success criteria
- [ ] Release version incremented
- [ ] Changelog updated
- [ ] README reflects current state

## Validation Gates

| Stage | Criteria | Automated? |
|-------|----------|-----------|
| Unit Tests | All pass | ✓ |
| Integration | All pass | ✓ |
| Coverage | ≥ 80% | ✓ |
| Robustness | Stable under perturbations | Manual |
| Backtest | Sharpe ≥ 0.4, positive return | Manual |
| Code Review | Approved | Manual |

## Common Commands

```bash
# Install in development mode
pip install -e ".[dev]"

# Run tests
pytest tests/ -v
pytest tests/ --cov=src --cov-report=html

# Format code
black src/ tests/

# Lint
flake8 src/ tests/

# Type check
mypy src/

# Run algorithm
python src/main.py --help

# Start Jupyter
jupyter notebook research/

# Build Docker image
docker build -t my-research:latest .

# Run in Docker
docker-compose up
```

## Best Practices

1. **Write tests first** (TDD): Define what "success" means before implementing
2. **Document assumptions**: What correlations, volatilities, prices do we assume?
3. **Validate robustness**: How stable are results under small input changes?
4. **Backtest rigorously**: Test on historical data, measure realized returns
5. **Create decision memos**: Summarize findings for portfolio managers
6. **Version everything**: Code, data, dependencies, results
7. **Review carefully**: Peer review catches errors and improves code quality
8. **Monitor production**: Track metrics over time to catch regressions

## Extensions

### Adding a New Feature

1. Create feature branch: `git checkout -b feature/my-feature`
2. Write tests for the feature
3. Implement the feature
4. Ensure tests pass: `pytest`
5. Update documentation
6. Create PR and request review

### Making the Algorithm Production-Ready

1. Move code from `research/` notebook to `src/`
2. Add unit tests with ≥80% coverage
3. Add integration tests for full workflow
4. Run robustness checks
5. Backtest on historical data
6. Get code review approval
7. Merge to main and tag release

## Troubleshooting

**Q: Tests pass locally but fail in CI/CD**
- Check Python version compatibility
- Verify dependencies match pyproject.toml
- Check for hardcoded paths

**Q: Coverage is low**
- Add tests for untested code paths
- Use `pytest --cov=src --cov-report=html` to identify gaps
- Priority: test critical algorithms first

**Q: Algorithm is slow**
- Profile: `python -m cProfile -s cumtime src/main.py`
- Optimize algorithms: vectorize with numpy/pandas
- Consider approximate algorithms for production speed

## Support

For questions or issues:
- Refer to sma-quant-core documentation: `https://github.com/john-stromberg/sma-quant-core`
- Check SMA Quantitative Research hub: `https://github.com/john-stromberg/sma-quantitative-research`
- Create an issue in this repository

## License

[Your Organization] Internal Use Only
