<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Python OOP Fundamentals – Part 1

## Table of Contents

1. [Introduction to Object-Oriented Programming](#introduction-to-object-oriented-programming)
2. [Classes](#classes)
    - [What is a Class](#what-is-a-class)
    - [Class Definition](#class-definition)
    - [Class Attributes vs Instance Attributes](#class-attributes-vs-instance-attributes)
    - [Methods](#methods)
    - [Constructor (`__init__`)](#constructor-__init__)
    - [Object Creation](#object-creation)
    - [The `self` Keyword](#the-self-keyword)
    - [Class Variables vs Instance Variables](#class-variables-vs-instance-variables)
    - [Class Methods](#class-methods)
    - [Static Methods](#static-methods)
    - [Internal Working of Classes](#internal-working-of-classes)
3. [Objects](#objects)
    - [What is an Object](#what-is-an-object)
    - [Object Lifecycle](#object-lifecycle)
    - [Object State, Behavior, and Identity](#object-state-behavior-and-identity)
    - [Creating Objects](#creating-objects)
    - [Accessing Members](#accessing-members)
    - [Updating Attributes](#updating-attributes)
    - [Internal Working of Objects](#internal-working-of-objects)
4. [Common OOP Mistakes](#common-oop-mistakes)
5. [Best Practices](#best-practices)
6. [Performance Considerations](#performance-considerations)
7. [UML-style Design Examples](#uml-style-design-examples)
8. [Real-world OOP Models](#real-world-oop-models)
9. [Coding Exercises](#coding-exercises)
10. [Interview Questions](#interview-questions)
11. [Summary](#summary)
12. [Cheat Sheet](#cheat-sheet)
13. [Practice Problems](#practice-problems)

***

## Introduction to Object-Oriented Programming

Object-Oriented Programming (OOP) is a programming paradigm based on the concept of **objects**, which contain **data** (attributes) and **code** (methods). Python is a multi-paradigm language that fully supports OOP.

**Why OOP exists:**

- Organizes code into reusable, modular components
- Models real-world entities naturally
- Enables code reuse through inheritance
- Improves maintainability and scalability

**Real-world analogy:** Think of a **car factory** (class) that produces **cars** (objects). The factory has a blueprint (class definition), and each car produced has its own color, mileage, and state (attributes) while sharing the same functionality (methods).

***

## Classes

### What is a Class

**Definition:** A class is a blueprint or template for creating objects. It defines the structure (attributes) and behavior (methods) that objects of that class will have.

**Why it exists:** Classes allow you to create multiple objects with the same structure without repeating code. They enable abstraction and encapsulation.

**Real-world analogy:** A **cookie cutter** (class) that creates multiple **cookies** (objects). The cutter defines the shape, but each cookie is a separate instance.

**Syntax:**

```python
class ClassName:
    """Optional docstring"""
    # class attributes and methods
```

**Step-by-step explanation:**

1. Use `class` keyword
2. Follow with class name (capitalized, CamelCase)
3. Add colon
4. Indent class body
5. Define attributes and methods

**Memory behavior:** When Python encounters a class definition, it creates a class object in memory and stores it in the namespace with the given name.

***

### Class Definition

**Syntax:**

```python
class Person:
    """A class representing a person"""
    species = "Human"  # class attribute
    
    def __init__(self, name, age):
        self.name = name  # instance attribute
        self.age = age
    
    def greet(self):
        return f"Hello, I'm {self.name}"
```

**Step-by-step explanation:**

1. `class Person:` - defines a new class named Person
2. `"""A class representing a person"""` - docstring describing the class
3. `species = "Human"` - class attribute (shared by all instances)
4. `def __init__(...):` - constructor method
5. `self.name = name` - creates instance attribute

**Memory behavior:** Python creates a class object containing:

- `__dict__`: dictionary of class attributes
- `__name__`: class name
- `__module__`: module where defined
- Methods as function objects

**Internal working:**

```python
# What happens internally
Person = type('Person', (), {'species': 'Human', '__init__': __init__, 'greet': greet})
```


***

### Class Attributes vs Instance Attributes

**Definition:**

- **Class attributes**: Defined in the class body, shared by all instances
- **Instance attributes**: Defined in methods (usually `__init__`), unique to each instance

**Why they exist:**

- Class attributes for data shared across all instances (constants, counters)
- Instance attributes for data specific to each object

**Real-world analogy:**

- Class attribute: All humans have the same species ("Human")
- Instance attribute: Each person has their own name and age

**Syntax:**

```python
class Car:
    # Class attribute
    wheels = 4
    company = "Toyota"
    
    def __init__(self, model, color):
        # Instance attributes
        self.model = model
        self.color = color
```

**Step-by-step explanation:**

1. `wheels = 4` - class attribute, same for all Cars
2. `self.model = model` - instance attribute, unique per Car

**Memory behavior:**

- Class attributes stored in class's `__dict__`
- Instance attributes stored in instance's `__dict__`

**Attribute lookup mechanism:**

```python
car1 = Car("Camry", "Red")
car2 = Car("Corolla", "Blue")

print(car1.wheels)  # 4 - looks in class __dict__
print(car1.model)   # "Camry" - looks in instance __dict__

# If instance doesn't have attribute, Python looks in class
car1.wheels = 6  # Creates instance attribute, overrides class attribute
print(car1.wheels)  # 6
print(car2.wheels)  # 4 - still uses class attribute
```

**Multiple examples:**

```python
# Example 1: Counter class
class Counter:
    count = 0  # class attribute
    
    def __init__(self):
        Counter.count += 1  # increment class attribute
    
    def get_count(self):
        return Counter.count

c1 = Counter()  # count = 1
c2 = Counter()  # count = 2
c3 = Counter()  # count = 3
```

```python
# Example 2: Employee class
class Employee:
    company = "Tech Corp"  # class attribute
    raise_percentage = 0.1  # class attribute
    
    def __init__(self, name, salary):
        self.name = name  # instance attribute
        self.salary = salary  # instance attribute
    
    def apply_raise(self):
        self.salary *= (1 + Employee.raise_percentage)
```

**Edge cases:**

```python
class Test:
    attr = 10
    
t1 = Test()
t2 = Test()

# Modifying class attribute through class
Test.attr = 20
print(t1.attr)  # 20
print(t2.attr)  # 20

# Creating instance attribute (overrides class attribute)
t1.attr = 30
print(t1.attr)  # 30 (instance attribute)
print(t2.attr)  # 20 (class attribute)
print(Test.attr)  # 20 (class attribute)
```

**Common mistakes:**

```python
# WRONG: Treating class attribute as instance attribute
class Bad:
    def __init__(self):
        self.list = []  # This is actually fine, but...

# WRONG: Modifying class attribute through instance (creates instance attribute)
class Wrong:
    shared = []
    
    def add(self, item):
        self.shared.append(item)  # Creates instance attribute!

# CORRECT:
class Correct:
    shared = []
    
    def add(self, item):
        Correct.shared.append(item)  # Modifies class attribute
```


***

### Methods

**Definition:** Methods are functions defined inside a class that define the behavior of objects.

**Why they exist:** Methods encapsulate functionality related to the object, enabling objects to perform actions.

**Real-world analogy:** A **phone** (object) has methods like `call()`, `send_message()`, `take_photo()`.

**Types of methods:**

1. **Instance methods** - most common, access instance data
2. **Class methods** - access class data
3. **Static methods** - no access to instance/class data

**Syntax (Instance method):**

```python
class MyClass:
    def instance_method(self, arg1, arg2):
        return arg1 + arg2
```

**Step-by-step explanation:**

1. Define function inside class
2. First parameter is `self` (reference to instance)
3. Can access instance attributes via `self`
4. Called on instance: `obj.method()`

**Memory behavior:** Methods are stored as function objects in the class's `__dict__`. When accessed via instance, they become **bound methods**.

**Method binding:**

```python
class Person:
    def greet(self):
        return f"Hello, {self.name}"

p = Person()
p.name = "Alice"

# Method is bound to instance
bound_method = p.greet
print(bound_method)  # <bound method Person.greet of <Person object>>
print(bound_method())  # "Hello, Alice"

# The bound method remembers p as self
```

**Bound methods vs unbound methods:**

```python
# Bound method (accessed through instance)
bound = p.greet  # self is automatically p

# Unbound function (accessed through class)
unbound = Person.greet  # requires explicit self
unbound(p)  # must pass instance explicitly
```

**Dynamic method addition:**

```python
class Dynamic:
    pass

obj = Dynamic()

# Add method dynamically
def new_method(self, x):
    return x * 2

obj.new_method = new_method
print(obj.new_method(5))  # 10

# Add to class (all instances get it)
Dynamic.class_method = lambda self: "class method"
print(obj.class_method())  # "class method"
```

**Multiple examples:**

```python
# Example 1: Bank Account
class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner = owner
        self.balance = balance
    
    def deposit(self, amount):
        if amount > 0:
            self.balance += amount
            return f"Deposited ${amount}"
        return "Invalid amount"
    
    def withdraw(self, amount):
        if 0 < amount <= self.balance:
            self.balance -= amount
            return f"Withdrawn ${amount}"
        return "Insufficient funds"
    
    def get_balance(self):
        return self.balance
```

```python
# Example 2: Rectangle
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def area(self):
        return self.width * self.height
    
    def perimeter(self):
        return 2 * (self.width + self.height)
    
    def is_square(self):
        return self.width == self.height
    
    def scale(self, factor):
        self.width *= factor
        self.height *= factor
```

**Edge cases:**

```python
class Edge:
    def method1(self):
        return self
    
    def method2(self):
        return method1()  # ERROR: method1 not defined
    
    def method3(self):
        return self.method1()  # CORRECT: must use self

# Calling method without instance (ERROR)
# Edge.method1()  # TypeError: missing 'self'
Edge.method1(Edge())  # Works: pass instance explicitly
```

**Common mistakes:**

```python
# WRONG: Missing self parameter
class Bad:
    def greet():  # Missing self!
        return "Hello"

# CORRECT:
class Good:
    def greet(self):
        return "Hello"

# WRONG: Not using self for instance attributes
class Wrong:
    def __init__(self, name):
        name = name  # Creates local variable, not attribute
    
    def get_name(self):
        return name  # ERROR: name not defined

# CORRECT:
class Right:
    def __init__(self, name):
        self.name = name  # Instance attribute
    
    def get_name(self):
        return self.name
```


***

### Constructor (`__init__`)

**Definition:** The `__init__` method is a special method (constructor) that's automatically called when a new object is created. It initializes the object's state.

**Why it exists:** To set up initial values for instance attributes when an object is created. Ensures objects start in a valid state.

**Real-world analogy:** When you **buy a new phone**, it comes with initial settings (battery level, default apps, language). The factory process is like `__init__`.

**Syntax:**

```python
class ClassName:
    def __init__(self, param1, param2):
        self.attr1 = param1
        self.attr2 = param2
```

**Step-by-step explanation:**

1. Method name is `__init__` (double underscore, init, double underscore)
2. First parameter is `self`
3. Additional parameters for initialization values
4. Assign values to instance attributes using `self`
5. No return statement (implicitly returns None)

**Memory behavior:**

1. Python creates a new empty object
2. Calls `__init__` with the new object as `self`
3. `__init__` populates the object's `__dict__`
4. Returns the initialized object

**Object creation process:**

```python
class Person:
    def __init__(self, name):
        print(f"Creating person: {name}")
        self.name = name

# What happens:
# 1. Python creates empty Person object
# 2. Calls Person.__init__(empty_object, "Alice")
# 3. empty_object.name = "Alice"
# 4. Returns initialized object
p = Person("Alice")
```

**Multiple examples:**

```python
# Example 1: Book with default values
class Book:
    def __init__(self, title, author, pages=0, read=False):
        self.title = title
        self.author = author
        self.pages = pages
        self.read = read
    
    def __str__(self):
        return f"{self.title} by {self.author}"
```

```python
# Example 2: Point with validation
class Point:
    def __init__(self, x=0, y=0):
        if not isinstance(x, (int, float)):
            raise TypeError("x must be numeric")
        if not isinstance(y, (int, float)):
            raise TypeError("y must be numeric")
        
        self.x = x
        self.y = y
    
    def distance_from_origin(self):
        return (self.x ** 2 + self.y ** 2) ** 0.5
```

```python
# Example 3: Optional parameters
class Config:
    def __init__(self, host="localhost", port=8080, debug=False, **kwargs):
        self.host = host
        self.port = port
        self.debug = debug
        self.extra = kwargs
```

**Edge cases:**

```python
# No __init__ defined (uses default)
class NoInit:
    pass

obj = NoInit()  # Works: creates empty object

# Multiple __init__ (only last one works - no overloading)
class MultiInit:
    def __init__(self, a):
        self.a = a
    
    def __init__(self, a, b):  # Overrides previous!
        self.a = a
        self.b = b

# obj = MultiInit(1)  # ERROR: missing b
obj = MultiInit(1, 2)  # Works
```

**Common mistakes:**

```python
# WRONG: Returning value from __init__
class Bad:
    def __init__(self, value):
        self.value = value
        return self.value  # ERROR!

# CORRECT:
class Good:
    def __init__(self, value):
        self.value = value
        # No return

# WRONG: Using __new__ instead of __init__ for initialization
class Wrong:
    def __new__(cls, value):
        instance = super().__new__(cls)
        instance.value = value  # Should use __init__
        return instance

# CORRECT:
class Right:
    def __init__(self, value):
        self.value = value
```

**Best practices:**

- Keep `__init__` simple - only initialization
- Use default values for optional parameters
- Validate input parameters
- Don't do heavy computation in `__init__`
- Name parameters clearly

***

### Object Creation

**Definition:** Object creation (instantiation) is the process of creating a new instance of a class.

**Why it exists:** To create concrete objects from class blueprints that can store data and perform actions.

**Real-world analogy:** Using a **mold** (class) to create a **statue** (object). The mold defines the shape, but you need to pour material to create the actual statue.

**Syntax:**

```python
object_name = ClassName(arguments)
```

**Step-by-step explanation:**

1. Call class name like a function
2. Pass required arguments
3. Python creates new object
4. Calls `__init__` with arguments
5. Returns initialized object

**Memory behavior:**

```python
class Person:
    def __init__(self, name):
        self.name = name

p1 = Person("Alice")
p2 = Person("Bob")

# Memory layout:
# Person class object: {__dict__: {__init__: function, ...}}
# p1 object: {__dict__: {name: "Alice"}, __class__: Person}
# p2 object: {__dict__: {name: "Bob"}, __class__: Person}
```

**Internal working:**

```python
# What happens internally:
# 1. Person.__new__(Person) creates empty object
# 2. Person.__init__(empty_object, "Alice") initializes it
# 3. Returns initialized object

p = Person("Alice")
# Equivalent to:
p = Person.__new__(Person)
Person.__init__(p, "Alice")
```

**Multiple examples:**

```python
# Example 1: Basic creation
class Car:
    def __init__(self, brand, model):
        self.brand = brand
        self.model = model

car1 = Car("Toyota", "Camry")
car2 = Car("Honda", "Civic")
```

```python
# Example 2: Creation with defaults
class Server:
    def __init__(self, ip="127.0.0.1", port=8080):
        self.ip = ip
        self.port = port

s1 = Server()  # Uses defaults
s2 = Server("192.168.1.1")  # Override ip
s3 = Server("10.0.0.1", 3000)  # Override both
```

```python
# Example 3: Creation returning same object (singleton pattern)
class Singleton:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

s1 = Singleton()
s2 = Singleton()
print(s1 is s2)  # True - same object
```

**Edge cases:**

```python
# Creation without arguments (all defaults)
class AllDefault:
    def __init__(self, a=1, b=2, c=3):
        self.a = a
        self.b = b
        self.c = c

obj = AllDefault()  # Works

# Creation with too many arguments
class FewArgs:
    def __init__(self, x):
        self.x = x

# obj = FewArgs(1, 2)  # ERROR: too many arguments
```

**Common mistakes:**

```python
# WRONG: Calling class without parentheses
class Person:
    def __init__(self, name):
        self.name = name

p = Person  # ERROR: p is the class, not an object
p = Person()  # ERROR: missing name argument
p = Person("Alice")  # CORRECT

# WRONG: Assuming __init__ returns something
class Bad:
    def __init__(self, value):
        self.value = value
        return self.value

obj = Bad(5)
print(obj)  # None (not 5!)
```

**Best practices:**

- Use meaningful variable names for objects
- Create objects when needed (not all at startup)
- Consider object pools for frequently created objects
- Use factory functions for complex creation

***

### The `self` Keyword

**Definition:** `self` is a reference to the current instance of the class. It's the first parameter of instance methods.

**Why it exists:** Python doesn't use `this` pointer like other languages. `self` explicitly refers to the instance, making code clearer and allowing dynamic behavior.

**Real-world analogy:** When you **write a letter to yourself**, you sign your own name. `self` is like your name - it identifies which instance you're referring to.

**Key points:**

- Must be first parameter in instance methods
- Automatically passed by Python when calling method
- Not a keyword (just a convention, but strongly recommended)
- References the instance, not the class

**Syntax:**

```python
class ClassName:
    def method_name(self, parameter1, parameter2):
        return self.attribute
```

**Step-by-step explanation:**

1. Define method with `self` as first parameter
2. Access instance attributes via `self.attribute`
3. Call other methods via `self.method()`
4. When calling `obj.method()`, Python passes `obj` as `self`

**Memory behavior:**

```python
class Test:
    def show(self):
        print(self)
        print(id(self))

obj1 = Test()
obj2 = Test()

obj1.show()  # self = obj1
obj2.show()  # self = obj2
```

**Internal working:**

```python
# Method call transformation:
obj.method(arg1, arg2)
# Becomes:
ClassName.method(obj, arg1, arg2)

# Self is automatically the instance
```

**Multiple examples:**

```python
# Example 1: Accessing attributes
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    def introduce(self):
        return f"I'm {self.name}, {self.age} years old"
    
    def celebrate_birthday(self):
        self.age += 1
        return f"{self.name} is now {self.age}"
```

```python
# Example 2: Calling methods
class Calculator:
    def __init__(self):
        self.result = 0
    
    def add(self, x):
        self.result += x
        return self
    
    def multiply(self, x):
        self.result *= x
        return self
    
    def get_result(self):
        return self.result

calc = Calculator()
calc.add(5).add(3).multiply(2)
print(calc.get_result())  # 16
```

```python
# Example 3: Self for chaining
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def add(self, other):
        self.x += other.x
        self.y += other.y
        return self
    
    def __str__(self):
        return f"Vector({self.x}, {self.y})"

v1 = Vector(1, 2)
v2 = Vector(3, 4)
v1.add(v2)
print(v1)  # Vector(4, 6)
```

**Edge cases:**

```python
# self can be named differently (but DON'T do this)
class Weird:
    def method(this):
        return this.value

obj = Weird()
obj.value = 42
print(obj.method())  # 42

# Missing self (ERROR)
class Bad:
    def method():  # No self!
        return "Hello"

# obj = Bad()
# obj.method()  # ERROR: missing self
```

**Common mistakes:**

```python
# WRONG: Using self as variable name
class Wrong:
    def __init__(self, self):  # Confusing!
        self.self = self

# CORRECT:
class Right:
    def __init__(self, value):
        self.value = value

# WRONG: Not using self for attributes
class Bad:
    def __init__(self, name):
        name = name  # Local variable, not attribute
    
    def get_name(self):
        return name  # ERROR

# CORRECT:
class Good:
    def __init__(self, name):
        self.name = name  # Instance attribute
    
    def get_name(self):
        return self.name

# WRONG: Passing self explicitly (DON'T do this)
obj = Person()
obj.greet()  # CORRECT: self passed automatically
Person.greet(obj)  # Works but unusual
```

**Best practices:**

- Always use `self` as first parameter
- Always use `self` to access instance attributes
- Don't pass `self` explicitly when calling methods
- Stick to `self` convention (don't rename it)

***

### Class Variables vs Instance Variables

**Definition:**

- **Class variables**: Variables shared by all instances of a class
- **Instance variables**: Variables unique to each instance

**Why they exist:**

- Class variables for data common to all instances (constants, counters, configuration)
- Instance variables for data specific to each object

**Real-world analogy:**

- Class variable: All students in a school share the same school name
- Instance variable: Each student has their own name and grade

**Syntax:**

```python
class ClassName:
    class_variable = value  # Class variable
    
    def __init__(self, instance_value):
        self.instance_variable = instance_value  # Instance variable
```

**Step-by-step explanation:**

1. Define class variable outside methods (in class body)
2. Define instance variable inside `__init__` using `self`
3. Access class variable via `ClassName.variable` or `self.variable`
4. Access instance variable via `self.variable`

**Memory behavior:**

```python
class Test:
    class_var = "class"  # Stored in class __dict__
    
    def __init__(self):
        self.instance_var = "instance"  # Stored in instance __dict__

obj = Test()

# Memory locations:
# Test.__dict__["class_var"] = "class"
# obj.__dict__["instance_var"] = "instance"
```

**Attribute lookup for class variables:**

```python
class Counter:
    total = 0  # Class variable
    
    def __init__(self):
        Counter.total += 1  # Modify class variable
    
    def get_total(self):
        return Counter.total  # Access class variable

c1 = Counter()
c2 = Counter()
print(c1.get_total())  # 2
print(c2.get_total())  # 2
```

**Multiple examples:**

```python
# Example 1: Employee with company
class Employee:
    company = "Tech Corp"  # Class variable
    raise_rate = 0.1  # Class variable
    
    def __init__(self, name, salary):
        self.name = name  # Instance variable
        self.salary = salary  # Instance variable
    
    def apply_raise(self):
        self.salary *= (1 + Employee.raise_rate)
    
    def get_info(self):
        return f"{self.name} at {Employee.company}"
```

```python
# Example 2: Product with counter
class Product:
    product_count = 0  # Class variable
    
    def __init__(self, name, price):
        self.name = name  # Instance variable
        self.price = price  # Instance variable
        Product.product_count += 1  # Update class variable
        self.product_id = Product.product_count
    
    def get_total_products(self):
        return Product.product_count
```

```python
# Example 3: Configuration
class Config:
    DEBUG = False  # Class variable
    VERSION = "1.0"  # Class variable
    DEFAULT_PORT = 8080  # Class variable
    
    def __init__(self, host="localhost"):
        self.host = host  # Instance variable
        self.timeout = 30  # Instance variable
```

**Edge cases:**

```python
class Edge:
    shared = []
    
    def add(self, item):
        self.shared.append(item)  # Creates instance attribute!

e1 = Edge()
e2 = Edge()

e1.add(1)
print(e1.shared)  # [1]
print(e2.shared)  # [] (different list!)

# CORRECT:
class Fixed:
    shared = []
    
    def add(self, item):
        Fixed.shared.append(item)  # Modifies class attribute

f1 = Fixed()
f2 = Fixed()

f1.add(1)
print(f1.shared)  # [1]
print(f2.shared)  # [1] (same list!)
```

**Common mistakes:**

```python
# WRONG: Modifying class variable through instance (creates instance var)
class Wrong:
    count = 0
    
    def increment(self):
        self.count += 1  # Creates instance attribute!

w1 = Wrong()
w1.increment()
print(w1.count)  # 1 (instance)
print(Wrong.count)  # 0 (class)

# CORRECT:
class Right:
    count = 0
    
    def increment(self):
        Right.count += 1  # Modifies class attribute

r1 = Right()
r1.increment()
print(r1.count)  # 1 (class)
print(Right.count)  # 1 (class)
```

**Best practices:**

- Use class variables for constants and shared data
- Access class variables via `ClassName.variable` for clarity
- Don't modify class variables through instances
- Use instance variables for object-specific data

***

### Class Methods

**Definition:** Class methods are methods that receive the class as the first argument instead of the instance. Defined using `@classmethod` decorator.

**Why they exist:** To create methods that operate on the class itself rather than instances, useful for factory methods, alternative constructors, and class-level operations.

**Real-world analogy:** A **company HR department** (class method) that hires new employees (creates instances) doesn't belong to any specific employee.

**Syntax:**

```python
class ClassName:
    @classmethod
    def class_method(cls, argument):
        return cls(argument)
```

**Step-by-step explanation:**

1. Add `@classmethod` decorator above method
2. First parameter is `cls` (not `self`)
3. `cls` refers to the class, not instance
4. Can access class variables via `cls`
5. Can create instances via `cls()`

**Memory behavior:** Class methods are stored in class `__dict__` as classmethod objects. When accessed, they receive the class automatically.

**Internal working:**

```python
@classmethod
def method(cls):
    return cls

# Internally:
ClassName.method = classmethod(method)
```

**Multiple examples:**

```python
# Example 1: Alternative constructor
class Date:
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day
    
    @classmethod
    def from_string(cls, date_string):
        year, month, day = map(int, date_string.split("-"))
        return cls(year, month, day)
    
    @classmethod
    def today(cls):
        from datetime import date
        today = date.today()
        return cls(today.year, today.month, today.day)

d1 = Date(2024, 1, 15)
d2 = Date.from_string("2024-01-15")
d3 = Date.today()
```

```python
# Example 2: Counter with class method
class Counter:
    count = 0
    
    def __init__(self):
        Counter.count += 1
    
    @classmethod
    def get_count(cls):
        return cls.count
    
    @classmethod
    def reset_count(cls):
        cls.count = 0

c1 = Counter()
c2 = Counter()
print(Counter.get_count())  # 2
Counter.reset_count()
print(Counter.get_count())  # 0
```

```python
# Example 3: Factory method
class Shape:
    def __init__(self, color):
        self.color = color
    
    @classmethod
    def red(cls):
        return cls("red")
    
    @classmethod
    def blue(cls):
        return cls("blue")
    
    @classmethod
    def green(cls):
        return cls("green")

red_shape = Shape.red()
blue_shape = Shape.blue()
```

**Edge cases:**

```python
# Class method can call other class methods
class A:
    @classmethod
    def method1(cls):
        return "method1"
    
    @classmethod
    def method2(cls):
        return cls.method1() + " and method2"

print(A.method2())  # "method1 and method2"

# Class method vs instance method
class B:
    count = 0
    
    def __init__(self):
        B.count += 1
    
    def instance_method(self):
        return f"Instance: {self.count}"
    
    @classmethod
    def class_method(cls):
        return f"Class: {cls.count}"

b = B()
print(b.instance_method())  # "Instance: 1"
print(b.class_method())  # "Class: 1"
```

**Common mistakes:**

```python
# WRONG: Missing @classmethod decorator
class Wrong:
    def class_method(cls):  # Missing decorator!
        return cls

# ERROR: called as instance method, gets self not cls
# obj = Wrong()
# obj.class_method()  # ERROR

# CORRECT:
class Right:
    @classmethod
    def class_method(cls):
        return cls

# WRONG: Using self instead of cls
class Bad:
    @classmethod
    def method(self):  # Should be cls!
        return self

# CORRECT:
class Good:
    @classmethod
    def method(cls):
        return cls
```

**Best practices:**

- Use `cls` as first parameter (convention)
- Use class methods for alternative constructors
- Use class methods for operations on class-level data
- Return `cls(...)` for factory methods

***

### Static Methods

**Definition:** Static methods are methods that don't receive `self` or `cls`. They're defined using `@staticmethod` decorator and behave like regular functions but belong to the class.

**Why they exist:** To create utility functions that are related to the class but don't need access to instance or class data. Good for namespace organization.

**Real-world analogy:** A **math utility function** like `calculate_area()` that's related to geometry but doesn't belong to any specific shape.

**Syntax:**

```python
class ClassName:
    @staticmethod
    def static_method(argument1, argument2):
        return argument1 + argument2
```

**Step-by-step explanation:**

1. Add `@staticmethod` decorator above method
2. No `self` or `cls` parameter
3. Works like a regular function
4. Can be called on class or instance
5. Can't access instance or class attributes

**Memory behavior:** Static methods are stored in class `__dict__` as staticmethod objects. When accessed, they return the original function without binding.

**Multiple examples:**

```python
# Example 1: Math utilities
class MathUtils:
    @staticmethod
    def add(a, b):
        return a + b
    
    @staticmethod
    def multiply(a, b):
        return a * b
    
    @staticmethod
    def factorial(n):
        if n <= 1:
            return 1
        return n * MathUtils.factorial(n - 1)

print(MathUtils.add(5, 3))  # 8
print(MathUtils.factorial(5))  # 120
```

```python
# Example 2: Temperature converter
class Temperature:
    @staticmethod
    def celsius_to_fahrenheit(c):
        return c * 9/5 + 32
    
    @staticmethod
    def fahrenheit_to_celsius(f):
        return (f - 32) * 5/9
    
    @staticmethod
    def celsius_to_kelvin(c):
        return c + 273.15

print(Temperature.celsius_to_fahrenheit(0))  # 32.0
print(Temperature.fahrenheit_to_celsius(32))  # 0.0
```

```python
# Example 3: String validation
class Validator:
    @staticmethod
    def is_empty(s):
        return s is None or len(s) == 0
    
    @staticmethod
    def is_valid_email(email):
        return email and "@" in email and "." in email
    
    @staticmethod
    def sanitize(s):
        return s.strip() if s else ""

print(Validator.is_valid_email("test@example.com"))  # True
print(Validator.sanitize("  hello  "))  # "hello"
```

**Edge cases:**

```python
# Static method can be called on class or instance
class Test:
    @staticmethod
    def method(x):
        return x * 2

t = Test()
print(Test.method(5))  # 10
print(t.method(5))  # 10

# Static method can call other static methods
class Utils:
    @staticmethod
    def square(x):
        return x * x
    
    @staticmethod
    def square_sum(x, y):
        return Utils.square(x) + Utils.square(y)

print(Utils.square_sum(3, 4))  # 25
```

**Common mistakes:**

```python
# WRONG: Missing @staticmethod decorator
class Wrong:
    def static_method(x, y):  # Missing decorator!
        return x + y

# ERROR: called as instance method, gets self
# obj = Wrong()
# obj.static_method(1, 2)  # ERROR: too many arguments

# CORRECT:
class Right:
    @staticmethod
    def static_method(x, y):
        return x + y

# WRONG: Using self or cls in static method
class Bad:
    @staticmethod
    def method(self):  # Should have no self!
        return self

# CORRECT:
class Good:
    @staticmethod
    def method(x):
        return x
```

**Best practices:**

- Use static methods for utility functions
- Don't use if you need `self` or `cls`
- Call on class (not instance) for clarity
- Group related utilities in a class

***

### Internal Working of Classes

**How Python stores class objects:**

Python stores classes as objects with the following structure:

```python
class __dict__:
    {
        '__name__': 'ClassName',
        '__module__': 'module_name',
        '__doc__': 'docstring',
        'attribute': value,
        'method': function_object,
        ...
    }
```

**Class namespace:**

```python
class Namespace:
    class_var = "class"
    
    def method(self):
        instance_var = "instance"

# Namespace.__dict__ contains:
# {
#   'class_var': 'class',
#   'method': <function method>,
#   '__dict__': <member_descriptor>,
#   '__weakref__': <member_descriptor>,
#   ...
# }
```

**`__dict__` attribute:**

```python
class Example:
    class_attr = 10
    
    def __init__(self, value):
        self.instance_attr = value

obj = Example(5)

# Class __dict__
print(Example.__dict__)
# Contains: class_attr, __init__, __module__, etc.

# Instance __dict__
print(obj.__dict__)
# {'instance_attr': 5}
```

**Attribute lookup mechanism:**

```python
class Lookup:
    class_var = "class"
    
    def __init__(self):
        self.instance_var = "instance"

obj = Lookup()

# Lookup order:
# 1. Instance __dict__
print(obj.instance_var)  # "instance" - found in obj.__dict__

# 2. Class __dict__
print(obj.class_var)  # "class" - found in Lookup.__dict__

# 3. Parent classes (inheritance)
class Parent:
    parent_var = "parent"

class Child(Parent):
    child_var = "child"

c = Child()
print(c.parent_var)  # "parent" - found in Parent.__dict__
```

**Method binding:**

```python
class Binding:
    def method(self):
        return f"self = {self}"

obj = Binding()

# Unbound function (from class)
unbound = Binding.method
print(unbound)  # <function Binding.method>

# Bound method (from instance)
bound = obj.method
print(bound)  # <bound method Binding.method of Binding object>

# Calling bound method automatically passes self
print(bound())  # "self = <Binding object>"

# Calling unbound function requires explicit self
print(unbound(obj))  # "self = <Binding object>"
```

**Bound methods vs unbound methods:**

```python
class Bound:
    def instance_method(self):
        return "instance"
    
    @classmethod
    def class_method(cls):
        return "class"
    
    @staticmethod
    def static_method():
        return "static"

b = Bound()

# Instance method: bound to instance
print(type(b.instance_method))  # <class 'method'>
print(b.instance_method())  # "instance"

# Class method: bound to class
print(type(b.class_method))  # <class 'method'>
print(b.class_method())  # "class"

# Static method: not bound
print(type(b.static_method))  # <class 'function'>
print(b.static_method())  # "static"
```

**Dynamic attribute creation:**

```python
class Dynamic:
    pass

obj = Dynamic()

# Create attribute dynamically
obj.new_attr = "value"
print(obj.new_attr)  # "value"
print(obj.__dict__)  # {'new_attr': 'value'}

# Create method dynamically
def new_method(self):
    return "dynamic"

obj.new_method = new_method
print(obj.new_method())  # "dynamic"
```

**Dynamic method addition:**

```python
class AddMethod:
    def existing(self):
        return "existing"

# Add method to class
def added_method(self):
    return "added"

AddMethod.added_method = added_method

obj1 = AddMethod()
obj2 = AddMethod()

print(obj1.added_method())  # "added"
print(obj2.added_method())  # "added"

# Add method to single instance
obj1.instance_method = lambda: "instance"
print(obj1.instance_method())  # "instance"
print(obj2.instance_method())  # ERROR: obj2 doesn't have it
```

**Class object lifecycle:**

```python
class Lifecycle:
    print("1. Class body executed")
    
    def __init__(self):
        print("3. __init__ called")
    
    def method(self):
        print("4. method called")

print("2. Class object created")

obj = Lifecycle()  # 3. __init__ called
obj.method()  # 4. method called
```

**Relationship between classes and objects:**

```python
class Relationship:
    pass

obj = Relationship()

# obj is instance of Relationship
print(isinstance(obj, Relationship))  # True

# Relationship is type of obj
print(type(obj))  # <class 'Relationship'>

# Relationship is instance of type
print(isinstance(Relationship, type))  # True

# type is the base class of all classes
print(type(Relationship))  # <class 'type'>
```

**Diagram: Class Structure**

```
Class
 ├── Attributes
 │    ├── Class attributes (shared)
 │    └── Instance attributes (unique)
 ├── Methods
 │    ├── Instance methods (need self)
 │    ├── Class methods (need cls)
 │    └── Static methods (no self/cls)
 └── Constructor (__init__)
      ├── Initializes instance
      ├── Sets default values
      └── Validates input
```

**UML-style Class Diagram:**

```
┌─────────────────────────────────┐
│           Person                │
├─────────────────────────────────┤
│ # Class Attributes:             │
│   species: str = "Human"        │
│   count: int = 0                │
├─────────────────────────────────┤
│ - Instance Attributes:          │
│   - name: str                   │
│   - age: int                    │
│   - email: str                  │
├─────────────────────────────────┤
│ + Methods:                      │
│   + __init__(name, age, email)  │
│   + greet(): str                │
│   + introduce(): str            │
│   + celebrate_birthday(): void  │
│   @classmethod                  │
│   + get_count(): int            │
│   @staticmethod                 │
│   + is_adult(age): bool         │
└─────────────────────────────────┘

Legend:
  # = Class attribute
  - = Private instance attribute
  + = Public method
  @classmethod = Class method
  @staticmethod = Static method
```


***

## Objects

### What is an Object

**Definition:** An object is a concrete instance of a class. It contains data (attributes) and can perform actions (methods).

**Why it exists:** Objects are the fundamental building blocks of OOP. They represent real-world entities and encapsulate state and behavior.

**Real-world analogy:** A **specific car** (object) like "my Toyota Camry" vs. the **car blueprint** (class). The object has specific color, mileage, and owner.

**Key characteristics:**

- Has **state** (attributes/properties)
- Has **behavior** (methods)
- Has **identity** (unique identifier)

**Syntax:**

```python
object_name = ClassName(arguments)
```

**Step-by-step explanation:**

1. Call class with arguments
2. Python creates new object
3. Calls `__init__` to initialize
4. Returns the object

**Memory behavior:** Objects are stored in memory with:

- `__dict__`: dictionary of attributes
- `__class__`: reference to class
- `__id__`: unique identifier

***

### Object Lifecycle

**Definition:** Object lifecycle is the sequence of stages an object goes through: creation, initialization, usage, and destruction.

**Why it exists:** Understanding lifecycle helps manage resources, avoid memory leaks, and implement proper initialization/cleanup.

**Stages:**

1. **Creation** - Object is allocated in memory
2. **Initialization** - `__init__` sets up state
3. **Usage** - Object performs actions
4. **Destruction** - Object is garbage collected

**Real-world analogy:** A **person's life**: birth → growth → living → death.

**Step-by-step explanation:**

```python
class Lifecycle:
    def __init__(self):
        print("1. Object created and initialized")
    
    def __del__(self):
        print("3. Object destroyed")
    
    def use(self):
        print("2. Object being used")

# Lifecycle:
obj = Lifecycle()  # 1. Creation + initialization
obj.use()          # 2. Usage
del obj            # 3. Destruction
```

**Memory behavior:**

```python
# Creation
obj = MyClass()  # Memory allocated, __dict__ created

# Usage
obj.attr = 5     # Attribute added to __dict__

# Destruction
del obj          # Reference removed, may be garbage collected
```


***

### Object State, Behavior, and Identity

**Object State:**

- Defined by attributes
- Can change over time
- Stored in `__dict__`

```python
class State:
    def __init__(self):
        self.value = 0  # Initial state
    
    def increment(self):
        self.value += 1  # State changes

obj = State()
print(obj.value)  # 0 (initial state)
obj.increment()
print(obj.value)  # 1 (changed state)
```

**Object Behavior:**

- Defined by methods
- What object can do
- Encapsulated functionality

```python
class Behavior:
    def __init__(self, name):
        self.name = name
    
    def greet(self):
        return f"Hello, I'm {self.name}"  # Behavior
    
    def work(self):
        return f"{self.name} is working"  # Behavior

obj = Behavior("Alice")
print(obj.greet())  # "Hello, I'm Alice"
print(obj.work())   # "Alice is working"
```

**Object Identity:**

- Unique identifier for object
- Remains constant during lifetime
- Accessed via `id()`

```python
obj1 = [1, 2, 3]
obj2 = [1, 2, 3]
obj3 = obj1

print(id(obj1))  # e.g., 140234567890
print(id(obj2))  # e.g., 140234567950 (different!)
print(id(obj3))  # Same as obj1

print(obj1 == obj2)  # True (equal values)
print(obj1 is obj2)  # False (different objects)
print(obj1 is obj3)  # True (same object)
```


***

### Creating Objects

**Syntax:**

```python
object_name = ClassName(arguments)
```

**Multiple examples:**

```python
# Beginner: Simple object
class Person:
    def __init__(self, name):
        self.name = name

person = Person("Alice")
```

```python
# Intermediate: Object with multiple attributes
class Product:
    def __init__(self, name, price, quantity):
        self.name = name
        self.price = price
        self.quantity = quantity

product = Product("Laptop", 999.99, 10)
```

```python
# Advanced: Object with complex initialization
class Config:
    def __init__(self, host="localhost", port=8080, **options):
        self.host = host
        self.port = port
        self.options = options

config = Config("192.168.1.1", 3000, debug=True, timeout=30)
```

**Real-world modeling:**

```python
class BankAccount:
    def __init__(self, account_number, owner, balance=0):
        self.account_number = account_number
        self.owner = owner
        self.balance = balance
        self.transactions = []

account = BankAccount("12345", "Alice", 1000)
```


***

### Accessing Members

**Accessing attributes:**

```python
class Access:
    def __init__(self):
        self.public = "public"
        self._private = "private"
        self.__very_private = "very private"

obj = Access()

# Public attribute
print(obj.public)  # "public"

# "Private" attribute (name masking)
print(obj._private)  # "_private"

# Very private (name doubling)
print(obj.__very_private)  # ERROR!
print(obj._Access__very_private)  # "very private" (forced access)
```

**Accessing methods:**

```python
class Methods:
    def __init__(self, value):
        self.value = value
    
    def get_value(self):
        return self.value
    
    def set_value(self, new_value):
        self.value = new_value

obj = Methods(10)
print(obj.get_value())  # 10
obj.set_value(20)
print(obj.get_value())  # 20
```

**Using `getattr` and `setattr`:**

```python
class Dynamic:
    def __init__(self):
        self.attr1 = "value1"

obj = Dynamic()

# Dynamic attribute access
value = getattr(obj, "attr1")  # "value1"
value = getattr(obj, "attr2", "default")  # "default"

# Dynamic attribute setting
setattr(obj, "attr2", "value2")
print(obj.attr2)  # "value2"

# Check if attribute exists
print(hasattr(obj, "attr1"))  # True
print(hasattr(obj, "attr3"))  # False
```


***

### Updating Attributes

**Direct update:**

```python
class Update:
    def __init__(self):
        self.value = 10

obj = Update()
obj.value = 20  # Direct update
print(obj.value)  # 20
```

**Through method:**

```python
class Counter:
    def __init__(self):
        self.count = 0
    
    def increment(self):
        self.count += 1
    
    def decrement(self):
        self.count -= 1
    
    def set_count(self, value):
        if value >= 0:
            self.count = value

c = Counter()
c.increment()
c.increment()
print(c.count)  # 2
c.set_count(10)
print(c.count)  # 10
```

**With validation:**

```python
class Validated:
    def __init__(self, age):
        self.age = age
    
    def set_age(self, new_age):
        if new_age < 0 or new_age > 150:
            raise ValueError("Invalid age")
        self.age = new_age

p = Validated(25)
p.set_age(30)  # Works
# p.set_age(200)  # ERROR: ValueError
```


***

### Internal Working of Objects

**`id()` function:**

```python
obj1 = [1, 2, 3]
obj2 = [1, 2, 3]

print(id(obj1))  # Unique identifier (memory address)
print(id(obj2))  # Different identifier

print(obj1 is obj2)  # False (different objects)
print(obj1 == obj2)  # True (equal values)
```

**`type()` function:**

```python
class Person:
    pass

obj = Person()

print(type(obj))  # <class 'Person'>
print(type(5))    # <class 'int'>
print(type("hi")) # <class 'str'>
```

**`isinstance()` function:**

```python
class Animal:
    pass

class Dog(Animal):
    pass

dog = Dog()

print(isinstance(dog, Dog))    # True
print(isinstance(dog, Animal)) # True (inheritance)
print(isinstance(dog, str))    # False
```

**Object memory layout:**

```
Object Memory:
┌─────────────────────┐
│ __class__ → Class   │
│ __dict__ → {        │
│   attr1: value1,    │
│   attr2: value2     │
│ }                   │
│ __id__ → 123456     │
└─────────────────────┘
```

**References vs objects:**

```python
obj1 = [1, 2, 3]  # Create object, obj1 references it
obj2 = obj1       # obj2 references same object

obj1.append(4)
print(obj2)  # [1, 2, 3, 4] (same object!)

obj2 = [1, 2, 3]  # obj2 references NEW object
obj1.append(5)
print(obj2)  # [1, 2, 3] (different object)
```

**Identity vs equality:**

```python
class Compare:
    def __init__(self, value):
        self.value = value
    
    def __eq__(self, other):
        return self.value == other.value

obj1 = Compare(5)
obj2 = Compare(5)
obj3 = obj1

print(obj1 == obj2)  # True (equality - same value)
print(obj1 is obj2)  # False (identity - different objects)
print(obj1 is obj3)  # True (identity - same object)
```

**Attribute storage:**

```python
class Storage:
    def __init__(self):
        self.a = 1
        self.b = 2

obj = Storage()

print(obj.__dict__)  # {'a': 1, 'b': 2}

# Add attribute
obj.c = 3
print(obj.__dict__)  # {'a': 1, 'b': 2, 'c': 3}
```

**Garbage collection basics:**

```python
import gc

class GC:
    def __del__(self):
        print("Object destroyed")

obj = GC()
del obj  # May trigger garbage collection
gc.collect()  # Force garbage collection
```


***

## Common OOP Mistakes

```python
# 1. Missing self parameter
class Bad1:
    def greet():  # Missing self!
        return "Hello"

# 2. Not using self for attributes
class Bad2:
    def __init__(self, name):
        name = name  # Local variable, not attribute
    
    def get_name(self):
        return name  # ERROR

# 3. Modifying class variable through instance
class Bad3:
    count = 0
    
    def increment(self):
        self.count += 1  # Creates instance variable!

# 4. Returning value from __init__
class Bad4:
    def __init__(self, value):
        self.value = value
        return self.value  # ERROR!

# 5. Using private attributes incorrectly
class Bad5:
    def __init__(self):
        self.__private = 5
    
    def get(self):
        return self.__private

obj = Bad5()
print(obj.__private)  # ERROR! Use obj.get()

# 6. Mutable default arguments
class Bad6:
    def __init__(self, items=[]):  # Mutable default!
        self.items = items

# 7. Not calling parent __init__
class Parent:
    def __init__(self):
        self.parent_attr = 1

class ChildBad(Parent):
    def __init__(self):
        self.child_attr = 2  # Missing parent.__init__()

# 8. Overusing inheritance
class Bad8:
    # Deep inheritance hierarchy
    pass

# 9. Not using data validation
class Bad9:
    def __init__(self, age):
        self.age = age  # No validation!

# 10. Violating encapsulation
class Bad10:
    def __init__(self):
        self.balance = 0  # Public, can be modified directly
    
    def withdraw(self, amount):
        self.balance -= amount  # No validation!
```


***

## Best Practices

```python
# 1. Use meaningful names
class Good1:
    def __init__(self, email_address, account_balance):
        self.email_address = email_address
        self.account_balance = account_balance

# 2. Keep classes focused (Single Responsibility)
class Good2_email:
    def __init__(self, email):
        self.email = email
    
    def send(self, message):
        pass

class Good2_balance:
    def __init__(self, balance):
        self.balance = balance
    
    def deposit(self, amount):
        pass

# 3. Use docstrings
class Good3:
    """Represents a user account."""
    
    def __init__(self, username):
        """Initialize user with username."""
        self.username = username

# 4. Validate input
class Good4:
    def __init__(self, age):
        if age < 0 or age > 150:
            raise ValueError("Invalid age")
        self.age = age

# 5. Use encapsulation
class Good5:
    def __init__(self):
        self._balance = 0  # Protected
    
    def get_balance(self):
        return self._balance
    
    def set_balance(self, value):
        if value >= 0:
            self._balance = value

# 6. Prefer composition over inheritance
class Good6_credit_card:
    def pay(self, amount):
        print(f"Payng ${amount} with credit card")

class Good6_user:
    def __init__(self):
        self.payment = Good6_credit_card()
    
    def buy(self, amount):
        self.payment.pay(amount)

# 7. Use class/static methods appropriately
class Good7:
    count = 0
    
    @classmethod
    def get_count(cls):
        return cls.count
    
    @staticmethod
    def is_valid(age):
        return 0 <= age <= 150

# 8. Implement __str__ and __repr__
class Good8:
    def __init__(self, name):
        self.name = name
    
    def __str__(self):
        return f"User: {self.name}"
    
    def __repr__(self):
        return f"Good8('{self.name}')"

# 9. Avoid mutable default arguments
class Good9:
    def __init__(self, items=None):
        self.items = items if items is not None else []

# 10. Use type hints
class Good10:
    def __init__(self, name: str, age: int):
        self.name = name
        self.age = age
    
    def greet(self) -> str:
        return f"Hello, {self.name}"
```


***

## Performance Considerations

```python
# 1. __slots__ for memory optimization
class Normal:
    def __init__(self, x, y):
        self.x = x
        self.y = y

class Optimized:
    __slots__ = ['x', 'y']
    
    def __init__(self, x, y):
        self.x = x
        self.y = y

# Normal: ~500 bytes per instance
# Optimized: ~100 bytes per instance

# 2. Avoid dynamic attribute creation
class Slow:
    def __init__(self):
        pass
    
    def add_attr(self, name, value):
        setattr(self, name, value)  # Slow

class Fast:
    def __init__(self):
        self.attr1 = None
        self.attr2 = None  # Pre-define

# 3. Use local variables in methods
class Local:
    def process(self, data):
        result = []
        for item in data:
            result.append(item * 2)
        return result

# 4. Minimize method calls
class MinCall:
    def get_value(self):
        return self._value
    
    def process(self):
        # Bad: multiple calls
        for i in range(1000):
            x = self.get_value()
            y = self.get_value()
            z = self.get_value()
        
        # Good: cache result
        value = self.get_value()
        for i in range(1000):
            x = value
            y = value
            z = value

# 5. Use built-in methods
class BuiltIn:
    def sum_list(self, data):
        # Bad: manual loop
        total = 0
        for x in data:
            total += x
        return total
        
        # Good: built-in
        return sum(data)

# 6. Avoid deep inheritance
class Base:
    pass

class Level1(Base):
    pass

class Level2(Level1):
    pass

# Attribute lookup slower with deep inheritance

# 7. Use generator expressions
class Generator:
    def process(self, data):
        # Bad: list
        result = [x * 2 for x in data]
        
        # Good: generator
        result = (x * 2 for x in data)

# 8. Cache computed values
class Cached:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self._area = None
    
    @property
    def area(self):
        if self._area is None:
            self._area = self.x * self.y
        return self._area
```


***

## UML-style Design Examples

```
┌─────────────────────────────────────┐
│          Vehicle                    │
├─────────────────────────────────────┤
│ # Class Attributes:                 │
│   total_vehicles: int = 0           │
├─────────────────────────────────────┤
│ - Instance Attributes:              │
│   - make: str                       │
│   - model: str                      │
│   - year: int                       │
│   - speed: int = 0                  │
├─────────────────────────────────────┤
│ + Methods:                          │
│   + __init__(make, model, year)     │
│   + start(): void                   │
│   + accelerate(amount): void        │
│   + brake(amount): void             │
│   + get_speed(): int                │
│   + __str__(): str                  │
│   @classmethod                      │
│   + get_total(): int                │
└─────────────────────────────────────┘
         ▲
         | inherits
         │
┌─────────────────────────────────────┐
│           Car                       │
├─────────────────────────────────────┤
│ - Instance Attributes:              │
│   - num_doors: int                  │
│   - car_type: str                   │
├─────────────────────────────────────┤
│ + Methods:                          │
│   + __init__(make, model, year,     │
│              num_doors, car_type)   │
│   + honk(): str                     │
│   + override accelerate(amount):    │
│       void                          │
└─────────────────────────────────────┘
         ▲
         | inherits
         │
┌─────────────────────────────────────┐
│        ElectricCar                  │
├─────────────────────────────────────┤
│ - Instance Attributes:              │
│   - battery_capacity: int           │
│   - battery_level: int              │
├─────────────────────────────────────┤
│ + Methods:                          │
│   + __init__(make, model, year,     │
│              doors, battery_cap)    │
│   + charge(amount): void            │
│   + override accelerate(amount):    │
│       void                          │
└─────────────────────────────────────┘
```

**Implementation:**

```python
class Vehicle:
    total_vehicles = 0
    
    def __init__(self, make, model, year):
        self.make = make
        self.model = model
        self.year = year
        self.speed = 0
        Vehicle.total_vehicles += 1
    
    def start(self):
        print(f"{self.make} {self.model} started")
    
    def accelerate(self, amount):
        self.speed += amount
    
    def brake(self, amount):
        self.speed = max(0, self.speed - amount)
    
    def get_speed(self):
        return self.speed
    
    def __str__(self):
        return f"{self.year} {self.make} {self.model}"
    
    @classmethod
    def get_total(cls):
        return cls.total_vehicles


class Car(Vehicle):
    def __init__(self, make, model, year, num_doors, car_type):
        super().__init__(make, model, year)
        self.num_doors = num_doors
        self.car_type = car_type
    
    def honk(self):
        return "Honk honk!"
    
    def accelerate(self, amount):
        super().accelerate(amount)
        print(f"{self.car_type} accelerating...")


class ElectricCar(Car):
    def __init__(self, make, model, year, num_doors, battery_capacity):
        super().__init__(make, model, year, num_doors, "Electric")
        self.battery_capacity = battery_capacity
        self.battery_level = battery_capacity
    
    def charge(self, amount):
        self.battery_level = min(self.battery_capacity, 
                                  self.battery_level + amount)
    
    def accelerate(self, amount):
        if self.battery_level > 0:
            super().accelerate(amount)
            self.battery_level -= 1
            print("Electric car accelerating silently...")
        else:
            print("Battery empty!")
```


***

## Real-world OOP Models

```python
# 1. Bank System
class Account:
    def __init__(self, account_number, owner, balance=0):
        self.account_number = account_number
        self.owner = owner
        self.balance = balance
        self.transactions = []
    
    def deposit(self, amount):
        if amount > 0:
            self.balance += amount
            self.transactions.append(('deposit', amount))
            return True
        return False
    
    def withdraw(self, amount):
        if 0 < amount <= self.balance:
            self.balance -= amount
            self.transactions.append(('withdraw', amount))
            return True
        return False
    
    def get_history(self):
        return self.transactions


class Bank:
    def __init__(self, name):
        self.name = name
        self.accounts = {}
    
    def create_account(self, owner, initial_balance```

