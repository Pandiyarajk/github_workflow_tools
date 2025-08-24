# isort Import Sorter Guide: Complete Workflow Integration

## What is isort?

**isort** is a Python utility library that automatically sorts and formats import statements in Python files. It follows PEP 8 import style guidelines and can be configured to work seamlessly with other code formatters like Black.

### Key Features
- **Automatic Import Sorting**: Organizes imports into logical groups (standard library, third-party, local)
- **PEP 8 Compliance**: Follows Python style guide recommendations
- **Black Compatibility**: Configurable to work with Black formatter
- **Customizable**: Extensive configuration options for different project needs
- **Multi-line Import Support**: Handles complex import statements elegantly
- **Pre-commit Integration**: Can be used in pre-commit hooks
- **IDE Integration**: Works with most Python IDEs and editors

## How isort Works

### **Import Organization Rules**
1. **Standard Library Imports**: Python built-in modules (e.g., `os`, `sys`, `logging`)
2. **Third-party Imports**: External packages (e.g., `rich`, `pytest`, `numpy`)
3. **Local/Application Imports**: Your project's modules (e.g., `src.core`)

### **Sorting Logic**
- **Alphabetical Order**: Within each group, imports are sorted alphabetically
- **Line Length**: Respects maximum line length (default 79, configurable)
- **Multi-line Formatting**: Breaks long import lines appropriately
- **Relative vs Absolute**: Configurable preference for import styles

## Installation and Setup

### **Basic Installation**
```bash
# Install isort
pip install isort

# Install with Black compatibility
pip install isort[black]

# Install for development
pip install isort[pyproject]
```

### **Verify Installation**
```bash
# Check isort version
isort --version

# Check available options
isort --help
```

## Basic Usage

### **Command Line Usage**
```bash
# Sort imports in a single file
isort src/main.py

# Sort imports in entire directory
isort src/

# Sort imports in entire project
isort .

# Check only (don't modify files)
isort --check-only .

# Show diff without modifying
isort --diff .

# Sort with Black compatibility
isort --profile black .
```

### **Interactive Mode**
```bash
# Interactive mode for multiple files
isort --interactive .

# Ask before making changes
isort --ask .
```

## Configuration

### **pyproject.toml Configuration (Recommended)**
```toml
[tool.isort]
# Basic settings
profile = "black"
line_length = 88
multi_line_output = 3
include_trailing_comma = true
force_grid_wrap = 0
use_parentheses = true
ensure_newline_before_comments = true

# Import grouping
known_first_party = ["src"]
known_third_party = ["rich", "pytest", "numpy"]
sections = ["FUTURE", "STDLIB", "THIRDPARTY", "FIRSTPARTY", "LOCALFOLDER"]

# File patterns
skip = ["__pycache__", "build", "dist", ".venv", "venv"]
skip_glob = ["*.pyc", "*.pyo", "*.pyd"]

# Import sorting
force_sort_within_sections = true
case_sensitive = false
force_wrap_imports = true

# Black compatibility
profile = "black"
line_length = 88
multi_line_output = 3
include_trailing_comma = true
force_grid_wrap = 0
use_parentheses = true
ensure_newline_before_comments = true
```

### **.isort.cfg Configuration (Legacy)**
```ini
[settings]
profile = black
line_length = 88
multi_line_output = 3
include_trailing_comma = True
force_grid_wrap = 0
use_parentheses = True
ensure_newline_before_comments = True

known_first_party = src
known_third_party = rich,pytest,numpy

sections = FUTURE,STDLIB,THIRDPARTY,FIRSTPARTY,LOCALFOLDER

skip = __pycache__,build,dist,.venv,venv
skip_glob = *.pyc,*.pyo,*.pyd

force_sort_within_sections = True
case_sensitive = False
force_wrap_imports = True
```

### **setup.cfg Configuration**
```ini
[isort]
profile = black
line_length = 88
multi_line_output = 3
include_trailing_comma = True
force_grid_wrap = 0
use_parentheses = True
ensure_newline_before_comments = True

known_first_party = src
known_third_party = rich,pytest,numpy

sections = FUTURE,STDLIB,THIRDPARTY,FIRSTPARTY,LOCALFOLDER

skip = __pycache__,build,dist,.venv,venv
skip_glob = *.pyc,*.pyo,*.pyd

force_sort_within_sections = True
case_sensitive = False
force_wrap_imports = True
```

## Integration in GitHub Actions Workflows

### **Basic isort Workflow**
```yaml
name: Import Sorting Check
on: [push, pull_request]

jobs:
  isort-check:
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
          pip install isort[black]
          
      - name: Check import sorting
        run: |
          isort --check-only --diff .
```

### **Advanced isort Workflow with Auto-fix**
```yaml
name: Import Sorting and Formatting
on: [push, pull_request]

jobs:
  code-formatting:
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
          pip install isort[black] black
          
      - name: Check import sorting
        id: isort-check
        run: |
          isort --check-only --diff .
          
      - name: Auto-fix import sorting
        if: failure()
        run: |
          isort .
          
      - name: Check Black formatting
        run: |
          black --check --diff .
          
      - name: Auto-fix formatting
        if: failure()
        run: |
          black .
          
      - name: Commit changes
        if: always()
        run: |
          git config --local user.email "action@github.com"
          git config --local user.name "GitHub Action"
          git add -A
          git diff --quiet && git diff --staged --quiet || git commit -m "Auto-fix code formatting"
```

### **isort with Matrix Strategy**
```yaml
name: Multi-Python Import Sorting
on: [push, pull_request]

jobs:
  isort-matrix:
    runs-on: windows-latest
    strategy:
      matrix:
        python-version: ['3.11', '3.12', '3.13']
        
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}
          
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install isort[black]
          
      - name: Check import sorting for Python ${{ matrix.python-version }}
        run: |
          isort --check-only --diff .
```

### **isort with Pre-commit Workflow**
```yaml
name: Pre-commit Checks
on: [push, pull_request]

jobs:
  pre-commit:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
          
      - name: Install pre-commit
        run: |
          python -m pip install --upgrade pip
          pip install pre-commit
          
      - name: Run pre-commit
        run: |
          pre-commit run --all-files
```

## Writing Code for isort Compatibility

### **1. Import Statement Best Practices**

#### ✅ **Good: Well-organized Imports**
```python
"""src module for enhanced logging functionality."""

# Standard library imports
import logging
import os
import queue
import sys
import threading
from pathlib import Path
from typing import Any, Dict, List, Optional, Union

# Third-party imports
import rich
from rich.console import Console
from rich.logging import RichHandler
from rich.table import Table

# Local imports
from src.config import ConfigManager
from src.handlers import ThreadSafeQueueHandler
from src.utils import setup_logging
```

#### ❌ **Poor: Disorganized Imports**
```python
# Mixed import order
from src.config import ConfigManager
import logging
from rich.console import Console
import os
from typing import Optional
import threading
from src.handlers import ThreadSafeQueueHandler
import queue
from rich.logging import RichHandler
```

### **2. Import Grouping Examples**

#### **Standard Library First**
```python
# Standard library imports
import abc
import asyncio
import contextlib
import functools
import hashlib
import json
import logging
import os
import pathlib
import queue
import re
import sys
import threading
import time
from collections import defaultdict, deque
from datetime import datetime, timedelta
from pathlib import Path
from typing import Any, Callable, Dict, List, Optional, Union
```

#### **Third-party Imports**
```python
# Third-party imports
import click
import colorama
import jinja2
import pytest
import rich
import toml
import yaml
from click import command, option
from colorama import Fore, Style, init
from jinja2 import Environment, FileSystemLoader
from pytest import fixture, mark, raises
from rich.console import Console
from rich.logging import RichHandler
from rich.table import Table
from toml import load, loads
from yaml import dump, load as yaml_load
```

#### **Local/Application Imports**
```python
# Local imports
from src import __version__
from src.config import ConfigManager, load_config
from src.core import src, get_logger
from src.handlers import (
    FileHandler,
    RichConsoleHandler,
    ThreadSafeQueueHandler,
)
from src.utils import (
    format_message,
    setup_logging,
    validate_config,
)
```

### **3. Multi-line Import Examples**

#### **Long Import Lists**
```python
from src.handlers import (
    AsyncHandler,
    FileHandler,
    RichConsoleHandler,
    RotatingFileHandler,
    ThreadSafeQueueHandler,
    WebSocketHandler,
)
```

#### **Complex Import Statements**
```python
from src.utils import (
    format_message,
    get_log_level,
    parse_config,
    setup_logging,
    validate_config,
    write_to_file,
)
```

#### **Conditional Imports**
```python
try:
    from rich.console import Console
    from rich.logging import RichHandler
    RICH_AVAILABLE = True
except ImportError:
    RICH_AVAILABLE = False

try:
    import yaml
    YAML_AVAILABLE = True
except ImportError:
    YAML_AVAILABLE = False
```

### **4. Import Style Examples**

#### **Absolute Imports (Recommended)**
```python
# Good: Absolute imports
from src.config import ConfigManager
from src.handlers import ThreadSafeQueueHandler
from src.utils import setup_logging

# Avoid: Relative imports (unless necessary)
# from .config import ConfigManager
# from ..handlers import ThreadSafeQueueHandler
```

#### **Import Aliases**
```python
# Good: Clear aliases
import logging as log
import pathlib as path
from typing import List as ListType, Dict as DictType

# Good: Common aliases
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
```

### **5. Special Import Cases**

#### **Future Imports**
```python
# Future imports (always first)
from __future__ import annotations
from __future__ import division
from __future__ import print_function
from __future__ import unicode_literals

# Standard library imports
import os
import sys
```

#### **Type Checking Imports**
```python
# Standard library imports
import logging
import queue
import threading
from typing import TYPE_CHECKING, Any, Optional

if TYPE_CHECKING:
    # Type checking only imports
    from src.config import ConfigManager
    from src.handlers import ThreadSafeQueueHandler
```

#### **Platform-specific Imports**
```python
# Standard library imports
import os
import sys

# Platform-specific imports
if sys.platform == "win32":
    import win32api
    import win32con
elif sys.platform == "darwin":
    import fcntl
else:
    import termios
```

## isort Configuration Profiles

### **Black Profile (Recommended)**
```toml
[tool.isort]
profile = "black"
line_length = 88
multi_line_output = 3
include_trailing_comma = true
force_grid_wrap = 0
use_parentheses = true
ensure_newline_before_comments = true
```

### **Django Profile**
```toml
[tool.isort]
profile = "django"
line_length = 88
known_django = "django"
known_first_party = "src"
```

### **Google Profile**
```toml
[tool.isort]
profile = "google"
line_length = 88
force_sort_within_sections = true
```

### **Custom Profile**
```toml
[tool.isort]
# Custom profile for src
profile = "black"
line_length = 88
multi_line_output = 3
include_trailing_comma = true
force_grid_wrap = 0
use_parentheses = true
ensure_newline_before_comments = true

# Import sections
sections = ["FUTURE", "STDLIB", "THIRDPARTY", "FIRSTPARTY", "LOCALFOLDER"]

# Known packages
known_first_party = ["src"]
known_third_party = ["rich", "pytest", "numpy", "pandas"]

# Skip patterns
skip = ["__pycache__", "build", "dist", ".venv", "venv"]
skip_glob = ["*.pyc", "*.pyo", "*.pyd"]

# Sorting options
force_sort_within_sections = true
case_sensitive = false
force_wrap_imports = true
```

## Integration with Other Tools

### **1. Black Integration**
```toml
# pyproject.toml
[tool.black]
line-length = 88
target-version = ['py311']
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
line_length = 88
multi_line_output = 3
include_trailing_comma = true
force_grid_wrap = 0
use_parentheses = true
ensure_newline_before_comments = true
```

### **2. Flake8 Integration**
```toml
# pyproject.toml
[tool.flake8]
max-line-length = 88
extend-ignore = ["E203", "W503"]
exclude = [
    ".git",
    "__pycache__",
    "build",
    "dist",
    ".venv",
    "venv",
]

[tool.isort]
profile = "black"
line_length = 88
```

### **3. Pre-commit Integration**
```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/pycqa/isort
    rev: 5.13.2
    hooks:
      - id: isort
        args: ["--profile", "black"]
        
  - repo: https://github.com/psf/black
    rev: 23.12.1
    hooks:
      - id: black
        language_version: python3.11
        
  - repo: https://github.com/pycqa/flake8
    rev: 7.0.0
    hooks:
      - id: flake8
        args: [--max-line-length=88, --extend-ignore=E203,W503]
```

## Common Issues and Solutions

### **1. Import Order Conflicts**
```bash
# Problem: Imports not in expected order
# Solution: Check isort configuration
isort --check-only --diff src/main.py

# Fix automatically
isort src/main.py
```

### **2. Black Compatibility Issues**
```bash
# Problem: isort and Black formatting conflicts
# Solution: Use Black profile
isort --profile black .

# Or configure both tools consistently
```

### **3. Multi-line Import Formatting**
```python
# Problem: Inconsistent multi-line formatting
from src.handlers import FileHandler, RichConsoleHandler, ThreadSafeQueueHandler

# Solution: Force grid wrap
from src.handlers import (
    FileHandler,
    RichConsoleHandler,
    ThreadSafeQueueHandler,
)
```

### **4. Relative vs Absolute Imports**
```python
# Problem: Mixed import styles
from .config import ConfigManager
from src.handlers import ThreadSafeQueueHandler

# Solution: Consistent absolute imports
from src.config import ConfigManager
from src.handlers import ThreadSafeQueueHandler
```

## Best Practices

### **1. Development Workflow**
```bash
# Pre-commit check
isort --check-only --diff .

# Auto-fix before commit
isort .

# Verify with Black
black --check .
```

### **2. CI/CD Integration**
```yaml
# Always check import sorting in CI
- name: Check import sorting
  run: isort --check-only --diff .
```

### **3. Team Standards**
- **Consistent Configuration**: Use same isort config across team
- **Pre-commit Hooks**: Enforce import sorting before commits
- **Documentation**: Document import style guidelines
- **Code Reviews**: Check import organization in reviews

### **4. IDE Integration**
```json
// VS Code settings.json
{
    "python.sortImports.args": ["--profile", "black"],
    "python.sortImports.path": "isort",
    "editor.codeActionsOnSave": {
        "source.organizeImports": true
    }
}
```

## Troubleshooting

### **1. Common Error Messages**
```bash
# "Imports are incorrectly sorted and/or formatted"
isort --diff .  # Show what needs to be fixed

# "Found 1 import sorting error"
isort .  # Fix automatically
```

### **2. Configuration Issues**
```bash
# Check current configuration
isort --show-config

# Validate configuration
isort --check-only --diff .
```

### **3. Performance Issues**
```bash
# Skip certain directories
isort --skip __pycache__ --skip build .

# Use specific file patterns
isort "*.py" "!__pycache__"
```

## Conclusion

isort is an essential tool for maintaining clean, organized import statements in Python projects. By integrating it into your development workflow and following import best practices, you can ensure consistent code style and improve code readability.

### **Key Takeaways**
1. **Use isort consistently** across your project
2. **Configure for Black compatibility** to avoid conflicts
3. **Integrate with CI/CD** for automated import checking
4. **Follow import organization best practices** in code development
5. **Use pre-commit hooks** to enforce import sorting
6. **Regular import reviews** and team training

### **Recommended Workflow**
1. **Local Development**: Run isort before committing
2. **Pre-commit**: Automatic import sorting
3. **CI/CD**: Automated import checking in pipelines
4. **Code Reviews**: Verify import organization

### **Resources**
- [isort Documentation](https://pycqa.github.io/isort/)
- [isort GitHub](https://github.com/pycqa/isort)
- [PEP 8 Import Guidelines](https://www.python.org/dev/peps/pep-0008/#imports)
- [Black Formatter](https://black.readthedocs.io/)
- [Pre-commit Framework](https://pre-commit.com/)

---

*This guide covers the essential aspects of using isort for import sorting. For more advanced usage and custom configurations, refer to the official isort documentation.*
