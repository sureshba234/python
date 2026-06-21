<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Python Modules

## Table of Contents

1. [Introduction to Modules](#introduction-to-modules)
    - [What is a Module](#what-is-a-module)
    - [Why Modules Exist](#why-modules-exist)
    - [Benefits of Modular Programming](#benefits-of-modular-programming)
    - [Built-in Modules](#built-in-modules)
    - [Standard Library Modules](#standard-library-modules)
    - [Third-party Modules](#third-party-modules)
    - [User-defined Modules](#user-defined-modules)
2. [Creating Modules](#creating-modules)
    - [Creating a .py File as a Module](#creating-a-py-file-as-a-module)
    - [Module Structure](#module-structure)
    - [Writing Reusable Code](#writing-reusable-code)
    - [Module Documentation](#module-documentation)
    - [Module-level Variables](#module-level-variables)
    - [Module-level Functions](#module-level-functions)
    - [Constants](#constants)
3. [Import System Basics](#import-system-basics)
    - [import module](#import-module)
    - [Import Multiple Modules](#import-multiple-modules)
    - [from module import name](#from-module-import-name)
    - [from module import *](#from-module-import-)
    - [Aliasing Imports using as](#aliasing-imports-using-as)
    - [Importing Specific Attributes](#importing-specific-attributes)
4. [Module Namespaces](#module-namespaces)
    - [Global Namespace](#global-namespace)
    - [Local Namespace](#local-namespace)
    - [Module Namespace](#module-namespace)
    - [Name Resolution](#name-resolution)
    - [Avoiding Name Collisions](#avoiding-name-collisions)
5. [Module Execution](#module-execution)
    - [How Python Executes Modules](#how-python-executes-modules)
    - [Module Loading Process](#module-loading-process)
    - [Import-time Execution](#import-time-execution)
    - [Re-import Behavior](#re-import-behavior)
6. [Special Module Attributes](#special-module-attributes)
    - [`__name__`](#__name__)
    - [`__file__`](#__file__)
    - [`__doc__`](#__doc__)
    - [`__package__`](#__package__)
    - [`__cached__`](#__cached__)
7. [`__main__`](#__main__)
    - [Meaning of `__main__`](#meaning-of-__main__)
    - [`if __name__ == "__main__"`](#if-name-main)
    - [Script vs Module](#script-vs-module)
    - [Entry Point Patterns](#entry-point-patterns)
8. [Practical Examples](#practical-examples)
    - [Utility Module](#utility-module)
    - [Math Module](#math-module)
    - [Configuration Module](#configuration-module)
    - [Helper Module](#helper-module)
    - [Logging Module](#logging-module)
9. [Real-World Usage](#real-world-usage)
    - [Organizing Application Code](#organizing-application-code)
    - [Reusable Utilities](#reusable-utilities)
    - [Shared Business Logic](#shared-business-logic)
10. [Best Practices](#best-practices)
    - [Naming Conventions](#naming-conventions)
    - [Documentation](#documentation)
    - [Avoiding Circular Imports](#avoiding-circular-imports)
    - [Import Organization](#import-organization)
    - [Code Reuse](#code-reuse)
11. [Common Mistakes](#common-mistakes)
    - [Wildcard Imports](#wildcard-imports)
    - [Circular Imports](#circular-imports-1)
    - [Namespace Pollution](#namespace-pollution)
    - [Running Import-heavy Code](#running-import-heavy-code)
12. [Interview Questions](#interview-questions)
13. [Exercises](#exercises)
    - [Beginner Exercises](#beginner-exercises)
    - [Intermediate Exercises](#intermediate-exercises)
    - [Advanced Exercises](#advanced-exercises)
14. [Summary](#summary)
15. [Modules Cheat Sheet](#modules-cheat-sheet)

***

## Introduction to Modules

### What is a Module

A **module** in Python is a file containing Python definitions and statements. The file name is the module name with the suffix `.py` added. Modules allow you to organize Python code into logical units that can be reused across different programs.

```python
# my_module.py - This is a module file
def greet(name):
    return f"Hello, {name}!"

CONSTANT = 42
```

When you import this module, all the definitions become available in your program .

### Why Modules Exist

Modules exist to solve several fundamental problems in software development:

1. **Code Organization**: Break large programs into manageable files
2. **Reusability**: Use the same code in multiple programs without copying
3. **Namespacing**: Prevent name collisions between different parts of code
4. **Maintainability**: Update one file without affecting others
5. **Collaboration**: Multiple developers can work on different modules simultaneously

Without modules, every Python program would be a single monolithic file, making it difficult to manage as it grows .

### Benefits of Modular Programming

| Benefit | Description |
| :-- | :-- |
| **Code Reorganization** | Logical separation of functionality into distinct files |
| **Reusability** | Write once, use everywhere - import modules across projects |
| **Maintainability** | Fix bugs in one place; changes don't affect other modules |
| **Testing** | Test modules independently before integration |
| **Collaboration** | Teams work on different modules without conflicts |
| **Scalability** | Add new features by creating new modules |
| **Readability** | Smaller, focused files are easier to understand |

### Built-in Modules

Built-in modules are part of the Python interpreter itself and are available immediately without installation. They provide essential functionality.

**Common built-in modules:**

```python
import sys          # System-specific parameters and functions
import os           # Operating system interfaces
import math         # Mathematical functions
import random       # Random variable generator
import datetime     # Date and time operations
import json         # JSON encoder/decoder
import collections  # High-performance container datatypes
import itertools    # Iterator functions
import functools    # Functional tools
import re           # Regular expression operations
```

Example usage:

```python
import math
import sys

print(math.sqrt(16))        # 4.0
print(sys.platform)         # 'linux' or 'darwin' or 'win32'
```


### Standard Library Modules

The Python Standard Library is a vast collection of modules that come with Python but are separate from the core interpreter. It contains over 200 modules .

**Key standard library modules:**

```python
# File and directory operations
import os.path
import shutil
import glob

# Network operations
import urllib.request
import socket
import http.client

# Data compression
import zipfile
import gzip
import bz2

# Database
import sqlite3
import pickle

# Testing
import unittest
import pytest
import doctest

# Logging
import logging

# Parsing
import csv
import xml.etree.ElementTree
```


### Third-party Modules

Third-party modules are created by the Python community and distributed via package repositories like PyPI (Python Package Index). They require installation using `pip`.

**Popular third-party modules:**

```bash
# Install using pip
pip install requests       # HTTP library
pip install numpy          # Numerical computing
pip install pandas         # Data analysis
pip install beautifulsoup4 # Web scraping
pip install flask          # Web framework
pip install django         # Web framework
pip install matplotlib     # Plotting
pip install pytest         # Testing framework
```

Example:

```python
import requests

response = requests.get('https://api.github.com')
print(response.status_code)  # 200
```


### User-defined Modules

User-defined modules are modules you create yourself. They are the foundation of custom Python applications.

**Creating your first module:**

```python
# my_utils.py
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

PI = 3.14159
```

**Using your module:**

```python
# main.py
import my_utils

result = my_utils.add(5, 3)
print(result)  # 8

print(my_utils.PI)  # 3.14159
```


***

## Creating Modules

### Creating a .py File as a Module

Any file with a `.py` extension can be a Python module. The process is simple:

1. Create a file with a `.py` extension
2. Add Python code (functions, classes, variables)
3. Import it in other files
```bash
# Create the module file
touch my_module.py
```

```python
# my_module.py
"""My first module documentation."""

def hello():
    print("Hello from my_module!")

version = "1.0.0"
```


### Module Structure

A well-structured module follows this pattern:

```python
# my_module.py

# 1. Module docstring (required for good documentation)
"""
Module description.
What it does and how to use it.
"""

# 2. Import statements
import os
import sys
from typing import List, Optional

# 3. Module-level constants
MAX_SIZE = 100
DEFAULT_NAME = "unknown"

# 4. Module-level variables
_counter = 0

# 5. Function definitions
def public_function(x: int) -> int:
    """Public function with docstring."""
    return x * 2

def _private_function(x: int) -> int:
    """Private function (prefixed with _)."""
    return x * 3

# 6. Class definitions
class MyClass:
    """Class with docstring."""
    
    def __init__(self, name: str):
        self.name = name
    
    def method(self):
        return f"Hello, {self.name}"

# 7. __all__ definition (optional, for controlling __import__)
__all__ = ['public_function', 'MyClass', 'MAX_SIZE']

# 8. Optional main execution block
if __name__ == "__main__":
    print("Module executed as script")
```


### Writing Reusable Code

To write reusable module code:

1. **Use clear function names** that describe what they do
2. **Add type hints** for better IDE support
3. **Include docstrings** explaining parameters and return values
4. **Avoid side effects** at module level
5. **Make functions independent** (don't rely on global state)
```python
# good_module.py
from typing import List, Optional

def filter_positive_numbers(numbers: List[int]) -> List[int]:
    """
    Return only positive numbers from the list.
    
    Args:
        numbers: List of integers to filter
        
    Returns:
        List containing only positive integers
        
    Example:
        >>> filter_positive_numbers([-1, 0, 1, 2, -3])
        [1, 2]
    """
    return [num for num in numbers if num > 0]

def calculate_average(numbers: List[float]) -> Optional[float]:
    """
    Calculate the average of a list of numbers.
    
    Args:
        numbers: List of floating-point numbers
        
    Returns:
        Average as float, or None if list is empty
    """
    if not numbers:
        return None
    return sum(numbers) / len(numbers)
```


### Module Documentation

Every module should have a docstring at the top:

```python
# math_utils.py
"""
Mathematical utility functions for common operations.

This module provides functions for:
- Basic arithmetic operations
- Statistical calculations
- Number transformations

Usage:
    from math_utils import add, calculate_mean
    
    result = add(5, 3)
    average = calculate_mean([1, 2, 3, 4, 5])
"""

def add(a: float, b: float) -> float:
    """Add two numbers and return the result."""
    return a + b
```

**Accessing module documentation:**

```python
import math_utils

print(math_utils.__doc__)  # Print the module docstring
help(math_utils)           # Show full help for the module
```


### Module-level Variables

Variables defined at the module level are **global** to that module:

```python
# config.py
"""Configuration module with module-level variables."""

# Constants (should not change)
API_VERSION = "2.0"
MAX_RETRIES = 5
DEFAULT_TIMEOUT = 30

# State variables (can change)
_connection_count = 0
_last_error = None

def increment_connection():
    """Increment the connection counter."""
    _connection_count += 1

def get_connection_count():
    """Get the current connection count."""
    return _connection_count
```

**Important:** Module-level variables persist across function calls within the same module but are isolated from other modules.

### Module-level Functions

Functions at module level are the primary way to expose functionality:

```python
# string_utils.py
"""String manipulation utilities."""

from typing import List, Optional

def capitalize_words(text: str) -> str:
    """Capitalize the first letter of each word."""
    return text.title()

def remove_whitespace(text: str) -> str:
    """Remove all whitespace from a string."""
    return text.replace(" ", "").replace("\n", "").replace("\t", "")

def split_into_lines(text: str) -> List[str]:
    """Split text into a list of lines."""
    return text.splitlines()

def is_valid_email(email: str) -> bool:
    """
    Check if a string is a valid email format.
    
    Note: This is a simple check, not RFC-compliant.
    """
    return "@" in email and "." in email.split("@")[1]
```


### Constants

Constants are module-level variables that should not change. Use uppercase names:

```python
# constants.py
"""Application constants."""

# Numeric constants
MAX_USERS = 1000
MIN_PASSWORD_LENGTH = 8
DEFAULT_PAGE_SIZE = 50

# String constants
API_ENDPOINT = "https://api.example.com"
APP_NAME = "MyApplication"
LOG_FORMAT = "%(asctime)s - %(name)s - %(levelname)s - %(message)s"

# Color constants (for CLI output)
COLOR_RED = "\033[91m"
COLOR_GREEN = "\033[92m"
COLOR_YELLOW = "\033[93m"
COLOR_RESET = "\033[0m"

# Boolean constants
DEBUG_MODE = False
ENABLE_CACHE = True

# Collection constants
SUPPORTED_FORMATS = ["json", "xml", "csv"]
ALLOWED_METHODS = ("GET", "POST", "PUT", "DELETE")
```

**Best practices for constants:**

- Use uppercase names with underscores
- Group related constants together
- Add comments explaining each constant
- Don't modify constants after definition

***

## Import System Basics

### import module

The basic import statement loads a module and makes it available under its name:

```python
# main.py
import math

# Access module contents using the module name
result = math.sqrt(16)
print(result)  # 4.0

value = math.pi
print(value)  # 3.141592653589793
```

**What happens:**

1. Python finds the module
2. Executes all code in the module (top to bottom)
3. Creates a module object in `sys.modules`
4. Makes the module available under the name `math`

### Import Multiple Modules

You can import multiple modules in several ways:

```python
# Method 1: Separate import statements (recommended)
import math
import os
import sys

# Method 2: Multiple imports on one line
import math, os, sys

# Method 3: Import with aliases
import math as m
import os as operating_system
import sys as system

# Using the imports
print(m.sqrt(25))
print(operating_system.getcwd())
print(system.platform)
```

**Best practice:** Use separate import statements for clarity and easier debugging.

### from module import name

Import specific names (functions, classes, variables) from a module:

```python
# main.py
from math import sqrt, pi, pow

# Now you can use these directly without module prefix
result = sqrt(16)
print(result)  # 4.0

circle_area = pi * 5 ** 2
print(circle_area)  # 78.53981633974483

raised = pow(2, 10)
print(raised)  # 1024
```

**Advantages:**

- Cleaner code (no module prefix)
- Faster access (no attribute lookup)

**Disadvantages:**

- Can cause name collisions
- Less clear where names come from


### from module import *

Import all public names from a module (wildcard import):

```python
# main.py
from math import *

# All math functions are now available directly
print(sqrt(16))   # 4.0
print(pi)         # 3.141592653589793
print(cos(0))     # 1.0
```

**⚠️ WARNING:** This is generally considered a **bad practice** because:

- Pollutes the namespace with unknown names
- Makes code harder to read and understand
- Can cause name collisions
- Violates explicit import principles

**When it might be acceptable:**

- Interactive experimentation (not production code)
- Very small modules with no collision risk


### Aliasing Imports using as

Use `as` to create aliases for modules or imported names:

```python
# Module aliasing
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

data = np.array([1, 2, 3, 4])
df = pd.DataFrame(data)
plt.plot(data)

# Name aliasing
from math import sqrt as square_root, pi as PI

result = square_root(25)
print(f"PI = {PI}")  # PI = 3.141592653589793

# Function aliasing for clarity
from datetime import datetime as DateTime

now = DateTime.now()
```

**Common conventions:**

- `numpy` → `np`
- `pandas` → `pd`
- `matplotlib.pyplot` → `plt`
- `sklearn` → `sk`


### Importing Specific Attributes

Import nested attributes from modules:

```python
# Import from submodules
from os.path import join, exists, isfile

# Import from packages
from collections import defaultdict, Counter

# Import class methods
from datetime import datetime, date

# Import specific functions
from random import random, randint, choice

# Using the imports
path = join("folder", "file.txt")
print(exists(path))

numbers = [1, 2, 3, 4, 5]
print(choice(numbers))  # Random element
```

**Importing with conditions:**

```python
# Conditional imports
try:
    from typing import List, Dict
except ImportError:
    # Python < 3.5 fallback
    List = dict = None

# Platform-specific imports
import sys
if sys.platform == "win32":
    import msvcrt
else:
    import termios
```


***

## Module Namespaces

### Global Namespace

The **global namespace** contains names defined at the top level of a module (not inside functions or classes):

```python
# module1.py
global_var = "I'm global"  # This is in the global namespace

def my_function():
    local_var = "I'm local"  # This is in the local namespace
    print(global_var)  # Can access global_var
```

**Key characteristics:**

- Created when the module is loaded
- Exists until the module is unloaded
- Accessible from any function in the module
- Each module has its own global namespace


### Local Namespace

The **local namespace** contains names defined inside a function:

```python
def calculate(a, b):
    # Parameters and local variables are in local namespace
    result = a + b
    temp = result * 2
    return temp

# 'result' and 'temp' exist only while calculate() is running
```

**Key characteristics:**

- Created when function is called
- Destroyed when function returns
- Not accessible outside the function
- Each function call creates a new local namespace


### Module Namespace

Each module has its own **module namespace**, which is the module's global namespace:

```python
# module_a.py
value_a = "from module A"

def func_a():
    return value_a

# module_b.py
value_b = "from module B"

def func_b():
    return value_b

# main.py
import module_a
import module_b

# Each module has its own namespace
print(module_a.value_a)  # "from module A"
print(module_b.value_b)  # "from module B"

# Namespaces are isolated
# module_a cannot access module_b.value_b directly
```

**Namespace dictionary:**

```python
import module_a

print(module_a.__dict__)  # Access the module's namespace as a dictionary
```


### Name Resolution

Python resolves names using **LEGB rule**:

1. **L**ocal - Names in the current function
2. **E**nclosing - Names in enclosing functions (for nested functions)
3. **G**lobal - Names in the current module's global namespace
4. **B**uilt-in - Names in Python's built-in namespace
```python
# Built-in
len = 10  # This overrides the built-in len() function

def outer():
    # Enclosing
    len = 5
    
    def inner():
        # Local
        len = 3
        print(len)  # 3 (local)
    
    inner()
    print(len)  # 5 (enclosing)

outer()
print(len)  # 10 (global)
```

**Using `global` keyword:**

```python
counter = 0

def increment():
    global counter  # Explicitly use global variable
    counter += 1

increment()
print(counter)  # 1
```

**Using `nonlocal` keyword (Python 3+):**

```python
def outer():
    count = 0
    
    def inner():
        nonlocal count  # Use enclosing (non-global) variable
        count += 1
    
    inner()
    print(count)  # 1

outer()
```


### Avoiding Name Collisions

Name collisions occur when two modules define the same name:

```python
# module_a.py
def process():
    return "from module A"

# module_b.py
def process():
    return "from module B"

# main.py - Problematic!
from module_a import process
from module_b import process

# Which process() is this?
result = process()  # "from module B" (last import wins!)
```

**Solutions:**

1. **Use module prefixes:**
```python
import module_a
import module_b

result_a = module_a.process()
result_b = module_b.process()
```

2. **Use aliases:**
```python
from module_a import process as process_a
from module_b import process as process_b

result_a = process_a()
result_b = process_b()
```

3. **Import specific names carefully:**
```python
from module_a import process as process_from_a
# Avoid: from module_a import *
```


***

## Module Execution

### How Python Executes Modules

When you import a module, Python:

1. **Finds** the module file
2. **Creates** a new module object
3. **Executes** all code in the module (top to bottom)
4. **Stores** the module in `sys.modules`
5. **Returns** the module object
```python
# my_module.py
print("Module is being executed!")

def my_function():
    return "Hello"

x = 42
print(f"Variable x = {x}")

# This code runs immediately when imported
```

```python
# main.py
import my_module

# Output:
# Module is being executed!
# Variable x = 42
```

**Important:** Code at the module level executes **every time** the module is imported (unless already cached).

### Module Loading Process

Python's import process:

```
1. Check sys.modules (cache)
   ├─ Found? → Return cached module
   └─ Not found? → Continue

2. Find the module
   ├─ Built-in module? → Load from interpreter
   ├─ In sys.path? → Load from file
   └─ Not found? → Raise ImportError

3. Create module object
   └─ Add to sys.modules

4. Execute module code
   └─ Top to bottom

5. Return module
```

```python
import sys

# Check if module is already loaded
if 'math' in sys.modules:
    print("math is already loaded")

# Import adds to sys.modules
import math

print('math' in sys.modules)  # True

print(sys.modules['math'])  # Module object
```


### Import-time Execution

Any code at module level executes during import:

```python
# config_loader.py
import requests

print("Loading configuration...")  # Runs on import

# This executes on import (might be slow!)
CONFIG = requests.get("https://api.example.com/config").json()

def get_config():
    return CONFIG

# Database connection on import (bad practice!)
connection = create_database_connection()
```

**Better approach - lazy initialization:**

```python
# config_loader.py
import requests

CONFIG = None

def load_config():
    """Load configuration on demand."""
    if CONFIG is None:
        print("Loading configuration...")
        CONFIG = requests.get("https://api.example.com/config").json()
    return CONFIG

def get_config():
    return load_config()
```


### Re-import Behavior

**Python does NOT re-execute modules on re-import:**

```python
# my_module.py
counter = 0
print("Module executed!")

# main.py
import my_module
print(my_module.counter)  # 0

import my_module  # Already cached!
print(my_module.counter)  # Still 0

# To force re-import (rarely needed):
import importlib
import my_module

importlib.reload(my_module)  # Re-executes module
```

**Why?** Performance optimization. Modules are cached in `sys.modules`.

```python
import sys

# Check import cache
print(len(sys.modules))  # Number of loaded modules

# Clear a module from cache
del sys.modules['my_module']
import my_module  # Now re-executes
```

**When re-import happens:**

- Module file is deleted during runtime
- Module is removed from `sys.modules`
- Using `importlib.reload()`

***

## Special Module Attributes

### `__name__`

The `__name__` attribute contains the module's name as a string:

```python
# my_module.py
print(__name__)  # "my_module"

def get_name():
    return __name__
```

```python
# main.py
import my_module

print(my_module.__name__)  # "my_module"
print(my_module.get_name())  # "my_module"
```

**When executed as script:**

```python
# script.py
print(__name__)  # "__main__" when run directly
```

```bash
python script.py  # Prints: __main__
python -c "import script"  # Prints: script
```


### `__file__`

The `__file__` attribute contains the path to the module file:

```python
# my_module.py
import os

print(__file__)  # "/path/to/my_module.py"
print(os.path.abspath(__file__))  # Absolute path
print(os.path.dirname(__file__))  # Directory containing module
print(os.path.basename(__file__))  # Just the filename
```

```python
# Useful for finding related files
def get_resource_path(filename):
    """Get path to a resource file in the same directory."""
    module_dir = os.path.dirname(__file__)
    return os.path.join(module_dir, filename)

resource = get_resource_path("config.json")
```


### `__doc__`

The `__doc__` attribute contains the module's docstring:

```python
# my_module.py
"""
This is my module's docstring.
It describes what the module does.
"""

def my_function():
    """Function docstring."""
    pass
```

```python
import my_module

print(my_module.__doc__)
# Output:
# This is my module's docstring.
# It describes what the module does.

# Access function docstring
print(my_module.my_function.__doc__)  # "Function docstring."

# Using help()
help(my_module)
```


### `__package__`

The `__package__` attribute indicates the package containing the module:

```python
# my_package/my_module.py
print(__package__)  # "my_package"

# my_package/subpackage/nested_module.py
print(__package__)  # "my_package.subpackage"
```

```python
# Top-level module (not in a package)
print(__package__)  # None or ""
```

**Used for:** Relative imports within packages.

### `__cached__`

The `__cached__` attribute contains the path to the compiled bytecode file (`.pyc`):

```python
# my_module.py
print(__cached__)  # "/path/to/__pycache__/my_module.cpython-39.pyc"
```

**Purpose:** Python caches compiled modules for faster imports.

```python
import os

if __cached__:
    cache_dir = os.path.dirname(__cached__)
    print(f"Cache directory: {cache_dir}")
```

**List all cached modules:**

```python
import sys
import os

for name, module in sys.modules.items():
    if hasattr(module, '__cached__') and module.__cached__:
        print(f"{name}: {module.__cached__}")
```


***

## `__main__`

### Meaning of `__main__`

`__main__` is the name of the **top-level script** or **entry point** of a Python program:

```python
# script.py
print(__name__)

if __name__ == "__main__":
    print("This script is being run directly")
else:
    print("This script is being imported")
```

```bash
# Run directly
python script.py
# Output:
# __main__
# This script is being run directly

# Import as module
python -c "import script"
# Output:
# script
# This script is being imported
```


### `if __name__ == "__main__"`

This is the most common pattern for making modules usable as both scripts and imports:

```python
# utils.py
"""Utility functions module."""

def add(a, b):
    return a + b

def multiply(a, b):
    return a * b

# This code only runs when executed as script
if __name__ == "__main__":
    print("Testing utils module...")
    print(f"add(2, 3) = {add(2, 3)}")
    print(f"multiply(4, 5) = {multiply(4, 5)}")
```

```bash
# Run as script
python utils.py
# Output:
# Testing utils module...
# add(2, 3) = 5
# multiply(4, 5) = 20

# Import as module
python -c "from utils import add; print(add(10, 20))"
# Output:
# 30
# (The if __name__ block is NOT executed)
```


### Script vs Module

| Aspect | Script | Module |
| :-- | :-- | :-- |
| **Primary purpose** | Execute program logic | Provide reusable code |
| **`__name__` value** | `"__main__"` | Module name (e.g., `"my_module"`) |
| **Execution** | `python script.py` | `import my_module` |
| **Entry point** | `if __name__ == "__main__"` | Function/class definitions |
| **Can be both?** | ✅ Yes, with pattern | ✅ Yes, with pattern |

**Dual-purpose module:**

```python
# calculator.py
"""A calculator that can be imported or run as script."""

def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def main():
    """Main function when run as script."""
    print("Calculator CLI")
    print(f"10 + 5 = {add(10, 5)}")
    print(f"10 - 5 = {subtract(10, 5)}")

# Entry point
if __name__ == "__main__":
    main()
```

```bash
# Run as CLI
python calculator.py
# Output:
# Calculator CLI
# 10 + 5 = 15
# 10 - 5 = 5

# Import and use functions
python -c "from calculator import add; print(add(7, 8))"
# Output:
# 15
```


### Entry Point Patterns

**Pattern 1: Simple main block**

```python
if __name__ == "__main__":
    # Direct execution
    print("Running...")
    main_function()
```

**Pattern 2: Dedicated main function**

```python
def main():
    """Application entry point."""
    # Setup
    # Execute
    # Cleanup
    pass

if __name__ == "__main__":
    main()
```

**Pattern 3: With argument parsing**

```python
import sys

def main(args=None):
    """Main function with argument handling."""
    if args is None:
        args = sys.argv[1:]
    
    if len(args) < 2:
        print("Usage: program <num1> <num2>")
        return 1
    
    result = add(int(args[0]), int(args[1]))
    print(f"Result: {result}")
    return 0

if __name__ == "__main__":
    sys.exit(main())
```

**Pattern 4: Using `argparse`**

```python
import argparse

def main():
    parser = argparse.ArgumentParser(description="Calculator")
    parser.add_argument("a", type=int, help="First number")
    parser.add_argument("b", type=int, help="Second number")
    parser.add_argument("--op", default="add", choices=["add", "sub"], help="Operation")
    
    args = parser.parse_args()
    
    if args.op == "add":
        result = args.a + args.b
    else:
        result = args.a - args.b
    
    print(f"Result: {result}")

if __name__ == "__main__":
    main()
```


***

## Practical Examples

### Utility Module

```python
# utils.py
"""
General utility functions for common operations.
"""

from typing import List, Any, Optional
import datetime

def flatten_list(nested: List[List[Any]]) -> List[Any]:
    """
    Flatten a nested list into a single list.
    
    Args:
        nested: List of lists to flatten
        
    Returns:
        Single flattened list
        
    Example:
        >>> flatten_list([[1, 2], [3, 4], [5]])
        [1, 2, 3, 4, 5]
    """
    return [item for sublist in nested for item in sublist]

def chunk_list(items: List[Any], size: int) -> List[List[Any]]:
    """
    Split a list into chunks of specified size.
    
    Args:
        items: List to chunk
        size: Maximum size of each chunk
        
    Returns:
        List of chunks
    """
    return [items[i:i + size] for i in range(0, len(items), size)]

def get_current_timestamp() -> int:
    """Get current Unix timestamp."""
    return int(datetime.datetime.now().timestamp())

def format_date(date_obj: datetime.datetime) -> str:
    """Format datetime object as readable string."""
    return date_obj.strftime("%Y-%m-%d %H:%M:%S")

def safe_get(dictionary: dict, key: Any, default: Any = None) -> Any:
    """
    Safely get a value from dictionary with default.
    
    Args:
        dictionary: Dictionary to access
        key: Key to look up
        default: Default value if key not found
        
    Returns:
        Value or default
    """
    return dictionary.get(key, default)

if __name__ == "__main__":
    # Test the utilities
    print("Testing utils module...")
    
    nested = [[1, 2], [3, 4], [5]]
    print(f"Flattened: {flatten_list(nested)}")
    
    items = [1, 2, 3, 4, 5, 6, 7]
    print(f"Chunks: {chunk_list(items, 3)}")
    
    print(f"Timestamp: {get_current_timestamp()}")
    print(f"Formatted: {format_date(datetime.datetime.now())}")
    
    data = {"name": "Alice", "age": 30}
    print(f"Safe get: {safe_get(data, 'email', 'unknown')}")
```


### Math Module

```python
# math_utils.py
"""
Extended mathematical utilities.
"""

from typing import List, Optional
import math

# Constants
E = math.e
PI = math.pi
SQRT2 = math.sqrt(2)
SQRT5 = math.sqrt(5)

def factorial(n: int) -> int:
    """
    Calculate factorial of n.
    
    Args:
        n: Non-negative integer
        
    Returns:
        n! (factorial of n)
        
    Raises:
        ValueError: If n is negative
    """
    if n < 0:
        raise ValueError("Factorial not defined for negative numbers")
    return math.factorial(n)

def is_prime(n: int) -> bool:
    """
    Check if a number is prime.
    
    Args:
        n: Integer to check
        
    Returns:
        True if prime, False otherwise
    """
    if n < 2:
        return False
    if n == 2:
        return True
    if n % 2 == 0:
        return False
    
    for i in range(3, int(math.sqrt(n)) + 1, 2):
        if n % i == 0:
            return False
    return True

def fibonacci(n: int) -> int:
    """
    Calculate the nth Fibonacci number.
    
    Args:
        n: Position in Fibonacci sequence (0-indexed)
        
    Returns:
        nth Fibonacci number
    """
    if n < 0:
        raise ValueError("n must be non-negative")
    if n <= 1:
        return n
    
    a, b = 0, 1
    for _ in range(2, n + 1):
        a, b = b, a + b
    return b

def mean(numbers: List[float]) -> Optional[float]:
    """Calculate arithmetic mean."""
    if not numbers:
        return None
    return sum(numbers) / len(numbers)

def median(numbers: List[float]) -> Optional[float]:
    """Calculate median value."""
    if not numbers:
        return None
    sorted_nums = sorted(numbers)
    n = len(sorted_nums)
    mid = n // 2
    
    if n % 2 == 0:
        return (sorted_nums[mid - 1] + sorted_nums[mid]) / 2
    return sorted_nums[mid]

def standard_deviation(numbers: List[float]) -> Optional[float]:
    """Calculate standard deviation."""
    if not numbers or len(numbers) < 2:
        return None
    
    m = mean(numbers)
    variance = sum((x - m) ** 2 for x in numbers) / len(numbers)
    return math.sqrt(variance)

if __name__ == "__main__":
    print("Math Utilities Tests")
    print(f"factorial(5) = {factorial(5)}")  # 120
    print(f"is_prime(17) = {is_prime(17)}")  # True
    print(f"fibonacci(10) = {fibonacci(10)}")  # 55
    print(f"mean([1,2,3,4,5]) = {mean([1,2,3,4,5])}")  # 3.0
    print(f"median([1,3,5,7]) = {median([1,3,5,7])}")  # 4.0
```


### Configuration Module

```python
# config.py
"""
Application configuration management.
"""

import os
from typing import Dict, Any, Optional
from pathlib import Path

# Default configuration values
DEFAULTS: Dict[str, Any] = {
    "debug": False,
    "log_level": "INFO",
    "max_connections": 100,
    "timeout": 30,
    "cache_enabled": True,
    "api_base_url": "https://api.example.com",
    "database_url": "sqlite:///app.db",
}

# Configuration instance (singleton pattern)
_config: Dict[str, Any] = {}

def load_config(config_file: Optional[str] = None) -> Dict[str, Any]:
    """
    Load configuration from file or defaults.
    
    Args:
        config_file: Path to configuration file (JSON)
        
    Returns:
        Configuration dictionary
    """
    global _config
    
    # Start with defaults
    _config = DEFAULTS.copy()
    
    # Override with environment variables
    for key in _config:
        env_value = os.environ.get(f"{key.upper()}")
        if env_value is not None:
            _config[key] = parse_env_value(env_value)
    
    # Override with config file if provided
    if config_file and Path(config_file).exists():
        import json
        with open(config_file, 'r') as f:
            file_config = json.load(f)
            _config.update(file_config)
    
    return _config

def parse_env_value(value: str) -> Any:
    """Parse environment variable string to appropriate type."""
    if value.lower() in ("true", "yes", "1"):
        return True
    if value.lower() in ("false", "no", "0"):
        return False
    try:
        return int(value)
    except ValueError:
        try:
            return float(value)
        except ValueError:
            return value

def get_config(key: str, default: Any = None) -> Any:
    """
    Get a configuration value.
    
    Args:
        key: Configuration key
        default: Default value if key not found
        
    Returns:
        Configuration value
    """
    if not _config:
        load_config()
    return _config.get(key, default)

def set_config(key: str, value: Any) -> None:
    """Set a configuration value."""
    if not _config:
        load_config()
    _config[key] = value

def get_all_config() -> Dict[str, Any]:
    """Get entire configuration dictionary."""
    if not _config:
        load_config()
    return _config.copy()

if __name__ == "__main__":
    # Test configuration
    config = load_config()
    
    print("Application Configuration:")
    for key, value in config.items():
        print(f"  {key}: {value}")
    
    print(f"\nDebug mode: {get_config('debug')}")
    set_config("debug", True)
    print(f"After change: {get_config('debug')}")
```


### Helper Module

```python
# helpers.py
"""
Helper functions for common programming tasks.
"""

from typing import List, Dict, Any, Callable, TypeVar
import functools
import time

T = TypeVar('T')

def memoize(func: Callable) -> Callable:
    """
    Memoization decorator for caching function results.
    
    Args:
        func: Function to memoize
        
    Returns:
        Memoized function
    """
    cache = {}
    
    @functools.wraps(func)
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    
    return wrapper

def retry(max_attempts: int = 3, delay: float = 1.0):
    """
    Retry decorator for handling transient failures.
    
    Args:
        max_attempts: Maximum number of attempts
        delay: Delay between attempts in seconds
        
    Returns:
        Retry decorator
    """
    def decorator(func: Callable) -> Callable:
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    time.sleep(delay)
            return None
        return wrapper
    return decorator

def debounce(delay: float):
    """
    Debounce decorator to limit function call frequency.
    
    Args:
        delay: Minimum delay between calls in seconds
        
    Returns:
        Debounced function
    """
    timer = None
    
    def decorator(func: Callable) -> Callable:
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            nonlocal timer
            
            def call():
                func(*args, **kwargs)
                timer = None
            
            if timer:
                timer.cancel()
            
            timer = threading.Timer(delay, call)
            timer.start()
        return wrapper
    return decorator

def chunker(iterable: List[T], size: int) -> List[List[T]]:
    """Split iterable into chunks."""
    return [iterable[i:i + size] for i in range(0, len(iterable), size)]

def merge_dicts(dict1: Dict[str, Any], dict2: Dict[str, Any]) -> Dict[str, Any]:
    """
    Merge two dictionaries.
    
    Args:
        dict1: First dictionary
        dict2: Second dictionary
        
    Returns:
        Merged dictionary (dict2 values override dict1)
    """
    result = dict1.copy()
    result.update(dict2)
    return result

def deep_get(dictionary: Dict, key: str, default: Any = None) -> Any:
    """
    Safely get value from nested dictionary using dot notation.
    
    Args:
        dictionary: Dictionary to access
        key: Dot-separated key path (e.g., "user.name")
        default: Default value if not found
        
    Returns:
        Value or default
    """
    keys = key.split(".")
    value = dictionary
    
    for k in keys:
        if isinstance(value, dict) and k in value:
            value = value[k]
        else:
            return default
    return value

if __name__ == "__main__":
    import threading
    
    # Test memoize
    @memoize
    def expensive_calc(n):
        print(f"Calculating for {n}")
        return n * n
    
    print(expensive_calc(5))  # Calculates
    print(expensive_calc(5))  # Cached
    
    # Test merge_dicts
    d1 = {"a": 1, "b": 2}
    d2 = {"b": 3, "c": 4}
    print(merge_dicts(d1, d2))  # {"a": 1, "b": 3, "c": 4}
    
    # Test deep_get
    data = {"user": {"name": "Alice", "age": 30}}
    print(deep_get(data, "user.name"))  # Alice
    print(deep_get(data, "user.email", "unknown"))  # unknown
```


### Logging Module

```python
# logging_config.py
"""
Logging configuration and utilities.
"""

import logging
import sys
from pathlib import Path
from typing import Optional
from datetime import datetime

# Default configuration
DEFAULT_LOG_LEVEL = logging.INFO
DEFAULT_LOG_FORMAT = "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
DEFAULT_LOG_FILE = "app.log"

# Logger registry
_loggers = {}

def get_logger(
    name: str = "app",
    level: int = DEFAULT_LOG_LEVEL,
    log_file: Optional[str] = None,
    format_string: Optional[str] = None,
    console: bool = True
) -> logging.Logger:
    """
    Get or create a configured logger.
    
    Args:
        name: Logger name
        level: Logging level
        log_file: Path to log file
        format_string: Log format string
        console: Whether to log to console
        
    Returns:
        Configured logger instance
    """
    if name in _loggers:
        return _loggers[name]
    
    # Create logger
    logger = logging.getLogger(name)
    logger.setLevel(level)
    logger.handlers = []  # Clear existing handlers
    
    # Format
    log_format = format_string or DEFAULT_LOG_FORMAT
    formatter = logging.Formatter(log_format)
    
    # Console handler
    if console:
        console_handler = logging.StreamHandler(sys.stdout)
        console_handler.setLevel(level)
        console_handler.setFormatter(formatter)
        logger.addHandler(console_handler)
    
    # File handler
    if log_file:
        log_path = Path(log_file)
        log_path.parent.mkdir(parents=True, exist_ok=True)
        
        file_handler = logging.FileHandler(log_file)
        file_handler.setLevel(level)
        file_handler.setFormatter(formatter)
        logger.addHandler(file_handler)
    
    # Register logger
    _loggers[name] = logger
    
    return logger

def setup_application_logging(
    log_file: str = DEFAULT_LOG_FILE,
    level: int = DEFAULT_LOG_LEVEL,
    debug: bool = False
) -> logging.Logger:
    """
    Set up application-wide logging.
    
    Args:
        log_file: Path to log file
        level: Logging level
        debug: Enable debug mode
        
    Returns:
        Application logger
    """
    if debug:
        level = logging.DEBUG
    
    return get_logger(
        name="app",
        level=level,
        log_file=log_file,
        console=True
    )

def log_exception(
    logger: logging.Logger,
    message: str = "An exception occurred",
    exception: Exception = None
) -> None:
    """
    Log an exception with full details.
    
    Args:
        logger: Logger instance
        message: Error message
        exception: Exception object (optional)
    """
    if exception:
        logger.exception(f"{message}: {exception}")
    else:
        logger.error(message)

# Pre-configured loggers for common use cases
def get_db_logger() -> logging.Logger:
    """Get database logger."""
    return get_logger("database", log_file="database.log")

def get_api_logger() -> logging.Logger:
    """Get API logger."""
    return get_logger("api", log_file="api.log")

def get_auth_logger() -> logging.Logger:
    """Get authentication logger."""
    return get_logger("auth", log_file="auth.log")

if __name__ == "__main__":
    # Test logging
    logger = setup_application_logging(debug=True)
    
    logger.info("Application started")
    logger.debug("Debug message")
    logger.warning("Warning message")
    
    try:
        result = 10 / 0
    except Exception as e:
        log_exception(logger, "Division error", e)
    
    logger.info("Application finished")
    
    print(f"\nLog file created: {DEFAULT_LOG_FILE}")
```


***

## Real-World Usage

### Organizing Application Code

**Project structure with modules:**

```
my_project/
├── main.py              # Entry point
├── config.py            # Configuration
├── requirements.txt     # Dependencies
├── utils/
│   ├── __init__.py      # Makes utils a package
│   ├── helpers.py       # Helper functions
│   ├── validators.py    # Validation logic
│   └── formatters.py    # Formatting utilities
├── services/
│   ├── __init__.py
│   ├── api_service.py   # API interactions
│   ├── db_service.py    # Database operations
│   └── cache_service.py # Caching logic
├── models/
│   ├── __init__.py
│   ├── user.py          # User model
│   └── product.py       # Product model
└── tests/
    ├── __init__.py
    ├── test_helpers.py
    └── test_api.py
```

**main.py:**

```python
"""Main entry point for the application."""

import logging
from config import load_config
from utils.helpers import memoize
from services.api_service import APIService
from services.db_service import DatabaseService
from models.user import User

def main():
    """Application main function."""
    # Load configuration
    config = load_config()
    
    # Setup logging
    logger = logging.getLogger(__name__)
    logger.info("Starting application")
    
    # Initialize services
    api = APIService(config["api_url"])
    db = DatabaseService(config["database_url"])
    
    # Process users
    users = api.fetch_users()
    for user_data in users:
        user = User.from_dict(user_data)
        db.save(user)
    
    logger.info("Application completed successfully")

if __name__ == "__main__":
    main()
```


### Reusable Utilities

**Distributed utilities package:**

```python
# mypackage/utils.py
"""Utilities for the entire package."""

from typing import List, Dict, Any
import json
import hashlib

def serialize(obj: Any) -> str:
    """Serialize object to JSON string."""
    return json.dumps(obj)

def deserialize(data: str) -> Any:
    """Deserialize JSON string to object."""
    return json.loads(data)

def hash_string(text: str) -> str:
    """Create SHA256 hash of string."""
    return hashlib.sha256(text.encode()).hexdigest()

def dict_merge(base: Dict, override: Dict) -> Dict:
    """Merge two dictionaries recursively."""
    result = base.copy()
    for key, value in override.items():
        if key in result and isinstance(result[key], dict) and isinstance(value, dict):
            result[key] = dict_merge(result[key], value)
        else:
            result[key] = value
    return result
```

**Usage across project:**

```python
from mypackage.utils import serialize, hash_string

user = {"name": "Alice", "age": 30}
user_json = serialize(user)
user_hash = hash_string(user_json)
```


### Shared Business Logic

**Central business logic module:**

```python
# business_logic.py
"""Shared business logic for the application."""

from typing import List, Optional
from decimal import Decimal
from models.order import Order
from models.customer import Customer

class PricingEngine:
    """Calculate prices with discounts and taxes."""
    
    def __init__(self, tax_rate: Decimal = Decimal("0.10")):
        self.tax_rate = tax_rate
    
    def calculate_total(
        self,
        order: Order,
        discount_percent: Decimal = Decimal("0")
    ) -> Decimal:
        """
        Calculate order total with discount and tax.
        
        Args:
            order: Order object
            discount_percent: Discount percentage (0-100)
            
        Returns:
            Total amount including tax
        """
        subtotal = order.get_subtotal()
        
        # Apply discount
        discount_amount = subtotal * discount_percent / Decimal("100")
        discounted = subtotal - discount_amount
        
        # Apply tax
        tax_amount = discounted * self.tax_rate
        total = discounted + tax_amount
        
        return total.quantize(Decimal("0.01"))
    
    def apply_coupon(
        self,
        order: Order,
        coupon_code: str,
        coupons: List[dict]
    ) -> Decimal:
        """Apply coupon discount to order."""
        coupon = self._find_coupon(coupon_code, coupons)
        
        if not coupon:
            return order.get_subtotal()
        
        if coupon["type"] == "percentage":
            discount = order.get_subtotal() * coupon["value"] / Decimal("100")
        elif coupon["type"] == "fixed":
            discount = coupon["value"]
        
        return order.get_subtotal() - discount
    
    def _find_coupon(
        self,
        code: str,
        coupons: List[dict]
    ) -> Optional[dict]:
        """Find coupon by code."""
        return next(
            (c for c in coupons if c["code"] == code),
            None
        )

class InventoryManager:
    """Manage inventory checks and updates."""
    
    def check_availability(
        self,
        product_id: str,
        quantity: int,
        inventory: dict
    ) -> bool:
        """Check if product is available in inventory."""
        return inventory.get(product_id, 0) >= quantity
    
    def reserve_inventory(
        self,
        product_id: str,
        quantity: int,
        inventory: dict
    ) -> bool:
        """Reserve inventory for an order."""
        if not self.check_availability(product_id, quantity, inventory):
            return False
        
        inventory[product_id] -= quantity
        return True

if __name__ == "__main__":
    # Test business logic
    engine = PricingEngine()
    
    order = Order(items=[
        {"product": "A", "price": 100, "quantity": 2},
        {"product": "B", "price": 50, "quantity": 3}
    ])
    
    total = engine.calculate_total(order, discount_percent=Decimal("10"))
    print(f"Order total: ${total}")
```


***

## Best Practices

### Naming Conventions

**Module naming:**


| Rule | Example | ✅ Good | ❌ Bad |
| :-- | :-- | :-- | :-- |
| Use lowercase | `module_name` | `utils.py` | `Utils.py` |
| Use underscores | `snake_case` | `data_parser.py` | `dataParser.py` |
| No special chars | alphanumeric + `_` | `config.py` | `config-v2.py` |
| Descriptive names | Clear purpose | `authentication.py` | `a.py` |
| No Python keywords | Avoid conflicts | `parser.py` | `import.py` |

```python
# ✅ Good module names
# utils.py
# data_parser.py
# authentication.py
# config_settings.py
# file_handler.py

# ❌ Bad module names
# Utils.py          (uppercase)
# dataParser.py     (camelCase)
# import.py         (keyword)
# my-module.py      (hyphen)
# a.py              (too short)
```

**Function and variable naming in modules:**

```python
# ✅ Good
def calculate_total_price():
    max_retries = 5
    user_name = "Alice"

# ❌ Bad
def calc():
    maxRetries = 5
    UserName = "Alice"
```


### Documentation

**Module-level documentation:**

```python
# data_processor.py
"""
Data processing utilities for transforming and analyzing datasets.

This module provides functions for:
- CSV and JSON data parsing
- Data validation and cleaning
- Statistical analysis
- Data transformation pipelines

Examples:
    from data_processor import parse_csv, calculate_statistics
    
    data = parse_csv("data.csv")
    stats = calculate_statistics(data)
    
Attributes:
    VERSION (str): Module version number
    DEFAULT_ENCODING (str): Default file encoding
"""

VERSION = "1.2.0"
DEFAULT_ENCODING = "utf-8"

def parse_csv(filepath: str) -> list:
    """
    Parse CSV file into list of dictionaries.
    
    Args:
        filepath: Path to CSV file
        
    Returns:
        List of dictionaries representing rows
        
    Raises:
        FileNotFoundError: If file doesn't exist
        ValueError: If CSV format is invalid
    """
    pass
```

**Function documentation:**

```python
def transform_data(
    data: list,
    transformations: list = None
) -> list:
    """
    Apply transformations to dataset.
    
    Args:
        data: Input dataset as list of dictionaries
        transformations: List of transformation functions
        
    Returns:
        Transformed dataset
        
    Example:
        >>> data = [{"value": 1}, {"value": 2}]
        >>> double = lambda x: {"value": x["value"] * 2}
        >>> transform_data(data, [double])
        [{"value": 2}, {"value": 4}]
    """
```


### Avoiding Circular Imports

**Circular import problem:**

```python
# module_a.py
from module_b import func_b

def func_a():
    return "A" + func_b()

# module_b.py
from module_a import func_a

def func_b():
    return "B" + func_a()

# This causes ImportError!
```

**Solutions:**

1. **Import at function level:**
```python
# module_a.py
def func_a():
    from module_b import func_b  # Delayed import
    return "A" + func_b()
```

2. **Use `import` instead of `from ... import`:**
```python
# module_a.py
import module_b

def func_a():
    return "A" + module_b.func_b()
```

3. **Create a shared module:**
```python
# shared.py
# Common imports and utilities

# module_a.py
import shared

def func_a():
    return shared.func_common()

# module_b.py
import shared

def func_b():
    return shared.func_common()
```

4. **Reorganize code hierarchy:**
```python
# base.py
# Common functionality

# module_a.py
import base

# module_b.py
import base
```


### Import Organization

**Standard import order:**

```python
# 1. Standard library imports
import os
import sys
import json
from typing import List, Dict, Optional

# 2. Third-party imports
import numpy
import pandas
from requests import get

# 3. Local/application imports
from utils.helpers import memoize
from services.api import APIService
from models.user import User

# 4. Relative imports (if in package)
from . import submodule
from .utils import helper
```

**Grouping imports:**

```python
# ✅ Good - organized and grouped
import os
import sys
from pathlib import Path
from typing import List, Dict, Optional, Any

import requests
import numpy as np
import pandas as pd

from utils.helpers import chunk_list
from config import load_config

# ❌ Bad - mixed and unordered
from typing import List
import os
from config import load_config
import sys
from utils.helpers import chunk_list
```

**Import comments:**

```python
import logging

# Type handling
from typing import List, Dict, Optional, Union

# External dependencies
import numpy as np
import pandas as pd

# Internal utilities
from utils.validators import validate_email
from config import get_settings
```


### Code Reuse

**Design modules for reuse:**

```python
# ✅ Good - reusable design
from typing import List, TypeVar, Callable

T = TypeVar('T')

def process_items(
    items: List[T],
    processor: Callable[[T], T]
) -> List[T]:
    """Generic item processor - highly reusable."""
    return [processor(item) for item in items]

# ❌ Bad - specific and not reusable
def process_user_names(names: List[str]) -> List[str]:
    """Process only user names."""
    return [name.upper() for name in names]
```

**Avoid side effects:**

```python
# ✅ Good - no side effects
def calculate_total(items: list) -> float:
    return sum(item["price"] * item["quantity"] for item in items)

# ❌ Bad - has side effects
global_total = 0

def calculate_total(items: list) -> float:
    global_total = sum(item["price"] * item["quantity"] for item in items)
    return global_total
```

**Use configuration parameters:**

```python
# ✅ Good - configurable
def format_currency(
    amount: float,
    currency: str = "USD",
    decimals: int = 2
) -> str:
    symbols = {"USD": "$", "EUR": "€", "GBP": "£"}
    symbol = symbols.get(currency, currency)
    return f"{symbol}{amount:.{decimals}f}"

# ❌ Bad - hardcoded
def format_currency(amount: float) -> str:
    return f"${amount:.2f}"
```


***

## Common Mistakes

### Wildcard Imports

**Problem:** `from module import *` pollutes namespace

```python
# ❌ Bad - wildcard import
from math import *
from random import *
from numpy import *

# Now you have hundreds of names in your namespace
# What does sqrt do? Where did it come from?
result = sqrt(16)  # Could be from math, numpy, or custom

# Name collision!
from math import pi
from some_other_module import pi  # Overrides math.pi!
```

**Solution:** Explicit imports

```python
# ✅ Good - explicit imports
from math import sqrt, pi
from random import random, randint
from numpy import array

result = sqrt(16)  # Clearly from math
math_pi = pi  # No ambiguity
```

**When wildcard imports might be acceptable:**

- Interactive experimentation (not production)
- Very small modules with no collision risk
- REPL/Tutorial examples


### Circular Imports

**Problem:** Modules importing each other

```python
# ❌ Bad - circular import
# users.py
from orders import get_user_orders

def get_user(name):
    orders = get_user_orders(name)
    return {"name": name, "orders": orders}

# orders.py
from users import get_user

def get_user_orders(name):
    user = get_user(name)
    return user.get("orders", [])

# Run: from users import get_user
# Result: ImportError: cannot import name
```

**Solutions:**

1. **Delay imports:**
```python
# ✅ Good - delayed import
# users.py
def get_user(name):
    from orders import get_user_orders  # Import inside function
    orders = get_user_orders(name)
    return {"name": name, "orders": orders}
```

2. **Use type checking imports:**
```python
# ✅ Good - TYPE_CHECKING
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from orders import Order  # Only for type checking

def get_user_orders(name: str) -> list:
    # Actual import at runtime
    from orders import get_orders
    return get_orders(name)
```

3. **Extract shared code:**
```python
# shared.py
# Common utilities imported by both

# users.py
import shared

# orders.py
import shared
```


### Namespace Pollution

**Problem:** Importing too many names

```python
# ❌ Bad - namespace pollution
import everything

def process():
    # Now everything is available: func1, func2, ..., func100
    # Which one is correct?
    result = func42()  # Why func42?
```

**Solution:** Import only what you need

```python
# ✅ Good - selective imports
from module import specific_func, another_func

def process():
    result = specific_func()  # Clear and intentional
```

**Avoid global modifications:**

```python
# ❌ Bad - modifies global namespace
import os
os.getcwd = lambda: "/custom/path"  # Breaks os module!

# ✅ Good - use local aliases
import os
original_getcwd = os.getcwd
custom_getcwd = lambda: "/custom/path"
```


### Running Import-heavy Code

**Problem:** Heavy operations at module level

```python
# ❌ Bad - heavy import-time execution
import requests
import sqlite3
import pandas

# These run EVERY time the module is imported!
CONFIG = requests.get("https://api.example.com/config").json()
DB = sqlite3.connect("large_database.db")
DATA = pandas.read_csv("gigantic_file.csv")

def process():
    return CONFIG, DB, DATA

# Importing this module takes 30 seconds!
import my_module
```

**Solution:** Lazy initialization

```python
# ✅ Good - lazy initialization
import requests
import sqlite3
import pandas

CONFIG = None
DB = None
DATA = None

def _load_config():
    if CONFIG is None:
        CONFIG = requests.get("https://api.example.com/config").json()
    return CONFIG

def _get_db():
    if DB is None:
        DB = sqlite3.connect("large_database.db")
    return DB

def get_data():
    if DATA is None:
        DATA = pandas.read_csv("gigantic_file.csv")
    return DATA

def process():
    return _load_config(), _get_db(), get_data()
```

**Alternative: Use `__init__` for packages**

```python
# mypackage/__init__.py
from .core import main_function

# Heavy setup happens when package is first imported
```


***

## Interview Questions

### Module Fundamentals

**1. What is a Python module?**
A module is a file containing Python code (definitions, statements) with a `.py` extension. It allows organizing code into reusable units that can be imported across programs.

**2. How do you create a module?**
Create a file with a `.py` extension containing Python code. Any `.py` file is automatically a module.

```python
# my_module.py
def greet(name):
    return f"Hello, {name}"
```

**3. What are the types of modules in Python?**

- **Built-in modules:** Part of interpreter (e.g., `sys`, `os`)
- **Standard library modules:** Come with Python (e.g., `json`, `datetime`)
- **Third-party modules:** From PyPI (e.g., `requests`, `

