<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Advanced Python Object Customization – Part 2

## Table of Contents

1. [Magic Methods (Advanced)](#magic-methods-advanced)
    - [Container Methods](#container-methods)
        - [`__len__`](#-len)
        - [`__iter__`](#-iter)
        - [`__next__`](#-next)
        - [`__getitem__`](#-getitem)
        - [`__setitem__`](#-setitem)
        - [`__contains__`](#-contains)
    - [Callable Objects](#callable-objects)
        - [`__call__`](#-call)
    - [Context Managers](#context-managers)
        - [`__enter__`](#-enter)
        - [`__exit__`](#-exit)
2. [Data Classes](#data-classes)
    - [Introduction](#introduction)
    - [dataclass Decorator](#dataclass-decorator)
    - [Fields](#fields)
    - [Default Values](#default-values)
    - [default_factory](#default_factory)
    - [Frozen Dataclasses](#frozen-dataclasses)
    - [Ordering](#ordering)
    - [Comparison Generation](#comparison-generation)
    - [`__post_init__`](#-post_init)
    - [Dataclass Internals](#dataclass-internals)
3. [__slots__](#slots)
    - [Introduction](#introduction-slots)
    - [Memory Optimization](#memory-optimization)
    - [Benchmarks](#benchmarks)
    - [Restrictions](#restrictions)
    - [Performance Impact](#performance-impact)
    - [When to Use](#when-to-use)
    - [When Not to Use](#when-not-to-use)
4. [Real-World Projects](#real-world-projects)
5. [Final Summary](#final-summary)
6. [Complete Cheat Sheet](#complete-cheat-sheet)
7. [Decision Matrix](#decision-matrix)

***

## Magic Methods (Advanced)

Magic methods (also called dunder methods) allow you to customize how Python objects behave with built-in operations. This section covers the remaining advanced magic methods beyond the basics.

### Container Methods

Container methods enable your custom classes to behave like Python's built-in containers (lists, dictionaries, sets).

#### `__len__`

Returns the "length" of an object. Called by `len()`.

```python
class Stack:
    def __init__(self):
        self._items = []
    
    def push(self, item):
        self._items.append(item)
    
    def pop(self):
        return self._items.pop()
    
    def __len__(self):
        return len(self._items)
    
    def __str__(self):
        return f"Stack({self._items})"

stack = Stack()
stack.push(1)
stack.push(2)
stack.push(3)

print(len(stack))  # Output: 3 [cite:web:1]
```

**Key points:**

- Must return a non-negative integer
- Called automatically by `len(obj)`
- Essential for containers that need size tracking


#### `__iter__`

Returns an iterator object. Called when you use `for` loops or `iter()`.

```python
class NumberRange:
    def __init__(self, start, end):
        self.start = start
        self.end = end
    
    def __iter__(self):
        return NumberRangeIterator(self.start, self.end)

class NumberRangeIterator:
    def __init__(self, start, end):
        self.current = start
        self.end = end
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.current >= self.end:
            raise StopIteration
        value = self.current
        self.current += 1
        return value

for num in NumberRange(1, 5):
    print(num)  # 1, 2, 3, 4
```

**Iterable Protocol:**

- An object is **iterable** if it has `__iter__()` that returns an iterator
- An **iterator** has both `__iter__()` and `__next__()`


#### `__next__`

Returns the next item from an iterator. Called by `next()`.

```python
class LazyDataPipeline:
    def __init__(self, data_source):
        self.data_source = data_source
        self.index = 0
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.index >= len(self.data_source):
            raise StopIteration
        
        # Simulate lazy loading
        item = self.data_source[self.index]
        self.index += 1
        return item.process()  # Process only when requested

class DataItem:
    def __init__(self, raw_data):
        self.raw_data = raw_data
    
    def process(self):
        # Heavy processing - only happens when __next__ is called
        return self.raw_data.upper()

pipeline = LazyDataPipeline([DataItem("a"), DataItem("b"), DataItem("c")])

for item in pipeline:
    print(item)  # A, B, C - processed lazily
```

**Iterator Protocol:**

- `__next__()` must return the next item
- Must raise `StopIteration` when no more items exist
- Can be called repeatedly by `for` loops


#### `__getitem__`

Allows indexing and slicing. Called by `obj[key]`.

```python
class CustomList:
    def __init__(self, items):
        self._items = items
    
    def __getitem__(self, index):
        # Handle negative indices
        if isinstance(index, int):
            if index < 0:
                index = len(self._items) + index
            return self._items[index]
        
        # Handle slicing
        elif isinstance(index, slice):
            return CustomList(self._items[index])
        
        raise TypeError("Index must be int or slice")
    
    def __len__(self):
        return len(self._items)

my_list = CustomList([10, 20, 30, 40, 50])

print(my_list[0])      # 10
print(my_list[-1])     # 50
print(my_list[1:4])    # CustomList([20, 30, 40])
```

**Indexing Internals:**

```
obj[key]  →  calls obj.__getitem__(key)
```


#### `__setitem__`

Allows setting values by index. Called by `obj[key] = value`.

```python
class IndexedData:
    def __init__(self):
        self._data = {}
    
    def __setitem__(self, key, value):
        self._data[key] = value
        print(f"Set {key} = {value}")
    
    def __getitem__(self, key):
        return self._data[key]
    
    def __len__(self):
        return len(self._data)

data = IndexedData()
data[0] = "first"   # Set 0 = first
data[1] = "second"  # Set 1 = second
data[2] = "third"   # Set 2 = third

print(data[1])  # second
```


#### `__contains__`

Checks if an element exists. Called by `in` operator.

```python
class SearchableCollection:
    def __init__(self, items):
        self._items = items
        self._search_cache = set(items)
    
    def __contains__(self, item):
        # Optimized search using cache
        return item in self._search_cache
    
    def __iter__(self):
        return iter(self._items)

collection = SearchableCollection([1, 2, 3, 4, 5])

print(3 in collection)  # True - uses __contains__
print(6 in collection)  # False

# Without __contains__, Python would iterate through all items
# With __contains__, we use optimized cache
```

**`in` Operator Internals:**

```
item in obj  →  calls obj.__contains__(item)
               if not defined, iterates with __iter__ and __eq__
```


### Container Customization Table

| Method | Called By | Purpose | Example |
| :-- | :-- | :-- | :-- |
| `__len__` | `len(obj)` | Get length | `len(stack)` |
| `__iter__` | `for` loops | Make iterable | `for x in obj` |
| `__next__` | `next(obj)` | Get next item | Iterator protocol |
| `__getitem__` | `obj[key]` | Indexing | `list[0]` |
| `__setitem__` | `obj[key] = v` | Set index | `list[0] = 1` |
| `__contains__` | `item in obj` | Membership | `3 in list` |

### Custom Dictionary Example

```python
class CaseInsensitiveDict:
    def __init__(self, data=None):
        self._data = {}
        if data:
            for key, value in data.items():
                self[key] = value
    
    def __setitem__(self, key, value):
        self._data[key.lower()] = value
    
    def __getitem__(self, key):
        return self._data[key.lower()]
    
    def __contains__(self, key):
        return key.lower() in self._data
    
    def __len__(self):
        return len(self._data)
    
    def __iter__(self):
        return iter(self._data)
    
    def __repr__(self):
        return f"CaseInsensitiveDict({self._data})"

cid = CaseInsensitiveDict({"Name": "Alice", "Age": 30})

print(cid["name"])    # Alice - case insensitive
print(cid["NAME"])    # Alice
print("age" in cid)   # True
print(len(cid))       # 2
```


### for-loop Internals Diagram

```
for item in iterable:
    # Step 1: Get iterator
    iterator = iterable.__iter__()
    
    # Step 2: Loop until StopIteration
    while True:
        try:
            # Step 3: Get next item
            item = iterator.__next__()
            # Execute loop body
            process(item)
        except StopIteration:
            # Step 4: Exit loop
            break
```


***

## Callable Objects

### `__call__`

Makes an object callable like a function. Called by `obj()`.

```python
class Predictor:
    """Machine learning predictor with state"""
    
    def __init__(self, model_weights):
        self.weights = model_weights
        self.predictions_made = 0
    
    def __call__(self, input_data):
        """Make a prediction"""
        self.predictions_made += 1
        
        # Simulate prediction logic
        prediction = sum(w * x for w, x in zip(self.weights, input_data))
        return prediction > 0.5
    
    def get_stats(self):
        return {
            "predictions": self.predictions_made,
            "weights": self.weights
        }

# Usage - looks like a function but has state
predictor = Predictor([0.5, 0.3, 0.2])

result1 = predictor([1, 0, 1])  # Calls __call__
result2 = predictor([0, 1, 1])  # Calls __call__ again

print(result1)  # True or False
print(predictor.get_stats())  # {'predictions': 2, 'weights': [0.5, 0.3, 0.2]}
```


### Stateful Callable Example

```python
class Counter:
    """Counter object that maintains state"""
    
    def __init__(self, start=0):
        self.count = start
    
    def __call__(self, increment=1):
        self.count += increment
        return self.count
    
    def reset(self):
        self.count = 0

counter = Counter(10)

print(counter())      # 11
print(counter(5))     # 16
print(counter(-3))    # 13
counter.reset()
print(counter.count)  # 0
```


### Custom Validator

```python
class Validator:
    """Stateful validator for data validation"""
    
    def __init__(self, min_val=None, max_val=None):
        self.min_val = min_val
        self.max_val = max_val
        self.validation_errors = []
    
    def __call__(self, value):
        """Validate a value"""
        self.validation_errors = []
        
        if self.min_val is not None and value < self.min_val:
            self.validation_errors.append(f"Value {value} < min {self.min_val}")
        
        if self.max_val is not None and value > self.max_val:
            self.validation_errors.append(f"Value {value} > max {self.max_val}")
        
        return len(self.validation_errors) == 0
    
    def is_valid(self):
        return len(self.validation_errors) == 0
    
    def get_errors(self):
        return self.validation_errors

# Usage
age_validator = Validator(min_val=0, max_val=120)

print(age_validator(25))   # True
print(age_validator.is_valid())  # True

print(age_validator(-5))   # False
print(age_validator.get_errors())  # ['Value -5 < min 0']
```


### Callable Objects vs Functions

| Aspect | Function | Callable Object |
| :-- | :-- | :-- |
| State | No (unless global) | Yes (instance attributes) |
| Persistence | No | Yes (between calls) |
| Methods | No | Yes (can have helper methods) |
| Inheritance | No | Yes |


***

## Context Managers

Context managers manage resources with the `with` statement.

### `__enter__`

Called when entering the `with` block. Returns the resource.

### `__exit__`

Called when exiting the `with` block. Handles cleanup and exceptions.

```python
class DatabaseConnection:
    """Database connection context manager"""
    
    def __init__(self, db_url):
        self.db_url = db_url
        self.connection = None
    
    def __enter__(self):
        """Establish connection"""
        print(f"Connecting to {self.db_url}")
        self.connection = self._create_connection()
        return self.connection
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        """Close connection"""
        print("Closing connection")
        if self.connection:
            self.connection.close()
        
        # Handle exceptions
        if exc_type is not None:
            print(f"Exception occurred: {exc_val}")
            # Return False to propagate exception, True to suppress
            return False
        
        return True
    
    def _create_connection(self):
        # Simulate connection creation
        class MockConnection:
            def close(self):
                pass
            def execute(self, query):
                print(f"Executing: {query}")
        return MockConnection()

# Usage
with DatabaseConnection("postgres://localhost/db") as conn:
    conn.execute("SELECT * FROM users")
    conn.execute("INSERT INTO users VALUES (1, 'Alice')")

# Connection automatically closed even if exception occurs
```


### File Management Context Manager

```python
class ManagedFile:
    """File management with automatic cleanup"""
    
    def __init__(self, filename, mode='r'):
        self.filename = filename
        self.mode = mode
        self.file = None
    
    def __enter__(self):
        self.file = open(self.filename, self.mode)
        return self.file
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.file:
            self.file.close()
        
        # Log exceptions
        if exc_type:
            print(f"Error processing {self.filename}: {exc_val}")
        
        return False  # Don't suppress exceptions

# Usage
with ManagedFile("data.txt", "w") as f:
    f.write("Hello, World!")

with ManagedFile("data.txt", "r") as f:
    content = f.read()
    print(content)  # Hello, World!
```


### Logging System Context Manager

```python
import logging
from contextlib import contextmanager

class LoggingContext:
    """Temporarily change logging level"""
    
    def __init__(self, logger_name, level):
        self.logger = logging.getLogger(logger_name)
        self.old_level = self.logger.level
        self.new_level = level
    
    def __enter__(self):
        self.logger.setLevel(self.new_level)
        print(f"Logging level set to {self.new_level}")
        return self.logger
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.logger.setLevel(self.old_level)
        print(f"Logging level restored to {self.old_level}")
        return False

# Usage
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger("myapp")

with LoggingContext("myapp", logging.DEBUG):
    logger.debug("This debug message is visible")
    logger.info("This info message is visible")

logger.debug("This debug message is NOT visible")
```


### with Statement Execution Flow

```
with expression as variable:
    block

# Execution:
# 1. Evaluate expression
# 2. Call __enter__() → variable = result
# 3. Execute block
# 4. If exception: call __exit__(exc_type, exc_val, exc_tb)
# 5. If no exception: call __exit__(None, None, None)
# 6. If __exit__ returns True: suppress exception
# 7. If __exit__ returns False: propagate exception
```


### Exception Propagation Diagram

```
with ctx:
    try:
        # block code
        if error:
            raise SomeError()
    except SomeError as e:
        # __exit__ receives:
        # exc_type = SomeError
        # exc_val = e
        # exc_tb = traceback
        ctx.__exit__(SomeError, e, traceback)
```


***

## Data Classes

### Introduction

Data classes were introduced in Python 3.7 to reduce boilerplate code for classes that mainly store data.

**Why dataclasses were introduced:**

- Remove repetitive `__init__`, `__repr__`, `__eq__` methods
- Make data-focused classes more readable
- Provide type hint integration
- Reduce errors in manual implementation

**Boilerplate reduction:**

```python
# Without dataclass (boilerplate)
class Person:
    def __init__(self, name: str, age: int, email: str):
        self.name = name
        self.age = age
        self.email = email
    
    def __repr__(self):
        return f"Person(name={self.name}, age={self.age}, email={self.email})"
    
    def __eq__(self, other):
        if not isinstance(other, Person):
            return False
        return (self.name, self.age, self.email) == \
               (other.name, other.age, other.email)

# With dataclass (clean)
from dataclasses import dataclass

@dataclass
class Person:
    name: str
    age: int
    email: str
```


### dataclass Decorator

The `@dataclass` decorator automatically generates:

- `__init__()`: Initializes attributes
- `__repr__()`: String representation
- `__eq__()`: Equality comparison
- `__hash__()`: If `eq=True` and `frozen=False` (not generated by default)
- `__setattr__()`: Not generated
- `__delattr__()`: Not generated

```python
from dataclasses import dataclass

@dataclass
class Product:
    id: int
    name: str
    price: float
    
    def apply_discount(self, percent: float):
        self.price = self.price * (1 - percent / 100)

product = Product(1, "Laptop", 999.99)
print(product)  # Product(id=1, name='Laptop', price=999.99)
print(product == Product(1, "Laptop", 999.99))  # True
```

**Internal behavior:**

- Decorator runs at class definition time
- Inspects type hints to determine fields
- Generates methods in the class body
- Preserves docstrings and method definitions


### Fields

#### `field()`

Customize field behavior with `field()`.

```python
from dataclasses import dataclass, field
from typing import List

@dataclass
class Team:
    name: str
    members: List[str] = field(default_factory=list)
    metadata: dict = field(default_factory=dict, repr=False)
    
    def add_member(self, member: str):
        self.members.append(member)

team = Team("Engineering")
team.add_member("Alice")
team.add_member("Bob")

print(team)  # Team(name='Engineering', members=['Alice', 'Bob'])
# metadata is not shown because repr=False
```


#### `metadata`

Attach arbitrary metadata to fields.

```python
from dataclasses import dataclass, field

@dataclass
class Sensor:
    name: str
    value: float = field(metadata={"unit": "degrees", "min": 0, "max": 100})
    
    def get_unit(self):
        return self.value_metadata["unit"]

sensor = Sensor("Temperature", 25.5)
print(sensor.value.metadata)  # {'unit': 'degrees', 'min': 0, 'max': 100}
```


#### Field Parameters

| Parameter | Default | Purpose |
| :-- | :-- | :-- |
| `init` | `True` | Include in `__init__` |
| `repr` | `True` | Include in `__repr__` |
| `cmp` | `True` | Include in comparison methods |
| `hash` | `None` | Include in `__hash__` (None = same as cmp) |
| `metadata` | `{}` | Custom metadata dict |
| `default` | `None` | Default value |
| `default_factory` | `None` | Factory for default value |

```python
from dataclasses import dataclass, field

@dataclass
class User:
    id: int
    username: str
    password: str = field(repr=False, init=False)  # Hidden in repr, not in init
    created_at: str = field(init=False, default="2024-01-01")
    
user = User(1, "alice", "secret123")
print(user)  # User(id=1, username='alice', created_at='2024-01-01')
# password is hidden
```


### Default Values

#### Immutable Defaults

Safe to use directly:

```python
from dataclasses import dataclass

@dataclass
class Config:
    name: str = "default"
    version: int = 1
    enabled: bool = True
```


#### Mutable Defaults - Common Pitfall

```python
# WRONG - all instances share the same list!
@dataclass
class BadExample:
    items: list = []  # All instances share this list!

bad1 = BadExample()
bad2 = BadExample()
bad1.items.append(1)
print(bad2.items)  # [1] - unexpected!
```


#### Correct Approach

```python
from dataclasses import dataclass

@dataclass
class GoodExample:
    items: list = None
    
    def __post_init__(self):
        if self.items is None:
            self.items = []

good1 = GoodExample()
good2 = GoodExample()
good1.items.append(1)
print(good2.items)  # [] - correct!
```


### default_factory

Safe way to use mutable defaults.

```python
from dataclasses import dataclass, field
from typing import List, Dict

@dataclass
class ShoppingCart:
    items: List[str] = field(default_factory=list)
    prices: Dict[str, float] = field(default_factory=dict)
    tags: List[str] = field(default_factory=lambda: ["default"])
    
    def add_item(self, name: str, price: float):
        self.items.append(name)
        self.prices[name] = price

cart1 = ShoppingCart()
cart2 = ShoppingCart()

cart1.add_item("Laptop", 999.99)
print(cart2.items)  # [] - independent lists!
```


#### Nested Structures

```python
from dataclasses import dataclass, field
from typing import Dict, List

@dataclass
class Department:
    name: str
    
@dataclass
class Company:
    departments: Dict[str, Department] = field(default_factory=dict)
    employees: List[str] = field(default_factory=list)
    
    def add_department(self, dept: Department):
        self.departments[dept.name] = dept
    
    def add_employee(self, name: str):
        self.employees.append(name)

company = Company()
company.add_department(Department("Engineering"))
company.add_employee("Alice")

print(company.departments)  # {'Engineering': Department(name='Engineering')}
```


### Frozen Dataclasses

Immutable objects that cannot be modified after creation.

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Point:
    x: float
    y: float
    
    def distance_from_origin(self):
        return (self.x**2 + self.y**2) ** 0.5

point = Point(3, 4)
print(point.distance_from_origin())  # 5.0

# Try to modify - raises error
try:
    point.x = 10  # dataclasses.FrozenInstanceError
except Exception as e:
    print(f"Error: {e}")
```


#### Hashability

Frozen dataclasses are automatically hashable:

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Coordinate:
    lat: float
    lon: float

coord1 = Coordinate(40.7, -74.0)
coord2 = Coordinate(34.0, -118.0)

# Can use in sets and as dict keys
locations = {coord1, coord2}
location_map = {coord1: "NYC", coord2: "LA"}

print(len(locations))  # 2
print(location_map[coord1])  # NYC
```


#### Thread Safety

Frozen dataclasses are thread-safe by default:

```python
from dataclasses import dataclass
from threading import Thread

@dataclass(frozen=True)
class Config:
    api_key: str
    timeout: int
    retries: int

# Safe to share across threads - no modification possible
config = Config("secret-key", 30, 3)

def worker(thread_id):
    print(f"Thread {thread_id}: {config.timeout}s timeout")

threads = [Thread(target=worker, args=(i,)) for i in range(5)]
for t in threads:
    t.start()
for t in threads:
    t.join()
```


### Ordering

Generate comparison methods for sorting.

```python
from dataclasses import dataclass

@dataclass(ordering=True)
class Employee:
    name: str
    salary: int
    rank: int

employees = [
    Employee("Alice", 80000, 3),
    Employee("Bob", 90000, 2),
    Employee("Charlie", 75000, 1),
]

# Generated methods: __lt__, __le__, __gt__, __ge__
print(employees[0] < employees[1])  # True (compares by all fields)

# Sort by all fields
sorted_employees = sorted(employees)
for emp in sorted_employees:
    print(emp)

# Sort by specific field with custom key
from functools import total_ordering

@total_ordering
@dataclass
class Product:
    name: str
    price: float
    
    def __lt__(self, other):
        return self.price < other.price

products = [
    Product("Laptop", 999.99),
    Product("Phone", 599.99),
    Product("Tablet", 399.99),
]

for p in sorted(products):
    print(p)  # Sorted by price
```


#### Employee Ranking

```python
from dataclasses import dataclass

@dataclass(ordering=True)
class Employee:
    name: str
    salary: int
    performance_score: float

employees = [
    Employee("Alice", 80000, 4.5),
    Employee("Bob", 90000, 4.8),
    Employee("Charlie", 75000, 4.2),
]

# Rank by salary (highest first)
top_paid = sorted(employees, key=lambda e: e.salary, reverse=True)
print(f"Top paid: {top_paid[0].name}")  # Bob

# Rank by performance (highest first)
top_performer = sorted(employees, key=lambda e: e.performance_score, reverse=True)
print(f"Top performer: {top_performer[0].name}")  # Bob
```


#### Product Sorting

```python
from dataclasses import dataclass

@dataclass(ordering=True)
class Product:
    name: str
    price: float
    rating: float

products = [
    Product("Laptop", 999.99, 4.5),
    Product("Phone", 599.99, 4.8),
    Product("Tablet", 399.99, 4.2),
]

# Sort by price (lowest first)
by_price = sorted(products)
print("By price:", [p.name for p in by_price])

# Sort by rating (highest first)
by_rating = sorted(products, key=lambda p: p.rating, reverse=True)
print("By rating:", [p.name for p in by_rating])
```


### Comparison Generation

Control which comparison methods are generated.

```python
from dataclasses import dataclass

@dataclass(eq=True, order=False)
class Point:
    x: float
    y: float
    
    # Only __eq__ and __ne__ generated
    # No __lt__, __le__, __gt__, __ge__

p1 = Point(1, 2)
p2 = Point(1, 2)

print(p1 == p2)  # True
print(p1 < p2)   # Error: no __lt__
```


#### Parameters

| Parameter | Default | Effect |
| :-- | :-- | :-- |
| `eq` | `True` | Generate `__eq__`, `__ne__` |
| `order` | `False` | Generate `__lt__`, `__le__`, `__gt__`, `__ge__` |
| `unsafe_hash` | `False` | Generate `__hash__` even if not frozen |

```python
from dataclasses import dataclass

@dataclass(eq=True, order=True, unsafe_hash=True)
class Rectangle:
    width: float
    height: float
    
    def area(self):
        return self.width * self.height

r1 = Rectangle(10, 5)
r2 = Rectangle(10, 5)
r3 = Rectangle(20, 5)

print(r1 == r2)  # True
print(r1 < r3)   # True (compares width first)
print(hash(r1))  # Works because unsafe_hash=True
```


### `__post_init__`

Called after `__init__` for validation and transformations.

```python
from dataclasses import dataclass

@dataclass
class User:
    username: str
    email: str
    
    def __post_init__(self):
        # Validation
        if not self.username.isalnum():
            raise ValueError("Username must be alphanumeric")
        
        if "@" not in self.email:
            raise ValueError("Invalid email format")
        
        # Derived attributes
        self.username_lower = self.username.lower()
        self.email_domain = self.email.split("@")[1]

user = User("Alice123", "alice@example.com")
print(user.username_lower)  # alice123
print(user.email_domain)    # example.com
```


#### User Registration

```python
from dataclasses import dataclass
from datetime import datetime
import re

@dataclass
class UserRegistration:
    username: str
    email: str
    password: str
    
    def __post_init__(self):
        # Validate username
        if len(self.username) < 3:
            raise ValueError("Username must be at least 3 characters")
        
        # Validate email
        if not re.match(r"^[\w\.-]+@[\w\.-]+\.\w+", self.email):
            raise ValueError("Invalid email format")
        
        # Validate password
        if len(self.password) < 8:
            raise ValueError("Password must be at least 8 characters")
        
        # Create derived attributes
        self.created_at = datetime.now()
        self.username_lower = self.username.lower()
        self.email_normalized = self.email.lower()

try:
    user = UserRegistration("ab", "bad@email", "short")
except ValueError as e:
    print(f"Validation error: {e}")
```


#### Financial Calculations

```python
from dataclasses import dataclass
from decimal import Decimal

@dataclass
class Order:
    item: str
    quantity: int
    price_per_unit: Decimal
    
    def __post_init__(self):
        # Calculate total
        self.total = self.quantity * self.price_per_unit
        
        # Apply tax
        self.tax_rate = Decimal("0.08")
        self.total_with_tax = self.total * (1 + self.tax_rate)
        
        # Round to 2 decimal places
        self.total = self.total.quantize(Decimal("0.01"))
        self.total_with_tax = self.total_with_tax.quantize(Decimal("0.01"))

order = Order("Laptop", 2, Decimal("999.99"))
print(f"Total: ${order.total}")  # $1999.98
print(f"With tax: ${order.total_with_tax}")  # $2159.98
```


### Dataclass Internals

#### Generated Code Explanation

```python
from dataclasses import dataclass
import inspect

@dataclass
class Point:
    x: float
    y: float

# View generated __init__
print(inspect.getsource(Point.__init__))
```

Generated `__init__`:

```python
def __init__(self, x: float, y: float) -> None:
    self.x = x
    self.y = y
```

Generated `__repr__`:

```python
def __repr__(self) -> str:
    return f'Point(x={self.x!r}, y={self.y!r})'
```

Generated `__eq__`:

```python
def __eq__(self, other) -> bool:
    if not isinstance(other, Point):
        return NotImplemented
    return (self.x, self.y) == (other.x, other.y)
```


#### Comparison with Manual Implementation

| Aspect | Manual Class | Dataclass |
| :-- | :-- | :-- |
| Lines of code | 30-50 | 3-5 |
| Readability | Lower | Higher |
| Type hints | Optional | Integrated |
| Maintenance | Higher effort | Lower effort |
| Custom methods | Easy | Easy |
| Debugging | More complex | Clearer |


***

# __slots__

## Introduction

`__slots__` optimizes memory by preventing the creation of `__dict__` for each instance.

#### Why slots exist

- Python objects normally have a `__dict__` for storing attributes
- `__dict__` uses significant memory (hundreds of bytes per object)
- `__slots__` replaces `__dict__` with a fixed structure
- Saves memory when creating many instances


#### Object Memory Model

```python
# Normal class - has __dict__
class NormalPoint:
    def __init__(self, x, y):
        self.x = x
        self.y = y

point = NormalPoint(1, 2)
print(point.__dict__)  # {'x': 1, 'y': 2} - uses __dict__
```

```python
# With slots - no __dict__
class SlottedPoint:
    __slots__ = ['x', 'y']
    
    def __init__(self, x, y):
        self.x = x
        self.y = y

point = SlottedPoint(1, 2)
print(point.__dict__)  # AttributeError - no __dict__
```


## Memory Optimization

### How `__dict__` works

```python
class NormalClass:
    def __init__(self):
        self.a = 1
        self.b = 2
        self.c = 3

obj = NormalClass()
print(obj.__dict__)  # {'a': 1, 'b': 2, 'c': 3}
print(type(obj.__dict__))  # dict
```

Each instance has its own dictionary (~240 bytes minimum).

### How slots replace dictionaries

```python
class SlottedClass:
    __slots__ = ['a', 'b', 'c']
    
    def __init__(self):
        self.a = 1
        self.b = 2
        self.c = 3

obj = SlottedClass()
# No __dict__ - attributes stored in fixed slots
```

Attributes are stored in a fixed array structure, similar to C structs.

### Memory Illustrations

```
Normal Object:
┌─────────────────┐
│ __dict__ pointer│ → ┌──────────────┐
│ class pointer   │   │ {'a': 1}     │
│ slots (none)    │   │ {'b': 2}     │
└─────────────────┘   │ {'c': 3}     │
                      └──────────────┘
                      ~240+ bytes

Slotted Object:
┌─────────────────┐
│ a: 1 (slot)     │
│ b: 2 (slot)     │
│ c: 3 (slot)     │
│ class pointer   │
└─────────────────┘
~48 bytes (no dict)
```


### Object Layout Diagrams

```
Normal Class Instance:
[Pointer to __dict__] [Pointer to class] [Extra space...]
         ↓
    ┌──────────┐
    │ 'attr':  │
    │  value   │
    └──────────┐
    │ 'attr2': │
    │  value2  │
    └──────────┘

Slotted Class Instance:
[attr1: value] [attr2: value] [Pointer to class]
(Fixed position, no dict)
```


## Benchmarks

### Memory Usage Benchmark

