# Code Quality & Security Workflows

This project uses a set of automated tools to maintain **code quality,
consistency, and security**.\
They are integrated into GitHub Actions for continuous inspection.

------------------------------------------------------------------------

## 🚀 Tools in Use

### 1. **Black** -- Code Formatter

-   Enforces a consistent Python code style.

-   Uncompromising, PEP 8 compliant, minimal configuration.

-   Run manually:

    ``` bash
    black .
    black --check --diff .
    ```

### 2. **isort** -- Import Sorter

-   Automatically sorts imports into **standard library**,
    **third-party**, and **local** groups.

-   Compatible with Black.

-   Run manually:

    ``` bash
    isort .
    isort --check-only --diff .
    ```

### 3. **Flake8** -- Style Checker

-   Enforces **PEP 8** and catches common code issues.

-   Configurable rules for team/project standards.

-   Run manually:

    ``` bash
    flake8 .
    ```

### 4. **mypy** -- Static Type Checker

-   Validates **Python type hints (PEP 484)**.

-   Detects type mismatches, missing attributes, and logic issues.

-   Run manually:

    ``` bash
    mypy repository_folder/
    ```

### 5. **Bandit** -- Security Analyzer

-   Scans Python code for **security vulnerabilities**.

-   Provides severity levels and remediation tips.

-   Run manually:

    ``` bash
    bandit -r repository_folder/ -f json -o bandit-report.json
    ```

### 6. **SonarQube** -- Code Quality Platform

-   Multi-language **quality & security analysis**.

-   Tracks **bugs, vulnerabilities, code smells, test coverage, and
    duplication**.

-   Run scan:

    ``` bash
    sonar-scanner
    ```

------------------------------------------------------------------------

## ⚡ GitHub Actions Workflows

Typical workflow includes:

``` yaml
name: Code Quality Checks
on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - run: |
          pip install black isort flake8 mypy bandit
          black --check --diff .
          isort --check-only --diff .
          flake8 .
          mypy repository_folder/ --ignore-missing-imports
          bandit -r repository_folder/ -f json -o bandit-report.json
```

For **SonarQube**, configure `sonar-project.properties` and add:

``` yaml
      - name: SonarQube Scan
        uses: sonarqube-quality-gate-action@master
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
```

------------------------------------------------------------------------

## 📂 Configuration Files

-   `pyproject.toml` → shared config for Black, isort, mypy, and Bandit.
-   `.flake8` → Flake8 style rules.
-   `.bandit` → Security scanning configuration.
-   `sonar-project.properties` → SonarQube project config.

------------------------------------------------------------------------

## ✅ Benefits

-   **Consistent formatting** (Black, isort)\
-   **Style enforcement** (Flake8)\
-   **Type safety** (mypy)\
-   **Security checks** (Bandit)\
-   **Comprehensive quality metrics** (SonarQube)

------------------------------------------------------------------------
