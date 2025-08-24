# Black Code Formatter: Complete Guide

## What is Black?

**Black** is an uncompromising Python code formatter that automatically formats your code to comply with a consistent style. Unlike other formatters, Black makes very few decisions and focuses on producing consistent, readable code with minimal configuration.

### Key Features
- **Uncompromising**: Few configuration options for consistency across projects
- **Fast**: Written in Rust for high performance
- **PEP 8 Compliant**: Follows Python style guidelines
- **Deterministic**: Same input always produces same output
- **CI/CD Friendly**: Can be used in automated workflows
- **IDE Integration**: Works with VS Code, PyCharm, and other editors

### Philosophy
> "Any color in a rainbow is fine, as long as it's black." - Black's motto

Black believes that code formatting is not a matter of personal preference but a tool for reducing cognitive load and improving code readability.

## How Black Works

### **Code Analysis Process**
1. **Parsing**: Converts Python code into an Abstract Syntax Tree (AST)
2. **Formatting**: Applies consistent formatting rules
3. **Output**: Generates formatted code with consistent style

### **Formatting Rules**
- **Line Length**: Default 88 characters (configurable)
- **String Quotes**: Prefers double quotes, converts single quotes
- **Spacing**: Consistent spacing around operators and brackets
- **Line Breaks**: Automatic line breaking for long expressions
- **Import Sorting**: Works with isort for import organization
- **Trailing Commas**: Adds trailing commas for cleaner diffs

## Installation

### **Basic Installation**
```bash
# Install Black
pip install black

# Install with additional features
pip install black[toml]  # TOML configuration support
pip install black[jupyter]  # Jupyter notebook support
```

### **Development Installation**
```bash
# Install in development environment
pip install -e ".[dev]"

# Install specific version
pip install black==23.12.1
```

## Basic Usage

### **Command Line Usage**
```bash
# Format a single file
black repository_folder/main.py

# Format entire directory
black .

# Check formatting without changing files
black --check --diff .

# Format with specific line length
black --line-length 100 .

# Format specific file types
black --include '\.pyi?$' .

# Exclude directories
black --exclude '/(\.eggs|\.git|\.hg|\.mypy_cache|\.tox|\.venv|venv|build|dist)/' .
```

### **Python API Usage**
```python
import black

# Format code string
code = """
def hello_world(name):
    print(f"Hello, {name}!")
"""

formatted_code = black.format_str(code, mode=black.FileMode())
print(formatted_code)

# Format file
black.format_file_in_place(
    "repository_folder/main.py",
    mode=black.FileMode(),
    write_back=black.WriteBack.YES
)
```

## Integration in GitHub Actions Workflows

### **Basic Black Workflow**
```yaml
name: Code Formatting Check
on: [push, pull_request]

jobs:
  format-check:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
          
      - name: Install Black
        run: |
          python -m pip install --upgrade pip
          pip install black
          
      - name: Check code formatting
        run: |
          echo "Checking code formatting with Black..."
          black --check --diff .
```

### **Advanced Black Workflow with Auto-fix**
```yaml
name: Code Formatting & Auto-fix
on: [push, pull_request]

jobs:
  format-check:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
          
      - name: Install formatting tools
        run: |
          python -m pip install --upgrade pip
          pip install black isort
          
      - name: Check Black formatting
        run: |
          echo "Checking Black formatting..."
          black --check --diff . || {
            echo "Black formatting check failed. Run 'black .' to fix."
            exit 1
          }
          
      - name: Check isort import sorting
        run: |
          echo "Checking import sorting..."
          isort --check-only --diff . || {
            echo "Import sorting check failed. Run 'isort .' to fix."
            exit 1
          }
```

### **Black with Matrix Strategy**
```yaml
name: Multi-Python Format Check
on: [push, pull_request]

jobs:
  format-matrix:
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
          
      - name: Install Black
        run: |
          python -m pip install --upgrade pip
          pip install black
          
      - name: Check formatting on Python ${{ matrix.python-version }}
        run: |
          echo "Checking formatting on Python ${{ matrix.python-version }}"
          black --check --diff .
```

### **Black with Auto-fix and Commit**
```yaml
name: Auto-format Code
on:
  workflow_dispatch:
  schedule:
    - cron: '0 2 * * 1'  # Every Monday at 2 AM

jobs:
  auto-format:
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
          
      - name: Install formatting tools
        run: |
          python -m pip install --upgrade pip
          pip install black isort
          
      - name: Auto-format with Black
        run: |
          echo "Auto-formatting with Black..."
          black .
          
      - name: Auto-sort imports with isort
        run: |
          echo "Auto-sorting imports..."
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
          git commit -m "Auto-format code with Black and isort"
          git push
```

## Configuration

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
  | venv
  | build
  | dist
)/
'''
skip-string-normalization = false
preview = false
```

### **.black Configuration File**
```ini
# .black
[black]
line-length = 88
target-version = py311, py312, py313
include = '\.pyi?$'
extend-exclude = '''
/(
  \.eggs
  | \.git
  | \.hg
  | \.mypy_cache
  | \.tox
  | \.venv
  | venv
  | build
  | dist
)/
'''
skip-string-normalization = false
preview = false
```

### **Command Line Configuration**
```bash
# Create Black configuration
black --generate-config

# Validate configuration
black --config pyproject.toml --check .
```

## Writing Code for Black Compatibility

### **1. Line Length Management**

#### ✅ **Good: Proper Line Breaking**
```python
# Long function definition with proper line breaks
def create_logger_with_multiple_handlers(
    name: str,
    level: int = logging.INFO,
    log_file: Optional[str] = None,
    console_output: bool = True,
    file_rotation: bool = False,
    max_bytes: int = 1024 * 1024,
    backup_count: int = 5,
) -> logging.Logger:
    """Create a logger with multiple handler options."""
    
    # Long dictionary with proper formatting
    config = {
        "name": name,
        "level": level,
        "handlers": ["console"],
        "formatters": ["detailed"],
        "filters": [],
        "propagate": True,
    }
    
    # Long list with proper formatting
    handler_types = [
        "console",
        "file",
        "email",
        "syslog",
        "http",
        "queue",
    ]
    
    return logger
```

#### ❌ **Poor: Lines Too Long**
```python
# Function definition too long for Black
def create_logger_with_multiple_handlers(name, level=logging.INFO, log_file=None, console_output=True, file_rotation=False, max_bytes=1024*1024, backup_count=5):

# Dictionary too long for Black
config = {"name": name, "level": level, "handlers": ["console"], "formatters": ["detailed"], "filters": [], "propagate": True}

# List too long for Black
handler_types = ["console", "file", "email", "syslog", "http", "queue"]
```

### **2. String Formatting**

#### ✅ **Good: Consistent String Quotes**
```python
# Black prefers double quotes
message = "Hello, World!"
filename = "config.ini"
user_input = "Enter your name: "

# f-strings with double quotes
greeting = f"Hello, {name}!"
log_message = f"Processing file: {filename}"
error_msg = f"Error in {function_name}: {error}"

# Multi-line strings
long_message = (
    "This is a very long message that needs to be "
    "broken across multiple lines for readability."
)

# Raw strings
regex_pattern = r"\d{3}-\d{2}-\d{4}"
file_path = r"C:\Users\username\Documents\file.txt"
```

#### ❌ **Poor: Inconsistent String Quotes**
```python
# Mixed quote styles (Black will standardize)
message = 'Hello, World!'
filename = "config.ini"
user_input = 'Enter your name: '

# Inconsistent f-string quotes
greeting = f'Hello, {name}!'
log_message = f"Processing file: {filename}"
```

### **3. Spacing and Operators**

#### ✅ **Good: Proper Spacing**
```python
# Proper spacing around operators
x = 1 + 2 * 3
result = (a + b) * (c - d)
power = 2 ** 10

# Proper spacing in comparisons
if x > 5 and y < 10:
    print("Condition met")

# Proper spacing in function calls
function_call(param1, param2, param3)
method_call().chain().another_chain()

# Proper spacing in assignments
variable = value
multiple_vars = 1, 2, 3
```

#### ❌ **Poor: Inconsistent Spacing**
```python
# Inconsistent spacing (Black will fix)
x=1+2*3
result=(a+b)*(c-d)
power=2**10

if x>5 and y<10:
    print("Condition met")

function_call(param1,param2,param3)
method_call().chain().another_chain()

variable=value
multiple_vars=1,2,3
```

### **4. List and Dictionary Formatting**

#### ✅ **Good: Clean List/Dict Formatting**
```python
# Clean list formatting
simple_list = [1, 2, 3, 4, 5]
nested_list = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
]

# Clean dictionary formatting
simple_dict = {"key1": "value1", "key2": "value2"}
nested_dict = {
    "level1": {
        "level2": {
            "level3": "value",
        },
    },
}

# Long lists with proper formatting
long_list = [
    "item1",
    "item2",
    "item3",
    "item4",
    "item5",
    "item6",
    "item7",
    "item8",
]
```

#### ❌ **Poor: Cramped Formatting**
```python
# Cramped formatting (Black will expand)
simple_list = [1,2,3,4,5]
nested_list = [[1,2,3],[4,5,6],[7,8,9]]

simple_dict = {"key1":"value1","key2":"value2"}
nested_dict = {"level1":{"level2":{"level3":"value"}}}

long_list = ["item1","item2","item3","item4","item5","item6","item7","item8"]
```

### **5. Function and Class Definitions**

#### ✅ **Good: Clean Function Definitions**
```python
# Clean function definition
def simple_function(param1, param2):
    return param1 + param2

# Function with many parameters
def complex_function(
    name: str,
    age: int,
    email: str,
    phone: str,
    address: str,
    city: str,
    country: str,
) -> dict:
    """Create a user profile."""
    return {
        "name": name,
        "age": age,
        "email": email,
        "phone": phone,
        "address": address,
        "city": city,
        "country": country,
    }

# Class with proper formatting
class UserManager:
    def __init__(
        self,
        database_url: str,
        max_connections: int = 10,
        timeout: float = 30.0,
    ):
        self.database_url = database_url
        self.max_connections = max_connections
        self.timeout = timeout
```

#### ❌ **Poor: Cramped Definitions**
```python
# Cramped function definition (Black will expand)
def complex_function(name:str,age:int,email:str,phone:str,address:str,city:str,country:str)->dict:

# Cramped class definition
class UserManager:
    def __init__(self,database_url:str,max_connections:int=10,timeout:float=30.0):
        self.database_url=database_url
        self.max_connections=max_connections
        self.timeout=timeout
```

### **6. Import Statements**

#### ✅ **Good: Clean Import Formatting**
```python
# Standard library imports
import os
import sys
import logging
from pathlib import Path
from typing import Optional, List, Dict

# Third-party imports
import rich
from rich.console import Console
from rich.logging import RichHandler

# Local imports
from .config import load_config
from .handlers import ThreadSafeQueueHandler
from .utils import setup_logging

# Long import with proper line breaking
from very_long_module_name import (
    VeryLongClassName,
    AnotherLongClassName,
    YetAnotherLongClassName,
)
```

#### ❌ **Poor: Cramped Imports**
```python
# Cramped imports (Black will expand)
from very_long_module_name import VeryLongClassName,AnotherLongClassName,YetAnotherLongClassName

# Mixed import styles
import os,sys,logging
from pathlib import Path
from typing import Optional,List,Dict
```

### **7. Control Flow Statements**

#### ✅ **Good: Clean Control Flow**
```python
# Clean if statements
if condition1 and condition2:
    do_something()
elif condition3 or condition4:
    do_something_else()
else:
    do_default()

# Clean for loops
for item in items:
    process_item(item)

for i in range(10):
    if i % 2 == 0:
        print(f"{i} is even")

# Clean while loops
while condition:
    do_work()
    update_condition()
```

#### ❌ **Poor: Cramped Control Flow**
```python
# Cramped control flow (Black will expand)
if condition1 and condition2:do_something()
elif condition3 or condition4:do_something_else()
else:do_default()

for item in items:process_item(item)
for i in range(10):if i%2==0:print(f"{i} is even")

while condition:do_work();update_condition()
```

## Black with Other Tools

### **1. Black + isort Integration**

#### **Configuration for Compatibility**
```toml
# pyproject.toml
[tool.black]
line-length = 88
target-version = ['py311', 'py312', 'py313']

[tool.isort]
profile = "black"
multi_line_output = 3
line_length = 88
known_first_party = ["repository_folder"]
known_third_party = ["rich", "configparser", "logging"]
```

#### **Workflow Integration**
```yaml
name: Format Check
on: [push, pull_request]

jobs:
  format:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
          
      - name: Install tools
        run: |
          pip install black isort
          
      - name: Check formatting
        run: |
          black --check --diff .
          isort --check-only --diff .
```

### **2. Black + Flake8 Integration**

#### **Configuration for Compatibility**
```ini
# .flake8
[flake8]
max-line-length = 88
extend-ignore = E203, W503
exclude = .git,__pycache__,build,dist,.venv,venv
```

#### **Workflow Integration**
```yaml
name: Code Quality
on: [push, pull_request]

jobs:
  quality:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
          
      - name: Install tools
        run: |
          pip install black flake8
          
      - name: Check formatting
        run: |
          black --check --diff .
          
      - name: Check style
        run: |
          flake8 . --count --exit-zero --max-complexity=10 --max-line-length=88
```

## Common Black Issues and Solutions

### **1. Line Length Violations**

#### **Problem: Function Definition Too Long**
```python
# ❌ Too long
def very_long_function_name_with_many_parameters(param1, param2, param3, param4, param5, param6, param7, param8, param9, param10, param11, param12, param13, param14, param15, param16, param17, param18, param19, param20):

# ✅ Solution: Break into multiple lines
def very_long_function_name_with_many_parameters(
    param1, param2, param3, param4, param5,
    param6, param7, param8, param9, param10,
    param11, param12, param13, param14, param15,
    param16, param17, param18, param19, param20
):
```

#### **Problem: Long Dictionary/List**
```python
# ❌ Too long
config = {"name": "app", "version": "1.0.0", "description": "A very long description that exceeds the line limit", "author": "Developer", "email": "dev@example.com"}

# ✅ Solution: Break into multiple lines
config = {
    "name": "app",
    "version": "1.0.0",
    "description": "A very long description that exceeds the line limit",
    "author": "Developer",
    "email": "dev@example.com",
}
```

### **2. String Quote Issues**

#### **Problem: Inconsistent Quotes**
```python
# ❌ Mixed quotes
message = 'Hello, World!'
filename = "config.ini"
user_input = 'Enter name: '

# ✅ Solution: Let Black standardize to double quotes
message = "Hello, World!"
filename = "config.ini"
user_input = "Enter name: "
```

### **3. Spacing Issues**

#### **Problem: Inconsistent Spacing**
```python
# ❌ Inconsistent spacing
x=1+2*3
if x>5:
    print("x is greater than 5")

# ✅ Solution: Let Black add proper spacing
x = 1 + 2 * 3
if x > 5:
    print("x is greater than 5")
```

## Best Practices

### **1. Development Workflow**
```bash
# Pre-commit formatting
black . --check

# Auto-format before committing
black .
git add .
git commit -m "Format code with Black"
```

### **2. CI/CD Integration**
```yaml
# Always check formatting in CI
- name: Check formatting
  run: black --check --diff .
```

### **3. IDE Integration**

#### **VS Code Settings**
```json
// .vscode/settings.json
{
    "python.formatting.provider": "black",
    "python.formatting.blackArgs": ["--line-length", "88"],
    "editor.formatOnSave": true,
    "python.linting.enabled": true
}
```

#### **PyCharm Settings**
- Install Black plugin
- Set Black as default formatter
- Enable format on save

### **4. Team Standards**
- **Consistent Configuration**: Use same Black settings across team
- **Pre-commit Hooks**: Enforce formatting before commits
- **CI Enforcement**: Fail builds on formatting violations
- **Documentation**: Document Black usage in project README

## Troubleshooting

### **1. Common Errors**

#### **Black Fails to Format**
```bash
# Check Black version
black --version

# Check configuration
black --config pyproject.toml --check .

# Verbose output
black --verbose .
```

#### **Configuration Conflicts**
```bash
# Validate configuration
black --config pyproject.toml --validate

# Check for syntax errors
python -c "import tomllib; tomllib.load(open('pyproject.toml', 'rb'))"
```

### **2. Performance Issues**
```bash
# Limit scope for faster runs
black repository_folder/ --check

# Use specific file patterns
black --include '\.py$' .

# Exclude large directories
black --exclude '/(venv|.git|build|dist)/' .
```

### **3. Integration Issues**

#### **Black vs isort Conflicts**
```toml
# Ensure isort uses Black profile
[tool.isort]
profile = "black"
multi_line_output = 3
line_length = 88
```

#### **Black vs Flake8 Conflicts**
```ini
# Configure Flake8 to work with Black
[flake8]
extend-ignore = E203, W503
max-line-length = 88
```

## Conclusion

Black is an essential tool for maintaining consistent Python code formatting. By integrating it into your development workflow and following its formatting guidelines, you can ensure your code is always readable and professionally formatted.

### **Key Takeaways**
1. **Use Black consistently** across your project
2. **Configure Black properly** with pyproject.toml
3. **Integrate with CI/CD** for automated formatting checks
4. **Use with isort** for import sorting compatibility
5. **Configure IDEs** for real-time formatting
6. **Train your team** on Black-compatible coding practices

### **Recommended Workflow**
1. **Local Development**: Use Black in IDE with format-on-save
2. **Pre-commit**: Run Black before committing
3. **CI/CD**: Check formatting in automated workflows
4. **Auto-fix**: Use GitHub Actions for automatic formatting

### **Resources**
- [Black Documentation](https://black.readthedocs.io/)
- [Black GitHub Repository](https://github.com/psf/black)
- [Black Configuration Guide](https://black.readthedocs.io/en/stable/usage_and_configuration/)
- [Black with isort](https://pycqa.github.io/isort/docs/configuration/profiles/black/)
- [PEP 8 Style Guide](https://www.python.org/dev/peps/pep-0008/)

---

*This guide covers the essential aspects of using Black for Python code formatting. For more advanced usage and custom configurations, refer to the official Black documentation.*
