<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Python Packages

## Table of Contents

1. [Introduction to Packages](#introduction-to-packages)
    - [What is a Package](#what-is-a-package)
    - [Why Packages Exist](#why-packages-exist)
    - [Package vs Module](#package-vs-module)
    - [Benefits of Packages](#benefits-of-packages)
2. [Package Structure](#package-structure)
    - [Package Directory](#package-directory)
    - [Package Hierarchy](#package-hierarchy)
    - [Package Organization](#package-organization)
    - [Package Discovery](#package-discovery)
3. [`__init__.py`](#__init__.py)
    - [Purpose](#purpose)
    - [Package Initialization](#package-initialization)
    - [Exposing APIs](#exposing-apis)
    - [Controlling Imports](#controlling-imports)
4. [Nested Packages](#nested-packages)
    - [Multi-level Package Design](#multi-level-package-design)
    - [Subpackages](#subpackages)
    - [Package Hierarchy Examples](#package-hierarchy-examples)
5. [Package Imports](#package-imports)
    - [Importing Packages](#importing-packages)
    - [Importing Subpackages](#importing-subpackages)
    - [Importing Modules from Packages](#importing-modules-from-packages)
    - [Importing Objects from Packages](#importing-objects-from-packages)
6. [Relative Imports](#relative-imports)
    - [Syntax](#syntax)
    - [Single Dot Imports](#single-dot-imports)
    - [Double Dot Imports](#double-dot-imports)
    - [Multi-level Relative Imports](#multi-level-relative-imports)
    - [Advantages and Limitations](#advantages-and-limitations)
7. [Absolute Imports](#absolute-imports)
    - [Syntax](#syntax)
    - [Best Practices](#best-practices)
    - [Comparison with Relative Imports](#comparison-with-relative-imports)
8. [Package Design](#package-design)
    - [Small Project Structure](#small-project-structure)
    - [Medium Project Structure](#medium-project-structure)
    - [Large Project Structure](#large-project-structure)
    - [Enterprise Package Architecture](#enterprise-package-architecture)
9. [Folder Structure Diagrams](#folder-structure-diagrams)
10. [Enterprise Project Layouts](#enterprise-project-layouts)
    - [Web Applications](#web-applications)
    - [Data Science Projects](#data-science-projects)
    - [Machine Learning Projects](#machine-learning-projects)
    - [CLI Tools](#cli-tools)
    - [Microservices](#microservices)
    - [Python Libraries](#python-libraries)
11. [Package Architecture](#package-architecture)
    - [Layered Architecture](#layered-architecture)
    - [Feature-based Architecture](#feature-based-architecture)
    - [Domain-driven Structure](#domain-driven-structure)
    - [Scalable Package Design](#scalable-package-design)
12. [Best Practices](#best-practices)
    - [Naming Conventions](#naming-conventions)
    - [Package Organization](#package-organization)
    - [API Exposure](#api-exposure)
    - [Dependency Management](#dependency-management)
    - [Documentation Standards](#documentation-standards)
13. [Common Mistakes](#common-mistakes)
    - [Missing `__init__.py`](#missing-initpy)
    - [Bad Package Structures](#bad-package-structures)
    - [Circular Dependencies](#circular-dependencies)
    - [Relative Import Misuse](#relative-import-misuse)
    - [Namespace Conflicts](#namespace-conflicts)
14. [Real-World Projects](#real-world-projects)
    - [Flask Project](#flask-project)
    - [FastAPI Project](#fastapi-project)
    - [Data Science Project](#data-science-project-1)
    - [Python SDK](#python-sdk)
15. [Interview Questions](#interview-questions)
16. [Exercises](#exercises)
    - [Beginner Exercises](#beginner-exercises)
    - [Intermediate Exercises](#intermediate-exercises)
    - [Advanced Exercises](#advanced-exercises)
17. [Summary](#summary)
18. [Packages Cheat Sheet](#packages-cheat-sheet)

***

## Introduction to Packages

### What is a Package

A **Python package** is a directory (folder) that contains Python module files and optionally an `__init__.py` file. It provides a hierarchical structure for organizing related modules into a single, reusable unit.

```python
# A package is simply a folder with Python files
my_package/          # ← This directory is a package
    __init__.py      # ← Package initialization (optional in Python 3.3+)
    module1.py       # ← Module 1
    module2.py       # ← Module 2
```


### Why Packages Exist

Packages exist to solve several critical problems:


| Problem | Solution |
| :-- | :-- |
| **File organization** | Group related modules together |
| **Namespace management** | Prevent naming conflicts between modules |
| **Code reuse** | Distribute functionality as installable units |
| **Maintainability** | Logical separation makes code easier to navigate |
| **Scaling** | Support large projects with hundreds of modules |

### Package vs Module

```
┌─────────────────────────────────────────────────────────┐
│                        Python Code Structure             │
├─────────────────────────────────────────────────────────┤
│  Program                                                │
│   ├─ Module (single .py file)                          │
│   │   └─ my_module.py                                  │
│   │                                                      │
│   └─ Package (directory with .py files)                │
│       ├─ my_package/                                   │
│       │   ├─ __init__.py                               │
│       │   ├─ module1.py                                │
│       │   └─ subpackage/                               │
│       │       ├─ __init__.py                           │
│       │       └─ module2.py                            │
└─────────────────────────────────────────────────────────┘
```

**Key Differences:**


| Aspect | Module | Package |
| :-- | :-- | :-- |
| **Definition** | Single `.py` file | Directory containing `.py` files |
| **Extension** | `module.py` | No extension (directory name) |
| **Contents** | Functions, classes, variables | Modules, subpackages, `__init__.py` |
| **Import** | `import module` | `import package` |
| **Hierarchy** | Flat | Can contain nested subpackages |

### Benefits of Packages

1. **Organized Code Structure**: Group related functionality logically
2. **Namespace Isolation**: `package.module` prevents naming conflicts
3. **Easy Distribution**: Packages can be published to PyPI
4. **Dependency Management**: Clear dependency boundaries
5. **Team Collaboration**: Multiple developers can work on different packages
6. **Testing**: Isolated packages enable focused unit tests
7. **Reusability**: Import entire packages across projects

***

## Package Structure

### Package Directory

A valid Python package directory must contain:

```
package_name/
├── __init__.py          # Package initialization (required for namespace packages)
├── module1.py           # Public module
├── module2.py           # Public module
├── private_module.py    # Internal module (prefix with _)
└── subpackage/          # Subpackage
    ├── __init__.py
    └── module3.py
```

**Python 3.3+ Note**: Starting with Python 3.3, `__init__.py` is optional for **namespace packages**, but still recommended for regular packages.

### Package Hierarchy

```
project/
│
├── src/                          # Source directory
│   └── my_package/               # Main package
│       ├── __init__.py           # Level 0: Package root
│       ├── core/                 # Level 1: Subpackage
│       │   ├── __init__.py
│       │   ├── engine.py
│       │   └── utils.py
│       ├── api/                  # Level 1: Subpackage
│       │   ├── __init__.py
│       │   ├── routes.py
│       │   └── controllers.py
│       └── utils/                # Level 1: Subpackage
│           ├── __init__.py
│           └── helpers.py
│
├── tests/                        # Test directory
│   ├── __init__.py
│   ├── test_core.py
│   └── test_api.py
│
├── setup.py                      # Package installation script
├── pyproject.toml                # Modern build configuration
└── requirements.txt              # Dependencies
```


### Package Organization

**Best Organization Principles:**

1. **Single Responsibility**: Each package/subpackage should have one clear purpose
2. **Logical Grouping**: Group by feature, domain, or functionality
3. **Depth Control**: Avoid excessive nesting (2-3 levels max)
4. **Public vs Private**: Use `_prefix` for internal modules
5. **API Surface**: Expose only what's needed via `__init__.py`
```python
# Good organization
my_package/
    ├── __init__.py          # Exposes public API
    ├── public_module.py     # Public functionality
    └── _internal.py         # Private, not exposed

# Bad organization
my_package/
    ├── utils_for_stuff.py   # Unclear purpose
    ├── helper2.py           # Undocumented
    ├── temp_code.py         # Temporary
    └── random_functions.py  # No organization
```


### Package Discovery

Python discovers packages through:

1. **`sys.path`**: Directories in the Python path
2. **`PYTHONPATH`**: Environment variable adding directories
3. **Installed packages**: Via `pip` in `site-packages`
```python
import sys
print(sys.path)  # See where Python looks for packages

import my_package
print(my_package.__path__)  # See package location
```

**Package Location Commands:**

```bash
# Find where a package is installed
python -c "import my_package; print(my_package.__file__)"

# List all installed packages
pip list

# Show package details
pip show my_package
```


***

## `__init__.py`

### Purpose

The `__init__.py` file serves multiple critical purposes:

1. **Package Marker**: Identifies a directory as a Python package (Python < 3.3)
2. **Initialization Code**: Runs when package is imported
3. **API Exposure**: Controls what's available when `import package` is used
4. **Variable Setup**: Defines package-level variables and constants
```python
# __init__.py
"""My Package - A comprehensive utility library."""

__version__ = "1.0.0"
__author__ = "John Doe"

# Import and expose public API
from .module1 import function_a
from .module2 import ClassB

# Make these available: import my_package; my_package.function_a()
__all__ = ["function_a", "ClassB", "__version__"]
```


### Package Initialization

`__init__.py` executes automatically when the package is imported:

```python
# my_package/__init__.py
print("Package initialized!")  # Runs on import

# Usage
import my_package  # Prints: "Package initialized!"
```

**Common Initialization Tasks:**

```python
# __init__.py
import logging
from .core.engine import Engine
from .utils.helpers import setup_config

# Initialize package-level resources
_log = logging.getLogger(__name__)
_config = setup_config()
_engine = Engine()

def get_engine():
    """Return the package engine."""
    return _engine

def log_message(msg):
    """Log a message using package logger."""
    _log.info(msg)
```


### Exposing APIs

Control what users see with `import package`:

```python
# my_package/__init__.py
"""Public API for my_package."""

# Import from submodules
from .validators import validate_email, validate_phone
from .database import Database, Connection
from .utils import format_date, parse_json

# Define public API
__all__ = [
    "validate_email",
    "validate_phone",
    "Database",
    "Connection",
    "format_date",
    "parse_json",
]

# Package metadata
__version__ = "2.1.0"
__author__ = "Team Name"
```

**Usage:**

```python
import my_package

# All these work because exposed in __init__.py
my_package.validate_email("test@example.com")
my_package.Database()
my_package.format_date("2024-01-01")

# Access metadata
print(my_package.__version__)  # "2.1.0"
```


### Controlling Imports

Prevent unwanted imports and enforce structure:

```python
# my_package/__init__.py
"""Package with controlled imports."""

# Only expose public functions
from .public import public_function

# Don't expose internal modules
# from .internal import internal_function  # ← Commented out

# Raise error for deprecated imports
import warnings

def __getattr__(name):
    """Handle deprecated attribute access."""
    if name == "old_function":
        warnings.warn(
            "old_function is deprecated, use new_function instead",
            DeprecationWarning,
            stacklevel=2
        )
        from .new_module import new_function
        return new_function
    raise AttributeError(f"module {__name__} has no attribute {name}")
```


***

## Nested Packages

### Multi-level Package Design

Deep package hierarchies for large applications:

```
enterprise_app/
├── __init__.py
├── core/                          # Level 1
│   ├── __init__.py
│   ├── database/                  # Level 2
│   │   ├── __init__.py
│   │   ├── models.py
│   │   ├── connections.py
│   │   └── queries/               # Level 3
│   │       ├── __init__.py
│   │       ├── users.py
│   │       └── orders.py
│   ├── auth/                      # Level 2
│   │   ├── __init__.py
│   │   ├── strategies.py
│   │   └── middleware.py
│   └── cache/                     # Level 2
│       ├── __init__.py
│       └── redis.py
├── api/                           # Level 1
│   ├── __init__.py
│   ├── v1/                        # Level 2
│   │   ├── __init__.py
│   │   ├── routes.py
│   │   └── controllers.py
│   └── v2/                        # Level 2
│       ├── __init__.py
│       └── routes.py
└── utils/                         # Level 1
    ├── __init__.py
    └── helpers.py
```


### Subpackages

Subpackages are packages inside packages:

```python
# my_package/__init__.py
from .subpackage import submodule

# my_package/subpackage/__init__.py
from .module3 import function_c

# my_package/subpackage/module3.py
def function_c():
    return "From subpackage"
```

**Importing Subpackages:**

```python
import my_package.subpackage
import my_package.subpackage.module3

from my_package import subpackage
from my_package.subpackage import module3
from my_package.subpackage.module3 import function_c
```


### Package Hierarchy Examples

**Example 1: Data Processing Library**

```
dataproc/
├── __init__.py
├── loaders/
│   ├── __init__.py
│   ├── csv.py
│   ├── json.py
│   └── excel.py
├── transformers/
│   ├── __init__.py
│   ├── clean.py
│   ├── aggregate.py
│   └── filter.py
└── exporters/
    ├── __init__.py
    ├── csv.py
    └── parquet.py
```

**Example 2: Web Framework**

```
webframework/
├── __init__.py
├── http/
│   ├── __init__.py
│   ├── request.py
│   ├── response.py
│   └── middleware.py
├── routing/
│   ├── __init__.py
│   ├── router.py
│   └── decorators.py
├── database/
│   ├── __init__.py
│   ├── orm.py
│   └── migrations.py
└── templates/
    ├── __init__.py
    ├── engine.py
    └── filters.py
```


***

## Package Imports

### Importing Packages

```python
# Basic package import
import my_package

# Access modules
my_package.module1.function_a()

# Import with alias
import my_package as mp
mp.module1.function_a()
```


### Importing Subpackages

```python
# Import subpackage
import my_package.subpackage

# Access submodule
my_package.subpackage.module3.function_c()

# Import subpackage with alias
import my_package.subpackage as sub
sub.module3.function_c()
```


### Importing Modules from Packages

```python
# Import specific module
from my_package import module1

# Use module
module1.function_a()

# Import multiple modules
from my_package import module1, module2

module1.function_a()
module2.function_b()
```


### Importing Objects from Packages

```python
# Import function from module
from my_package.module1 import function_a

# Direct usage
function_a()

# Import class
from my_package.module2 import ClassB

obj = ClassB()

# Import multiple objects
from my_package.module1 import function_a, function_b
from my_package.module2 import ClassB, ClassC

# Import all from module (uses __all__)
from my_package.module1 import *
```

**Package-level Imports (via `__init__.py`):**

```python
# If __init__.py exposes these:
from my_package import validate_email, Database

# Direct usage
validate_email("test@example.com")
db = Database()
```


***

## Relative Imports

### Syntax

Relative imports use dots to reference packages relative to current file:

```python
# Single dot: current package
from . import module1
from .module1 import function_a

# Double dot: parent package
from .. import submodule
from ..module2 import function_b

# Three dots: grandparent package
from ... import other_package
```


### Single Dot Imports

Reference modules in the **same package**:

```python
# my_package/module1.py
from . import module2          # Import module2 from same package
from .module2 import function_b # Import function from module2
from .utils import helper       # Import from utils in same package
```

```python
# my_package/utils.py
def helper():
    return "Helper function"
```


### Double Dot Imports

Reference modules in **parent package**:

```python
# my_package/core/database.py
from .. import auth                 # Import auth from parent (my_package)
from ..auth import middleware       # Import middleware from auth
from ..utils import helpers         # Import helpers from parent's utils
```

```python
# my_package/__init__.py
from .core import database
from .auth import middleware
from .utils import helpers
```


### Multi-level Relative Imports

```python
# my_package/core/database/queries/users.py
from ... import auth                    # Level 3: my_package.auth
from ..connections import connect       # Level 2: my_package.core.database.connections
from ....utils import helpers           # Level 4: project.utils (if in src/)
```

**Dot Reference Chart:**

```
project/
├── my_package/                # Level 0 (current)
│   ├── __init__.py
│   ├── module1.py             # from .module2 → same package
│   └── core/                  # Level 1 (subpackage)
│       ├── __init__.py
│       ├── module2.py         # from .module3 → same package
│       │                      # from ..module1 → parent package
│       └── database/          # Level 2 (subpackage)
│           ├── __init__.py
│           └── module3.py     # from .utils → same package
│                              # from ..module2 → parent (core)
│                              # from ...module1 → grandparent (my_package)
```


### Advantages and Limitations

**Advantages:**


| Advantage | Description |
| :-- | :-- |
| **Refactoring** | Move packages without changing imports |
| **Readability** | Clear relationship between modules |
| **Portability** | Package works regardless of installation path |
| **Maintainability** | Less brittle than absolute paths |

**Limitations:**


| Limitation | Description |
| :-- | :-- |
| **Limited Scope** | Only works within package hierarchy |
| **Top-level Error** | `ImportError` in script run directly |
| **Debugging** | harder to trace import paths |
| **Tool Support** | Some IDEs/linters struggle |

```python
# ❌ This will fail when running module.py directly
from . import sibling_module  # ImportError: can't have relative import in __main__

# ✅ Use absolute import for top-level scripts
import my_package.sibling_module
```


***

## Absolute Imports

### Syntax

Absolute imports specify the full path from the project root:

```python
# Full path from package root
import my_package.module1
import my_package.subpackage.module3

# Import specific objects
from my_package import function_a
from my_package.module1 import function_a, function_b
from my_package.subpackage.module3 import function_c
```


### Best Practices

**1. Use Absolute Imports for Public APIs:**

```python
# ✅ Best: Clear, explicit, works everywhere
from my_package.validators import validate_email
from my_package.database import Database

# ❌ Avoid: Relative imports in public modules
from .validators import validate_email  # Only works within package
```

**2. Use Relative Imports for Internal Modules:**

```python
# my_package/core/engine.py
from .utils import helper      # ✅ Same package, relative is fine
from ..auth import middleware  # ✅ Parent package, relative works

# But for public-facing code:
from my_package.core.utils import helper  # ✅ More explicit
```

**3. Consistent Import Style:**

```python
# ✅ Consistent absolute imports
import my_package
from my_package import module1
from my_package.module1 import function_a
from my_package.core.database import Database

# ❌ Mixed style (confusing)
import my_package
from . import module1          # Relative
from my_package.module2 import function_b  # Absolute
from ..utils import helper     # Relative
```

**4. Import Order:**

```python
# Standard library imports
import os
import sys
from typing import List, Dict

# Third-party imports
import requests
from flask import Flask

# Local package imports
import my_package
from my_package import module1
from my_package.core import Database
```


### Comparison with Relative Imports

| Aspect | Absolute Imports | Relative Imports |
| :-- | :-- | :-- |
| **Syntax** | `from package.module import obj` | `from .module import obj` |
| **Clarity** | Explicit full path | Implicit relative path |
| **Refactoring** | Must update imports | Works after move |
| **Top-level Scripts** | ✅ Works | ❌ Fails |
| **Public APIs** | ✅ Recommended | ❌ Avoid |
| **Internal Modules** | ✅ Works | ✅ Recommended |
| **IDE Support** | ✅ Excellent | ⚠️ Variable |
| **Readability** | Clear for newcomers | Clear for package members |

**Recommendation:**

```python
# Public modules & entry points: Use absolute
from my_package.validators import validate_email

# Internal package modules: Use relative
from .validators import validate_email
from ..utils import helper
```


***

## Package Design

### Small Project Structure

Perfect for utilities, scripts, and simple libraries:

```
small_project/
├── my_package/
│   ├── __init__.py
│   ├── core.py
│   ├── utils.py
│   └── config.py
├── tests/
│   ├── __init__.py
│   └── test_core.py
├── setup.py
├── requirements.txt
└── README.md
```

**`__init__.py` for small package:**

```python
# my_package/__init__.py
"""Small utility package."""

from .core import process_data
from .utils import format_output
from .config import settings

__all__ = ["process_data", "format_output", "settings"]
__version__ = "1.0.0"
```


### Medium Project Structure

For applications with multiple features:

```
medium_project/
├── src/
│   └── my_package/
│       ├── __init__.py
│       ├── api/
│       │   ├── __init__.py
│       │   ├── routes.py
│       │   └── controllers.py
│       ├── core/
│       │   ├── __init__.py
│       │   ├── engine.py
│       │   └── processor.py
│       ├── db/
│       │   ├── __init__.py
│       │   ├── models.py
│       │   └── connections.py
│       └── utils/
│           ├── __init__.py
│           ├── helpers.py
│           └── validators.py
├── tests/
│   ├── __init__.py
│   ├── test_api/
│   │   ├── test_routes.py
│   │   └── test_controllers.py
│   ├── test_core/
│   │   ├── test_engine.py
│   │   └── test_processor.py
│   └── test_db/
│       ├── test_models.py
│       └── test_connections.py
├── pyproject.toml
├── requirements.txt
├── requirements-dev.txt
└── README.md
```


### Large Project Structure

Enterprise-grade architecture with clear separation:

```
large_project/
├── src/
│   └── my_package/
│       ├── __init__.py
│       ├── __main__.py
│       ├── public_api.py
│       │
│       ├── core/                    # Core business logic
│       │   ├── __init__.py
│       │   ├── engine.py
│       │   ├── processor.py
│       │   ├── scheduler.py
│       │   └── services/
│       │       ├── __init__.py
│       │       ├── user_service.py
│       │       └── order_service.py
│       │
│       ├── api/                     # API layer
│       │   ├── __init__.py
│       │   ├── v1/
│       │   │   ├── __init__.py
│       │   │   ├── routes.py
│       │   │   ├── controllers.py
│       │   │   └── schemas.py
│       │   ├── v2/
│       │   │   ├── __init__.py
│       │   │   └── routes.py
│       │   └── middleware/
│       │       ├── __init__.py
│       │       ├── auth.py
│       │       └── logging.py
│       │
│       ├── database/                # Data layer
│       │   ├── __init__.py
│       │   ├── models/
│       │   │   ├── __init__.py
│       │   │   ├── user.py
│       │   │   └── order.py
│       │   ├── connections.py
│       │   ├── queries.py
│       │   └── migrations/
│       │       ├── __init__.py
│       │       └── v001_init.py
│       │
│       ├── infrastructure/          # External integrations
│       │   ├── __init__.py
│       │   ├── cache/
│       │   │   ├── __init__.py
│       │   │   └── redis.py
│       │   ├── messaging/
│       │   │   ├── __init__.py
│       │   │   └── kafka.py
│       │   └── storage/
│       │       ├── __init__.py
│       │       └── s3.py
│       │
│       ├── utils/                   # Shared utilities
│       │   ├── __init__.py
│       │   ├── validators.py
│       │   ├── helpers.py
│       │   ├── logging.py
│       │   └── exceptions.py
│       │
│       └── cli/                     # Command-line interface
│           ├── __init__.py
│           ├── main.py
│           └── commands.py
│
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   ├── unit/
│   │   ├── test_core/
│   │   ├── test_api/
│   │   └── test_database/
│   ├── integration/
│   │   ├── test_api_integration.py
│   │   └── test_database_integration.py
│   └── e2e/
│       └── test_end_to_end.py
│
├── docker/
│   ├── Dockerfile
│   ├── docker-compose.yml
│   └── entrypoint.sh
│
├── docs/
│   ├── index.md
│   ├── api.md
│   └── architecture.md
│
├── pyproject.toml
├── setup.cfg
├── requirements.txt
├── requirements-dev.txt
├── Makefile
└── README.md
```


### Enterprise Package Architecture

**Key Principles:**

1. **Layered Separation**: Clear boundaries between layers
2. **Feature Modules**: Group by business capability
3. **Dependency Injection**: Avoid hard dependencies
4. **Interface Abstraction**: Define contracts, not implementations
5. **Testability**: Every layer has tests
```
enterprise/
├── src/
│   └── enterprise_package/
│       ├── interfaces/              # Abstract interfaces (contracts)
│       │   ├── __init__.py
│       │   ├── repository.py
│       │   ├── service.py
│       │   └── controller.py
│       │
│       ├── application/             # Application layer (use cases)
│       │   ├── __init__.py
│       │   ├── use_cases/
│       │   │   ├── __init__.py
│       │   │   ├── create_user.py
│       │   │   └── process_order.py
│       │   └── services/
│       │       ├── __init__.py
│       │       └── user_service.py
│       │
│       ├── domain/                  # Domain layer (business logic)
│       │   ├── __init__.py
│       │   ├── entities/
│       │   │   ├── __init__.py
│       │   │   ├── user.py
│       │   │   └── order.py
│       │   ├── repositories/
│       │   │   ├── __init__.py
│       │   │   └── user_repository.py
│       │   └── services/
│       │       ├── __init__.py
│       │       └── order_service.py
│       │
│       ├── infrastructure/          # Infrastructure layer (implementations)
│       │   ├── __init__.py
│       │   ├── repositories/
│       │   │   ├── __init__.py
│       │   │   ├── sql_user_repository.py
│       │   │   └── redis_cache_repository.py
│       │   ├── databases/
│       │   │   ├── __init__.py
│       │   │   ├── postgres.py
│       │   │   └── migrations.py
│       │   └── external/
│       │       ├── __init__.py
│       │       ├── payment_api.py
│       │       └── email_service.py
│       │
│       ├── presentation/            # Presentation layer (APIs, CLI)
│       │   ├── __init__.py
│       │   ├── api/
│       │   │   ├── __init__.py
│       │   │   ├── routes.py
│       │   │   └── controllers.py
│       │   └── cli/
│       │       ├── __init__.py
│       │       └── main.py
│       │
│       └── shared/                  # Shared kernel
│           ├── __init__.py
│           ├── config.py
│           ├── exceptions.py
│           ├── logging.py
│           └── events.py
│
└── ... (tests, docs, config)
```


***

## Folder Structure Diagrams

### Basic Package Structure

```
project/
│
├── package/
│   ├── __init__.py
│   ├── module1.py
│   ├── module2.py
│   └── subpackage/
│       ├── __init__.py
│       └── module3.py
```


### Standard Library Layout

```
my_library/
├── src/
│   └── my_library/
│       ├── __init__.py
│       ├── core.py
│       ├── utils.py
│       └── models.py
├── tests/
│   ├── __init__.py
│   ├── test_core.py
│   └── test_utils.py
├── docs/
│   ├── index.md
│   └── api.md
├── pyproject.toml
├── requirements.txt
└── README.md
```


### Python Application Layout

```
my_app/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── api/
│   │   ├── __init__.py
│   │   ├── routes.py
│   │   └── controllers.py
│   ├── core/
│   │   ├── __init__.py
│   │   ├── config.py
│   │   └── security.py
│   ├── db/
│   │   ├── __init__.py
│   │   ├── models.py
│   │   └── repository.py
│   └── services/
│       ├── __init__.py
│       └── user_service.py
├── tests/
│   ├── __init__.py
│   ├── test_api.py
│   └── test_services.py
├── .env
├── .gitignore
├── docker-compose.yml
├── requirements.txt
└── README.md
```


### Complex Package with Multiple Subpackages

```
complex_package/
├── __init__.py
├── a/
│   ├── __init__.py
│   ├── b/
│   │   ├── __init__.py
│   │   ├── c/
│   │   │   ├── __init__.py
│   │   │   └── d.py
│   │   └── e.py
│   └── f.py
└── g.py
```

**Import Paths:**

```python
import complex_package
import complex_package.a
import complex_package.a.b
import complex_package.a.b.c
import complex_package.a.b.c.d
import complex_package.a.b.e
import complex_package.a.f
import complex_package.g
```


***

## Enterprise Project Layouts

### Web Applications

**Flask Enterprise Application:**

```
flask_enterprise/
├── src/
│   └── flask_app/
│       ├── __init__.py
│       ├── __main__.py
│       ├── config/
│       │   ├── __init__.py
│       │   ├── base.py
│       │   ├── development.py
│       │   ├── production.py
│       │   └── testing.py
│       ├── extensions/
│       │   ├── __init__.py
│       │   ├── db.py
│       │   ├── auth.py
│       │   └── cache.py
│       ├── modules/
│       │   ├── __init__.py
│       │   ├── users/
│       │   │   ├── __init__.py
│       │   │   ├── routes.py
│       │   │   ├── models.py
│       │   │   ├── schemas.py
│       │   │   └── services.py
│       │   ├── orders/
│       │   │   ├── __init__.py
│       │   │   ├── routes.py
│       │   │   ├── models.py
│       │   │   └── services.py
│       │   └── products/
│       │       ├── __init__.py
│       │       ├── routes.py
│       │       └── models.py
│       ├── core/
│       │   ├── __init__.py
│       │   ├── security.py
│       │   ├── logging.py
│       │   └── exceptions.py
│       ├── api/
│       │   ├── __init__.py
│       │   ├── v1/
│       │   │   ├── __init__.py
│       │   │   ├── routes.py
│       │   │   └── decorators.py
│       │   └── middleware/
│       │       ├── __init__.py
│       │       ├── auth.py
│       │       └── logging.py
│       └── utils/
│           ├── __init__.py
│           ├── validators.py
│           └── helpers.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   ├── unit/
│   │   ├── test_users.py
│   │   └── test_orders.py
│   └── integration/
│       ├── test_api.py
│       └── test_db.py
├── migrations/
│   ├── __init__.py
│   └── versions/
├── docker/
│   ├── Dockerfile
│   └── docker-compose.yml
├── requirements.txt
├── requirements-dev.txt
├── Makefile
└── README.md
```


### Data Science Projects

**Data Science Package:**

```
data_science_project/
├── src/
│   └── ds_package/
│       ├── __init__.py
│       ├── config/
│       │   ├── __init__.py
│       │   └── settings.py
│       ├── data/
│       │   ├── __init__.py
│       │   ├── loaders.py
│       │   ├── cleaners.py
│       │   ├── transformers.py
│       │   └── validators.py
│       ├── features/
│       │   ├── __init__.py
│       │   ├── extraction.py
│       │   ├── selection.py
│       │   └── engineering.py
│       ├── models/
│       │   ├── __init__.py
│       │   ├── trainer.py
│       │   ├── evaluator.py
│       │   ├── registry.py
│       │   └── pipelines/
│       │       ├── __init__.py
│       │       ├── classification.py
│       │       └── regression.py
│       ├── visualization/
│       │   ├── __init__.py
│       │   ├── charts.py
│       │   ├── plots.py
│       │   └── dashboards.py
│       ├── evaluation/
│       │   ├── __init__.py
│       │   ├── metrics.py
│       │   ├── reports.py
│       │   └── comparison.py
│       └── utils/
│           ├── __init__.py
│           ├── helpers.py
│           ├── logging.py
│           └── serialization.py
├── notebooks/
│   ├── 01_exploration.ipynb
│   ├── 02_preprocessing.ipynb
│   ├── 03_modeling.ipynb
│   └── 04_evaluation.ipynb
├── tests/
│   ├── __init__.py
│   ├── test_data/
│   ├── test_features/
│   └── test_models/
├── data/
│   ├── raw/
│   ├── processed/
│   └── models/
├── requirements.txt
├── environment.yml
└── README.md
```


### Machine Learning Projects

**ML Package Architecture:**

```
ml_project/
├── src/
│   └── ml_package/
│       ├── __init__.py
│       ├── infrastructure/
│       │   ├── __init__.py
│       │   ├── database.py
│       │   ├── storage.py
│       │   └── streaming.py
│       ├── data_processing/
│       │   ├── __init__.py
│       │   ├── ingestion.py
│       │   ├── preprocessing.py
│       │   ├── augmentation.py
│       │   └── pipelines.py
│       ├── feature_engineering/
│       │   ├── __init__.py
│       │   ├── extraction.py
│       │   ├── transformation.py
│       │   └── selection.py
│       ├── model_training/
│       │   ├── __init__.py
│       │   ├── trainers/
│       │   │   ├── __init__.py
│       │   │   ├── base.py
│       │   │   ├── supervised.py
│       │   │   └── unsupervised.py
│       │   ├── hyperparameter/
│       │   │   ├── __init__.py
│       │   │   ├── search.py
│       │   │   └── optimization.py
│       │   └── callbacks/
│       │       ├── __init__.py
│       │       ├── early_stop.py
│       │       └── checkpoint.py
│       ├── model_evaluation/
│       │   ├── __init__.py
│       │   ├── metrics.py
│       │   ├── validators.py
│       │   └── reports.py
│       ├── model_deployment/
│       │   ├── __init__.py
│       │   ├── serving/
│       │   │   ├── __init__.py
│       │   │   ├── api.py
│       │   │   └── batch.py
│       │   ├── monitoring/
│       │   │   ├── __init__.py
│       │   │   ├── drift.py
│       │   │   └── performance.py
│       │   └── versioning.py
│       └── utils/
│           ├── __init__.py
│           ├── config.py
│           ├── logging.py
│           └── serialization.py
├── tests/
├── scripts/
│   ├── train.py
│   ├── evaluate.py
│   └── deploy.py
├── configs/
│   ├── training.yaml
│   ├── evaluation.yaml
│   └── deployment.yaml
├── requirements.txt
└── README.md
```


### CLI Tools

**CLI Package Structure:**

```
cli_tool/
├── src/
│   └── cli_tool/
│       ├── __init__.py
│       ├── __main__.py
│       ├── cli/
│       │   ├── __init__.py
│       │   ├── main.py
│       │   ├── commands/
│       │   │   ├── __init__.py
│       │   │   ├── init.py
│       │   │   ├── run.py
│       │   │   ├── build.py
│       │   │   └── deploy.py
│       │   └── decorators.py
│       ├── core/
│       │   ├── __init__.py
│       │   ├── engine.py
│       │   └── processor.py
│       ├── config/
│       │   ├── __init__.py
│       │   ├── loader.py
│       │   └── validator.py
│       ├── utils/
│       │   ├── __init__.py
│       │   ├── filesystem.py
│       │   ├── logging.py
│       │   └── errors.py
│       └── version.py
├── tests/
│   ├── __init__.py
│   ├── test_cli/
│   └── test_core/
├── bin/
│   └── cli_tool
├── requirements.txt
├── setup.py
└── README.md
```


### Microservices

**Microservices Package:**

```
microservices/
├── src/
│   ├── user_service/
│   │   ├── __init__.py
│   │   ├── main.py
│   │   ├── api/
│   │   │   ├── __init__.py
│   │   │   ├── routes.py
│   │   │   └── schemas.py
│   │   ├── core/
│   │   │   ├── __init__.py
│   │   │   ├── config.py
│   │   │   └── security.py
│   │   ├── database/
│   │   │   ├── __init__.py
│   │   │   ├── models.py
│   │   │   └── repository.py
│   │   ├── services/
│   │   │   ├── __init__.py
│   │   │   └── user_service.py
│   │   └── utils/
│   │       ├── __init__.py
│   │       └── logging.py
│   │
│   ├── order_service/
│   │   ├── __init__.py
│   │   ├── main.py
│   │   ├── api/
│   │   │   ├── __init__.py
│   │   │   └── routes.py
│   │   ├── core/
│   │   │   └── __init__.py
│   │   ├── database/
│   │   │   └── __init__.py
│   │   └── services/
│   │       └── __init__.py
│   │
│   ├── payment_service/
│   │   ├── __init__.py
│   │   └── main.py
│   │
│   └── shared/
│       ├── __init__.py
│       ├── messaging/
│       │   ├── __init__.py
│       │   ├── producer.py
│       │   └── consumer.py
│       ├── events/
│       │   ├── __init__.py
│       │   ├── user_events.py
│       │   └── order_events.py
│       ├── config/
│       │   ├── __init__.py
│       │   └── base.py
│       └── utils/
│           ├── __init__.py
│           └── logging.py
├── docker/
│   ├── docker-compose.yml
│   └── Kubernetes/
├── tests/
├── requirements.txt
└── README.md
```


### Python Libraries

**Publishable Library Structure:**

```
python_library/
├── src/
│   └── library_name/
│       ├── __init__.py
│       ├── __version__.py
│       ├── public_api.py
│       ├── core/
│       │   ├── __init__.py
│       │   ├── engine.py
│       │   └── processor.py
│       ├── api/
│       │   ├── __init__.py
│       │   └── client.py
│       ├── models/
│       │   ├── __init__.py
│       │   ├── base.py
│       │   └── data.py
│       ├── utils/
│       │   ├── __init__.py
│       │   ├── validators.py
│       │   └── helpers.py
│       └── exceptions.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   ├── unit/
│   └── integration/
├── docs/
│   ├── index.md
│   ├── api.md
│   ├── usage.md
│   └── contributing.md
├── examples/
│   ├── basic_usage.py
│   ├── advanced_usage.py
│   └── integration.py
├── notebooks/
│   └── tutorial.ipynb
├── pyproject.toml
├── setup.cfg
├── requirements.txt
├── requirements-dev.txt
├── tox.ini
├── .pre-commit-config.yaml
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── publish.yml
└── README.md
```


***

## Package Architecture

### Layered Architecture

Separate concerns into distinct layers:

```
layered_package/
├── __init__.py
├── presentation/      # Layer 1: API, CLI, UI
│   ├── __init__.py
│   ├── api.py
│   └── cli.py
├── application/       # Layer 2: Use cases, business logic
│   ├── __init__.py
│   ├── services.py
│   └── use_cases.py
├── domain/            # Layer 3: Core business entities
│   ├── __init__.py
│   ├── entities.py
│   └── repositories.py
└── infrastructure/    # Layer 4: Implementations
    ├── __init__.py
    ├── db_repository.py
    └── external_api.py
```

**Import Flow:**

```python
# presentation层 imports application层
from layered_package.application import UserService

# application层 imports domain层
from layered_package.domain import User, UserRepository

# infrastructure层 implements domain interfaces
from layered_package.domain.repositories import UserRepository
```


### Feature-based Architecture

Group by business features:

```
feature_package/
├── __init__.py
├── users/               # Feature: User management
│   ├── __init__.py
│   ├── api.py
│   ├── service.py
│   ├── models.py
│   └── repository.py
├── orders/              # Feature: Order processing
│   ├── __init__.py
│   ├── api.py
│   ├── service.py
│   ├── models.py
│   └── repository.py
├── products/            # Feature: Product catalog
│   ├── __init__.py
│   ├── api.py
│   ├── service.py
│   └── models.py
└── shared/              # Shared across features
    ├── __init__.py
    ├── exceptions.py
    └── utils.py
```


### Domain-driven Structure

Follow Domain-Driven Design (DDD):

```
ddd_package/
├── __init__.py
├── user_domain/
│   ├── __init__.py
│   ├── entities/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── role.py
│   ├── services/
│   │   ├── __init__.py
│   │   ├── user_service.py
│   │   └── auth_service.py
│   ├── repositories/
│   │   ├── __init__.py
│   │   └── user_repository.py
│   └── events/
│       ├── __init__.py
│       └── user_events.py
├── order_domain/
│   ├── __init__.py
│   ├── entities/
│   │   ├── __init__.py
│   │   └── order.py
│   └── services/
│       └── __init__.py
└── shared_kernel/
    ├── __init__.py
    ├── domain_event.py
    └── repository.py
```


### Scalable Package Design

Design for growth:

```python
# Key principles for scalability:

# 1. Interface-based design
class BaseService:
    """Abstract base service."""
    def create(self, data): pass
    def get(self, id): pass
    def update(self, id, data): pass
    def delete(self, id): pass

# 2. Dependency injection
class UserService(BaseService):
    def __init__(self, repository: UserRepository):
        self._repository = repository

# 3. Modular features
# Each feature can be developed independently

# 4. Clear public API
# __init__.py exposes only what's needed
```


***

## Best Practices

### Naming Conventions

**Package Names:**


| Rule | Example | ✅/❌ |
| :-- | :-- | :-- |
| Lowercase only | `my_package` | ✅ |
| No hyphens | `my_package` (not `my-package`) | ✅ |
| Short \& descriptive | `utils`, `api` | ✅ |
| No Python keywords | `my_utils` (not `import`) | ✅ |
| AvoidAmbiguity | `data_processors` (not `dp`) | ✅ |

```python
# ✅ Good names
import requests
import numpy
import pandas
from my_utils import helper

# ❌ Bad names
import My_Package  # Capital letters
import my-package  # Hyphen
import import    # Keyword
import dp        # Too ambiguous
```


### Package Organization

**1. Group by Functionality:**

```python
# ✅ Good: Logical grouping
package/
    ├── api/
    ├── database/
    ├── auth/
    └── utils/

# ❌ Bad: Random grouping
package/
    ├── helpers.py
    ├── utils.py
    ├── stuff.py
    └──斯坦.py
```

**2. Keep Depth Manageable:**

```python
# ✅ Good: 2-3 levels max
package/
    ├── core/
    │   └── engine.py

# ❌ Bad: Excessive nesting
package/
    ├── a/
    │   ├── b/
    │   │   ├── c/
    │   │   │   ├── d/
    │   │   │   └── e.py
```

**3. Separate Public \& Private:**

```python
# ✅ Good
package/
    ├── public_module.py    # Exposed in __init__.py
    └── _internal.py        # Not exposed (prefix with _)

# ❌ Bad
package/
    ├── public.py
    ├── internal.py         # No indication it's private
    └── temp.py             # Temporary code in package
```


### API Exposure

**Control what's public:**

```python
# package/__init__.py
"""Public API."""

# Explicit imports
from .public_module import public_function
from .public_module import PublicClass

# Define __all__
__all__ = ["public_function", "PublicClass"]

# Package metadata
__version__ = "1.0.0"
```

**Avoid:**

```python
# ❌ Don't do this
from . import *  # Exposes everything

# ❌ Don't expose internal modules
from ._internal import internal_function
```


### Dependency Management

**1. Use `requirements.txt`:**

```txt
# requirements.txt
requests>=2.28.0
flask==2.3.0
numpy>=1.24.0
pydantic>=2.0.0
```

**2. Use `pyproject.toml` (Modern):**

```toml
# pyproject.toml
[project]
name = "my_package"
version = "1.0.0"
dependencies = [
    "requests>=2.28.0",
    "flask==2.3.0",
    "numpy>=1.24.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0.0",
    "black>=23.0.0",
]
```

**3. Separate Dev Dependencies:**

```txt
# requirements-dev.txt
pytest>=7.0.0
black>=23.0.0
mypy>=1.0.0
pre-commit>=3.0.0
```


### Documentation Standards

**1. Package-level docstring:**

```python
# package/__init__.py
"""
My Package - A comprehensive utility library.

This package provides utilities for data processing,
validation, and transformation.

Example:
    from my_package import validate_email
    validate_email("test@example.com")
"""
```

**2. Module docstrings:**

```python
# package/validators.py
"""
Validation utilities.

Provides functions for validating emails, phones, URLs.
"""
```

**3. Function docstrings:**

```python
def validate_email(email: str) -> bool:
    """
    Validate an email address.

    Args:
        email: The email address to validate.

    Returns:
        True if valid, False otherwise.

    Example:
        >>> validate_email("test@example.com")
        True
    """
```

**4. README.md:**

```markdown
# My Package

## Installation
```bash
pip install my-package
```


## Usage

```python
from my_package import validate_email
```


## API Reference

See [docs/api.md](docs/api.md)

```

***

## Common Mistakes

### Missing `__init__.py`

**Problem:** Package not recognized (Python < 3.3) or no API exposed.

```python
# ❌ Bad: Missing __init__.py
package/
    ├── module1.py
    └── module2.py

# import package  # ❌ ImportError
```

```python
# ✅ Good: Include __init__.py
package/
    ├── __init__.py
    ├── module1.py
    └── module2.py

# import package  # ✅ Works
```

**Python 3.3+ Note:** Namespace packages work without `__init__.py`, but regular packages should include it for API control.

### Bad Package Structures

**Problem:** Unorganized, confusing structures.

```python
# ❌ Bad: Random organization
package/
    ├── utils_for_stuff.py
    ├── helper2.py
    ├── temp_code.py
    ├── random_functions.py
    ├── class_definitions.py
    └── data_handler_backup.py

# ✅ Good: Logical organization
package/
    ├── utils/
    │   ├── helpers.py
    │   └── validators.py
    ├── core/
    │   ├── engine.py
    │   └── processor.py
    ├── models/
    │   ├── user.py
    │   └── order.py
    └── api/
        ├── routes.py
        └── controllers.py
```


### Circular Dependencies

**Problem:** Modules import each circularly.

```python
# ❌ Bad: Circular dependency
# package/module_a.py
from package.module_b import function_b

def function_a():
    return function_b()

# package/module_b.py
from package.module_a import function_a

def function_b():
    return function_a()

# Import: ImportError due to circular dependency
```

**Solutions:**

```python
# ✅ Solution 1: Lazy imports
# package/module_a.py
def function_a():
    from package.module_b import function_b  # Import inside function
    return function_b()

# ✅ Solution 2: Common interface
# package/interfaces.py
def interface_function():
    pass

# package/module_a.py
from package.interfaces import interface_function

# package/module_b.py
from package.interfaces import interface_function

# ✅ Solution 3: Extract shared code
# package/shared.py
def shared_function():
    pass

# Both modules import from shared
```


### Relative Import Misuse

**Problem:** Using relative imports incorrectly.

```python
# ❌ Bad: Relative import in top-level script
# script.py
from package import module  # ✅ Absolute
from . import module  # ❌ ImportError: can't have relative import in __main__

# ❌ Bad: Too many dots
# package/core/database.py
from ....utils import helper  # ❌ Too deep, likely wrong

# ✅ Good: Correct relative imports
# package/core/database.py
from ..utils import helper  # ✅ Parent's utils
from .queries import query  # ✅ Same package
```


### Namespace Conflicts

**Problem:** Naming conflicts between packages.

```python
# ❌ Bad: Conflicting names
# package1/utils.py
def helper():
    return "From package1"

# package2/utils.py
def helper():
    return "From package2"

# Usage conflict
from package1.utils import helper
from package2.utils import helper  # ❌ Overwrites first helper
```

```python
# ✅ Good: Clear namespacing
from package1.utils import helper as helper1
from package2.utils import helper as helper2

helper1()  # "From package1"
helper2()  # "From package2"
```

**Prevention:**

```python
# ✅ Use package prefixes
import package1.utils
import package2.utils

package1.utils.helper()
package2.utils.helper()
```


***

## Real-World Projects

### Flask Project

**Complete Flask Package Structure:**

```
flask_project/
├── src/
│   └── flask_app/
│       ├── __init__.py
│       ├── __main__.py
│       ├── config.py
│       ├── extensions.py
│       ├── application.py
│       ├── api/
│       │   ├── __init__.py
│       │   ├── routes.py
│       │   ├── controllers/
│       │   │   ├── __init__.py
│       │   │   ├── user_controller.py
│       │   │   └── order_controller.py
│       │   └── schemas/
│       │       ├── __init__.py
│       │       ├── user_schema.py
│       │       └── order_schema.py
│       ├── core/
│       │   ├── __init__.py
│       │   ├── security.py
│       │   ├── logging.py
│       │   └── exceptions.py
│       ├── models/
│       │   ├── __init__.py
│       │   ├── user.py
│       │   ├── order.py
│       │   └── product.py
│       ├── services/
│       │   ├── __init__.py
│       │   ├── user_service.py
│       │   └── order_service.py
│       ├── repositories/
│       │   ├── __init__.py
│       │   ├── user_repo.py
│       │   └── order_repo.py
│       └── utils/
│           ├── __init__.py
│           ├── validators.py
│           └── helpers.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   ├── test_api/
│   │   ├── test_routes.py
│   │   └── test_controllers.py
│   ├── test_services/
│   │   ├── test_user_service.py
│   │   └── test_order_service.py
│   └── test_models/
│       ├── test_user.py
│       └── test_order.py
├── migrations/
│   ├── __init__.py
│   └── versions/
├── requirements.txt
├── requirements-dev.txt
├── docker-compose.yml
├── .env
├── .gitignore
└── README.md
```

**`__init__.py` Example:**

```python
# src/flask_app/__init__.py
"""Flask Application Package."""

from flask import Flask
from .config import load_config
from .extensions import init_extensions
from .api.routes import api_routes
from .core.logging import setup_logging
from .core.exceptions import register_error_handlers

__version__ = "1.0.0"

def create_app():
    """Create and configure the Flask application."""
    app = Flask(__name__)
    
    # Load configuration
    app.config.update(load_config())
    
    # Initialize extensions (DB, Auth, etc.)
    init_extensions(app)
    
    # Register routes
    app.register_blueprint(api_routes)
    
    # Register error handlers
    register_error_handlers(app)
    
    # Setup logging
    setup_logging(app)
    
    return app
```


### FastAPI Project

**Complete FastAPI Package Structure:**

```
fastapi_project/
├── src/
│   └── fastapi_app/
│       ├── __init__.py
│       ├── __main__.py
│       ├── main.py
│       ├── config/
│       │   ├── __init__.py
│       │   ├── settings.py
│       │   └── database.py
│       ├── api/
│       │   ├── __init__.py
│       │   ├── v1/
│       │   │   ├── __init__.py
│       │   │   ├── routes.py
│       │   │   ├── endpoints/
│       │   │   │   ├── __init__.py
│       │   │   │   ├── users.py
│       │   │   │   └── orders.py
│       │   │   └── middlewares.py
│       │   └── deps.py
│       ├── core/
│       │   ├── __init__.py
│       │   ├── security.py
│       │   ├── logging.py
│       │   └── exceptions.py
│       ├── db/
│       │   ├── __init__.py
│       │   ├── base.py
│       │   ├── session.py
│       │   └── models/
│       │       ├── __init__.py
│       │       ├── user.py
│       │       └── order.py
│       ├── services/
│       │   ├── __init__.py
│       │   ├── user_service.py
│       │   └── order_service.py
│       ├── schemas/
│       │   ├── __init__.py
│       │   ├── user.py
│       │   └── order.py
│       └── utils/
│           ├── __init__.py
│           └── validators.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   ├── test_api/
│   └── test_services/
├── requirements.txt
├── docker-compose.yml
└── README.md
```

**`__init__.py` Example:**

```python
# src/fastapi_app/__init__.py
"""FastAPI Application Package."""

from fastapi import FastAPI
from .main import create_app
from .config.settings import settings
from .api.v1.routes import api_v1
from .core.exceptions import AppException

__version__ = "1.0.0"

# Public API
__all__ = ["create_app", "settings", "api_v1", "AppException"]
```


### Data Science Project

**Complete Data Science Package:**

```
data_science_project/
├── src/
│   └── ds_project/
│       ├── __init__.py
│       ├── config/
│       │   ├── __init__.py
│       │   └── settings.yaml
│       ├── data/
│       │   ├── __init__.py
│       │   ├── loaders/
│       │   │   ├── __init__.py
│       │   │   ├── csv_loader.py
│       │   │   └── parquet_loader.py
│       │   ├── cleaners/
│       │   │   ├── __init__.py
│       │   │   └── data_cleaner.py
│       │   └── transformers/
│       │       ├── __init__.py
│       │       ├── normalizer.py
│       │       └── encoder.py
│       ├── features/
│       │   ├── __init__.py
│       │   ├── extraction.py
│       │   ├── selection.py
│       │   └── engineering.py
│       ├── models/
│       │   ├── __init__.py
│       │   ├── trainer.py
│       │   ├── evaluator.py
│       │   └── pipelines/
│       │       ├── __init__.py
│       │       ├── classification.py
│       │       └── regression.py
│       ├── visualization/
│       │   ├── __init__.py
│       │   ├── charts.py
│       │   └── plots.py
│       └── utils/
│           ├── __init__.py
│           ├── helpers.py
│           └── logging.py
├── notebooks/
│   ├── 01_exploration.ipynb
│   ├── 02_preprocessing.ipynb
│   └── 03_modeling.ipynb
├── tests/
│   ├── __init__.py
│   ├── test_data/
│   ├── test_features/
│   └── test_models/
├── data/
│   ├── raw/
│   ├── processed/
│   └── models/
├── requirements.txt
├── environment.yml
└── README.md
```


### Python SDK

**Complete SDK Package Structure:**

```
python_sdk/
├── src/
│   └── sdk_client/
│       ├── __init__.py
│       ├── __version__.py
│       ├── client.py
│       ├── config.py
│       ├── api/
│       │   ├── __init__.py
│       │   ├── base_client.py
│       │   ├── users.py
│       │   ├── orders.py
│       │   └── products.py
│       ├── models/
│       │   ├── __init__.py
│       │   ├── base.py
│       │   ├── user.py
│       │   └── order.py
│       ├── auth/
│       │   ├── __init__.py
│       │   ├── token.py
│       │   └── credentials.py
│       ├── utils/
│       │   ├── __init__.py
│       │   ├── validators.py
│       │   ├── serializers.py
│       │   └── logging.py
│       └── exceptions.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   ├── test_api/
│   ├── test_models/
│   └── test_auth/
├── examples/
│   ├── basic_usage.py
│   ├── advanced_usage.py
│   └── error_handling.py
├── docs/
│   ├── index.md
│   ├── api.md
│   ├── authentication.md
│   └── examples.md
├── pyproject.toml
├── requirements.txt
├── requirements-dev.txt
└── README.md
```

**`__

