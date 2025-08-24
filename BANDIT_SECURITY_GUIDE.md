# Bandit Security Analysis Guide

## What is Bandit?

**Bandit** is a security linter designed to find common security issues in Python code. It scans your code for patterns that could lead to security vulnerabilities and provides detailed reports with severity levels and remediation guidance.

### Key Features
- **Security-Focused**: Specifically designed to catch security-related code patterns
- **Configurable**: Customizable rules and severity levels
- **CI/CD Integration**: Easy to integrate into automated workflows
- **Python-Specific**: Built specifically for Python code analysis
- **Rule-Based**: Uses a comprehensive set of security rules

## How Bandit Works

Bandit analyzes Python code by:
1. **Parsing**: Converts Python code into an Abstract Syntax Tree (AST)
2. **Rule Application**: Applies security rules to identify problematic patterns
3. **Issue Detection**: Flags code that matches security vulnerability patterns
4. **Reporting**: Generates detailed reports with severity levels and line numbers

## Installation

```bash
# Install Bandit
pip install bandit

# Install with additional security tools
pip install bandit[toml]  # TOML configuration support
```

## Basic Usage

### Command Line
```bash
# Scan current directory
bandit .

# Scan specific file
bandit src/main.py

# Scan with specific severity levels
bandit -r . -f json -o bandit-report.json

# Scan with custom configuration
bandit -c .bandit -r .
```

### Python API
```python
import bandit.core.manager
import bandit.core.config_manager

# Configure Bandit
config = bandit.core.config_manager.BanditConfig()
manager = bandit.core.manager.BanditManager(config, 'file', debug=False)

# Run analysis
manager.run_tests()
```

## Integration in GitHub Actions Workflows

### Basic Workflow Integration
```yaml
name: Security Analysis
on: [push, pull_request]

jobs:
  security:
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
          pip install bandit
          
      - name: Run Bandit security analysis
        run: |
          bandit -r . -f json -o bandit-report.json
          
      - name: Upload security report
        uses: actions/upload-artifact@v3
        with:
          name: bandit-security-report
          path: bandit-report.json
```

### Advanced Workflow with Multiple Tools
```yaml
name: Code Quality & Security
on: [push, pull_request]

jobs:
  security-analysis:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
          
      - name: Install security tools
        run: |
          python -m pip install --upgrade pip
          pip install bandit safety
          
      - name: Run Bandit analysis
        run: |
          echo "Running Bandit security analysis..."
          bandit -r . -f json -o bandit-report.json || true
          
      - name: Run Safety dependency check
        run: |
          echo "Checking for vulnerable dependencies..."
          safety check --json --output bandit-safety-report.json || true
          
      - name: Generate security summary
        run: |
          echo "## Security Analysis Results" >> $env:GITHUB_STEP_SUMMARY
          echo "" >> $env:GITHUB_STEP_SUMMARY
          
          if (Test-Path "bandit-report.json") {
            echo "### Bandit Security Issues" >> $env:GITHUB_STEP_SUMMARY
            $banditData = Get-Content "bandit-report.json" | ConvertFrom-Json
            if ($banditData.results.Count -gt 0) {
              foreach ($issue in $banditData.results) {
                echo "- **$($issue.issue_severity)**: $($issue.issue_text) in $($issue.filename):$($issue.line_number)" >> $env:GITHUB_STEP_SUMMARY
              }
            } else {
              echo "- No security issues found" >> $env:GITHUB_STEP_SUMMARY
            }
          }
          
          if (Test-Path "bandit-safety-report.json") {
            echo "### Dependency Vulnerabilities" >> $env:GITHUB_STEP_SUMMARY
            $safetyData = Get-Content "bandit-safety-report.json" | ConvertFrom-Json
            if ($safetyData.Count -gt 0) {
              foreach ($vuln in $safetyData) {
                echo "- **$($vuln.severity)**: $($vuln.package) - $($vuln.description)" >> $env:GITHUB_STEP_SUMMARY
              }
            } else {
              echo "- No vulnerable dependencies found" >> $env:GITHUB_STEP_SUMMARY
            }
          }
          
      - name: Upload security reports
        uses: actions/upload-artifact@v3
        with:
          name: security-reports
          path: |
            bandit-report.json
            bandit-safety-report.json
```

## Configuration

### .bandit Configuration File
```ini
# .bandit
[bandit]
exclude_dirs = tests,venv,.git
skips = B101,B601
targets = src

# Severity levels: LOW, MEDIUM, HIGH
# Confidence levels: LOW, MEDIUM, HIGH
```

### pyproject.toml Configuration
```toml
[tool.bandit]
exclude_dirs = ["tests", "venv", ".git"]
skips = ["B101", "B601"]
targets = ["src"]

[tool.bandit.assert_used]
skips = ["*_test.py", "*/test_*.py"]

[tool.bandit.bandit_baseline]
# Path to baseline file
baseline = "bandit-baseline.json"
```

## Common Bandit Issues and Solutions

### 1. **B101: assert_used**
**Issue**: Use of `assert` statements in production code
```python
# ❌ Problematic
def divide(a, b):
    assert b != 0, "Division by zero"
    return a / b

# ✅ Secure
def divide(a, b):
    if b == 0:
        raise ValueError("Division by zero")
    return a / b
```

### 2. **B102: exec_used**
**Issue**: Use of `exec()` function
```python
# ❌ Problematic
user_input = input("Enter code: ")
exec(user_input)

# ✅ Secure
def safe_eval(expression):
    # Use ast.literal_eval for safe evaluation
    import ast
    return ast.literal_eval(expression)
```

### 3. **B103: set_bad_file_permissions**
**Issue**: Setting overly permissive file permissions
```python
# ❌ Problematic
import os
os.chmod("config.ini", 0o777)

# ✅ Secure
import os
os.chmod("config.ini", 0o600)  # Owner read/write only
```

### 4. **B104: hardcoded_bind_all_interfaces**
**Issue**: Binding to all network interfaces
```python
# ❌ Problematic
import socket
s = socket.socket()
s.bind(('0.0.0.0', 8080))

# ✅ Secure
import socket
s = socket.socket()
s.bind(('127.0.0.1', 8080))  # Localhost only
```

### 5. **B105: hardcoded_password_string**
**Issue**: Hardcoded passwords in source code
```python
# ❌ Problematic
PASSWORD = "secret123"

# ✅ Secure
import os
PASSWORD = os.environ.get('DB_PASSWORD')
if not PASSWORD:
    raise ValueError("Database password not configured")
```

### 6. **B601: paramiko_calls**
**Issue**: Insecure SSH key handling
```python
# ❌ Problematic
import paramiko
client = paramiko.SSHClient()
client.set_missing_host_key_policy(paramiko.AutoAddPolicy())

# ✅ Secure
import paramiko
client = paramiko.SSHClient()
client.load_system_host_keys()
client.set_missing_host_key_policy(paramiko.RejectPolicy())
```

### 7. **B701: jinja2_autoescape_false**
**Issue**: Disabled Jinja2 auto-escape
```python
# ❌ Problematic
from jinja2 import Environment
env = Environment(autoescape=False)

# ✅ Secure
from jinja2 import Environment
env = Environment(autoescape=True)
```

## Writing Secure Code for Bandit

### General Principles

#### 1. **Input Validation**
```python
# ❌ Insecure
def process_user_input(data):
    return eval(data)

# ✅ Secure
def process_user_input(data):
    import ast
    try:
        return ast.literal_eval(data)
    except (ValueError, SyntaxError):
        raise ValueError("Invalid input format")
```

#### 2. **Exception Handling**
```python
# ❌ Problematic
def cleanup_loggers():
    try:
        # Some cleanup code
        pass
    except Exception:
        continue  # Bandit B110: try_except_continue

# ✅ Secure
def cleanup_loggers():
    try:
        # Some cleanup code
        pass
    except (ValueError, TypeError) as e:
        print(f"Cleanup error: {e}")
        # Handle specific exceptions appropriately
```

#### 3. **File Operations**
```python
# ❌ Problematic
def read_file(filename):
    with open(filename, 'r') as f:
        return f.read()

# ✅ Secure
def read_file(filename):
    import os
    # Validate file path
    if not os.path.exists(filename):
        raise FileNotFoundError(f"File not found: {filename}")
    
    # Use absolute path to prevent directory traversal
    abs_path = os.path.abspath(filename)
    if not abs_path.startswith(os.getcwd()):
        raise ValueError("Invalid file path")
    
    with open(abs_path, 'r') as f:
        return f.read()
```

#### 4. **Network Security**
```python
# ❌ Problematic
import socket
def create_server():
    s = socket.socket()
    s.bind(('0.0.0.0', 8080))
    return s

# ✅ Secure
import socket
def create_server():
    s = socket.socket()
    s.bind(('127.0.0.1', 8080))  # Localhost only
    return s
```

#### 5. **Configuration Management**
```python
# ❌ Problematic
DATABASE_URL = "postgresql://user:password@localhost/db"

# ✅ Secure
import os
DATABASE_URL = os.environ.get('DATABASE_URL')
if not DATABASE_URL:
    raise ValueError("DATABASE_URL environment variable not set")
```

## Bandit Rule Categories

### **B1xx: General Issues**
- **B101**: `assert_used` - Assert statements
- **B102**: `exec_used` - Exec function usage
- **B103**: `set_bad_file_permissions` - File permissions

### **B2xx: Crypto Issues**
- **B201**: `flask_debug_true` - Flask debug mode
- **B202**: `assert_used` - Assert in tests
- **B203**: `assert_used` - Assert in production

### **B3xx: Network Issues**
- **B301**: `pickle` - Pickle usage
- **B302**: `marshal` - Marshal usage
- **B303**: `md5` - MD5 usage

### **B4xx: Shell Issues**
- **B401**: `import_telnetlib` - Telnet import
- **B402**: `import_ftplib` - FTP import
- **B403**: `import_xml_etree` - XML parsing

### **B5xx: SQL Issues**
- **B501**: `request_with_no_cert_validation` - No SSL verification
- **B502**: `ssl_with_bad_version` - Bad SSL version
- **B503**: `ssl_with_bad_defaults` - Bad SSL defaults

## Suppressing False Positives

### Inline Suppression
```python
# Suppress specific rule for this line
password = "temp123"  # nosec B105

# Suppress multiple rules
exec(code)  # nosec B102,B601

# Suppress with explanation
assert user.is_admin  # nosec B101 - This is a test file
```

### File-Level Suppression
```python
# -*- coding: utf-8 -*-
# nosec B101,B102,B601
"""
This file contains test code where security warnings are expected.
"""

def test_function():
    assert True  # nosec B101
    exec("print('test')")  # nosec B102
```

## Best Practices

### 1. **Regular Scanning**
- Run Bandit in CI/CD pipelines
- Scan before each release
- Monitor for new security issues

### 2. **Baseline Management**
```bash
# Generate baseline
bandit -r . -f json -o bandit-baseline.json

# Use baseline to ignore known issues
bandit -r . -b bandit-baseline.json
```

### 3. **Rule Customization**
- Disable rules that don't apply to your project
- Add custom rules for project-specific concerns
- Maintain rule documentation

### 4. **Team Training**
- Educate developers on security best practices
- Review Bandit reports in code reviews
- Use Bandit findings for security training

## Integration with Other Tools

### **Pre-commit Hooks**
```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/PyCQA/bandit
    rev: v1.7.5
    hooks:
      - id: bandit
        args: [-r, ., -f, json, -o, bandit-report.json]
```

### **VS Code Integration**
```json
// .vscode/settings.json
{
    "python.linting.banditEnabled": true,
    "python.linting.banditArgs": ["-r", ".", "-f", "json"]
}
```

### **PyCharm Integration**
- Install Bandit plugin
- Configure inspection settings
- Enable real-time analysis

## Troubleshooting Common Issues

### **False Positives**
```python
# Use nosec comments for legitimate cases
import subprocess
subprocess.run(['ls'])  # nosec B607 - This is a trusted command
```

### **Performance Issues**
```bash
# Limit scan scope
bandit -r src/ -x "*/tests/*"

# Use parallel processing
bandit -r . -p 4
```

### **Configuration Conflicts**
```bash
# Check configuration
bandit -c .bandit --help

# Validate configuration
bandit -c .bandit --validate
```

## Conclusion

Bandit is an essential tool for maintaining Python code security. By integrating it into your development workflow and following secure coding practices, you can significantly reduce security vulnerabilities in your applications.

### **Key Takeaways**
1. **Run Bandit regularly** in CI/CD pipelines
2. **Fix high-severity issues** immediately
3. **Document suppressions** with clear explanations
4. **Train your team** on security best practices
5. **Use Bandit as part** of a comprehensive security strategy

### **Resources**
- [Bandit Documentation](https://bandit.readthedocs.io/)
- [Bandit GitHub Repository](https://github.com/PyCQA/bandit)
- [Python Security Best Practices](https://python-security.readthedocs.io/)
- [OWASP Python Security](https://owasp.org/www-project-python-security-top-10/)

---

*This guide covers the essential aspects of using Bandit for Python security analysis. For more advanced usage and custom rules, refer to the official Bandit documentation.*
