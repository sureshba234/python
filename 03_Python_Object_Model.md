# 03 — Python Object Model

> **Audience:** Python Full Stack · Backend Development · AI/ML · System Design interviews
> **Depth:** Beginner → Advanced · CPython internals · Interview-ready

---

## Table of Contents

1. [Everything is an Object](#1-everything-is-an-object)
2. [Type Hierarchy](#2-type-hierarchy)
3. [Metatype](#3-metatype)
4. [Object Lifecycle](#4-object-lifecycle)
5. [Attribute Lookup](#5-attribute-lookup)
6. [Method Resolution](#6-method-resolution)
7. [Descriptor Protocol](#7-descriptor-protocol)

---

## 1. Everything is an Object

### Definition

In Python, **every value — no matter how simple or complex — is an object**. Integers, strings, functions, classes, modules, `None`, lambdas, and even types themselves are all first-class objects. There are no primitive types hiding underneath.

### Why It Exists

Python was designed for simplicity and uniformity. By treating everything as an object, Python avoids the dual-world problem (primitives vs objects) that languages like Java have. This uniformity lets you pass functions around, store classes in lists, inspect any value with the same tools, and build highly dynamic programs without special casing.

### How It Works

Every object in CPython is a C struct called `PyObject` at its core:

```c
typedef struct _object {
    Py_ssize_t ob_refcnt;   // Reference count for memory management
    PyTypeObject *ob_type;  // Pointer to the object's type
} PyObject;
```

When you write `x = 42`, Python:

1. Allocates a `PyLongObject` struct in memory.
2. Sets `ob_refcnt = 1`.
3. Sets `ob_type = &PyLong_Type`.
4. Binds the name `x` in the local namespace to that memory address.

`x` is not a box holding 42 — it is a **label pointing to an object**.

### Example 1 — Basic

```python
# Integers are objects
x = 42
print(type(x))        # <class 'int'>
print(id(x))          # memory address, e.g. 140234567890
print(isinstance(x, object))  # True — everything is an object

# Strings are objects
s = "hello"
print(type(s))        # <class 'str'>
print(id(s))          # different memory address

# None is an object
print(type(None))     # <class 'NoneType'>
print(id(None))       # same id every time — None is a singleton
```

### Example 2 — Intermediate (Functions, Classes, Modules as Objects)

```python
import math

# Functions are objects
def greet(name):
    return f"Hello, {name}"

print(type(greet))           # <class 'function'>
print(id(greet))             # memory address
print(isinstance(greet, object))  # True

# You can store functions in variables, lists, dicts
operations = [greet, str.upper, len]
print(operations[0]("Alice"))   # Hello, Alice

# Classes are objects
class Dog:
    pass

print(type(Dog))             # <class 'type'>
print(isinstance(Dog, object))  # True

# Modules are objects
print(type(math))            # <class 'module'>
print(id(math))              # memory address

# Methods are objects too
print(type(greet.__call__))  # <class 'method-wrapper'>
```

### Internal Working

- CPython stores **all Python objects on the heap** (not the stack).
- Small integers (-5 to 256) and interned strings are **cached** (singleton optimization).
- Every name (variable) lives in a **namespace dictionary** (`__dict__` or `locals()`). The name maps to an object's memory address.
- Assigning `y = x` does **not** copy the object — it creates a second reference to the same object.

```python
x = [1, 2, 3]
y = x
print(id(x) == id(y))   # True — same object
y.append(4)
print(x)                 # [1, 2, 3, 4] — mutation is visible via both names
```

### Visualization

```
Variable names (namespace dict)        Objects in memory (heap)
┌────────────────────────────┐         ┌──────────────────────────┐
│  "x"  ──────────────────────────────►│  PyLongObject (42)       │
│                            │         │  ob_refcnt = 2           │
│  "y"  ──────────────────────────────►│  ob_type = &PyLong_Type  │
└────────────────────────────┘         └──────────────────────────┘
```

### Common Mistakes

```python
# ❌ Mistake 1: Thinking = copies the object
a = [1, 2, 3]
b = a           # b points to the SAME list
b.append(99)
print(a)        # [1, 2, 3, 99] — surprise!

# ✅ Fix: Use copy() or slicing for a shallow copy
b = a.copy()

# ❌ Mistake 2: Comparing identity when you mean equality
x = 1000
y = 1000
print(x is y)   # May be False (not cached above 256)
print(x == y)   # True — always use == for value comparison

# ❌ Mistake 3: Assuming functions can't have attributes
def my_func():
    pass

my_func.metadata = "version 1.0"   # Functions ARE objects — this works!
print(my_func.metadata)            # version 1.0
```

### Interview Questions

**Beginner**

**Q1: Is everything in Python really an object? Give three examples.**
A: Yes. An integer like `42` is an instance of `int`. A function `def f(): pass` is an instance of `function`. The class `int` itself is an instance of `type`. All pass `isinstance(x, object)`.

**Q2: What is the difference between `==` and `is`?**
A: `==` compares **value** (calls `__eq__`). `is` compares **identity** (same memory address via `id()`). Use `is` only for singletons like `None`, `True`, `False`.

**Q3: What does `id()` return?**
A: The memory address of the object in CPython. It is unique for any two objects that exist simultaneously but may be reused after an object is destroyed.

**Intermediate**

**Q4: Why does `a = 256; b = 256; a is b` return `True` but `a = 257; b = 257; a is b` may return `False`?**
A: CPython caches small integer objects in the range -5 to 256 at startup. Integers outside this range are freshly allocated each time, so they are different objects in memory.

**Q5: How do you check an object's type and its class hierarchy at once?**
A: Use `type(obj)` for the immediate type and `isinstance(obj, SomeClass)` to check the full inheritance chain.

**Q6: Can a function have attributes? How?**
A: Yes. Functions are objects. You can set attributes directly: `func.tag = "hello"`. Python decorators use this pattern frequently (e.g., `functools.wraps` copies `__name__`, `__doc__` etc.).

**Advanced**

**Q7: Explain how Python variable assignment differs from C variable assignment at the memory level.**
A: In C, `int x = 42` allocates memory on the stack and stores 42 there. `x` refers to the storage location. In Python, `x = 42` creates a `PyLongObject` on the heap and stores a pointer to it in the local namespace dict under the key `"x"`. Reassigning `x = 99` does not modify the existing object — it creates a new one and rebinds the name.

**Q8: What is object interning and when does Python apply it automatically?**
A: Interning means reusing the same object for equal values. CPython interns: small integers (-5 to 256), string literals that look like identifiers (compile-time), and `None`/`True`/`False`. You can manually intern strings with `sys.intern()`. Interning saves memory and makes `is` comparisons safe for those values.

### Key Takeaways

- Every Python value is a `PyObject` with a reference count and a type pointer.
- Variables are names (labels) in namespace dictionaries pointing to objects — not boxes.
- `type()`, `id()`, and `isinstance()` are the primary introspection tools.
- Assignment never copies — it creates an additional reference.
- Functions, classes, and modules are all first-class objects you can pass, store, and inspect.

---

## 2. Type Hierarchy

### Definition

Python's type system is a **single-rooted hierarchy** with `object` at the top. Every class — whether built-in or user-defined — ultimately inherits from `object`. Types form a directed acyclic graph (DAG) of inheritance relationships.

### Why It Exists

A unified hierarchy guarantees that every object has a predictable minimum interface (`__str__`, `__repr__`, `__eq__`, `__hash__`, etc.). It enables polymorphism, duck typing, and the `isinstance`/`issubclass` checks that power Python's runtime type system.

### How It Works

1. `object` is the root — it is the base class of all classes.
2. Built-in types (`int`, `str`, `list`, `dict`, etc.) inherit directly from `object`.
3. User-defined classes that omit a base automatically inherit from `object` (Python 3 default).
4. Multiple inheritance creates a diamond or tree structure above `object`.
5. `type` is both a class and the metaclass (see section 3).

### Example 1 — Basic

```python
# object is the root
print(isinstance(42, object))       # True
print(isinstance("hello", object))  # True
print(isinstance([], object))       # True
print(isinstance(None, object))     # True

# Built-in inheritance
print(int.__bases__)    # (<class 'object'>,)
print(str.__bases__)    # (<class 'object'>,)
print(bool.__bases__)   # (<class 'int'>,) — bool inherits from int!

# Check the chain
print(bool.__mro__)
# (<class 'bool'>, <class 'int'>, <class 'object'>)
```

### Example 2 — Intermediate

```python
class Animal:
    def breathe(self):
        return "breathing"

class Mammal(Animal):
    def feed_young(self):
        return "nursing"

class Dog(Mammal):
    def speak(self):
        return "woof"

d = Dog()

# isinstance checks the entire chain
print(isinstance(d, Dog))     # True
print(isinstance(d, Mammal))  # True
print(isinstance(d, Animal))  # True
print(isinstance(d, object))  # True

# issubclass checks class hierarchy
print(issubclass(Dog, Animal))   # True
print(issubclass(Dog, object))   # True

# MRO (Method Resolution Order)
print(Dog.__mro__)
# (<class 'Dog'>, <class 'Mammal'>, <class 'Animal'>, <class 'object'>)

# object provides default implementations
print(d.__class__)   # <class '__main__.Dog'>
print(repr(d))       # <__main__.Dog object at 0x...>
```

### Internal Working

CPython stores the inheritance chain in `PyTypeObject.tp_bases` (a tuple) and the linearized MRO in `PyTypeObject.tp_mro`. When Python looks up an attribute, it walks `tp_mro` in order. The `object` type provides default slot implementations like `tp_repr`, `tp_str`, and `tp_hash` that all types fall back to.

`bool` is a subclass of `int` — this is why `True + True == 2` works in Python:

```python
print(True + True)   # 2
print(type(True))    # <class 'bool'>
print(True.__class__.__bases__)  # (<class 'int'>,)
```

### Visualization

```
                    ┌──────────┐
                    │  object  │  ← root of all types
                    └────┬─────┘
          ┌──────────────┼──────────────────────┐
          │              │                      │
       ┌──┴──┐        ┌──┴──┐              ┌───┴───┐
       │ int │        │ str │              │ list  │   (built-ins)
       └──┬──┘        └─────┘              └───────┘
          │
       ┌──┴──┐
       │bool │   ← bool inherits from int
       └─────┘

User-defined:
       ┌──────────┐
       │  object  │
       └────┬─────┘
            │
       ┌────┴────┐
       │ Animal  │
       └────┬────┘
            │
       ┌────┴────┐
       │ Mammal  │
       └────┬────┘
            │
       ┌────┴────┐
       │   Dog   │
       └─────────┘
```

### Common Mistakes

```python
# ❌ Mistake 1: Forgetting that bool is a subclass of int
print(isinstance(True, int))   # True — may surprise you
print(True == 1)               # True
print(False == 0)              # True

# ❌ Mistake 2: Checking type equality instead of isinstance
def process(x):
    if type(x) == int:   # Fails for bool, which IS an int
        print("integer")

process(True)  # prints nothing — type(True) is bool, not int

# ✅ Fix
def process(x):
    if isinstance(x, int):   # Correct: includes bool
        print("integer")

process(True)   # integer

# ❌ Mistake 3: Assuming user classes don't inherit from object
class MyClass:
    pass

print(issubclass(MyClass, object))  # True — always, in Python 3
```

### Interview Questions

**Beginner**

**Q1: What is the root class in Python's type hierarchy?**
A: `object`. Every class in Python implicitly or explicitly inherits from `object`.

**Q2: Why does `isinstance(True, int)` return `True`?**
A: Because `bool` is a subclass of `int` (`bool.__bases__ == (int,)`). `isinstance` checks the full inheritance chain, not just the immediate type.

**Q3: What does `__bases__` contain vs `__mro__`?**
A: `__bases__` contains only the direct parent classes. `__mro__` contains the full linearized lookup order including all ancestors up to and including `object`.

**Intermediate**

**Q4: Can you create a class that does NOT inherit from `object` in Python 3?**
A: No. In Python 3, all classes implicitly inherit from `object` even if you write `class Foo:` with no base. In Python 2, "old-style" classes didn't inherit from `object`, but that distinction is gone in Python 3.

**Q5: How would you verify that `str` and `int` are siblings in the hierarchy?**
A: `int.__bases__ == (object,)` and `str.__bases__ == (object,)` — both have `object` as their sole direct parent.

**Q6: What method does `object` provide that every class inherits?**
A: `__repr__`, `__str__`, `__eq__`, `__ne__`, `__hash__`, `__init__`, `__new__`, `__class__`, `__doc__`, `__dir__`, `__delattr__`, `__setattr__`, `__getattribute__`, among others.

**Advanced**

**Q7: How does Python's type hierarchy relate to the Liskov Substitution Principle?**
A: The hierarchy is designed so that subclass instances can be used wherever superclass instances are expected. `isinstance(d, Animal)` being `True` for a `Dog` instance means you can call `animal.breathe()` on it safely. Violating LSP (e.g., overriding a method to raise `NotImplementedError`) breaks polymorphic code.

**Q8: What happens internally when Python evaluates `isinstance(obj, cls)` for a deep hierarchy?**
A: CPython calls `type_is_subtype_base_chain` which walks `obj.__class__.__mro__` and checks if `cls` appears anywhere in that tuple. This is an O(depth) operation in the hierarchy depth.

### Key Takeaways

- `object` is the universal base class — every object is an instance of it.
- `bool` subclasses `int` — this is intentional and has arithmetic implications.
- Use `isinstance()` over `type() ==` to respect subclass relationships.
- `__mro__` gives the complete, linearized lookup order for any class.
- User classes inherit from `object` automatically in Python 3.

---

## 3. Metatype

### Definition

A **metaclass** is the class of a class. Just as a regular class defines how its instances behave, a metaclass defines how classes themselves behave. In Python, the built-in metaclass is `type`. Every class you write — unless you specify otherwise — is an instance of `type`.

### Why It Exists

Metaclasses give you a hook into **class creation itself**. They let you automatically add methods, enforce coding standards, register classes, modify class attributes, or implement ORMs (like Django's Model) — all at class-definition time without touching the class body.

### How It Works

When Python processes a `class` statement, it:

1. Collects the class name, base classes, and namespace (body).
2. Determines the metaclass (default: `type`, or whatever `metaclass=` specifies).
3. Calls `metaclass(name, bases, namespace)` to create the class object.
4. Binds the resulting class object to the class name in the enclosing scope.

```python
# These two are equivalent:
class Foo:
    x = 10

Foo = type("Foo", (), {"x": 10})   # Manual class creation via type()
```

### Example 1 — Basic

```python
# type is the metaclass of all built-in types
print(type(int))    # <class 'type'>
print(type(str))    # <class 'type'>
print(type(list))   # <class 'type'>
print(type(type))   # <class 'type'>  ← type is its own metaclass!

# User-defined classes are also instances of type
class Dog:
    pass

print(type(Dog))            # <class 'type'>
print(isinstance(Dog, type))  # True

# Creating a class dynamically using type()
Cat = type("Cat", (object,), {"speak": lambda self: "meow"})
c = Cat()
print(c.speak())    # meow
print(type(Cat))    # <class 'type'>
```

### Example 2 — Intermediate (Custom Metaclass)

```python
# A metaclass that auto-uppercases all method names
class UpperMeta(type):
    def __new__(mcs, name, bases, namespace):
        upper_ns = {}
        for key, value in namespace.items():
            if not key.startswith("__"):
                upper_ns[key.upper()] = value
            else:
                upper_ns[key] = value
        return super().__new__(mcs, name, bases, upper_ns)

class MyClass(metaclass=UpperMeta):
    def hello(self):
        return "hi"

obj = MyClass()
print(obj.HELLO())    # hi
# print(obj.hello())  # AttributeError — renamed to HELLO

# Singleton metaclass (classic use case)
class SingletonMeta(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Config(metaclass=SingletonMeta):
    def __init__(self):
        self.debug = False

a = Config()
b = Config()
print(a is b)   # True — same instance every time
```

### Internal Working

`type` is implemented in C as `PyType_Type`. When `type.__call__(metaclass, name, bases, ns)` is invoked during class creation:

1. `type.__new__` allocates a new `PyTypeObject` struct.
2. It computes the MRO via C3 linearization.
3. It copies slots and descriptors from bases.
4. It calls `type.__init__` to finalize.

The circular relationship — `type` is an instance of itself (`type(type) is type`) and `type` is a subclass of `object` (`issubclass(type, object)`) — is hardwired at C level during CPython startup, bootstrapping the type system.

### Visualization

```
┌──────────────────────────────────────────────────────┐
│                  METACLASS LAYER                     │
│                                                      │
│   type ◄──────── type's own metaclass is type       │
│     │                                                │
│     │  (is metaclass of)                             │
│     ▼                                                │
│   int    str    list    dict    MyClass   ...        │
│    │                               │                 │
│    │  (is class of / type of)      │                 │
│    ▼                               ▼                 │
│   42    "hi"    [1,2]           Dog()   ...          │
│                  INSTANCE LAYER                      │
└──────────────────────────────────────────────────────┘

Inheritance (IS-A) hierarchy:
type ──inherits──► object
int  ──inherits──► object

type(type)   → type   (type is its own metaclass)
type(int)    → type   (int is an instance of type)
type(42)     → int    (42 is an instance of int)
```

### Common Mistakes

```python
# ❌ Mistake 1: Confusing type() as a function vs as a metaclass
print(type(42))           # <class 'int'>  — called with 1 arg: returns type of obj
MyClass = type("X", (), {})  # called with 3 args: creates a new class
# These are different modes of the same built-in

# ❌ Mistake 2: Forgetting metaclass conflicts in multiple inheritance
class Meta1(type): pass
class Meta2(type): pass

class A(metaclass=Meta1): pass
class B(metaclass=Meta2): pass

# class C(A, B): pass   # TypeError: metaclass conflict
# Fix: create a combined metaclass
class CombinedMeta(Meta1, Meta2): pass
class C(A, B, metaclass=CombinedMeta): pass   # works

# ❌ Mistake 3: Using metaclasses when __init_subclass__ or class decorators suffice
# Metaclasses are powerful but complex — prefer simpler hooks for most tasks
class Base:
    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        print(f"New subclass registered: {cls.__name__}")

class Child(Base): pass   # prints: New subclass registered: Child
```

### Interview Questions

**Beginner**

**Q1: What is `type` in Python?**
A: `type` is both a built-in function (returns the type of an object) and a built-in metaclass (the class of all classes). When called with one argument it returns the type; with three arguments `(name, bases, dict)` it creates a new class.

**Q2: What does `type(type)` return and why?**
A: `type`. `type` is its own metaclass — it is the class of all classes, including itself. This circular relationship is bootstrapped at the C level when CPython starts up.

**Q3: What is a metaclass in plain terms?**
A: A metaclass is a class whose instances are classes. Just as `int` creates integers, `type` (the default metaclass) creates class objects.

**Intermediate**

**Q4: What are the three hooks you can override in a metaclass?**
A: `__new__` (called to allocate the class object), `__init__` (called to initialize it after creation), and `__call__` (called when the class is used to create instances — useful for singletons).

**Q5: How does Django's ORM use metaclasses?**
A: `django.db.models.base.ModelBase` is a metaclass. When you define `class MyModel(models.Model)`, the metaclass inspects the class body, collects `Field` descriptors, registers the model in a global registry, and sets up the database mapping — all automatically.

**Q6: How would you create a class at runtime without using the `class` keyword?**
A: `MyClass = type("MyClass", (BaseClass,), {"attr": value, "method": func})`. This is exactly what Python's `class` statement does internally.

**Advanced**

**Q7: Explain the metaclass resolution algorithm when a class has multiple bases with different metaclasses.**
A: Python computes the "winner" metaclass: it must be a subclass of all base metaclasses. If no single metaclass satisfies this, Python raises `TypeError: metaclass conflict`. You resolve it by creating an explicit metaclass that inherits from all conflicting metaclasses.

**Q8: How would you use a metaclass to enforce that all subclasses implement a specific method (without ABCs)?**
A:
```python
class EnforceMeta(type):
    def __init__(cls, name, bases, namespace):
        super().__init__(name, bases, namespace)
        if bases:  # skip the base class itself
            if "process" not in namespace:
                raise TypeError(f"{name} must implement process()")

class Base(metaclass=EnforceMeta): pass
class Child(Base):
    def process(self): pass      # OK
# class Bad(Base): pass          # TypeError
```

### Key Takeaways

- `type` is the metaclass of all classes — including itself.
- Every class is an instance of `type` (or a custom metaclass).
- `type(name, bases, dict)` creates a class dynamically — exactly what `class` does.
- Metaclasses intercept class creation via `__new__`, `__init__`, `__call__`.
- Prefer `__init_subclass__` or class decorators before reaching for metaclasses.

---

## 4. Object Lifecycle

### Definition

Every Python object goes through a lifecycle: **creation → initialization → usage → reference counting → garbage collection → destruction**. Python manages memory automatically using reference counting with a cyclic garbage collector as a fallback.

### Why It Exists

Automatic memory management frees developers from manual `malloc`/`free`. Reference counting provides immediate, deterministic deallocation for most objects. The cyclic GC handles the rare but real case of reference cycles that reference counting alone cannot break.

### How It Works

**Step 1 — Allocation (`__new__`)**
`type.__call__` invokes `cls.__new__(cls, ...)` which allocates memory for the object and returns the new (uninitialized) instance.

**Step 2 — Initialization (`__init__`)**
`cls.__init__(instance, ...)` initializes the instance's attributes. It receives the already-allocated object.

**Step 3 — Usage**
The object is used. Each time a new reference is created (assignment, argument passing, container insertion), `ob_refcnt` is incremented via `Py_INCREF`. Each time a reference is dropped, `ob_refcnt` is decremented via `Py_DECREF`.

**Step 4 — Destruction (`__del__`)**
When `ob_refcnt` reaches zero, CPython calls `tp_dealloc` which calls `__del__` if defined, then frees the memory.

### Example 1 — Basic

```python
import sys

class MyObj:
    def __new__(cls, *args, **kwargs):
        print(f"__new__ called — allocating {cls.__name__}")
        instance = super().__new__(cls)
        return instance

    def __init__(self, name):
        print(f"__init__ called — initializing with name={name}")
        self.name = name

    def __del__(self):
        print(f"__del__ called — destroying {self.name}")

obj = MyObj("Alice")
# Output:
# __new__ called — allocating MyObj
# __init__ called — initializing with name=Alice

print(sys.getrefcount(obj))   # 2 (obj + temporary in getrefcount)

del obj
# Output:
# __del__ called — destroying Alice
```

### Example 2 — Intermediate (Reference Counting & Cycles)

```python
import sys
import gc

# Reference counting demo
a = [1, 2, 3]
print(sys.getrefcount(a))   # 2 (a + getrefcount argument)

b = a
print(sys.getrefcount(a))   # 3 (a + b + getrefcount argument)

del b
print(sys.getrefcount(a))   # 2 again

# Reference cycles — reference counting alone can't handle this
class Node:
    def __init__(self, val):
        self.val = val
        self.next = None

n1 = Node(1)
n2 = Node(2)
n1.next = n2
n2.next = n1   # cycle!

del n1
del n2
# n1 and n2 still have refcount > 0 due to the cycle
# The cyclic GC will collect them eventually

gc.collect()   # Force a collection cycle
print("Cycle collected")

# Checking GC generations
print(gc.get_count())      # (gen0, gen1, gen2) counts
print(gc.get_threshold())  # thresholds for promoting to next generation
```

### Internal Working

**Reference counting** is the primary mechanism:

```
obj created  →  ob_refcnt = 1
b = obj      →  ob_refcnt = 2
func(obj)    →  ob_refcnt = 3 (during call)
return       →  ob_refcnt = 2
del b        →  ob_refcnt = 1
del obj      →  ob_refcnt = 0  →  tp_dealloc called → memory freed
```

**Cyclic GC** — CPython's `gc` module uses a **tri-generational collector**:
- Gen 0: new objects — collected most frequently.
- Gen 1: survived one Gen 0 collection.
- Gen 2: long-lived objects — collected least frequently.

The GC finds cycles by computing the set of objects whose reference count would drop to zero if all intra-cycle references were subtracted.

**`__new__` vs `__init__`:**
- `__new__` returns the instance — it can return an instance of a different class (used in singletons and immutables like `int`/`str`).
- `__init__` only initializes — it must return `None`.

```python
# __new__ for immutable customization
class PositiveInt(int):
    def __new__(cls, value):
        if value <= 0:
            raise ValueError("Must be positive")
        return super().__new__(cls, value)

n = PositiveInt(5)
print(n)             # 5
print(type(n))       # <class 'PositiveInt'>
# PositiveInt(-1)    # ValueError
```

### Visualization

```
Object Lifecycle
────────────────

  type.__call__(cls, *args)
          │
          ▼
  cls.__new__(cls, *args)       ← allocates memory, returns instance
          │
          ▼
  cls.__init__(instance, *args) ← initializes attributes
          │
          ▼
  [USAGE PHASE]
  ┌─────────────────────────────────┐
  │  ob_refcnt incremented on:      │
  │    - assignment (x = obj)       │
  │    - function argument          │
  │    - container append           │
  │  ob_refcnt decremented on:      │
  │    - del x                      │
  │    - end of scope               │
  │    - reassignment (x = other)   │
  └─────────────────────────────────┘
          │
          ▼  ob_refcnt == 0 ?
  cls.__del__(instance)          ← cleanup (not guaranteed timing!)
          │
          ▼
  tp_dealloc                     ← memory returned to allocator


CPython Generational GC
───────────────────────
Gen 0 ──[survived]──► Gen 1 ──[survived]──► Gen 2
(young)                                    (old)
   ▲                                          │
   │ new objects                    GC sweeps periodically
```

### Common Mistakes

```python
# ❌ Mistake 1: Relying on __del__ for critical cleanup
class FileWrapper:
    def __init__(self, path):
        self.f = open(path, "w")

    def __del__(self):
        self.f.close()   # NOT guaranteed to run promptly!

# ✅ Fix: Use context managers
class FileWrapper:
    def __init__(self, path):
        self.f = open(path, "w")

    def __enter__(self):
        return self

    def __exit__(self, *args):
        self.f.close()

with FileWrapper("/tmp/test.txt") as fw:
    fw.f.write("data")   # guaranteed close

# ❌ Mistake 2: Creating accidental reference cycles
class Parent:
    def __init__(self):
        self.child = Child(self)   # child holds reference to parent

class Child:
    def __init__(self, parent):
        self.parent = parent       # cycle!

# Use weakref to break cycles
import weakref

class Child:
    def __init__(self, parent):
        self.parent = weakref.ref(parent)   # weak reference — doesn't bump refcount

# ❌ Mistake 3: Thinking del destroys the object immediately
a = [1, 2, 3]
b = a
del a          # only removes name 'a'; object still alive because b references it
print(b)       # [1, 2, 3] — object is fine
```

### Interview Questions

**Beginner**

**Q1: What is the difference between `__new__` and `__init__`?**
A: `__new__` allocates and returns the object instance. `__init__` initializes the already-created instance. `__new__` runs first; if it returns an instance of `cls`, Python then calls `__init__` on it.

**Q2: What does `del x` do in Python?**
A: It removes the name `x` from the namespace and decrements the reference count of the object it pointed to. If the reference count reaches zero, the object is deallocated. It does not necessarily destroy the object immediately if other references exist.

**Q3: What is reference counting?**
A: Every Python object maintains a count of how many references point to it (`ob_refcnt`). The count is incremented when a new reference is created and decremented when one is dropped. When it reaches zero, the object is immediately deallocated.

**Intermediate**

**Q4: Why does Python have a cyclic garbage collector in addition to reference counting?**
A: Reference counting cannot handle circular references (A→B→A). In such cycles, the reference count never reaches zero even after all external references are removed. The cyclic GC detects and breaks these cycles.

**Q5: What are the three GC generations and why are there three?**
A: Gen 0 (youngest), Gen 1, Gen 2 (oldest). The generational hypothesis states that most objects die young. Collecting Gen 0 frequently is cheap. Long-lived objects are promoted and collected rarely, reducing GC overhead.

**Q6: When would you override `__new__` instead of `__init__`?**
A: When subclassing immutable types (`int`, `str`, `tuple`, `frozenset`) whose value is set at allocation time, or to implement singletons (returning an existing instance instead of allocating a new one).

**Advanced**

**Q7: How would you implement a singleton using `__new__`?**
A:
```python
class Singleton:
    _instance = None

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

a = Singleton()
b = Singleton()
print(a is b)   # True
```

**Q8: What happens to `__del__` during interpreter shutdown?**
A: During CPython shutdown, global variables are set to `None` before `__del__` is called. This means inside `__del__`, names you expect to be available may be `None`. This is a known footgun — never rely on globals in `__del__`, and always prefer context managers for resource cleanup.

### Key Takeaways

- Object lifecycle: `__new__` → `__init__` → usage → `__del__` → deallocation.
- Reference counting is immediate; cyclic GC is periodic and handles cycles.
- `del x` removes a name binding, not necessarily the object.
- Prefer `with` statements and `__exit__` over `__del__` for resource cleanup.
- Use `weakref` to reference objects without preventing their collection.

---

## 5. Attribute Lookup

### Definition

**Attribute lookup** is the process Python follows when you write `obj.attr` — searching through a well-defined sequence of locations to find the value bound to that name. Python checks the instance, then the class, then each parent class in MRO order.

### Why It Exists

The lookup protocol enables inheritance, descriptors, class attributes, and instance attributes to coexist cleanly. The ordered search means subclasses can override parent attributes, and descriptors (like `property`) can intercept access transparently.

### How It Works

Python calls `object.__getattribute__(obj, "attr")` which follows this search order:

1. **Data descriptors** in `type(obj).__mro__` (descriptor with `__get__` AND `__set__` or `__delete__`).
2. **Instance `__dict__`** (`obj.__dict__["attr"]`).
3. **Non-data descriptors** and other class attributes in `type(obj).__mro__`.

If none found → `__getattr__` is called (if defined), otherwise `AttributeError`.

```
Priority: data descriptor > instance dict > non-data descriptor/class attr
```

### Example 1 — Basic

```python
class Person:
    species = "Homo sapiens"     # class attribute

    def __init__(self, name):
        self.name = name         # instance attribute

p = Person("Alice")

# Instance attribute lookup
print(p.name)          # Alice  (from p.__dict__)
print(p.__dict__)      # {'name': 'Alice'}

# Class attribute lookup (not in instance dict — goes to class)
print(p.species)       # Homo sapiens
print(Person.species)  # Homo sapiens

# getattr / hasattr / setattr
print(getattr(p, "name"))           # Alice
print(getattr(p, "age", "unknown")) # unknown (default if missing)
print(hasattr(p, "name"))           # True
print(hasattr(p, "age"))            # False

setattr(p, "age", 30)
print(p.age)    # 30
print(p.__dict__)  # {'name': 'Alice', 'age': 30}
```

### Example 2 — Intermediate (Lookup Priority)

```python
class MyDescriptor:
    def __get__(self, obj, objtype=None):
        return "descriptor value"

    def __set__(self, obj, value):
        print(f"Setting blocked by descriptor: {value}")

class MyClass:
    attr = MyDescriptor()   # data descriptor (has __get__ and __set__)

obj = MyClass()

# Try to set instance attribute directly
obj.__dict__["attr"] = "instance value"
print(obj.__dict__)     # {'attr': 'instance value'}
print(obj.attr)         # "descriptor value" — data descriptor wins over __dict__!

# Non-data descriptor (only __get__) — instance dict WINS
class NonDataDesc:
    def __get__(self, obj, objtype=None):
        return "non-data descriptor"

class MyClass2:
    attr = NonDataDesc()

obj2 = MyClass2()
obj2.__dict__["attr"] = "instance wins"
print(obj2.attr)   # "instance wins" — instance dict beats non-data descriptor

# __getattr__ as fallback
class Flexible:
    def __getattr__(self, name):
        return f"No attribute '{name}' — generated dynamically"

f = Flexible()
print(f.anything)   # No attribute 'anything' — generated dynamically
```

### Internal Working

`object.__getattribute__` in CPython (simplified):

```python
def __getattribute__(self, name):
    # 1. Look in type's MRO for a data descriptor
    for base in type(self).__mro__:
        if name in base.__dict__:
            attr = base.__dict__[name]
            if hasattr(type(attr), "__get__") and (
               hasattr(type(attr), "__set__") or hasattr(type(attr), "__delete__")):
                return type(attr).__get__(attr, self, type(self))  # data descriptor wins

    # 2. Look in instance dict
    if name in self.__dict__:
        return self.__dict__[name]

    # 3. Look in type's MRO for non-data descriptor or plain class attr
    for base in type(self).__mro__:
        if name in base.__dict__:
            attr = base.__dict__[name]
            if hasattr(type(attr), "__get__"):
                return type(attr).__get__(attr, self, type(self))  # non-data descriptor
            return attr  # plain class attribute

    raise AttributeError(name)
```

### Visualization

```
obj.attr  →  object.__getattribute__(obj, "attr")
                          │
             ┌────────────┴───────────────────────┐
             │  Search type(obj).__mro__ for       │
             │  DATA DESCRIPTOR (has __get__+__set__)│
             └────────────┬───────────────────────┘
                    Found?│
                    YES ──┴──► Return descriptor.__get__(obj, type(obj))
                    NO
                          │
             ┌────────────┴───────────────────────┐
             │  Search obj.__dict__                │
             └────────────┬───────────────────────┘
                    Found?│
                    YES ──┴──► Return obj.__dict__[attr]
                    NO
                          │
             ┌────────────┴───────────────────────┐
             │  Search type(obj).__mro__ for       │
             │  NON-DATA DESCRIPTOR or class attr  │
             └────────────┬───────────────────────┘
                    Found?│
                    YES ──┴──► Return it (invoke __get__ if descriptor)
                    NO
                          │
             ┌────────────┴──────────┐
             │  Call obj.__getattr__ │ (if defined)
             └────────────┬──────────┘
                    Raises AttributeError if not found
```

### Common Mistakes

```python
# ❌ Mistake 1: Shadowing a class attribute with an instance attribute unintentionally
class Counter:
    count = 0

    def increment(self):
        self.count += 1   # WRONG: creates instance attribute, doesn't touch class attr

c1 = Counter()
c2 = Counter()
c1.increment()
print(Counter.count)  # 0 — class attribute unchanged!
print(c1.count)       # 1 — only c1's instance dict

# ✅ Fix: modify the class attribute explicitly
class Counter:
    count = 0

    def increment(self):
        Counter.count += 1

# ❌ Mistake 2: Confusing __getattr__ and __getattribute__
class Bad:
    def __getattribute__(self, name):
        return super().__getattribute__(name)   # always call super!

    # __getattr__ is ONLY called when normal lookup fails
    # __getattribute__ is called for EVERY attribute access

# ❌ Mistake 3: Infinite recursion in __getattribute__
class Infinite:
    def __getattribute__(self, name):
        return self.name   # self.name calls __getattribute__ again → RecursionError!

    # ✅ Fix: use object.__getattribute__ to avoid recursion
    def __getattribute__(self, name):
        return object.__getattribute__(self, name)
```

### Interview Questions

**Beginner**

**Q1: What is the attribute lookup order in Python?**
A: Data descriptors (in class/MRO) → instance `__dict__` → non-data descriptors and class attributes (in MRO) → `__getattr__` fallback.

**Q2: What is the difference between `getattr(obj, "x")` and `obj.x`?**
A: Functionally identical — `obj.x` desugars to `type(obj).__getattribute__(obj, "x")`. `getattr` additionally accepts a default value: `getattr(obj, "x", default)` returns `default` instead of raising `AttributeError`.

**Q3: What does `setattr(obj, "name", value)` do?**
A: It calls `type(obj).__setattr__(obj, "name", value)`, which normally stores `value` in `obj.__dict__["name"]`. If a data descriptor with `__set__` exists, it is invoked instead.

**Intermediate**

**Q4: What is the difference between `__getattr__` and `__getattribute__`?**
A: `__getattribute__` is called on **every** attribute access. `__getattr__` is called only when `__getattribute__` raises `AttributeError` (i.e., attribute not found through normal channels). Use `__getattr__` for dynamic fallback attributes; override `__getattribute__` rarely and carefully.

**Q5: Why does a data descriptor take priority over the instance `__dict__`?**
A: By design. Data descriptors (like `property`) are meant to intercept both getting and setting, providing controlled access. If instance `__dict__` had priority, users could silently bypass the descriptor by direct dict assignment.

**Q6: How would you make an object's attribute read-only?**
A:
```python
class ReadOnly:
    def __init__(self, value):
        object.__setattr__(self, "_value", value)

    @property
    def value(self):
        return self._value

    # property without a setter is a read-only data descriptor
```

**Advanced**

**Q7: Explain how `__slots__` affects attribute lookup.**
A: `__slots__ = ["x", "y"]` causes Python to allocate fixed memory slots instead of a per-instance `__dict__`. Each slot becomes a data descriptor on the class. This speeds up attribute access, reduces memory usage, and prevents creation of arbitrary instance attributes. Classes with `__slots__` have no `__dict__` unless `"__dict__"` is listed in slots explicitly.

**Q8: How does Python handle attribute lookup across a multiple-inheritance diamond?**
A: It uses the MRO (C3 linearization). All bases in the MRO are searched in order. The first match (data descriptor, then instance dict, then class attr) wins. The diamond is resolved because each class appears exactly once in the MRO — avoiding the "deadly diamond" problem.

### Key Takeaways

- Lookup order: data descriptor → `__dict__` → non-data descriptor → `__getattr__`.
- `__getattr__` is a fallback; `__getattribute__` intercepts everything.
- Data descriptors (with `__set__`) beat instance dictionaries.
- `__slots__` replaces `__dict__` with fixed C-level slots for efficiency.
- Always call `super().__getattribute__` or `object.__getattribute__` to avoid recursion.

---

## 6. Method Resolution

### Definition

**Method Resolution Order (MRO)** is the deterministic sequence in which Python searches base classes to find a method or attribute. Python uses the **C3 Linearization algorithm** to compute a consistent, predictable MRO for any class hierarchy — including complex multiple inheritance.

### Why It Exists

Without a consistent lookup order, multiple inheritance creates ambiguity. Which parent's `method()` is called in `class C(A, B)`? C3 linearization provides a mathematically proven algorithm that is monotonic (preserves local ordering) and consistent (respects the order bases are listed).

### How It Works

For `class C(A, B)`, C3 linearization computes:

```
L(C) = C + merge(L(A), L(B), [A, B])
```

**Merge rule:** at each step, take the head of the first list if it does not appear in the tail of any other list. Remove it from all lists and add it to the result.

`super()` in Python 3 uses the MRO to call the next class in the chain — not necessarily the parent.

### Example 1 — Basic

```python
class A:
    def hello(self):
        return "A"

class B(A):
    def hello(self):
        return "B"

class C(A):
    def hello(self):
        return "C"

class D(B, C):
    pass

d = D()
print(d.hello())          # B (first in MRO that has hello)
print(D.__mro__)
# (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)
print(D.mro())            # same as list(D.__mro__)
```

### Example 2 — Intermediate (super() in cooperative multiple inheritance)

```python
class Base:
    def process(self):
        print("Base.process")

class Mixin1(Base):
    def process(self):
        print("Mixin1.process - before")
        super().process()
        print("Mixin1.process - after")

class Mixin2(Base):
    def process(self):
        print("Mixin2.process - before")
        super().process()
        print("Mixin2.process - after")

class MyClass(Mixin1, Mixin2):
    def process(self):
        print("MyClass.process")
        super().process()

obj = MyClass()
obj.process()
# MyClass.process
# Mixin1.process - before
# Mixin2.process - before
# Base.process
# Mixin2.process - after
# Mixin1.process - after

print(MyClass.__mro__)
# (MyClass, Mixin1, Mixin2, Base, object)
```

### Internal Working

**C3 Linearization example — `class D(B, C)` where `B(A)` and `C(A)`:**

```
L(object) = [object]
L(A)      = [A, object]
L(B)      = [B] + merge([A, object], [object])
          = [B, A, object]
L(C)      = [C, A, object]
L(D)      = [D] + merge([B, A, object], [C, A, object], [B, C])

Step 1: head = B, not in any tail → take B
        → [D, B], merge([A, object], [C, A, object], [C])
Step 2: head = A, is in tail of [C, A, object] → skip
        head = C, not in any tail → take C
        → [D, B, C], merge([A, object], [A, object], [])
Step 3: head = A → take A
        → [D, B, C, A, object]
```

**`super()` mechanics in Python 3:**
- `super()` (no arguments, Python 3) uses `__class__` cell variable and the first argument of the enclosing method.
- It returns a proxy that searches the MRO **after** the current class.

```python
class A:
    def greet(self):
        return "A"

class B(A):
    def greet(self):
        return "B → " + super().greet()  # super() finds A in MRO after B

print(B().greet())   # B → A
```

### Visualization

```
Diamond Problem:
                ┌──────────┐
                │  object  │
                └────┬─────┘
          ┌──────────┘└──────────┐
          │                      │
       ┌──┴──┐               ┌───┴──┐
       │  A  │               │  A   │  (same A)
       └──┬──┘               └───┬──┘
          │                      │
       ┌──┴──┐               ┌───┴──┐
       │  B  │               │  C   │
       └──┬──┘               └───┬──┘
          └──────────┐ ┌─────────┘
                  ┌──┴─┴──┐
                  │   D   │
                  └───────┘

C3 MRO for D:  D → B → C → A → object
               (each class appears exactly ONCE)

super() call chain for MyClass(Mixin1, Mixin2):
MRO: MyClass → Mixin1 → Mixin2 → Base → object

MyClass.process()
    └─ super() → Mixin1.process()
           └─ super() → Mixin2.process()
                   └─ super() → Base.process()
```

### Common Mistakes

```python
# ❌ Mistake 1: Calling parent method directly instead of super()
class A:
    def method(self):
        print("A")

class B(A):
    def method(self):
        A.method(self)   # Hardcodes A — breaks cooperative inheritance

# ✅ Fix
class B(A):
    def method(self):
        super().method()  # Uses MRO — cooperative and flexible

# ❌ Mistake 2: Inconsistent MRO (Python will raise TypeError)
class X: pass
class Y: pass
class A(X, Y): pass
class B(Y, X): pass
# class C(A, B): pass   # TypeError: Cannot create a consistent MRO
# Because A says X before Y, B says Y before X — contradiction

# ❌ Mistake 3: Forgetting super() in __init__ with multiple inheritance
class Animal:
    def __init__(self, name):
        self.name = name

class Flyable:
    def __init__(self, speed):
        self.speed = speed

class FlyingAnimal(Animal, Flyable):
    def __init__(self, name, speed):
        Animal.__init__(self, name)    # Flyable.__init__ never called!
        # ✅ Fix: use super() and keyword arguments for cooperative init
        # This requires all classes in the chain to cooperate

# ✅ Cooperative __init__
class Animal:
    def __init__(self, name, **kwargs):
        super().__init__(**kwargs)
        self.name = name

class Flyable:
    def __init__(self, speed, **kwargs):
        super().__init__(**kwargs)
        self.speed = speed

class FlyingAnimal(Animal, Flyable):
    def __init__(self, name, speed):
        super().__init__(name=name, speed=speed)

fa = FlyingAnimal("Eagle", 200)
print(fa.name, fa.speed)   # Eagle 200
```

### Interview Questions

**Beginner**

**Q1: What is MRO and why is it important?**
A: Method Resolution Order is the sequence Python follows to look up methods/attributes in a class hierarchy. It ensures predictability in single and multiple inheritance by defining exactly which class's method is called.

**Q2: How do you check the MRO of a class?**
A: `MyClass.__mro__` returns a tuple; `MyClass.mro()` returns a list. Both show the complete lookup order from the class itself up through all bases to `object`.

**Q3: What does `super()` do?**
A: It returns a proxy object that delegates method calls to the next class in the MRO of the current class. In Python 3 with no arguments, it automatically detects the enclosing class and instance.

**Intermediate**

**Q4: What is the C3 linearization algorithm?**
A: An algorithm that computes a consistent MRO for any class hierarchy by taking the merge of parent MROs, selecting the head of the first list that doesn't appear in the tail of any other list at each step. It ensures monotonicity and local precedence order.

**Q5: When would Python raise a `TypeError` related to MRO?**
A: When the requested inheritance creates an inconsistent MRO — e.g., two parents disagree on the relative order of a shared ancestor. Python raises `TypeError: Cannot create a consistent method resolution order`.

**Q6: What is cooperative multiple inheritance?**
A: A pattern where every class in a hierarchy uses `super()` and accepts `**kwargs`, forwarding any unrecognized arguments up the chain. This ensures all `__init__` methods are called even with complex multiple inheritance.

**Advanced**

**Q7: Explain how `super()` works without arguments in Python 3 at the bytecode level.**
A: Python 3 compiles `super()` using `__class__` — a cell variable implicitly created by the compiler for any method that uses `super()`. It refers to the class in which the method is defined. Combined with the first local variable (the instance), the compiler generates `LOAD_GLOBAL super` and `LOAD_DEREF __class__` instructions, so no arguments are needed.

**Q8: How would you resolve a metaclass conflict in multiple inheritance?**
A: Create a combined metaclass that inherits from all conflicting metaclasses, and explicitly pass it as the metaclass:
```python
class Meta1(type): pass
class Meta2(type): pass
class CombinedMeta(Meta1, Meta2): pass
class MyClass(ClassWithMeta1, ClassWithMeta2, metaclass=CombinedMeta): pass
```

### Key Takeaways

- MRO defines the deterministic method lookup order using C3 linearization.
- `super()` delegates to the **next class in MRO**, not just the parent.
- Inconsistent inheritance orders raise `TypeError` at class definition time.
- Cooperative inheritance requires every class to use `super()` and `**kwargs`.
- Check MRO with `ClassName.__mro__` or `ClassName.mro()`.

---

## 7. Descriptor Protocol

### Definition

A **descriptor** is any object that defines `__get__`, `__set__`, or `__delete__`. When a descriptor is stored as a class attribute, Python invokes these methods instead of normal attribute access. Descriptors power `property`, `staticmethod`, `classmethod`, and virtually all of Python's attribute machinery.

### Why It Exists

Descriptors allow you to attach behavior to attribute access and assignment without changing the client code. They are the mechanism behind computed properties, validation, type coercion, lazy loading, and the entire function/method binding system.

### How It Works

- **Data descriptor**: defines `__get__` AND (`__set__` or `__delete__`). Takes priority over `instance.__dict__`.
- **Non-data descriptor**: defines only `__get__`. Instance `__dict__` takes priority.

When Python evaluates `obj.attr`:

1. `type(obj).__getattribute__` checks if `type(obj).__dict__["attr"]` (or in its MRO) is a descriptor.
2. If data descriptor → calls `descriptor.__get__(obj, type(obj))`.
3. Otherwise checks `obj.__dict__`.
4. Otherwise calls `__get__` for non-data descriptors.

### Example 1 — Basic (`property`)

```python
class Temperature:
    def __init__(self, celsius=0):
        self._celsius = celsius  # note: _celsius, not celsius, to avoid recursion

    @property
    def celsius(self):
        return self._celsius

    @celsius.setter
    def celsius(self, value):
        if value < -273.15:
            raise ValueError("Temperature below absolute zero!")
        self._celsius = value

    @property
    def fahrenheit(self):
        return self._celsius * 9/5 + 32

t = Temperature(25)
print(t.celsius)     # 25    — calls __get__
print(t.fahrenheit)  # 77.0
t.celsius = 100      # calls __set__ (setter)
print(t.celsius)     # 100

# t.celsius = -300   # ValueError

# property is a data descriptor
print(type(Temperature.__dict__["celsius"]))  # <class 'property'>
print(hasattr(property, "__get__"))   # True
print(hasattr(property, "__set__"))   # True — makes it a data descriptor
```

### Example 2 — Intermediate (Custom Descriptor + staticmethod/classmethod)

```python
# Custom validator descriptor
class Typed:
    def __init__(self, name, expected_type):
        self.name = name
        self.expected_type = expected_type

    def __set_name__(self, owner, name):
        self.name = name        # auto-capture attribute name (Python 3.6+)

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self         # accessed from class — return descriptor itself
        return obj.__dict__.get(self.name)

    def __set__(self, obj, value):
        if not isinstance(value, self.expected_type):
            raise TypeError(
                f"{self.name} must be {self.expected_type.__name__}, "
                f"got {type(value).__name__}"
            )
        obj.__dict__[self.name] = value

class Person:
    name = Typed("name", str)
    age  = Typed("age", int)

    def __init__(self, name, age):
        self.name = name   # triggers Typed.__set__
        self.age  = age

p = Person("Alice", 30)
print(p.name)    # Alice
print(p.age)     # 30

# p.age = "thirty"    # TypeError: age must be int, got str

# staticmethod and classmethod are descriptors
class MyClass:
    @staticmethod
    def static_greet():
        return "static"

    @classmethod
    def class_greet(cls):
        return f"class: {cls.__name__}"

print(type(MyClass.__dict__["static_greet"]))  # <class 'staticmethod'>
print(type(MyClass.__dict__["class_greet"]))   # <class 'classmethod'>

# Both are non-data descriptors — they only define __get__
print(MyClass.static_greet())    # static
print(MyClass.class_greet())     # class: MyClass
obj = MyClass()
print(obj.static_greet())        # static — __get__ strips obj
print(obj.class_greet())         # class: MyClass — __get__ passes cls
```

### Internal Working

**How functions become bound methods — the descriptor protocol in action:**

```python
class Dog:
    def bark(self):
        return "woof"

d = Dog()

# bark is stored as a plain function in Dog.__dict__
print(type(Dog.__dict__["bark"]))    # <class 'function'>

# Functions define __get__ — they are non-data descriptors
print(hasattr(Dog.__dict__["bark"], "__get__"))  # True

# When accessed via instance, __get__ is called, binding self
method = Dog.__dict__["bark"].__get__(d, Dog)
print(method)         # <bound method Dog.bark of <__main__.Dog object>>
print(method())       # woof

# Equivalent to:
print(d.bark())       # woof — Python calls __get__ automatically
```

**`property` implementation sketch:**

```python
class property:
    def __init__(self, fget=None, fset=None, fdel=None):
        self.fget = fget
        self.fset = fset
        self.fdel = fdel

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self          # class access — return the property itself
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
        return type(self)(self.fget, fset, self.fdel)
```

### Visualization

```
Descriptor Taxonomy
───────────────────
Descriptor
├── Data Descriptor (defines __get__ + __set__ or __delete__)
│       Examples: property, custom validators, slot members
│       Priority: HIGHER than instance __dict__
│
└── Non-Data Descriptor (defines only __get__)
        Examples: function, staticmethod, classmethod
        Priority: LOWER than instance __dict__

How function binding works via descriptor:
──────────────────────────────────────────
d = Dog()
d.bark
  │
  ▼
Dog.__getattribute__("bark")
  │
  ├── Finds "bark" in Dog.__dict__
  ├── bark is a function (has __get__)
  └── Calls function.__get__(bark, d, Dog)
              │
              └── Returns <bound method> with self=d pre-filled

property access on obj.x:
──────────────────────────
obj.x
  │
  ▼  (data descriptor wins over __dict__)
property.__get__(obj, type(obj))
  │
  └── Calls fget(obj)  →  returns computed value
```

### Common Mistakes

```python
# ❌ Mistake 1: Infinite recursion in property by using the same name
class Bad:
    @property
    def x(self):
        return self.x   # calls property.__get__ again → RecursionError!

    @x.setter
    def x(self, val):
        self.x = val    # calls property.__set__ again → RecursionError!

# ✅ Fix: use a private backing attribute
class Good:
    @property
    def x(self):
        return self._x

    @x.setter
    def x(self, val):
        self._x = val

# ❌ Mistake 2: Sharing descriptor instances between attributes
class Typed:
    def __init__(self, expected_type):
        self.expected_type = expected_type
        self.name = None   # must be set per-attribute

    def __set_name__(self, owner, name):
        self.name = name   # ← this is critical

class Person:
    # If __set_name__ is NOT used, you must pass the name manually
    # or all attributes share the same 'name' string → conflict
    name = Typed(str)   # Typed.__set_name__ is called automatically
    age  = Typed(int)   # with the attribute name "age"

# ❌ Mistake 3: Accessing descriptor from the class returns the descriptor object
class MyClass:
    @property
    def value(self):
        return 42

print(MyClass.value)   # <property object> — not 42!
# To get the value, access via an INSTANCE:
obj = MyClass()
print(obj.value)       # 42

# This is why property.__get__ checks: if obj is None: return self
```

### Interview Questions

**Beginner**

**Q1: What is a descriptor in Python?**
A: An object that defines `__get__`, `__set__`, or `__delete__`. When stored as a class attribute, Python calls these methods instead of performing normal attribute access, allowing computed or validated attributes.

**Q2: What is the difference between `@property` and a regular attribute?**
A: A regular attribute is stored directly in `__dict__`. A `property` is a descriptor stored in the class — when accessed, Python calls the getter function, allowing computed, read-only, or validated attributes without changing the calling syntax (`obj.x` instead of `obj.get_x()`).

**Q3: What is `__set_name__` and when is it called?**
A: `__set_name__(self, owner, name)` is called by Python automatically during class creation when a descriptor is assigned as a class attribute. It lets the descriptor know the attribute name it was assigned to — without requiring the name to be passed manually to `__init__`.

**Intermediate**

**Q4: What is the difference between a data descriptor and a non-data descriptor?**
A: A data descriptor defines both `__get__` and `__set__` (or `__delete__`) — it takes priority over the instance `__dict__`. A non-data descriptor defines only `__get__` — the instance `__dict__` takes priority over it. `property` is a data descriptor; plain functions are non-data descriptors.

**Q5: How does Python turn a function into a bound method using descriptors?**
A: Functions define `__get__`. When accessed via an instance, `function.__get__(instance, owner)` is called, which returns a `method` object with `self` pre-bound. This is why `obj.method()` automatically receives `obj` as the first argument.

**Q6: How do `staticmethod` and `classmethod` use the descriptor protocol?**
A: Both are non-data descriptors. `staticmethod.__get__` ignores `obj` and returns the bare function. `classmethod.__get__` returns a bound method with `cls` (the class) as the first argument instead of the instance.

**Advanced**

**Q7: Implement a `cached_property` descriptor that computes a value once and caches it in the instance dict.**
A:
```python
class cached_property:
    def __init__(self, func):
        self.func = func
        self.attrname = None
        self.__doc__ = func.__doc__

    def __set_name__(self, owner, name):
        self.attrname = name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        val = self.func(obj)
        obj.__dict__[self.attrname] = val   # cache in instance dict
        return val
        # NOTE: because this is a non-data descriptor (no __set__),
        # the instance __dict__ entry will win on next access — cached!

class Circle:
    def __init__(self, radius):
        self.radius = radius

    @cached_property
    def area(self):
        import math
        print("computing...")
        return math.pi * self.radius ** 2

c = Circle(5)
print(c.area)   # computing... 78.539...
print(c.area)   # 78.539... (no "computing" — served from __dict__)
```

**Q8: Explain how Django model fields use descriptors.**
A: Each Django `Field` (e.g., `CharField`, `IntegerField`) is a descriptor assigned as a class attribute on the `Model` class. `__get__` retrieves the value from the model's internal `__dict__` or database state. `__set__` validates and stores the value. This allows `instance.name = "Alice"` to trigger validation and type coercion transparently.

### Key Takeaways

- Descriptors intercept attribute access via `__get__`, `__set__`, `__delete__`.
- Data descriptors (with `__set__`) beat instance `__dict__`; non-data descriptors lose to it.
- Functions are non-data descriptors — `__get__` is how they become bound methods.
- `property`, `staticmethod`, `classmethod` are all built on the descriptor protocol.
- `__set_name__` lets descriptors auto-capture their attribute name at class creation time.
- Descriptors are the foundation of Django fields, SQLAlchemy columns, and many ORMs.

---

## Summary Table

| Topic | Core Concept | Key Tool | CPython Hook |
|---|---|---|---|
| Everything is Object | All values are `PyObject` | `type()`, `id()`, `isinstance()` | `ob_refcnt`, `ob_type` |
| Type Hierarchy | `object` is root; single DAG | `__bases__`, `__mro__` | `tp_bases`, `tp_mro` |
| Metatype | `type` is the class of classes | `type(name, bases, ns)` | `PyType_Type` |
| Object Lifecycle | Create → Init → Use → Destroy | `__new__`, `__init__`, `__del__` | `tp_new`, `tp_dealloc` |
| Attribute Lookup | data desc > `__dict__` > non-data desc | `getattr`, `__getattr__` | `__getattribute__` |
| Method Resolution | C3 linearization; MRO | `__mro__`, `super()` | `tp_mro` |
| Descriptor Protocol | `__get__`/`__set__`/`__delete__` | `property`, `classmethod` | `tp_descr_get` |

---

*End of 03 — Python Object Model*
