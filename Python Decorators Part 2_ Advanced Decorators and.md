<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Python Decorators Part 2: Advanced Decorators and Descriptor System

## Table of Contents

1. [Class-Based Decorators](#section-1---class-based-decorators)
2. [Built-In Decorators](#section-2---built-in-decorators)
3. [Descriptor Protocol](#section-3---descriptor-protocol)
4. [Why Property Works](#section-4---why-property-works)
5. [Bound Methods](#section-5---bound-methods)
6. [Decorator Design Patterns](#section-6---decorator-design-patterns)
7. [functools Decorators](#section-7---functools-decorators)
8. [Testing Decorators](#section-8---testing-decorators)
9. [Common Mistakes](#section-9---common-mistakes)
10. [Exercises](#section-10---exercises)
11. [Interview Questions](#section-11---interview-questions)
12. [Comparison Tables \& Cheat Sheets](#final---comparison-tables-and-cheat-sheets)

***

## Section 1 — Class-Based Decorators

Class-based decorators leverage Python's descriptor protocol and `__call__` method to create stateful, reusable decorators with internal state.

### Two Primary Patterns

#### Pattern 1: `__call__` Only (Function-Scoped)

```python
class Decorator:
    def __call__(self, func):
        """Called when decorator is applied to a function"""
        self.func = func
        return self
    
    def __getattr__(self, name):
        """Forward attribute access to wrapped function"""
        return getattr(self.func, name)
```

**Usage:**

```python
@Decorator
def my_function():
    pass
```

**Object Lifecycle:**

- `Decorator()` instance created when class is defined
- `__call__(func)` invoked when `@Decorator` is applied
- Instance stores `func` and returns itself

**Memory Model:**

```python
# Before decoration
decorator_instance = Decorator()  # One instance in memory

# After decoration
@decorator_instance
def my_function():
    pass

# my_function is now decorator_instance
# decorator_instance.func points to original function
```


#### Pattern 2: `__init__` + `__call__` (Classic Decorator)

```python
class Decorator:
    def __init__(self, func):
        """Called immediately when @Decorator is applied"""
        self.func = func
        self.state = {}
    
    def __call__(self, *args, **kwargs):
        """Called when wrapped function is invoked"""
        self.state['calls'] = self.state.get('calls', 0) + 1
        return self.func(*args, **kwargs)
```

**Usage:**

```python
@Decorator
def my_function(x):
    return x * 2
```

**Object Lifecycle:**

```python
# Step 1: Class definition
# Decorator class exists in memory

# Step 2: Decoration (module load time)
@Decorator
def my_function(x):
    return x * 2

# Equivalent to:
# my_function = Decorator(my_function)
# __init__ called with func=my_function
# Instance created and stored as my_function

# Step 3: Invocation
my_function(5)  # __call__ invoked with args=(5,), kwargs={}
```


### Stateful Decorators

```python
class CountCalls:
    def __init__(self, func):
        self.func = func
        self.count = 0
        functools.update_wrapper(self, func)
    
    def __call__(self, *args, **kwargs):
        self.count += 1
        print(f"Call {self.count} to {self.func.__name__}")
        return self.func(*args, **kwargs)
```

**Key Features:**

- **State persistence**: `self.count` persists across calls
- **Wrapped function metadata**: `update_wrapper` preserves `__name__`, `__doc__`
- **Memory**: Each decorated function gets its own instance


### Memory Model Deep Dive

```python
class StatefulDecorator:
    def __init__(self, func):
        self.func = func
        self.cache = {}
    
    def __call__(self, *args):
        if args not in self.cache:
            self.cache[args] = self.func(*args)
        return self.cache[args]

@StatefulDecorator
def expensive(x):
    return x ** 2

# Memory layout:
# expensive -> StatefulDecorator instance
# StatefulDecorator instance.func -> original expensive function
# StatefulDecorator instance.cache -> { (1,): 1, (2,): 4, ... }
```


### Instance Creation Flow

```python
def demonstrate_flow():
    print("1. Class definition")
    
    class MyDecorator:
        def __init__(self, func):
            print(f"   __init__ called with {func.__name__}")
            self.func = func
        
        def __call__(self, *args):
            print(f"   __call__ called with {args}")
            return self.func(*args)
    
    print("2. Function definition")
    
    @MyDecorator
    def test(x):
        return x * 2
    
    print("3. Function invocation")
    result = test(5)  # __call__ invoked
    print(f"   Result: {result}")

demonstrate_flow()
```

**Output:**

```
1. Class definition
2. Function definition
   __init__ called with test
3. Function invocation
   __call__ called with (5,)
   Result: 10
```


***

## Section 2 — Built-In Decorators

### @staticmethod

**Internals:**

```python
class staticmethod:
    def __init__(self, func):
        self.func = func
    
    def __get__(self, obj, objtype=None):
        """Returns function without binding obj"""
        return self.func
```

**Behavior:**

- Binds to class, not instance
- No `self` parameter injected
- Accessible via `obj.method()` or `Class.method()`

```python
class MyClass:
    @staticmethod
    def static_add(x, y):
        return x + y

MyClass.static_add(1, 2)  # 3
MyInstance = MyClass()
MyInstance.static_add(1, 2)  # 3
```


### @classmethod

**Internals:**

```python
class classmethod:
    def __init__(self, func):
        self.func = func
    
    def __get__(self, obj, objtype=None):
        """Returns function bound to class"""
        if objtype is None:
            objtype = type(obj)
        return lambda *args: self.func(objtype, *args)
```

**Behavior:**

- Receives class as first argument
- Can access class attributes
- Used for factory methods

```python
class MyClass:
    count = 0
    
    @classmethod
    def increment_count(cls):
        cls.count += 1
        return cls.count

MyClass.increment_count()  # 1
```


### @property, @setter, @deleter

**Complete Internals:**

```python
class Property:
    def __init__(self, fget=None, fset=None, fdel=None, doc=None):
        self.fget = fget
        self.fset = fset
        self.fdel = fdel
        self.__doc__ = doc or fget.__doc__ if fget else None
    
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        if self.fget is None:
            raise AttributeError("unreadable attribute")
        return self.fget(obj)
    
    def __set__(self, obj, value):
        if self.fset is None:
            raise AttributeError("can't set attribute")
        self.fset(obj, value)
    
    def __delete__(self, obj):
        if self.fdel is None:
            raise AttributeError("can't delete attribute")
        self.fdel(obj)
    
    def setter(self, fset):
        return Property(self.fget, fset, self.fdel, self.__doc__)
    
    def deleter(self, fdel):
        return Property(self.fget, self.fset, fdel, self.__doc__)
```

**Usage:**

```python
class Circle:
    def __init__(self, radius):
        self._radius = radius
    
    @property
    def radius(self):
        """Get radius"""
        return self._radius
    
    @radius.setter
    def radius(self, value):
        """Set radius with validation"""
        if value <= 0:
            raise ValueError("Radius must be positive")
        self._radius = value
    
    @radius.deleter
    def radius(self):
        """Delete radius"""
        del self._radius
```

**Internal Flow:**

```python
circle = Circle(5)

# circle.radius calls Property.__get__(circle, Circle)
#   → returns self.fget(circle) → circle._radius

# circle.radius = 10 calls Property.__set__(circle, 10)
#   → calls self.fset(circle, 10)

# del circle.radius calls Property.__delete__(circle)
#   → calls self.fdel(circle)
```


***

## Section 3 — Descriptor Protocol

### Descriptor Types

**Data Descriptors:** Have `__set__` or `__delete__`

```python
class DataDescriptor:
    def __get__(self, obj, objtype=None): ...
    def __set__(self, obj, value): ...
    def __delete__(self, obj): ...
```

**Non-Data Descriptors:** Only `__get__`

```python
class NonDataDescriptor:
    def __get__(self, obj, objtype=None): ...
```


### Descriptor Protocol Methods

```python
class Descriptor:
    def __get__(self, obj, objtype=None):
        """Access descriptor via obj.attr or objtype.attr"""
        if obj is None:
            return self  # Return descriptor itself for class access
        return obj._value
    
    def __set__(self, obj, value):
        """Set descriptor value via obj.attr = value"""
        obj._value = value
    
    def __delete__(self, obj):
        """Delete descriptor via del obj.attr"""
        del obj._value
```


### Attribute Lookup Order

```python
# Priority order for obj.attr:
# 1. Data descriptors in class dict
# 2. Instance dict (__dict__)
# 3. Non-data descriptors in class dict
# 4. Regular attributes in class dict

class Example:
    data = DataDescriptor()
    non_data = NonDataDescriptor()
    
    def __init__(self):
        self.instance_attr = "instance"

obj = Example()
obj.data = 10  # Calls DataDescriptor.__set__
obj.non_data = 20  # Sets obj.__dict__['non_data'] = 20
```


### Decorator-Descriptor Interaction

```python
class ValidatedProperty:
    def __init__(self, validator):
        self.validator = validator
        self.data = {}
    
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return self.data.get(obj)
    
    def __set__(self, obj, value):
        if not self.validator(value):
            raise ValueError("Invalid value")
        self.data[obj] = value

def positive(x):
    return x > 0

class Number:
    value = ValidatedProperty(positive)
```

**How decorators use descriptors:**

```python
def logged_property(func):
    """Decorator that creates a logged property"""
    return Property(
        fget=lambda obj: func(obj),
        fset=lambda obj, val: func(obj, val)
    )
```


***

## Section 4 — Why Property Works

### Building Property from Scratch

```python
class MyProperty:
    def __init__(self, fget):
        self.fget = fget
        functools.update_wrapper(self, fget)
    
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return self.fget(obj)
    
    def setter(self, fset):
        return MyProperty(self.fget, fset)

class MyClass:
    def __init__(self):
        self._value = 0
    
    @MyProperty
    def value(self):
        return self._value
    
    @value.setter
    def value(self, val):
        self._value = val
```


### Descriptor Machinery

**Step-by-step attribute lookup:**

```python
class Trace:
    def __get__(self, obj, objtype=None):
        print(f"__get__ called: {obj}, {objtype}")
        return obj._value
    
    def __set__(self, obj, value):
        print(f"__set__ called: {obj}, {value}")
        obj._value = value

class MyClass:
    attr = Trace()
    
    def __init__(self):
        self._value = 10

obj = MyClass()

# obj.attr triggers:
# 1. Check class dict for 'attr' → Found Trace instance
# 2. Trace instance is data descriptor (has __set__)
# 3. Call Trace.__get__(obj, MyClass)
value = obj.attr  # Prints: __get__ called: <obj>, MyClass

# obj.attr = 20 triggers:
# 1. Check class dict for 'attr' → Found Trace instance
# 2. Trace is data descriptor
# 3. Call Trace.__set__(obj, 20)
obj.attr = 20  # Prints: __set__ called: <obj>, 20
```


### Attribute Lookup Algorithm

```python
def attribute_lookup(obj, name):
    """Simplified Python attribute lookup"""
    cls = type(obj)
    
    # 1. Check data descriptors in class
    for class_in_hierarchy in cls.__mro__:
        if name in class_in_hierarchy.__dict__:
            descriptor = class_in_hierarchy.__dict__[name]
            if hasattr(descriptor, '__set__'):
                return descriptor.__get__(obj, cls)
    
    # 2. Check instance dict
    if name in obj.__dict__:
        return obj.__dict__[name]
    
    # 3. Check non-data descriptors
    for class_in_hierarchy in cls.__mro__:
        if name in class_in_hierarchy.__dict__:
            descriptor = class_in_hierarchy.__dict__[name]
            if hasattr(descriptor, '__get__'):
                return descriptor.__get__(obj, cls)
    
    # 4. Regular class attribute
    if name in cls.__dict__:
        return cls.__dict__[name]
    
    raise AttributeError(name)
```


### Binding Process

```python
class BoundMethod:
    def __init__(self, func, obj):
        self.func = func
        self.obj = obj
    
    def __call__(self, *args):
        return self.func(self.obj, *args)

class Descriptor:
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return BoundMethod(self.func, obj)
    
    def __set__(self, obj, value):
        pass

def method(self):
    return f"Called on {self}"

class MyClass:
    attr = Descriptor()
    attr.func = method

obj = MyClass()
obj.attr()  # Returns: "Called on <obj>"
```


***

## Section 5 — Bound Methods

### Method Binding

```python
class MyClass:
    def method(self, x):
        return x * 2

obj = MyClass()

# obj.method creates a bound method
bound = obj.method

# bound is <bound-method MyClass.method of obj>
# When called: bound(5) → MyClass.method(obj, 5)
```


### Internal Representation

```python
import types

class MyClass:
    def method(self):
        pass

obj = MyClass()

# Unbound (class method)
unbound = MyClass.method
print(type(unbound))  # function

# Bound method
bound = obj.method
print(type(bound))  # method
print(bound.__self__)  # obj
print(bound.__func__)  # original function
```


### Self Injection Mechanism

```python
def demonstrate_self_injection():
    class Tracker:
        def __init__(self):
            self.calls = []
        
        def method(self, x):
            self.calls.append((self, x))
            return x * 2
    
    tracker = Tracker()
    
    # Direct call
    result1 = tracker.method(5)
    
    # Bound method call
    bound = tracker.method
    result2 = bound(5)
    
    # Both inject self automatically
    assert result1 == result2 == 10
    assert tracker.calls[0][0] is tracker
    assert tracker.calls[1][0] is tracker

demonstrate_self_injection()
```


### Method Objects

```python
class MethodObject:
    def __init__(self, func, obj):
        self.__func__ = func
        self.__self__ = obj
    
    def __call__(self, *args, **kwargs):
        return self.__func__(self.__self__, *args, **kwargs)
    
    def __repr__(self):
        return f"<bound-method {self.__func__.__name__} of {self.__self__}>"

class MyClass:
    def method(self, x):
        return x

obj = MyClass()
bound = MethodObject(obj.method, obj)
bound(5)  # Returns 5
```


### functools.wrap for Methods

```python
import functools

def method_decorator(func):
    @functools.wraps(func)
    def wrapper(self, *args, **kwargs):
        print(f"Calling {func.__name__}")
        return func(self, *args, **kwargs)
    return wrapper

class MyClass:
    @method_decorator
    def method(self, x):
        return x * 2

obj = MyClass()
obj.method(5)  # Prints: Calling method, Returns 10
```


***

## Section 6 — Decorator Design Patterns

### Logging

```python
import functools
import logging

def log_calls(logger=logging.getLogger(__name__)):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            logger.info(f"Calling {func.__name__} with args={args}, kwargs={kwargs}")
            result = func(*args, **kwargs)
            logger.info(f"{func.__name__} returned {result}")
            return result
        return wrapper
    return decorator

@log_calls()
def add(x, y):
    return x + y
```


### Timing

```python
import functools
import time

def timing(logger=None):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            start = time.perf_counter()
            result = func(*args, **kwargs)
            end = time.perf_counter()
            duration = end - start
            msg = f"{func.__name__} took {duration:.4f}s"
            if logger:
                logger.info(msg)
            else:
                print(msg)
            return result
        return wrapper
    return decorator

@timing()
def slow_function():
    time.sleep(0.5)
    return "done"
```


### Tracing

```python
import functools

def trace(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"→ {func.__name__}({args}, {kwargs})")
        result = func(*args, **kwargs)
        print(f"← {func.__name__} returned {result}")
        return result
    return wrapper

@trace
def recursive(n):
    if n <= 0:
        return 1
    return n * recursive(n - 1)
```


### Validation

```python
import functools
from typing import Callable

def validate(**validators):
    def decorator(func: Callable):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            # Get parameter names
            import inspect
            sig = inspect.signature(func)
            params = list(sig.parameters.keys())
            
            # Bind arguments
            bound = sig.bind(*args, **kwargs)
            bound.apply_defaults()
            
            # Validate each parameter
            for param_name, value in bound.arguments.items():
                if param_name in validators:
                    validator = validators[param_name]
                    if not validator(value):
                        raise ValueError(f"{param_name} invalid: {value}")
            
            return func(*args, **kwargs)
        return wrapper
    return decorator

@validate(x=lambda x: x > 0, y=lambda y: y < 100)
def range_func(x, y):
    return x + y
```


### Authentication

```python
import functools
from typing import Optional

class AuthManager:
    def __init__(self):
        self.users = {}
    
    def add_user(self, username, token):
        self.users[username] = token
    
    def verify_token(self, token):
        return token in self.users.values()

auth_manager = AuthManager()

def require_auth(token_header: str):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            if not auth_manager.verify_token(token_header):
                raise PermissionError("Authentication required")
            return func(*args, **kwargs)
        return wrapper
    return decorator

@require_auth("valid_token_123")
def protected_api():
    return "Secret data"
```


### Authorization

```python
import functools
from enum import Enum
from typing import List

class Role(Enum):
    USER = "user"
    ADMIN = "admin"
    SUPERADMIN = "superadmin"

def require_role(required_roles: List[Role]):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(user_role: Role, *args, **kwargs):
            if user_role not in required_roles:
                raise PermissionError(f"Required roles: {required_roles}")
            return func(user_role, *args, **kwargs)
        return wrapper
    return decorator

@require_role([Role.ADMIN, Role.SUPERADMIN])
def delete_user(user_role: Role, user_id: int):
    return f"Deleted user {user_id}"
```


### Retry

```python
import functools
import time
from typing import Callable, Type

def retry(max_attempts: int = 3, 
          delay: float = 1.0,
          exceptions: Type[Exception] = Exception):
    def decorator(func: Callable):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    if attempt == max_attempts - 1:
                        raise
                    print(f"Attempt {attempt + 1} failed: {e}")
                    time.sleep(delay * (attempt + 1))
        return wrapper
    return decorator

@retry(max_attempts=5, delay=0.5, exceptions=(ConnectionError, TimeoutError))
def unstable_api_call():
    import random
    if random.random() < 0.7:
        raise ConnectionError("Network failed")
    return "Success"
```


### Monitoring

```python
import functools
from collections import defaultdict
from typing import Dict

class MetricCollector:
    def __init__(self):
        self.metrics: Dict[str, Dict] = defaultdict(lambda: {
            'calls': 0,
            'errors': 0,
            'total_time': 0
        })

metrics = MetricCollector()

def monitor(func_name: str = None):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            name = func_name or func.__name__
            start = time.perf_counter()
            metrics[name]['calls'] += 1
            
            try:
                result = func(*args, **kwargs)
                return result
            except Exception as e:
                metrics[name]['errors'] += 1
                raise
            finally:
                end = time.perf_counter()
                metrics[name]['total_time'] += end - start
        return wrapper
    return decorator

@monitor()
def monitored_function():
    return 42
```


### Profiling

```python
import functools
import cProfile
import io
from typing import Optional

def profile(output: Optional[str] = None):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            profiler = cProfile.Profile()
            profiler.enable()
            
            try:
                result = func(*args, **kwargs)
                return result
            finally:
                profiler.disable()
                
                if output:
                    profiler.dump_stats(output)
                else:
                    stream = io.StringIO()
                    profiler.print_stats(stream=stream)
                    print(stream.getvalue())
        return wrapper
    return decorator

@profile()
def profiled_function():
    sum(i ** 2 for i in range(10000))
```


### Benchmarking

```python
import functools
import time
from typing import List
from dataclasses import dataclass

@dataclass
class BenchmarkResult:
    func_name: str
    iterations: int
    total_time: float
    avg_time: float
    min_time: float
    max_time: float

def benchmark(iterations: int = 100):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            times: List[float] = []
            
            for _ in range(iterations):
                start = time.perf_counter()
                func(*args, **kwargs)
                end = time.perf_counter()
                times.append(end - start)
            
            result = BenchmarkResult(
                func_name=func.__name__,
                iterations=iterations,
                total_time=sum(times),
                avg_time=sum(times) / len(times),
                min_time=min(times),
                max_time=max(times)
            )
            
            print(f"Benchmark: {result}")
            return func(*args, **kwargs)
        return wrapper
    return decorator

@benchmark(iterations=1000)
def benchmarked_sum():
    sum(range(1000))
```


***

## Section 7 — functools Decorators

### @lru_cache

```python
from functools import lru_cache
import time

@lru_cache(maxsize=128)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

# Performance comparison
def fib_no_cache(n):
    if n < 2:
        return n
    return fib_no_cache(n-1) + fib_no_cache(n-2)

# Time cached version
start = time.perf_counter()
fibonacci(50)
cached_time = time.perf_counter() - start

# Time uncached version (much slower)
start = time.perf_counter()
fib_no_cache(30)  # Reduced to avoid timeout
uncached_time = time.perf_counter() - start

print(f"Cached: {cached_time:.6f}s, Uncached: {uncached_time:.6f}s")
```

**Internals:**

```python
class LRUCache:
    def __init__(self, maxsize=128):
        self.maxsize = maxsize
        self.cache = {}
        self.order = []
    
    def __call__(self, func):
        @functools.wraps(func)
        def wrapper(*args):
            if args in self.cache:
                return self.cache[args]
            
            result = func(*args)
            self.cache[args] = result
            self.order.append(args)
            
            if len(self.cache) > self.maxsize:
                oldest = self.order.pop(0)
                del self.cache[oldest]
            
            return result
        return wrapper
```


### @cache

```python
from functools import cache

@cache
def expensive_computation(x):
    return x ** x

# Unbounded cache (same as lru_cache(maxsize=None))
```


### @cached_property

```python
from functools import cached_property

class DataProcessor:
    def __init__(self, data):
        self.data = data
        self._computed = None
    
    @cached_property
    def processed(self):
        print("Computing processed data...")
        return [x * 2 for x in self.data]

processor = DataProcessor([1, 2, 3])
processor.processed  # Computes once
processor.processed  # Returns cached value
```

**Internals:**

```python
class CachedProperty:
    def __init__(self, func):
        self.func = func
        self.attrname = f"_cache_{func.__name__}"
    
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        
        cache = getattr(obj, self.attrname, None)
        if cache is None:
            cache = self.func(obj)
            setattr(obj, self.attrname, cache)
        
        return cache
```


### @singledispatch

```python
from functools import singledispatch

@singledispatch
def process(data: int):
    return f"Processing int: {data}"

@process.register(str)
def _(data: str):
    return f"Processing str: {data}"

@process.register(list)
def _(data: list):
    return f"Processing list: {len(data)} items"

process(5)      # "Processing int: 5"
process("hi")   # "Processing str: hi"
process([1,2])  # "Processing list: 2 items"
```

**Internals:**

```python
class singledispatch:
    def __init__(self, func):
        self.func = func
        self.registry = {}
    
    def register(self, cls):
        def decorator(wrapper_func):
            self.registry[cls] = wrapper_func
            return wrapper_func
        return decorator
    
    def __call__(self, *args, **kwargs):
        if not args:
            raise TypeError("At least one argument required")
        
        first_type = type(args[0])
        handler = self.registry.get(first_type, self.func)
        return handler(*args, **kwargs)
```


### @singledispatchmethod

```python
from functools import singledispatchmethod

class Calculator:
    @singledispatchmethod
    def add(self, a: int, b: int):
        return a + b
    
    @add.register(float, float)
    def _(self, a: float, b: float):
        return a + b
    
    @add.register(str, str)
    def _(self, a: str, b: str):
        return a + b

calc = Calculator()
calc.add(1, 2)        # 3
calc.add(1.5, 2.5)    # 4.0
calc.add("a", "b")    # "ab"
```


### Performance Analysis

```python
import time
from functools import lru_cache, cache

# Benchmark lru_cache vs cache
@lru_cache(maxsize=128)
def cached_lru(n):
    return n ** n

@cache
def cached_all(n):
    return n ** n

def benchmark(func, iterations=10000):
    start = time.perf_counter()
    for i in range(iterations):
        func(i % 100)  # Cycle through 100 values
    return time.perf_counter() - start

lu_time = benchmark(cached_lru)
ca_time = benchmark(cached_all)

print(f"lru_cache(128): {lu_time:.4f}s")
print(f"cache (unbounded): {ca_time:.4f}s")
```


***

## Section 8 — Testing Decorators

### Unit Testing

```python
import unittest
from unittest.mock import Mock, patch
import functools

def counter(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        wrapper.calls += 1
        return func(*args, **kwargs)
    wrapper.calls = 0
    return wrapper

@counter
def add(x, y):
    return x + y

class CounterDecoratorTest(unittest.TestCase):
    def test_call_count(self):
        add.calls = 0  # Reset
        add(1, 2)
        add(3, 4)
        self.assertEqual(add.calls, 2)
    
    def test_wraps_preserves_metadata(self):
        self.assertEqual(add.__name__, 'add')
        self.assertEqual(add.__doc__, None)

if __name__ == '__main__':
    unittest.main()
```


### Mocking

```python
from unittest.mock import Mock, call

def mock_decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        wrapper.mock_call(*args, **kwargs)
        return func(*args, **kwargs)
    wrapper.mock_call = Mock()
    return wrapper

@mock_decorator
def service_call(data):
    return f"Processed: {data}"

# Test
service_call.mock_call.reset_mock()
service_call("test1")
service_call("test2")

assert service_call.mock_call.call_count == 2
service_call.mock_call.assert_has_calls([
    call("test1"),
    call("test2")
])
```


### Debugging Wrappers

```python
import inspect

def debug_decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"=== Calling {func.__name__} ===")
        print(f"Signature: {inspect.signature(func)}")
        print(f"Args: {args}")
        print(f"Kwargs: {kwargs}")
        
        result = func(*args, **kwargs)
        
        print(f"Result: {result}")
        print(f"=== End {func.__name__} ===\n")
        return result
    return wrapper

@debug_decorator
def example(x, y, z=10):
    return x + y + z

example(1, 2, z=3)
```


### Testing Stateful Decorators

```python
class TestStatefulDecorator(unittest.TestCase):
    def setUp(self):
        self.decorator = CountCalls(lambda x: x * 2)
    
    def test_state_persistence(self):
        self.decorator(1)
        self.decorator(2)
        self.assertEqual(self.decorator.count, 2)
    
    def test_wrapped_function(self):
        result = self.decorator(5)
        self.assertEqual(result, 10)
```


***

## Section 9 — Common Mistakes

### Lost Metadata

**Problem:**

```python
def bad_decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@bad_decorator
def my_func():
    """My docstring"""
    pass

print(my_func.__name__)  # 'wrapper' (wrong)
print(my_func.__doc__)   # None (wrong)
```

**Solution:**

```python
def good_decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```


### Incorrect Forwarding

**Problem:**

```python
def bad_forward(func):
    def wrapper(*args):  # Missing **kwargs
        return func(*args)
    return wrapper

@bad_forward
def func_with_kwargs(x, y=10):
    return x + y

func_with_kwargs(1, y=20)  # TypeError
```

**Solution:**

```python
def good_forward(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```


### Mutable State Bugs

**Problem:**

```python
def bad_mutable(func):
    cache = {}  # Shared across all decorated functions
    
    @functools.wraps(func)
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    return wrapper

@bad_mutable
def func1(x):
    return x * 2

@bad_mutable
def func2(x):
    return x * 3

# Both share same cache!
```

**Solution:**

```python
def good_mutable(func):
    cache = {}  # Per-function cache
    
    @functools.wraps(func)
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    return wrapper
```


### Thread Safety Issues

**Problem:**

```python
import threading

def bad_thread_unsafe(func):
    count = 0
    
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        count += 1  # Not thread-safe!
        return func(*args, **kwargs)
    return wrapper
```

**Solution:**

```python
def thread_safe(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        with wrapper.lock:
            wrapper.count += 1
        return func(*args, **kwargs)
    wrapper.count = 0
    wrapper.lock = threading.Lock()
    return wrapper
```


### Class Method Decorator Issues

**Problem:**

```python
def bad_class_method(func):
    def wrapper(self, *args):
        return func(self, *args)
    return wrapper

class MyClass:
    @bad_class_method
    def method(self):
        pass
```

**Solution:**

```python
def good_class_method(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```


***

## Section 10 — Exercises

### Exercise 1: Class-Based Stateful Decorator

**Problem:** Create a decorator that caches function results per-instance when used on class methods.

```python
# Solution
class InstanceCache:
    def __init__(self, func):
        self.func = func
        functools.update_wrapper(self, func)
    
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        
        if not hasattr(obj, '_instance_cache'):
            obj._instance_cache = {}
        
        cache = obj._instance_cache
        
        @functools.wraps(self.func)
        def wrapper(*args, **kwargs):
            key = (args, kwargs)
            if key not in cache:
                cache[key] = self.func(obj, *args, **kwargs)
            return cache[key]
        return wrapper
    
    def __call__(self, *args, **kwargs):
        raise TypeError("Cannot use @InstanceCache on standalone functions")

class DataProcessor:
    def __init__(self, data):
        self.data = data
    
    @InstanceCache
    def compute(self, multiplier):
        print(f"Computing for {multiplier}")
        return sum(self.data) * multiplier

processor = DataProcessor([1, 2, 3, 4, 5])
processor.compute(2)  # Computes: 30
processor.compute(2)  # Cached: 30
processor.compute(3)  # Computes: 45
```


### Exercise 2: Descriptor-Based Validation

**Problem:** Create a validated attribute using descriptors.

```python
# Solution
class Validated:
    def __init__(self, validator, default=None):
        self.validator = validator
        self.default = default
        self.name = None
    
    def __set_name__(self, owner, name):
        self.name = name
    
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, f'_{self.name}', self.default)
    
    def __set__(self, obj, value):
        if not self.validator(value):
            raise ValueError(f"{self.name} is invalid")
        setattr(obj, f'_{self.name}', value)

class Person:
    age = Validated(lambda x: 0 <= x <= 150, default=0)
    name = Validated(lambda x: len(x) > 0 and len(x) < 100)
    
    def __init__(self, name, age):
        self.name = name
        self.age = age

p = Person("Alice", 25)
p.age = 30  # OK
try:
    p.age = 200  # ValueError
except ValueError as e:
    print(e)
```


### Exercise 3: Async Retry Decorator

**Problem:** Create an async retry decorator.

```python
# Solution
import asyncio
import functools

def async_retry(max_attempts=3, delay=1.0):
    def decorator(func):
        @functools.wraps(func)
        async def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return await func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    print(f"Attempt {attempt + 1} failed: {e}")
                    await asyncio.sleep(delay * (attempt + 1))
        return wrapper
    return decorator

@async_retry(max_attempts=5, delay=0.5)
async def unstable_async_api():
    import random
    if random.random() < 0.8:
        raise ConnectionError("Network failed")
    return "Success"

async def main():
    result = await unstable_async_api()
    print(result)

asyncio.run(main())
```


### Exercise 4: Decorator Stack Meter

**Problem:** Measure decorator stack depth.

```python
# Solution
import functools
import inspect

def measure_depth(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        depth = 0
        current = wrapper
        while hasattr(current, '__wrapped__'):
            depth += 1
            current = current.__wrapped__
        print(f"Decorator depth: {depth}")
        return func(*args, **kwargs)
    return wrapper

@measure_depth
@measure_depth
@measure_depth
def deeply_decorated():
    pass

deeply_decorated()  # Prints: Decorator depth: 3
```


### Exercise 5: Context Manager Decorator

**Problem:** Create a decorator that converts functions to use context managers.

```python
# Solution
import functools
from contextlib import contextmanager

@contextmanager
def transaction():
    print("Starting transaction")
    try:
        yield "tx"
        print("Committing")
    except:
        print("Rolling back")
        raise

def with_context(ctx_manager):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            with ctx_manager() as ctx:
                return func(ctx, *args, **kwargs)
        return wrapper
    return decorator

@with_context(transaction)
def process_data(tx, data):
    print(f"Processing {data} with {tx}")
    return data.upper()

result = process_data("hello")
print(result)  # "HELLO"
```


***

## Section 11 — Interview Questions

### Question 1: What is the difference between `__init__` + `__call__` and `__call__` only decorator patterns?

**Answer:**

- `__init__` + `__call__`: Decorator instance created at decoration time, stores function in `__init__`. Best for stateful decorators.
- `__call__` only: Decorator instance created once, function passed to `__call__`. Best for function-scoped state.


### Question 2: Explain how `@property` uses the descriptor protocol.

**Answer:** `Property` implements `__get__`, `__set__`, and `__delete__`. When accessed via `obj.attr`, Python finds `Property` in class dict, detects it's a data descriptor, and calls `__get__(obj, cls)`.

### Question 3: Why do data descriptors have higher priority than instance attributes?

**Answer:** Data descriptors have `__set__`, so Python routes attribute access to them to ensure consistent behavior. This prevents instance attributes from overriding descriptor semantics.

### Question 4: What is `functools.wraps` and why is it important?

**Answer:** `wraps` copies `__name__`, `__doc__`, `__module__`, `__qualname__`, `__annotations__`, and `__dict__` from wrapped function to wrapper. Essential for preserving metadata.

### Question 5: How does method binding work in Python?

**Answer:** When accessing `obj.method`, Python finds function in class dict, sees it has `__get__`, calls `__get__(obj, cls)`, which returns a `bound_method` object with `__self__` = obj and `__func__` = original function.

### Question 6: Implement a decorator that tracks execution time per function.

**Answer:**

```python
import functools
import time

def timing(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        end = time.perf_counter()
        wrapper.time_taken = end - start
        return result
    wrapper.time_taken = 0
    return wrapper
```


### Question 7: What is the difference between `@staticmethod` and `@classmethod`?

**Answer:**

- `@staticmethod`: No automatic first argument, behaves like regular function.
- `@classmethod`: Receives class as first argument (`cls`).


### Question 8: Explain `lru_cache` vs `cache`.

**Answer:** `lru_cache(maxsize=128)` has bounded cache with LRU eviction. `cache` is `lru_cache(maxsize=None)` - unbounded cache.

### Question 9: How would you make a decorator thread-safe?

**Answer:** Use `threading.Lock()` to protect shared state:

```python
def thread_safe(func):
    lock = threading.Lock()
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        with lock:
            return func(*args, **kwargs)
    return wrapper
```


### Question 10: What is `__set_name__` and when is it called?

**Answer:** `__set_name__(owner, name)` is called when descriptor is assigned to class attribute. Provides descriptor with its attribute name.

### Question 11: Implement a decorator that requires async functions.

**Answer:**

```python
import functools
import inspect

def async_only(func):
    if not inspect.iscoroutinefunction(func):
        raise TypeError("Must be async function")
    return func
```


### Question 12: How does `singledispatch` work internally?

**Answer:** Uses a registry dict mapping types to handlers. At call time, inspects first argument type and looks up handler in registry.

### Question 13: What problem does `cached_property` solve?

**Answer:** Caches property computation per instance, computed once on first access, then cached in instance dict.

### Question 14: Why is `functools.update_wrapper` needed in class decorators?

**Answer:** Copies metadata from wrapped function to class instance, preserving `__name__`, `__doc__`, etc.

### Question 15: Implement a decorator that validates argument types.

**Answer:**

```python
import functools
import inspect

def type_check(func):
    sig = inspect.signature(func)
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        bound = sig.bind(*args, **kwargs)
        bound.apply_defaults()
        for name, param in sig.parameters.items():
            if param.annotation != inspect.Parameter.empty:
                value = bound.arguments[name]
                if not isinstance(value, param.annotation):
                    raise TypeError(f"{name} must be {param.annotation}")
        return func(*args, **kwargs)
    return wrapper
```


### Question 16: What is the purpose of `__wrapped__` attribute?

**Answer:** Points to original wrapped function, used by `inspect.getcoroutine`, `functools.wraps`, and debugging tools.

### Question 17: Explain decorator stacking order.

**Answer:** `@a @b @c def f()` applies as `f = a(b(c(f)))`. Outer decorator (`a`) wraps innermost.

### Question 18: How to preserve function signature in decorators?

**Answer:** Use `@functools.wraps(func)` which copies `__annotations__` and `__qualname__`. For full signature, use `inspect.signature`.

### Question 19: What's the difference between data and non-data descriptors?

**Answer:** Data descriptors have `__set__` or `__delete__`. Non-data only have `__get__`. Data descriptors have higher priority in attribute lookup.

### Question 20: Implement a decorator that counts exceptions.

**Answer:**

```python
import functools

def count_exceptions(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        try:
            return func(*args, **kwargs)
        except Exception:
            wrapper.exception_count += 1
            raise
    wrapper.exception_count = 0
    return wrapper
```


### Question 21: How does `@property.setter` work?

**Answer:** `setter` returns new `Property` instance with same `fget`, new `fset`, same `fdel`, same `__doc__`.

### Question 22: What is the MRO and how does it affect descriptors?

**Answer:** Method Resolution Order (MRO) is class hierarchy order. Descriptors searched in MRO order; first found descriptor wins.

***

## Final — Comparison Tables and Cheat Sheets

### Decorator Type Comparison

| Type | Stateful | Metadata | Best For |
| :-- | :-- | :-- | :-- |
| Function decorator | No (use closure) | Needs `wraps` | Simple decorators |
| Class decorator (`__init__` + `__call__`) | Yes | Needs `update_wrapper` | Complex state |
| Class decorator (`__call__` only) | Yes (per instance) | Needs `update_wrapper` | Function-scoped state |
| Descriptor | Yes | Built-in | Attribute management |

### Built-in Decorators Comparison

| Decorator | Takes Args | Returns | Use Case |
| :-- | :-- | :-- | :-- |
| `@staticmethod` | No | Function | Class-level function |
| `@classmethod` | No | Function | Factory methods |
| `@property` | No | Property | Read-only attribute |
| `@property.setter` | No | Property | Writeable attribute |
| `@property.deleter` | No | Property | Deleteable attribute |

### functools Decorators Comparison

| Decorator | Bounded | Per-Function | Use Case |
| :-- | :-- | :-- | :-- |
| `@lru_cache(maxsize=N)` | Yes | Yes | Memory-constrained caching |
| `@cache` | No | Yes | Unlimited caching |
| `@cached_property` | No | Per-instance | Cached property |
| `@singledispatch` | No | Yes | Type-based dispatch |
| `@singledispatchmethod` | No | Yes | Type-based method dispatch |

### Descriptor Priority Table

| Lookup Source | Priority | Example |
| :-- | :-- | :-- |
| Data descriptor | 1 | `@property`, `@staticmethod` |
| Instance dict | 2 | `obj.__dict__['attr']` |
| Non-data descriptor | 3 | Bound methods |
| Class dict | 4 | Regular class attributes |

### Common Mistakes Cheat Sheet

| Mistake | Symptom | Fix |
| :-- | :-- | :-- |
| Missing `@wraps` | Lost `__name__`, `__doc__` | Add `@functools.wraps(func)` |
| Missing `**kwargs` | TypeError on kwargs | Use `*args, **kwargs` |
| Shared mutable state | Cross-function cache pollution | Use per-function cache |
| No thread lock | Race conditions | Add `threading.Lock()` |
| Wrong decorator order | Unexpected behavior | Remember: outer wraps inner |

### Quick Reference: Decorator Patterns

```python
# Simple function decorator
def simple(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

# Decorator with arguments
def with_args(arg1, arg2):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            return func(*args, **kwargs)
        return wrapper
    return decorator

# Class-based decorator
class ClassDecorator:
    def __init__(self, func):
        self.func = func
        functools.update_wrapper(self, func)
    
    def __call__(self, *args, **kwargs):
        return self.func(*args, **kwargs)

# Descriptor
class MyDescriptor:
    def __get__(self, obj, objtype=None): ...
    def __set__(self, obj, value): ...
    def __delete__(self, obj): ...
```


### Performance Guidelines

| Pattern | Memory | Speed | Recommendation |
| :-- | :-- | :-- | :-- |
| Function decorator | Low | Fast | Simple cases |
| Class decorator | Medium | Fast | State needed |
| `@lru_cache` | Medium | Very fast | Repeated calls |
| `@cache` | High | Very fast | Unlimited cache OK |
| Descriptor | Medium | Fast | Attribute control |

Choose based on:

- **State needed?** → Class decorator
- **Caching?** → `@lru_cache` or `@cache`
- **Attribute control?** → Descriptor
- **Simple wrapping?** → Function decorator

