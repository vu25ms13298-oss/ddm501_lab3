# Testing Strategy for Movie Rating Prediction API

## 1. Overview

This document describes the testing strategy for the Movie Rating Prediction API, a FastAPI-based service that uses SVD (Singular Value Decomposition) collaborative filtering to predict movie ratings.

## 2. Testing Pyramid

We follow the ML Testing Pyramid, which extends the traditional testing pyramid with ML-specific concerns:

```
         /\
        /  \        System/E2E Tests (Docker smoke test in CI)
       /    \
      /------\      Model Behavioral Tests (20 tests)
     /        \         - Invariance, directional, minimum functionality
    /----------\    Data Validation Tests (17 tests)
   /            \       - Quality, distribution, uniqueness, types
  /--------------\  Integration Tests (23 tests)
 /                \     - API endpoints, error handling, batch
/------------------\Unit Tests (29 tests)
                        - Model class, Pydantic schemas
```

## 3. Test Categories

### 3.1 Unit Tests (29 tests)

**Purpose:** Test individual functions and classes in isolation.

**Coverage:**
- **Model Class** (`test_model.py`): Model loading, prediction return types, rating range validation, batch predictions, `is_loaded()` behavior, error handling for edge cases (None, empty strings).
- **Pydantic Schemas** (`test_schemas.py`): Request/response validation, missing fields, empty/invalid inputs, type validation, boundary values, batch request limits.

**Key Fixtures:** `trained_model`, `known_user_movie_pairs`

### 3.2 Integration Tests (23 tests)

**Purpose:** Test component interactions through API endpoints.

**Coverage:**
- **Health Endpoint**: Status code, response structure, field types
- **Root Endpoint**: API information completeness
- **Predict Endpoint**: Valid requests, response fields, rating range, validation errors (missing fields, empty body, invalid JSON), multiple sequential requests
- **Batch Predict Endpoint**: Status code, result count, rating range
- **Error Handling**: 404 for unknown endpoints, 405 for wrong HTTP methods
- **Model Info Endpoint**: Version and status fields

**Key Fixtures:** `test_client` (with pre-loaded model), `sample_prediction_request`, `sample_batch_request`

### 3.3 Data Validation Tests (17 tests)

**Purpose:** Validate data quality before it reaches the model.

**Coverage:**
- **Rating Quality**: Valid range (1.0-5.0), no negatives, no exceeding maximum
- **ID Validation**: No missing user/movie IDs, correct string types
- **Data Completeness**: No null ratings, all required fields present
- **Distribution**: Mean rating reasonableness (2.0-4.5), standard deviation bounds, multiple distinct values
- **Uniqueness**: Unique user-movie pairs, multiple users and movies present
- **Type Safety**: Numeric ratings, float compatibility

### 3.4 Model Behavioral Tests (20 tests)

**Purpose:** Verify the model behaves correctly following the CheckList approach.

**Coverage:**
- **Invariance Tests**: Deterministic outputs (same input = same output), consistency across multiple calls, batch order independence, individual vs batch consistency
- **Directional Tests**: Predictions close to actual ratings (within 2.5), different movies yield different predictions, different users yield different predictions
- **Minimum Functionality**: Predictions for known users, multiple user predictions, non-uniform predictions, graceful handling of unknown users/movies
- **Performance**: Mean absolute error < 1.5, no extreme errors (> 3.0)
- **Robustness**: String numeric IDs, leading zeros in IDs

## 4. CI/CD Pipeline

### 4.1 Continuous Integration (ci.yml)

Triggered on: push to `main`/`develop`, pull requests to `main`.

```
Lint (flake8, black, isort)
         |
    Type Check (mypy)       [parallel with lint]
         |
   Test (pytest + coverage)  [depends on lint]
         |
   Build (Docker)            [depends on test]
```

- **Lint Job**: Code style enforcement (flake8, black, isort)
- **Type Check Job**: Static type analysis with mypy
- **Test Job**: Full test suite with coverage report (minimum 80%)
- **Build Job**: Docker image build and health check smoke test

### 4.2 Continuous Deployment (cd.yml)

Triggered on: version tags (`v*`).

```
Build & Push Docker Image → Create GitHub Release → Deploy Staging → Deploy Production
```

### 4.3 Model Validation (model-validation.yml)

Triggered on: changes to `models/`, `pipeline/`, `scripts/`.

Ensures model trains successfully and produces valid predictions.

## 5. Code Quality

### Pre-commit Hooks
- **pre-commit-hooks**: Trailing whitespace, end-of-file fixer, YAML/JSON checks, large file detection, merge conflict detection, private key detection
- **Black**: Code formatting (line length 100)
- **isort**: Import sorting (black profile)
- **Flake8**: Linting (line length 100)
- **mypy**: Type checking
- **pytest**: Unit tests before commit

### Configuration
All tool configurations are centralized in `pyproject.toml`.

## 6. Coverage Target

- **Minimum**: 80% code coverage
- **Current**: 84% code coverage
- **Coverage Tool**: pytest-cov with XML and HTML reports

## 7. Test Execution

```bash
# All tests
pytest tests/ -v --cov=app --cov-report=html

# By category
pytest tests/unit/ -v           # Unit tests only
pytest tests/integration/ -v     # Integration tests only
pytest tests/data/ -v            # Data validation tests only
pytest tests/model/ -v           # Model behavioral tests only

# With markers
pytest -m "not slow"             # Skip slow tests
pytest -m integration            # Integration tests only
```
