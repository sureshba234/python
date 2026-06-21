<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Python Import Internals – Part 2

## Table of Contents

1. [Introduction](#introduction)
2. [Beginner: Import Hooks Fundamentals](#beginner-import-hooks-fundamentals)
    - [What are Import Hooks?](#what-are-import-hooks)
    - [Custom Importers Overview](#custom-importers-overview)
    - [First Custom Finder Example](#first-custom-finder-example)
3. [Intermediate: Finder \& Loader Protocols](#intermediate-finder-loader-protocols)
    - [Finder Protocol Deep Dive](#finder-protocol-deep-dive)
    - [Loader Protocol Deep Dive](#loader-protocol-deep-dive)
    - [Custom Finder Implementation](#custom-finder-implementation)
    - [Custom Loader Implementation](#custom-loader-implementation)
    - [Execution Trace: How Finders \& Loaders Work](#execution-trace-how-finders-loaders-work)
4. [Intermediate: importlib Hooks](#intermediate-importlib-hooks)
    - [importlib.meta_path](#importlib-meta_path)
    - [importlib.abc Classes](#importlib-abc-classes)
    - [Hook Injection Patterns](#hook-injection-patterns)
5. [Intermediate: Plugin Systems with Import Hooks](#intermediate-plugin-systems-with-import-hooks)
    - [Plugin Architecture](#plugin-architecture)
    - [Real-world Plugin System](#real-world-plugin-system)
6. [Beginner: sys.modules Basics](#beginner-sys-modules-basics)
    - [What is sys.modules?](#what-is-sys-modules)
    - [sys.modules Lifecycle](#sys-modules-lifecycle)
    - [Internal Diagram: sys.modules Structure](#internal-diagram-sys-modules-structure)
7. [Intermediate: sys.modules Advanced Usage](#intermediate-sys-modules-advanced-usage)
    - [How Python Uses sys.modules](#how-python-uses-sys-modules)
    - [Import Cache Mechanism](#import-cache-mechanism)
    - [Circular Imports Analysis](#circular-imports-analysis)
    - [Circular Import Execution Trace](#circular-import-execution-trace)
8. [Intermediate: Reloading Modules](#intermediate-reloading-modules)
    - [Why Reload?](#why-reload)
    - [importlib.reload()](#importlib-reload)
    - [Reloading Demonstration](#reloading-demonstration)
    - [Memory Explanations: What Changes on Reload](#memory-explanations-what-changes-on-reload)
9. [Advanced: Module Caching](#advanced-module-caching)
    - [Caching Mechanism Deep Dive](#caching-mechanism-deep-dive)
    - [Import Cache Architecture](#import-cache-architecture)
    - [Import Optimization Techniques](#import-optimization-techniques)
    - [Reload Behavior \& Cache](#reload-behavior-cache)
    - [Performance Implications](#performance-implications)
    - [Performance Benchmarks](#performance-benchmarks)
10. [Advanced: Debugging Techniques](#advanced-debugging-techniques)
    - [Debugging Import Issues](#debugging-import-issues)
    - [Import Tracing Tools](#import-tracing-tools)
    - [Common Mistakes](#common-mistakes)
11. [Framework Examples](#framework-examples)
    - [Django's Import System](#django-s-import-system)
    - [pytest Plugin System](#pytest-plugin-system)
12. [Best Practices](#best-practices)
13. [Interview Questions](#interview-questions)
14. [Exercises](#exercises)
15. [Summary](#summary)
16. [Cheat Sheets](#cheat-sheets)
    - [Import Hooks Cheat Sheet](#import-hooks-cheat-sheet)
    - [sys.modules Cheat Sheet](#sys-modules-cheat-sheet)
    - [Module Caching Cheat Sheet](#module-caching-cheat-sheet)
17. [Advanced Practice Problems](#advanced-practice-problems)

***

## Introduction

Python's import system is one of its most powerful and flexible features. This guide dives deep into **Import Hooks**, **sys.modules**, and **Module Caching** — the internal mechanisms that make Python's import system work. You'll progress from beginner concepts to expert-level implementations including custom finders, loaders, plugin systems, and performance optimization.

***

## Beginner: Import Hooks Fundamentals

### What are Import Hooks?

Import hooks are Python's mechanism for intercepting and customizing the import process. They allow you to:

- Load modules from non-standard locations (ZIP files, databases, networks)
- Transform code before execution
- Implement plugin systems
- Mock modules for testing
- Create virtual modules

```python
# Standard import flow
import module  # Python searches, finds, loads, and executes

# With import hooks
sys.meta_path.append(custom_finder)  # Intercept the import
import module  # custom_finder handles it
```


### Custom Importers Overview

A custom importer consists of two parts:

1. **Finder**: Locates the module (returns a module spec or loader)
2. **Loader**: Loads and executes the module code
```
Import Request → Finders (meta_path) → Loader → Module Object → sys.modules
```


### First Custom Finder Example

Let's create a finder that loads modules from a dictionary:

```python
import sys
import importlib.abc
import importlib.util
from types import ModuleType

class DictFinder(importlib.abc.MetaPathFinder):
    """Finder that loads modules from a dictionary."""
    
    def __init__(self, module_dict):
        self.module_dict = module_dict
    
    def find_spec(self, fullname, path, target=None):
        if fullname in self.module_dict:
            code = self.module_dict[fullname]
            loader = DictLoader(fullname, code)
            spec = importlib.util.spec_from_loader(fullname, loader)
            return spec
        return None

class DictLoader(importlib.abc.Loader):
    """Loader that executes code from a string."""
    
    def __init__(self, fullname, code):
        self.fullname = fullname
        self.code = code
    
    def create_module(self, spec):
        return ModuleType(self.fullname)
    
    def exec_module(self, module):
        exec(self.code, module.__dict__)

# Usage
module_dict = {
    'magic_module': '''
def hello():
    return "Hello from magic_module!"
value = 42
'''
}

sys.meta_path.append(DictFinder(module_dict))

# Now import works!
import magic_module
print(magic_module.hello())  # "Hello from magic_module!"
print(magic_module.value)    # 42
```

**Key Points:**

- `find_spec()` is the modern finder protocol (Python 3.4+)
- `create_module()` creates the module object
- `exec_module()` executes the code in the module's namespace

***

## Intermediate: Finder \& Loader Protocols

### Finder Protocol Deep Dive

The finder protocol uses `find_spec()` with these parameters:


| Parameter | Description |
| :-- | :-- |
| `fullname` | Fully qualified module name (e.g., `package.submodule`) |
| `path` | The search path if importing from a package (None for top-level) |
| `target` | Optional target module for optimization |

**Return values:**

- `importlib.ModuleSpec` object if found
- `None` if not found (let next finder try)

```python
import importlib.abc
import importlib.util

class MyFinder(importlib.abc.MetaPathFinder):
    def find_spec(self, fullname, path, target=None):
        # Your logic to find the module
        if self._module_exists(fullname):
            loader = MyLoader(fullname)
            spec = importlib.util.spec_from_loader(fullname, loader)
            spec.origin = '/path/to/module.py'
            spec.loader_state = {'version': '1.0'}
            return spec
        return None
    
    def _module_exists(self, fullname):
        # Check if module exists
        return hasattr(self, fullname)
```


### Loader Protocol Deep Dive

The loader protocol has these methods:


| Method | Purpose | When to Override |
| :-- | :-- | :-- |
| `create_module(spec)` | Create module object | Usually return `None` (use default) |
| `exec_module(module)` | Execute module code | **Always override** |
| `get_code(fullname)` | Get code object | For custom code loading |
| `get_source(fullname)` | Get source string | For source-based loaders |
| `is_package(fullname)` | Check if package | For package detection |

```python
import importlib.abc

class MyLoader(importlib.abc.Loader):
    def create_module(self, spec):
        # Return None to use default module creation
        return None
    
    def exec_module(self, module):
        # Execute code in module's namespace
        code = self.get_code(module.__name__)
        exec(code, module.__dict__)
    
    def get_code(self, fullname):
        # Return compiled code object
        source = self.get_source(fullname)
        return compile(source, fullname, 'exec')
    
    def get_source(self, fullname):
        # Return source code string
        return "# Module source for " + fullname
```


### Custom Finder Implementation

Here's a finder that loads modules from ZIP files:

```python
import sys
import zipimport
import importlib.abc
import importlib.util
import zipfile
from pathlib import Path

class ZipFinder(importlib.abc.MetaPathFinder):
    """Finder for modules in ZIP files."""
    
    def __init__(self, zip_path):
        self.zip_path = Path(zip_path)
        if not self.zip_path.exists():
            raise FileNotFoundError(f"ZIP file not found: {zip_path}")
    
    def find_spec(self, fullname, path, target=None):
        # Convert module name to path (e.g., 'package.module' → 'package/module.py')
        module_path = fullname.replace('.', '/') + '.py'
        
        with zipfile.ZipFile(self.zip_path) as zf:
            if module_path in zf.namelist():
                # Read source code
                source = zf.read(module_path).decode('utf-8')
                
                # Create loader and spec
                loader = ZipLoader(fullname, source, str(self.zip_path))
                spec = importlib.util.spec_from_loader(fullname, loader)
                spec.origin = str(self.zip_path / module_path)
                spec.loader_state = {'zip_path': str(self.zip_path)}
                return spec
        
        return None

class ZipLoader(importlib.abc.Loader):
    """Loader for ZIP module sources."""
    
    def __init__(self, fullname, source, zip_path):
        self.fullname = fullname
        self.source = source
        self.zip_path = zip_path
    
    def create_module(self, spec):
        return None  # Use default
    
    def exec_module(self, module):
        code = compile(self.source, self.fullname, 'exec')
        exec(code, module.__dict__)
    
    def get_source(self, fullname):
        return self.source

# Usage
# Create a ZIP file with modules first
# zip_path = 'my_modules.zip'

# sys.meta_path.append(ZipFinder(zip_path))
# import module_from_zip
```


### Custom Loader Implementation

Here's a loader that decrypts encrypted Python modules:

```python
import importlib.abc
import importlib.util
import base64
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.backends import default_backend

class EncryptedLoader(importlib.abc.Loader):
    """Loader for encrypted Python modules."""
    
    def __init__(self, fullname, encrypted_source, key):
        self.fullname = fullname
        self.encrypted_source = encrypted_source
        self.key = key  # 32-byte key for AES-256
    
    def create_module(self, spec):
        return None
    
    def exec_module(self, module):
        # Decrypt source
        decrypted = self._decrypt(self.encrypted_source)
        source = base64.b64decode(decrypted).decode('utf-8')
        
        # Compile and execute
        code = compile(source, self.fullname, 'exec')
        exec(code, module.__dict__)
    
    def _decrypt(self, encrypted_data):
        # Simple AES decryption (simplified for example)
        from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
        backend = default_backend()
        cipher = Cipher(algorithms.AES(self.key), modes.ECB(), backend)
        decryptor = cipher.decryptor()
        return decryptor.update(encrypted_data)
    
    def get_source(self, fullname):
        decrypted = self._decrypt(self.encrypted_source)
        return base64.b64decode(decrypted).decode('utf-8')

# Usage pattern
# encrypted_data = load_encrypted_file('module.enc')
# key = b'your-32-byte-key-here!!!!'
# loader = EncryptedLoader('encrypted_module', encrypted_data, key)
# spec = importlib.util.spec_from_loader('encrypted_module', loader)
# module = importlib.util.module_from_spec(spec)
# loader.exec_module(module)
```


### Execution Trace: How Finders \& Loaders Work

```
import my_module

┌─────────────────────────────────────────────────────────────┐
│ 1. Python checks sys.modules for 'my_module'                │
│    └─ Found? → Return cached module (skip rest)              │
│    └─ Not found? → Continue                                  │
└─────────────────────────────────────────────────────────────┘
         ↓
┌─────────────────────────────────────────────────────────────┐
│ 2. Iterate sys.meta_path (finder list)                       │
│    for finder in sys.meta_path:                              │
│        spec = finder.find_spec('my_module', None, None)      │
│        if spec is not None: break                            │
└─────────────────────────────────────────────────────────────┘
         ↓ (first finder that returns spec)
┌─────────────────────────────────────────────────────────────┐
│ 3. Create module from spec                                   │
│    module = spec.loader.create_module(spec)                  │
│    └─ Usually returns None → use default ModuleType          │
└─────────────────────────────────────────────────────────────┘
         ↓
┌─────────────────────────────────────────────────────────────┐
│ 4. Execute module code                                       │
│    spec.loader.exec_module(module)                           │
│    └─ Your custom code runs here!                            │
└─────────────────────────────────────────────────────────────┘
         ↓
┌─────────────────────────────────────────────────────────────┐
│ 5. Cache in sys.modules                                      │
│    sys.modules['my_module'] = module                         │
└─────────────────────────────────────────────────────────────┘
         ↓
┌─────────────────────────────────────────────────────────────┐
│ 6. Return module to caller                                   │
└─────────────────────────────────────────────────────────────┘
```

**Trace with debugging:**

```python
import sys

class TracingFinder(importlib.abc.MetaPathFinder):
    def find_spec(self, fullname, path, target=None):
        print(f"[FINDER] Looking for: {fullname}")
        print(f"[FINDER] Path: {path}")
        print(f"[FINDER] Target: {target}")
        
        # Your logic
        if fullname == 'traced_module':
            print(f"[FINDER] Found traced_module!")
            loader = TracingLoader(fullname)
            return importlib.util.spec_from_loader(fullname, loader)
        
        print(f"[FINDER] Not found, returning None")
        return None

class TracingLoader(importlib.abc.Loader):
    def create_module(self, spec):
        print(f"[LOADER] Creating module: {spec.name}")
        return None
    
    def exec_module(self, module):
        print(f"[LOADER] Executing module: {module.__name__}")
        print(f"[LOADER] Module dict before: {module.__dict__.keys()}")
        
        # Execute actual code
        exec('value = 42\ndef foo(): return "bar"', module.__dict__)
        
        print(f"[LOADER] Module dict after: {module.__dict__.keys()}")

# Add tracer
sys.meta_path.insert(0, TracingFinder())

# Import and see the trace
import traced_module
print(f"Result: {traced_module.value}, {traced_module.foo()}")
```

**Output:**

```
[FINDER] Looking for: traced_module
[FINDER] Path: None
[FINDER] Target: None
[FINDER] Found traced_module!
[LOADER] Creating module: traced_module
[LOADER] Executing module: traced_module
[LOADER] Module dict before: {'__name__': 'traced_module', '__doc__': None, '__package__': '', '__loader__': ..., '__spec__': ...}
[LOADER] Module dict after: {'__name__': 'traced_module', '__doc__': None, '__package__': '', '__loader__': ..., '__spec__': ..., 'value': 42, 'foo': <function>}
Result: 42, bar
```


***

## Intermediate: importlib Hooks

### importlib.meta_path

`sys.meta_path` is the list of finder objects:

```python
import sys

print("Current meta_path:")
for i, finder in enumerate(sys.meta_path):
    print(f"  {i}: {finder.__class__.__name__} - {finder}")

# Add your finder
sys.meta_path.insert(0, MyFinder())  # Higher priority

# Remove a finder
sys.meta_path = [f for f in sys.meta_path if not isinstance(f, OldFinder)]
```

**Priority order:** First finder in the list gets priority.

### importlib.abc Classes

Base classes for creating custom finders/loaders:

```python
import importlib.abc

# Finder bases
importlib.abc.MetaPathFinder    # For sys.meta_path (Python 3.4+)
importlib.abc.FindLoader        # Legacy (Python 2.x)

# Loader bases
importlib.abc.Loader            # Modern loader protocol
importlib.abc.ExecutionLoader   # Adds get_code()
importlib.abc.SourceLoader      # Adds get_source()
importlib.abc.InputLoader       # For file-based loading

# Mixed patterns
importlib.abc.FileLoader        # File-based loader + source
importlib.abc.SourcelessLoader  # Loader without source
```

**Example with SourceLoader:**

```python
import importlib.abc
import importlib.util

class FileSourceLoader(importlib.abc.SourceLoader):
    """Loader that reads source from a file."""
    
    def __init__(self, filepath):
        self.filepath = filepath
    
    def create_module(self, spec):
        return None
    
    def exec_module(self, module):
        code = self.get_code(module.__name__)
        exec(code, module.__dict__)
    
    def get_code(self, fullname):
        source = self.get_source(fullname)
        return self.compile_source(source, fullname)
    
    def get_source(self, fullname):
        with open(self.filepath, 'r') as f:
            return f.read()
    
    def compile_source(self, source, fullname):
        return compile(source, self.filepath, 'exec')
    
    def get_filename(self, fullname):
        return self.filepath

# Usage
loader = FileSourceLoader('/path/to/module.py')
spec = importlib.util.spec_from_loader('file_module', loader)
module = importlib.util.module_from_spec(spec)
loader.exec_module(module)
```


### Hook Injection Patterns

**Pattern 1: Add finder to meta_path**

```python
sys.meta_path.append(MyFinder())  # Low priority
sys.meta_path.insert(0, MyFinder())  # High priority
```

**Pattern 2: Replace sys.importer**

```python
# Legacy (Python 2.x, not recommended)
import __builtin__
__builtin__.__import__ = my_custom_import
```

**Pattern 3: Use importlib.import_module with custom loader**

```python
import importlib.util
import importlib

loader = MyLoader('custom_module')
spec = importlib.util.spec_from_loader('custom_module', loader)
module = importlib.util.module_from_spec(spec)
loader.exec_module(module)

# Now it's in sys.modules
import custom_module  # Will use cached version
```

**Pattern 4: Namespace-based injection**

```python
class NamespaceFinder(importlib.abc.MetaPathFinder):
    def __init__(self, namespace, finder_map):
        self.namespace = namespace
        self.finder_map = finder_map  # {submodule: finder}
    
    def find_spec(self, fullname, path, target=None):
        if fullname.startswith(self.namespace):
            subname = fullname[len(self.namespace) + 1:]  # Remove 'namespace.'
            if subname in self.finder_map:
                return self.finder_map[subname].find_spec(fullname, path, target)
        return None

# Usage
finders = {
    'plugin1': Plugin1Finder(),
    'plugin2': Plugin2Finder(),
}

sys.meta_path.append(NamespaceFinder('myapp.plugins', finders))
import myapp.plugins.plugin1
import myapp.plugins.plugin2
```


***

## Intermediate: Plugin Systems with Import Hooks

### Plugin Architecture

```
┌─────────────────────────────────────────────────────────────┐
│ Plugin System Architecture                                   │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Plugin Directory                                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ plugin_a.py  │  │ plugin_b.py  │  │ plugin_c.py  │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│         ↓                  ↓                  ↓              │
│  ┌──────────────────────────────────────────────────┐       │
│  │  PluginFinder (scans directory)                  │       │
│  │  - Lists .py files                               │       │
│  │  - Creates specs for each                        │       │
│  └──────────────────────────────────────────────────┘       │
│         ↓                                                    │
│  ┌──────────────────────────────────────────────────┐       │
│  │  PluginLoader (loads each plugin)                │       │
│  │  - Reads source                                  │       │
│  │  - Executes in module namespace                  │       │
│  │  - Validates plugin interface                    │       │
│  └──────────────────────────────────────────────────┘       │
│         ↓                                                    │
│  ┌──────────────────────────────────────────────────┐       │
│  │  sys.modules['plugins.plugin_a']                 │       │
│  │  sys.modules['plugins.plugin_b']                 │       │
│  └──────────────────────────────────────────────────┘       │
│         ↓                                                    │
│  ┌──────────────────────────────────────────────────┐       │
│  │  Application discovers plugins                   │       │
│  │  importlib.import_module('plugins.plugin_a')     │       │
│  └──────────────────────────────────────────────────┘       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```


### Real-world Plugin System

```python
import sys
import importlib.abc
import importlib.util
from pathlib import Path
import inspect

class PluginFinder(importlib.abc.MetaPathFinder):
    """Finder for plugins in a directory."""
    
    def __init__(self, plugin_dir, plugin_namespace='plugins'):
        self.plugin_dir = Path(plugin_dir)
        self.plugin_namespace = plugin_namespace
        if not self.plugin_dir.exists():
            self.plugin_dir.mkdir(parents=True)
    
    def find_spec(self, fullname, path, target=None):
        # Check if this is a plugin module
        if fullname.startswith(self.plugin_namespace):
            # Extract plugin name
            plugin_name = fullname.split('.')[-1]
            plugin_file = self.plugin_dir / f"{plugin_name}.py"
            
            if plugin_file.exists():
                loader = PluginLoader(fullname, plugin_file)
                spec = importlib.util.spec_from_loader(fullname, loader)
                spec.origin = str(plugin_file)
                spec.submodule_search_locations = []
                return spec
        
        return None
    
    def discover_plugins(self):
        """Return list of available plugin names."""
        plugins = []
        for py_file in self.plugin_dir.glob('*.py'):
            if py_file.name != '__init__.py':
                plugins.append(py_file.stem)
        return plugins

class PluginLoader(importlib.abc.SourceLoader):
    """Loader for plugin modules with validation."""
    
    PLUGIN_INTERFACE = {
        'name': str,
        'execute': callable,
        'version': str,
    }
    
    def __init__(self, fullname, filepath):
        self.fullname = fullname
        self.filepath = filepath
    
    def create_module(self, spec):
        return None
    
    def exec_module(self, module):
        # Load and execute
        source = self.get_source(module.__name__)
        code = compile(source, self.filepath, 'exec')
        exec(code, module.__dict__)
        
        # Validate plugin interface
        self._validate_plugin(module)
    
    def get_source(self, fullname):
        with open(self.filepath, 'r') as f:
            return f.read()
    
    def get_filename(self, fullname):
        return str(self.filepath)
    
    def _validate_plugin(self, module):
        """Check that plugin has required interface."""
        errors = []
        
        for attr, attr_type in self.PLUGIN_INTERFACE.items():
            if not hasattr(module, attr):
                errors.append(f"Missing attribute: {attr}")
            elif not isinstance(getattr(module, attr), attr_type):
                errors.append(f"Wrong type for {attr}: expected {attr_type}")
        
        if errors:
            raise ImportError(f"Plugin validation failed for {module.__name__}: {errors}")

# Plugin registration
class PluginRegistry:
    def __init__(self, plugin_dir='plugins'):
        self.plugin_dir = plugin_dir
        self.plugins = {}
        
        # Register finder
        sys.meta_path.append(
            PluginFinder(plugin_dir, 'plugins')
        )
    
    def load_all(self):
        """Load all discovered plugins."""
        finder = sys.meta_path[-1]  # Our PluginFinder
        plugin_names = finder.discover_plugins()
        
        for name in plugin_names:
            fullname = f'plugins.{name}'
            try:
                module = importlib.import_module(fullname)
                self.plugins[name] = module
                
                print(f"Loaded plugin: {module.name} v{module.version}")
            except Exception as e:
                print(f"Failed to load {name}: {e}")
    
    def get_plugin(self, name):
        """Get a specific plugin."""
        return self.plugins.get(name)
    
    def execute_all(self):
        """Execute all plugins."""
        for name, plugin in self.plugins.items():
            print(f"Executing {name}:")
            plugin.execute()

# Example plugin (plugins/my_plugin.py)
"""
name = "My Plugin"
version = "1.0.0"

def execute():
    print("My Plugin is running!")

def helper():
    return "Helper function"
"""

# Usage
if __name__ == '__main__':
    registry = PluginRegistry('plugins')
    registry.load_all()
    registry.execute_all()
    
    # Use specific plugin
    plugin = registry.get_plugin('my_plugin')
    if plugin:
        result = plugin.helper()
        print(f"Result: {result}")
```


***

## Beginner: sys.modules Basics

### What is sys.modules?

`sys.modules` is a dictionary that maps module names to their module objects:

```python
import sys

print(type(sys.modules))  # dict
print(len(sys.modules))   # Number of loaded modules

# Access a module
import os
print(sys.modules['os'])  # <module 'os' from '/path/to/os.py'>

# Check if module is loaded
print('json' in sys.modules)  # True after import json
```

**Key properties:**

- **Type**: `dict` (actually `dict` subclass)
- **Keys**: Module names (strings)
- **Values**: Module objects (`types.ModuleType`)
- **Purpose**: Import cache and module registry


### sys.modules Lifecycle

```
┌─────────────────────────────────────────────────────────────┐
│ sys.modules Lifecycle                                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Import Request                                          │
│     import my_module                                        │
│                                                              │
│  2. Check sys.modules                                       │
│     if 'my_module' in sys.modules:                          │
│         return sys.modules['my_module']  # Cached!          │
│                                                              │
│  3. Module Not Found → Load                                 │
│     - Finder finds module                                   │
│     - Loader creates module object                          │
│     - Loader executes code                                  │
│                                                              │
│  4. Cache in sys.modules                                    │
│     sys.modules['my_module'] = module_object                │
│                                                              │
│  5. Return to Caller                                        │
│     return module_object                                    │
│                                                              │
│  6. Subsequent Imports (Cached)                             │
│     import my_module  # Returns from sys.modules            │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```


### Internal Diagram: sys.modules Structure

```
sys.modules
├── '__main__' → Module('__main__')
├── 'sys' → Module('sys')  # Built-in
├── 'builtins' → Module('builtins')  # Built-in
├── 'os' → Module('os')
│   ├── __name__: 'os'
│   ├── __doc__: 'Standard os module'
│   ├── __file__: '/path/to/os.py'
│   ├── __package__: ''
│   ├── __loader__: FileLoader(...)
│   ├── __spec__: ModuleSpec(...)
│   ├── path: ['/path/to/os']
│   └── functions: open(), read(), write()...
├── 'json' → Module('json')
├── 'package' → Module('package')
│   ├── __name__: 'package'
│   ├── __path__: ['/path/to/package']
│   ├── __submodules__: {'submodule1', 'submodule2'}
│   └── submodule1: Module('package.submodule1')
├── 'package.submodule1' → Module('package.submodule1')
├── 'package.submodule2' → Module('package.submodule2')
└── ... (all imported modules)
```

**Module object structure:**

```python
import types
import os

module = sys.modules['os']

print(type(module))  # <class 'module'>
print(module.__name__)      # 'os'
print(module.__doc__)       # Module docstring
print(module.__file__)      # Path to .py file
print(module.__package__)   # Package name (empty for top-level)
print(module.__loader__)    # Loader object
print(module.__spec__)      # ModuleSpec object
print(module.__path__)      # Package path (None for non-packages)
```


***

## Intermediate: sys.modules Advanced Usage

### How Python Uses sys.modules

Python checks `sys.modules` **before** any finder:

```python
# Simplified import logic (from CPython)
def _import(name):
    # Step 1: Check cache
    if name in sys.modules:
        return sys.modules[name]
    
    # Step 2: Create placeholder (prevents infinite recursion)
    sys.modules[name] = None
    
    # Step 3: Find and load
    spec = _find_spec(name)
    module = spec.loader.create_module(spec)
    sys.modules[name] = module  # Update placeholder
    
    # Step 4: Execute
    spec.loader.exec_module(module)
    
    # Step 5: Return
    return module
```

**Why the placeholder?** Prevents circular imports from causing infinite recursion.

### Import Cache Mechanism

```python
import sys

# First import - loads from disk
import time
print(f"First import: {time.__file__}")
print(f"In sys.modules: {'time' in sys.modules}")

# Second import - returns from cache
import time
print(f"Second import: Same object? {time is sys.modules['time']}")

# Verify it's the same object
import time as t2
print(f"time is t2: {time is t2}")  # True
```

**Cache behavior:**

- **Single object per module name**: Only one module object exists per name
- **Fast lookup**: Dictionary lookup is O(1)
- **Global cache**: All imports share the same cache


### Circular Imports Analysis

Circular imports occur when modules import each other:

```python
# a.py
import b  # Circular!

def func_a():
    return "a"

# b.py
import a  # Circular!

def func_b():
    return "b"
```

**What happens:**

```
import a
  ↓
Check sys.modules['a'] → None
  ↓
Create placeholder: sys.modules['a'] = None
  ↓
Execute a.py
  ↓
import b
  ↓
Check sys.modules['b'] → None
  ↓
Create placeholder: sys.modules['b'] = None
  ↓
Execute b.py
  ↓
import a
  ↓
Check sys.modules['a'] → None (placeholder, not loaded!)
  ↓
Try to access a.func_a() → AttributeError!
```


### Circular Import Execution Trace

```python
import sys

# Enable import tracing
import traceback

class TracingModule(dict):
    """Trace sys.modules access."""
    
    def __getitem__(self, key):
        print(f"[sys.modules] Getting: {key}")
        traceback.print_stack(limit=5)
        return super().__getitem__(key)
    
    def __setitem__(self, key, value):
        print(f"[sys.modules] Setting: {key} = {value}")
        if value is None:
            print("  → Creating placeholder!")
        else:
            print("  → Loading module!")
        super().__setitem__(key, value)

# Replace sys.modules temporarily
original_modules = sys.modules
sys.modules = TracingModule(original_modules)

# Create circular import files
import tempfile
import os

with tempfile.TemporaryDirectory() as tmpdir:
    # a.py
    a_path = os.path.join(tmpdir, 'circular_a.py')
    with open(a_path, 'w') as f:
        f.write('''
import circular_b

def func_a():
    return "a from func_a"

print("a.py: Loaded")
''')
    
    # b.py
    b_path = os.path.join(tmpdir, 'circular_b.py')
    with open(b_path, 'w') as f:
        f.write('''
import circular_a

def func_b():
    return "b from func_b"

print("b.py: Loaded")
''')
    
    # Add to path
    sys.path.insert(0, tmpdir)
    
    # Try circular import
    try:
        import circular_a
    except Exception as e:
        print(f"Error: {e}")
    
    # Restore
    sys.modules = original_modules
    sys.path.remove(tmpdir)
```

**Solutions for circular imports:**

```python
# Solution 1: Lazy import (import inside function)
# a.py
def func_a():
    import circular_b  # Lazy!
    return circular_b.func_b()

# Solution 2: Import at end of module
# a.py
def func_a():
    return "a"

# b.py (imports a at end)
import circular_a  # After functions defined

def func_b():
    return circular_a.func_a()

# Solution 3: Use __init__.py to centralize
# package/__init__.py
from . import a
from . import b

# Solution 4: Rearrange dependencies
# a.py → no import of b
# b.py → imports a (one-way)
```


***

## Intermediate: Reloading Modules

### Why Reload?

Module reloading is useful for:

- **Hot-reloading** in development servers
- **Plugin updates** without restart
- **Testing** with modified code
- **Configuration changes** at runtime

**Warning:** Reloading is fragile and can break references to old objects.

### importlib.reload()

```python
import importlib
import sys

# First import
import my_module
print(f"Original: {my_module.value}")  # 42

# Modify my_module.py on disk
# Change: value = 100

# Reload
importlib.reload(my_module)
print(f"Reloaded: {my_module.value}")  # 100

# Check sys.modules
print(sys.modules['my_module'])  # Same object (updated)
```


### Reloading Demonstration

```python
import importlib
import tempfile
import os
import sys

# Create temporary module
with tempfile.TemporaryDirectory() as tmpdir:
    module_path = os.path.join(tmpdir, 'reload_demo.py')
    
    # Initial version
    with open(module_path, 'w') as f:
        f.write('''
value = 42

def hello():
    return "Hello v1"

class MyClass:
    def __init__(self):
        self.version = 1
''')
    
    sys.path.insert(0, tmpdir)
    
    # Import
    import reload_demo
    print(f"v1 value: {reload_demo.value}")          # 42
    print(f"v1 hello: {reload_demo.hello()}")        # "Hello v1"
    print(f"v1 MyClass.version: {reload_demo.MyClass().version}")  # 1
    
    # Store original references
    original_myclass = reload_demo.MyClass
    original_hello = reload_demo.hello
    
    # Modify file
    with open(module_path, 'w') as f:
        f.write('''
value = 100

def hello():
    return "Hello v2"

class MyClass:
    def __init__(self):
        self.version = 2
''')
    
    # Reload
    print("\n--- Reloading ---")
    importlib.reload(reload_demo)
    
    print(f"v2 value: {reload_demo.value}")          # 100
    print(f"v2 hello: {reload_demo.hello()}")        # "Hello v2"
    print(f"v2 MyClass.version: {reload_demo.MyClass().version}")  # 2
    
    # Check what changed
    print("\n--- Reference Analysis ---")
    print(f"Same module object? {reload_demo is original_myclass.__module__}")
    print(f"Same hello function? {reload_demo.hello is original_hello}")  # False!
    print(f"Same MyClass? {reload_demo.MyClass is original_myclass}")     # False!
    
    # Old references still point to old classes
    old_instance = original_myclass()
    print(f"Old instance version: {old_instance.version}")  # 1 (old!)
    new_instance = reload_demo.MyClass()
    print(f"New instance version: {new_instance.version}")  # 2 (new!)
    
    sys.path.remove(tmpdir)
```

**Output:**

```
v1 value: 42
v1 hello: Hello v1
v1 MyClass.version: 1

--- Reloading ---
v2 value: 100
v2 hello: Hello v2
v2 MyClass.version: 2

--- Reference Analysis ---
Same module object? True
Same hello function? False
Same MyClass? False
Old instance version: 1
New instance version: 2
```


### Memory Explanations: What Changes on Reload

```
┌─────────────────────────────────────────────────────────────┐
│ Memory Changes During reload()                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Before reload():                                            │
│  ┌────────────────────────────────────────────────────┐    │
│  │ sys.modules['my_module']                           │    │
│  │ ─────────────────────────────────────────────────  │    │
│  │ __name__: 'my_module'                              │    │
│  │ value: 42                                          │    │
│  │ hello: <function hello_v1 at 0x7f1>                │    │
│  │ MyClass: <class MyClass_v1 at 0x7f2>               │    │
│  └────────────────────────────────────────────────────┘    │
│         ↑                                                    │
│         │ (references held by user code)                     │
│  ┌──────┴──────┐  ┌──────┴──────┐                           │
│  │ hello_v1    │  │ MyClass_v1  │                           │
│  │ 0x7f1       │  │ 0x7f2       │                           │
│  └─────────────┘  └─────────────┘                           │
│                                                              │
│  After reload():                                             │
│  ┌────────────────────────────────────────────────────┐    │
│  │ sys.modules['my_module'] ← SAME OBJECT (updated)   │    │
│  │ ─────────────────────────────────────────────────  │    │
│  │ __name__: 'my_module'                              │    │
│  │ value: 100         ← CHANGED                       │    │
│  │ hello: <function hello_v2 at 0x7f3>  ← NEW!        │    │
│  │ MyClass: <class MyClass_v2 at 0x7f4> ← NEW!        │    │
│  └────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌──────┬──────┐  ┌──────┬──────┐                           │
│  │hello_v1│    │  │MyClass_v1│  │ ← OLD objects still exist│
│  │ 0x7f1│    │  │ 0x7f2│  │  │   (if referenced)          │
│  └──────┘    │  └──────┘    │                           │
│                                                              │
│  ⚠️ User code with old references continues using old!     │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Key behaviors:**

1. **Module object is reused**: Same object in `sys.modules`
2. **Names are updated**: All names in `module.__dict__` are replaced
3. **Old objects persist**: If you have references to old functions/classes, they still work
4. **No cleanup**: Old objects aren't automatically garbage collected

***

## Advanced: Module Caching

### Caching Mechanism Deep Dive

Python's import caching has multiple layers:

```
┌─────────────────────────────────────────────────────────────┐
│ Import Cache Layers                                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Layer 1: sys.modules (Primary Cache)                      │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ 'module_name' → ModuleObject                         │  │
│  │ 'package.sub' → ModuleObject                         │  │
│  └──────────────────────────────────────────────────────┘  │
│         ↓ O(1) lookup                                       │
│                                                              │
│  Layer 2: Module Spec Cache (importlib)                    │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Cached ModuleSpec objects from finders               │  │
│  │ Reused for subsequent imports                        │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  Layer 3: File System Cache (__.pycache__)                 │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Compiled .pyc files in __pycache__/                  │  │
│  │ Avoids re-compiling source                           │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  Layer 4: Finder Cache (custom)                            │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Custom finders may cache file listings, etc.         │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```


### Import Cache Architecture

```python
import sys
import types

# Examine sys.modules internals
print(f"Type: {type(sys.modules)}")
print(f"MRO: {type(sys.modules).__mro__}")

# Check for special methods
print(f"Has __dict__: {hasattr(sys.modules, '__dict__')}")
print(f"Has __weakref__: {hasattr(sys.modules, '__weakref__')}")

# Module object cache behavior
import os
module = sys.modules['os']

# Module's internal cache
print(f"Module __dict__ size: {len(module.__dict__)}")
print(f"Module __dict__ type: {type(module.__dict__)}")

# Check if module is in dict
print(f"'os' in sys.modules: {'os' in sys.modules}")
print(f"sys.modules.get('os'): {sys.modules.get('os')}")
print(f"sys.modules['os']: {sys.modules['os']}")
```


### Import Optimization Techniques

**1. Lazy imports**

```python
# Don't import at module level
# import heavy_module

# Import when needed
def use_heavy():
    import heavy_module  # Only imported once
    return heavy_module.process()
```

**2. Import caching in functions**

```python
# Bad: Re-imports every call (actually uses cache, but still checks)
def bad():
    import os
    return os.path

# Good: Cache at module level
import os

def good():
    return os.path

# Better: Cache in function (for very expensive imports)
def cached():
    if not hasattr(cached, '_os'):
        cached._os = importlib.import_module('os')
    return cached._os.path
```

**3. Pre-import common modules**

```python
# In __init__.py or startup script
import sys
_common = ['os', 'sys', 'json', 'pathlib', 'typing']
for name in _common:
    if name not in sys.modules:
        importlib.import_module(name)
```


### Reload Behavior \& Cache

```python
import importlib
import sys

# Import
import my_module
print(f"Before: {my_module.value}")  # 42

# Check cache state
print(f"In cache: {'my_module' in sys.modules}")
print(f"Cache key: {sys.modules['my_module']}")

# Modify file
# my_module.value = 100

# Reload
importlib.reload(my_module)
print(f"After: {my_module.value}")  # 100

# Cache state after reload
print(f"Still in cache: {'my_module' in sys.modules}")
print(f"Same object: {my_module is sys.modules['my_module']}")

# What gets cleared?
print(f"Module dict keys before reload: {...}")  # Previous keys
print(f"Module dict keys after reload: {...}")   # New keys
```

**Reload caveats:**

- `sys.modules` entry is **not** cleared
- Module's `__dict__` is **updated** (old keys may persist)
- Submodule references may become stale
- Import hooks are **not** re-invoked


### Performance Implications

```python
import time
import sys
import importlib

# Benchmark 1: Import cache benefit
def benchmark_first_import():
    # Remove from cache
    if 'benchmark_mod' in sys.modules:
        del sys.modules['benchmark_mod']
    
    # Create module
    import tempfile
    import os
    tmpdir = tempfile.mkdtemp()
    path = os.path.join(tmpdir, 'benchmark_mod.py')
    with open(path, 'w') as f:
        f.write('value = 42\n')
    
    sys.path.insert(0, tmpdir)
    
    # Time import
    start = time.perf_counter()
    import benchmark_mod
    end = time.perf_counter()
    
    sys.path.remove(tmpdir)
    return end - start

def benchmark_cached_import():
    import benchmark_mod  # Already cached
    return time.perf_counter() - time.perf_counter()  # ~0

# Run benchmarks
first = benchmark_first_import()
cached = benchmark_cached_import()

print(f"First import: {first:.6f}s")
print(f"Cached import: {cached:.6f}s")
print(f"Speedup: {first/cached if cached > 0 else 'inf'}x")
```


### Performance Benchmarks

Let me create a proper benchmark:

