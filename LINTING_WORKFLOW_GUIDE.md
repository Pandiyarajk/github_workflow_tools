# Linting in Workflows: Complete Guide

## What is Linting?

**Linting** is a static code analysis tool that examines your source code to flag programming errors, bugs, stylistic errors, and suspicious constructs. Linters help maintain code quality, consistency, and catch potential issues before they become problems.

### Key Benefits
- **Code Quality**: Catch errors and bugs early
- **Style Consistency**: Enforce coding standards across the team
- **Best Practices**: Ensure adherence to language-specific guidelines
- **CI/CD Integration**: Automated quality checks in workflows
- **Team Productivity**: Reduce code review time and technical debt

## Types of Linting Tools

### **Python Linters**
- **Flake8**: Style guide enforcement (PEP 8)
- **Black**: Code formatter
- **isort**: Import sorting
- **mypy**: Static type checking
- **Bandit**: Security analysis
- **Pylint**: Comprehensive code analysis

### **General Code Quality**
- **ESLint**: JavaScript/TypeScript linting
- **RuboCop**: Ruby code style
- **StyleCop**: C# code analysis
- **SonarQube**: Multi-language code quality

## Linting in GitHub Actions Workflows

### Basic Linting Workflow
```yaml
name: Code Quality & Linting
on: [push, pull_request]

jobs:
  lint:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
          
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install flake8 black isort mypy bandit
          
      - name: Run Black formatter check
        run: |
          echo "Checking code formatting with Black..."
          black --check --diff .
          
      - name: Run isort import sorting check
        run: |
          echo "Checking import sorting with isort..."
          isort --check-only --diff .
          
      - name: Run Flake8 style check
        run: |
          echo "Checking code style with Flake8..."
          flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
          flake8 . --count --exit-zero --max-complexity=10 --max-line-length=88 --statistics
          
      - name: Run mypy type checking
        run: |
          echo "Running type checking with mypy..."
          mypy src/ --ignore-missing-imports
          
      - name: Run Bandit security analysis
        run: |
          echo "Running security analysis with Bandit..."
          bandit -r src/ -f json -o bandit-report.json || true
```

### Advanced Linting with Matrix Strategy
```yaml
name: Comprehensive Code Quality
on: [push, pull_request]

jobs:
  lint-matrix:
    runs-on: windows-latest
    strategy:
      matrix:
        python-version: ['3.11', '3.12', '3.13']
        lint-tool: [flake8, black, isort, mypy, bandit]
        
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}
          
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install -r requirements-dev.txt
          
      - name: Run ${{ matrix.lint-tool }}
        run: |
          echo "Running ${{ matrix.lint-tool }} on Python ${{ matrix.python-version }}"
          
          if ("${{ matrix.lint-tool }}" -eq "black") {
            black --check --diff .
          } elseif ("${{ matrix.lint-tool }}" -eq "isort") {
            isort --check-only --diff .
          } elseif ("${{ matrix.lint-tool }}" -eq "flake8") {
            flake8 . --count --exit-zero --max-complexity=10 --max-line-length=88 --statistics
          } elseif ("${{ matrix.lint-tool }}" -eq "mypy") {
            mypy src/ --ignore-missing-imports
          } elseif ("${{ matrix.lint-tool }}" -eq "bandit") {
            bandit -r src/ -f json -o bandit-report-${{ matrix.python-version }}.json || true
          }
```

### Linting with Auto-fix and Commit
```yaml
name: Auto-fix Linting Issues
on:
  workflow_dispatch:
  schedule:
    - cron: '0 2 * * 1'  # Every Monday at 2 AM

jobs:
  auto-fix:
    runs-on: windows-latest
    permissions:
      contents: write
      pull-requests: write
      
    steps:
      - uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
          
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install black isort flake8
          
      - name: Auto-fix with Black
        run: |
          echo "Auto-fixing code formatting..."
          black .
          
      - name: Auto-fix with isort
        run: |
          echo "Auto-fixing import sorting..."
          isort .
          
      - name: Check for changes
        id: check_changes
        run: |
          git add -A
          if (git diff --staged --quiet) {
            echo "has_changes=false" >> $env:GITHUB_OUTPUT
          } else {
            echo "has_changes=true" >> $env:GITHUB_OUTPUT
          }
          
      - name: Commit and push changes
        if: steps.check_changes.outputs.has_changes == 'true'
        run: |
          git config --local user.email "action@github.com"
          git config --local user.name "GitHub Action"
          git commit -m "Auto-fix linting issues with Black and isort"
          git push
```

## Configuration Files

### **pyproject.toml Configuration**
```toml
[tool.black]
line-length = 88
target-version = ['py311', 'py312', 'py313']
include = '\.pyi?$'
extend-exclude = '''
/(
  # directories
  \.eggs
  | \.git
  | \.hg
  | \.mypy_cache
  | \.tox
  | \.venv
  | build
  | dist
)/
'''

[tool.isort]
profile = "black"
multi_line_output = 3
line_length = 88
known_first_party = ["src"]
known_third_party = ["rich", "configparser", "logging", "threading", "queue"]
sections = ["FUTURE", "STDLIB", "THIRDPARTY", "FIRSTPARTY", "LOCALFOLDER"]

[tool.mypy]
python_version = "3.11"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
disallow_incomplete_defs = true
check_untyped_defs = true
disallow_untyped_decorators = true
no_implicit_optional = true
warn_redundant_casts = true
warn_unused_ignores = true
warn_no_return = true
warn_unreachable = true
strict_equality = true

[tool.flake8]
max-line-length = 88
extend-ignore = ["E203", "W503"]
exclude = [
    ".git",
    "__pycache__",
    "build",
    "dist",
    ".venv",
    "venv"
]

[tool.bandit]
exclude_dirs = ["tests", "venv", ".git"]
skips = ["B101", "B601"]
targets = ["src"]
```

### **.flake8 Configuration**
```ini
[flake8]
max-line-length = 88
extend-ignore = E203, W503
exclude = 
    .git,
    __pycache__,
    build,
    dist,
    .venv,
    venv,
    *.egg-info
per-file-ignores =
    # Ignore line length for specific files
    __init__.py:E501
    # Ignore specific rules for test files
    tests/*:E501,W503
```

### **.isort.cfg Configuration**
```ini
[settings]
profile = black
multi_line_output = 3
line_length = 88
known_first_party = src
known_third_party = rich,configparser,logging,threading,queue
sections = FUTURE,STDLIB,THIRDPARTY,FIRSTPARTY,LOCALFOLDER
```

## Writing Code for Linting

### **1. Black Code Formatting**

#### ✅ **Good Formatting**
```python
# Proper line length (88 characters max)
def create_logger_with_handlers(
    name: str,
    level: int = logging.INFO,
    log_file: Optional[str] = None,
    console_output: bool = True,
) -> logging.Logger:
    """Create a logger with both file and console handlers."""
    
    # Proper spacing around operators
    if level < logging.DEBUG:
        level = logging.DEBUG
    elif level > logging.CRITICAL:
        level = logging.CRITICAL
        
    # Proper list/dict formatting
    config = {
        "name": name,
        "level": level,
        "handlers": ["file", "console"],
        "formatters": ["detailed", "simple"],
    }
    
    return logger
```

#### ❌ **Poor Formatting**
```python
# Too long lines
def create_logger_with_handlers(name,level,log_file,console_output):
    """Create a logger with both file and console handlers."""
    
    # Inconsistent spacing
    if level<logging.DEBUG:
        level=logging.DEBUG
    elif level>logging.CRITICAL:
        level=logging.CRITICAL
        
    # Poor list/dict formatting
    config={"name":name,"level":level,"handlers":["file","console"],"formatters":["detailed","simple"]}
    
    return logger
```

### **2. isort Import Sorting**

#### ✅ **Good Import Organization**
```python
# Standard library imports
import configparser
import logging
import os
import queue
import threading
from pathlib import Path
from typing import Optional

# Third-party imports
from rich.console import Console
from rich.highlighter import RegexHighlighter
from rich.logging import RichHandler

# Local imports
from .config import load_config
from .handlers import ThreadSafeQueueHandler
from .utils import setup_logging
```

#### ❌ **Poor Import Organization**
```python
# Mixed and unsorted imports
from .config import load_config
import logging
from rich.console import Console
import os
from .handlers import ThreadSafeQueueHandler
import queue
from rich.logging import RichHandler
import threading
from .utils import setup_logging
from pathlib import Path
from typing import Optional
from rich.highlighter import RegexHighlighter
import configparser
```

### **3. Flake8 Style Compliance**

#### ✅ **Good Style**
```python
# Proper naming conventions
class WindowsSafeRotatingFileHandler(logging.Handler):
    """Handler for safe file rotation on Windows."""
    
    def __init__(self, filename, max_bytes=0, backup_count=0):
        super().__init__()
        self.filename = filename
        self.max_bytes = max_bytes
        self.backup_count = backup_count
        
    def emit(self, record):
        try:
            msg = self.format(record)
            with open(self.filename, 'a', encoding='utf-8') as f:
                f.write(msg + '\n')
        except Exception as e:
            self.handleError(record)
```

#### ❌ **Poor Style**
```python
# Bad naming and style
class windowsSafeRotatingFileHandler(logging.Handler):
    """Handler for safe file rotation on Windows."""
    
    def __init__(self,filename,max_bytes=0,backup_count=0):
        super().__init__()
        self.filename=filename
        self.max_bytes=max_bytes
        self.backup_count=backup_count
        
    def emit(self,record):
        try:
            msg=self.format(record)
            with open(self.filename,'a',encoding='utf-8') as f:
                f.write(msg+'\n')
        except Exception as e:
            self.handleError(record)
```

### **4. mypy Type Annotations**

#### ✅ **Good Type Hints**
```python
from typing import Optional, List, Dict, Any, Union
import logging
import queue
import threading

class ThreadSafeQueueHandler(logging.Handler):
    """Thread-safe queue handler for logging."""
    
    def __init__(
        self,
        queue_size: int = 1000,
        flush_interval: float = 1.0,
        level: int = logging.NOTSET
    ) -> None:
        super().__init__(level)
        self.queue: queue.Queue = queue.Queue(maxsize=queue_size)
        self.worker_thread: Optional[threading.Thread] = None
        self.handlers: List[logging.Handler] = []
        self.flush_interval = flush_interval
        
    def emit(self, record: logging.LogRecord) -> None:
        try:
            self.queue.put_nowait(record)
        except queue.Full:
            # Handle queue full scenario
            pass
            
    def get_logger(self, name: str) -> logging.Logger:
        logger = logging.getLogger(name)
        logger.addHandler(self)
        return logger
```

#### ❌ **Poor Type Hints**
```python
# Missing or incorrect type hints
class ThreadSafeQueueHandler(logging.Handler):
    """Thread-safe queue handler for logging."""
    
    def __init__(self, queue_size, flush_interval, level):
        super().__init__(level)
        self.queue = queue.Queue(maxsize=queue_size)
        self.worker_thread = None
        self.handlers = []
        self.flush_interval = flush_interval
        
    def emit(self, record):
        try:
            self.queue.put_nowait(record)
        except:
            pass
            
    def get_logger(self, name):
        logger = logging.getLogger(name)
        logger.addHandler(self)
        return logger
```

### **5. Bandit Security Compliance**

#### ✅ **Secure Code**
```python
import os
import ast
from typing import Any

def safe_eval(expression: str) -> Any:
    """Safely evaluate expressions without security risks."""
    try:
        return ast.literal_eval(expression)
    except (ValueError, SyntaxError):
        raise ValueError("Invalid expression format")

def read_config_file(filename: str) -> str:
    """Read configuration file with path validation."""
    # Validate file path
    if not os.path.exists(filename):
        raise FileNotFoundError(f"Config file not found: {filename}")
    
    # Use absolute path to prevent directory traversal
    abs_path = os.path.abspath(filename)
    if not abs_path.startswith(os.getcwd()):
        raise ValueError("Invalid file path")
    
    with open(abs_path, 'r', encoding='utf-8') as f:
        return f.read()

def get_database_url() -> str:
    """Get database URL from environment variable."""
    db_url = os.environ.get('DATABASE_URL')
    if not db_url:
        raise ValueError("DATABASE_URL environment variable not set")
    return db_url
```

#### ❌ **Insecure Code**
```python
# Security issues that Bandit will flag
def unsafe_eval(expression):
    """Unsafe evaluation - Bandit B102: exec_used"""
    return eval(expression)  # nosec B102

def read_any_file(filename):
    """Unsafe file reading - potential path traversal"""
    with open(filename, 'r') as f:  # nosec B108
        return f.read()

def get_password():
    """Hardcoded password - Bandit B105: hardcoded_password_string"""
    return "secret123"  # nosec B105

def create_server():
    """Unsafe network binding - Bandit B104: hardcoded_bind_all_interfaces"""
    import socket
    s = socket.socket()
    s.bind(('0.0.0.0', 8080))  # nosec B104
    return s
```

## Common Linting Issues and Solutions

### **Black Issues**

#### **Line Too Long**
```python
# ❌ Problem
def very_long_function_name_with_many_parameters(param1, param2, param3, param4, param5, param6, param7, param8, param9, param10, param11, param12, param13, param14, param15, param16, param17, param18, param19, param20):

# ✅ Solution
def very_long_function_name_with_many_parameters(
    param1, param2, param3, param4, param5,
    param6, param7, param8, param9, param10,
    param11, param12, param13, param14, param15,
    param16, param17, param18, param19, param20
):
```

#### **Inconsistent Spacing**
```python
# ❌ Problem
x=1+2*3
if x>5:
    print("x is greater than 5")

# ✅ Solution
x = 1 + 2 * 3
if x > 5:
    print("x is greater than 5")
```

### **isort Issues**

#### **Import Order**
```python
# ❌ Problem
from .local_module import LocalClass
import os
from third_party import ThirdPartyClass
import sys

# ✅ Solution
import os
import sys

from third_party import ThirdPartyClass

from .local_module import LocalClass
```

### **Flake8 Issues**

#### **Unused Imports**
```python
# ❌ Problem
import os
import sys
import json  # Unused import
from typing import List, Dict, Tuple  # Tuple unused

# ✅ Solution
import os
import sys
from typing import List, Dict
```

#### **Undefined Variables**
```python
# ❌ Problem
def process_data(data):
    result = data * 2
    return reslt  # Typo: undefined variable

# ✅ Solution
def process_data(data):
    result = data * 2
    return result
```

### **mypy Issues**

#### **Missing Type Annotations**
```python
# ❌ Problem
def calculate_total(items, discount):
    total = sum(items)
    if discount:
        total *= (1 - discount)
    return total

# ✅ Solution
from typing import List, Optional

def calculate_total(items: List[float], discount: Optional[float]) -> float:
    total = sum(items)
    if discount:
        total *= (1 - discount)
    return total
```

#### **Type Mismatch**
```python
# ❌ Problem
def get_user_age(user_id: str) -> int:
    user_data = fetch_user_data(user_id)
    return user_data.get("age", "unknown")  # Returns str, not int

# ✅ Solution
def get_user_age(user_id: str) -> int:
    user_data = fetch_user_data(user_id)
    age = user_data.get("age")
    if age is None:
        return 0
    return int(age)
```

## Linting Workflow Best Practices

### **1. Pre-commit Hooks**
```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/psf/black
    rev: 23.12.1
    hooks:
      - id: black
        language_version: python3
        
  - repo: https://github.com/pycqa/isort
    rev: 5.13.2
    hooks:
      - id: isort
        args: ["--profile", "black"]
        
  - repo: https://github.com/pycqa/flake8
    rev: 7.0.0
    hooks:
      - id: flake8
        args: [--max-line-length=88, --extend-ignore=E203,W503]
        
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.8.0
    hooks:
      - id: mypy
        additional_dependencies: [types-all]
```

### **2. IDE Integration**

#### **VS Code Settings**
```json
// .vscode/settings.json
{
    "python.linting.enabled": true,
    "python.linting.flake8Enabled": true,
    "python.linting.mypyEnabled": true,
    "python.formatting.provider": "black",
    "python.sortImports.args": ["--profile", "black"],
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
        "source.organizeImports": true
    }
}
```

#### **PyCharm Settings**
- Enable Black formatter
- Configure isort for import sorting
- Enable mypy type checking
- Set up Flake8 inspection

### **3. CI/CD Integration**

#### **Fail Fast Strategy**
```yaml
name: Quick Lint Check
on: [pull_request]

jobs:
  quick-lint:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
          
      - name: Install linting tools
        run: |
          pip install black isort flake8
          
      - name: Quick format check
        run: |
          black --check --diff .
          isort --check-only --diff .
          flake8 . --count --exit-zero --max-complexity=10 --max-line-length=88
```

#### **Comprehensive Quality Check**
```yaml
name: Full Quality Check
on: [push, pull_request]

jobs:
  quality:
    runs-on: windows-latest
    needs: quick-lint
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
          
      - name: Install all tools
        run: |
          pip install -r requirements-dev.txt
          
      - name: Run all checks
        run: |
          black --check --diff .
          isort --check-only --diff .
          flake8 . --count --exit-zero --max-complexity=10 --max-line-length=88
          mypy src/ --ignore-missing-imports
          bandit -r src/ -f json -o bandit-report.json
```

## Troubleshooting Common Issues

### **1. Black vs isort Conflicts**
```toml
# pyproject.toml
[tool.isort]
profile = "black"
multi_line_output = 3
line_length = 88
```

### **2. Flake8 vs Black Conflicts**
```ini
# .flake8
[flake8]
extend-ignore = E203, W503
max-line-length = 88
```

### **3. mypy Import Issues**
```toml
# pyproject.toml
[tool.mypy]
ignore_missing_imports = true
exclude = ["tests/", "venv/", ".venv/"]
```

### **4. Performance Issues**
```bash
# Limit scope for faster runs
black src/ --check
isort src/ --check-only
flake8 src/ --count
mypy src/ --ignore-missing-imports
```

## Conclusion

Linting is essential for maintaining code quality and consistency in modern development workflows. By integrating multiple linting tools and configuring them properly, you can ensure your code meets high standards for style, security, and correctness.

### **Key Takeaways**
1. **Use multiple tools** for comprehensive coverage
2. **Configure tools to work together** (Black + isort + Flake8)
3. **Integrate into CI/CD** for automated quality checks
4. **Set up pre-commit hooks** to catch issues early
5. **Configure IDEs** for real-time feedback
6. **Train your team** on linting best practices

### **Recommended Tool Stack**
- **Black**: Code formatting
- **isort**: Import sorting
- **Flake8**: Style and error checking
- **mypy**: Type checking
- **Bandit**: Security analysis
- **Pre-commit**: Local hooks

### **Resources**
- [Black Documentation](https://black.readthedocs.io/)
- [isort Documentation](https://pycqa.github.io/isort/)
- [Flake8 Documentation](https://flake8.pycqa.org/)
- [mypy Documentation](https://mypy.readthedocs.io/)
- [Bandit Documentation](https://bandit.readthedocs.io/)
- [Pre-commit Documentation](https://pre-commit.com/)

---

*This guide covers the essential aspects of linting in workflows. For more advanced usage and custom configurations, refer to the individual tool documentation.*
