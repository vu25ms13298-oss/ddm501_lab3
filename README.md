# Lab 3: Testing & CI/CD for ML Systems

[![CI Pipeline](../../actions/workflows/ci.yml/badge.svg)](../../actions/workflows/ci.yml)

## Overview

Comprehensive testing strategies and CI/CD pipelines for the movie rating prediction system to ensure quality and automate deployment.

**Course:** DDM501 - AI in Production: From Models to Systems
**Weight:** 15% of total grade
**Format:** Team Lab (3-4 members per team)
**Prerequisites:** Lab 1 and Lab 2 completed

## Project Structure

```
ddm501-lab3-starter/
├── app/
│   ├── __init__.py
│   ├── main.py             # FastAPI application
│   ├── model.py            # ML model class (SVD collaborative filtering)
│   ├── schemas.py          # Pydantic schemas
│   └── config.py           # Configuration
├── tests/
│   ├── __init__.py
│   ├── conftest.py         # Shared fixtures
│   ├── unit/
│   │   ├── test_model.py   # Model unit tests (13 tests)
│   │   └── test_schemas.py # Schema tests (16 tests)
│   ├── integration/
│   │   └── test_api.py     # API endpoint tests (23 tests)
│   ├── data/
│   │   └── test_data_quality.py  # Data quality tests (17 tests)
│   └── model/
│       └── test_model_behavior.py  # Behavioral tests (20 tests)
├── .github/
│   └── workflows/
│       ├── ci.yml              # CI pipeline
│       ├── cd.yml              # CD pipeline
│       └── model-validation.yml # Model validation pipeline
├── scripts/
│   └── train_model.py      # Model training script
├── models/                 # Saved models (svd_model.pkl)
├── .pre-commit-config.yaml # Pre-commit hooks
├── pyproject.toml          # Project configuration
├── requirements.txt
├── requirements-dev.txt
├── Dockerfile
├── TESTING_STRATEGY.md     # Testing strategy document
└── README.md
```

## Quick Start

### 1. Setup

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate  # Linux/Mac
venv\Scripts\activate     # Windows

# Install dependencies
pip install -r requirements.txt
pip install -r requirements-dev.txt
```

### 2. Train Model

```bash
python scripts/train_model.py
```

### 3. Run Tests

```bash
# Run all tests
pytest tests/ -v

# Run with coverage
pytest tests/ -v --cov=app --cov-report=html

# Run specific test category
pytest tests/unit/ -v
pytest tests/integration/ -v
pytest tests/data/ -v
pytest tests/model/ -v
```

### 4. Code Quality Checks

```bash
# Install pre-commit hooks
pre-commit install

# Run all checks manually
pre-commit run --all-files

# Individual tools
black app/ tests/
flake8 app/ tests/ --max-line-length=100
isort app/ tests/
mypy app/ --ignore-missing-imports
```

### 5. Run the API

```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

## Test Summary

| Category | Tests | Description |
|----------|-------|-------------|
| Unit Tests | 29 | Model class, Pydantic schemas |
| Integration Tests | 23 | API endpoints, error handling |
| Data Tests | 17 | Data quality, distribution, uniqueness |
| Model Behavioral Tests | 20 | Invariance, directional, minimum functionality |
| **Total** | **89** | **84% code coverage** |

## CI/CD Pipeline

### CI Pipeline (`ci.yml`)
- **Lint**: flake8, black, isort
- **Type Check**: mypy
- **Test**: pytest with coverage (needs lint to pass first)
- **Build**: Docker image build and smoke test

### CD Pipeline (`cd.yml`)
- Triggered on version tags (`v*`)
- Builds and pushes Docker image to Docker Hub
- Creates GitHub Release
- Deploys to staging then production

### Model Validation (`model-validation.yml`)
- Triggered when `models/`, `pipeline/`, or `scripts/` change
- Trains model and validates performance

## Completed Tasks

- [x] `tests/unit/test_model.py` - Unit tests for model class
- [x] `tests/unit/test_schemas.py` - Schema validation tests
- [x] `tests/integration/test_api.py` - API endpoint tests
- [x] `tests/data/test_data_quality.py` - Data quality tests
- [x] `tests/model/test_model_behavior.py` - Behavioral tests
- [x] `.github/workflows/ci.yml` - CI pipeline
- [x] `.github/workflows/cd.yml` - CD pipeline
- [x] `.github/workflows/model-validation.yml` - Model validation
- [x] `.pre-commit-config.yaml` - Pre-commit hooks
- [x] `TESTING_STRATEGY.md` - Testing strategy document

## License

MIT License - For educational purposes only.
