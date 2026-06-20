# The Complete Guide to Python Dictionaries

*From Absolute Beginner to Advanced Professional*

---

## Table of Contents

1. [Introduction to Dictionaries](#1-introduction-to-dictionaries)
2. [Creating Dictionaries](#2-creating-dictionaries)
3. [Dictionary Anatomy](#3-dictionary-anatomy)
4. [Accessing Dictionary Data](#4-accessing-dictionary-data)
5. [Adding and Updating Data](#5-adding-and-updating-data)
6. [Removing Dictionary Items](#6-removing-dictionary-items)
7. [Dictionary Views](#7-dictionary-views)
8. [Dictionary Iteration](#8-dictionary-iteration)
9. [Dictionary Methods](#9-dictionary-methods)
10. [Dictionary Comprehensions](#10-dictionary-comprehensions)
11. [Nested Dictionaries](#11-nested-dictionaries)
12. [Dictionary Merging](#12-dictionary-merging)
13. [Dictionary Copying](#13-dictionary-copying)
14. [Dictionary Sorting](#14-dictionary-sorting)
15. [Dictionary Searching](#15-dictionary-searching)
16. [Dictionary Performance](#16-dictionary-performance)
17. [Advanced Dictionary Features](#17-advanced-dictionary-features)
18. [collections Module Dictionaries](#18-collections-module-dictionaries)
19. [Dictionaries in Real Applications](#19-dictionaries-in-real-applications)
20. [Dictionaries and JSON](#20-dictionaries-and-json)
21. [Common Mistakes](#21-common-mistakes)
22. [Best Practices](#22-best-practices)
23. [Interview Preparation](#23-interview-preparation)
24. [Coding Exercises](#24-coding-exercises)
25. [Mini Projects](#25-mini-projects)
26. [Dictionary Cheat Sheet](#26-dictionary-cheat-sheet)
27. [Final Summary](#27-final-summary)

---

## 1. Introduction to Dictionaries

### What is a Dictionary

A **dictionary** in Python is a built-in data structure that stores data as **key-value pairs**. Think of it like a real-world dictionary: you look up a *word* (the key) to find its *definition* (the value). Instead of accessing data by a numeric position (like in a list), you access it by a meaningful label.

```python
person = {
    "name": "Alice",
    "age": 30,
    "city": "New York"
}
print(person["name"])
```

**Output:**
```
Alice
```

### Why Dictionaries Exist

Dictionaries solve a fundamental problem: **fast lookup of data using a meaningful identifier instead of a numeric index**. If you had to store a phone book using a list, you would need to scan every entry to find a name. With a dictionary, you go straight to the value using the key, almost instantly, regardless of how large the dictionary is.

### Dictionary Characteristics

| Characteristic | Description |
|---|---|
| Ordered (3.7+) | Insertion order is preserved |
| Mutable | Can be changed after creation |
| Indexed by key | Not by integer position |
| Keys are unique | No duplicate keys allowed |
| Keys must be hashable | Strings, numbers, tuples (of hashables) |
| Values can be anything | Any data type, including other dicts |
| Dynamic size | Grows and shrinks as needed |

### Key-Value Mapping

A dictionary is fundamentally a **mapping**: each key maps to exactly one value.

```
Key          Value
-----------------------
"name"   --> "Alice"
"age"    --> 30
"city"   --> "New York"
```

### Real-world Examples

- A **phone book**: name → phone number
- A **student record**: subject → grade
- A **shopping cart**: product → price
- A **JSON API response**: field → value
- A **configuration file**: setting name → setting value

### Dictionary vs List

| Feature | Dictionary | List |
|---|---|---|
| Access by | Key | Index (integer) |
| Order | Insertion-ordered | Insertion-ordered |
| Lookup speed | O(1) average | O(n) for search |
| Duplicates | Keys must be unique | Allows duplicate values |
| Use case | Labeled data | Sequential data |

```python
# List - access by position
fruits = ["apple", "banana", "cherry"]
print(fruits[1])  # banana

# Dictionary - access by key
fruit_colors = {"apple": "red", "banana": "yellow"}
print(fruit_colors["banana"])  # yellow
```

### Dictionary vs Tuple

| Feature | Dictionary | Tuple |
|---|---|---|
| Mutability | Mutable | Immutable |
| Structure | Key-value pairs | Ordered sequence of values |
| Access | By key | By index |
| Use case | Labeled, changeable data | Fixed grouping of values |

### Dictionary vs Set

| Feature | Dictionary | Set |
|---|---|---|
| Stores | Key-value pairs | Unique values only |
| Syntax | `{"a": 1}` | `{"a", "b"}` |
| Lookup | By key | By value membership |
| Duplicates | Keys unique | Elements unique |

> **Note:** `{}` creates an **empty dictionary**, not an empty set. To create an empty set, use `set()`.

---

## 2. Creating Dictionaries

### 2.1 Using `{}` (Literal Syntax)

The most common and readable way to create a dictionary.

```python
empty_dict = {}
person = {"name": "Bob", "age": 25}
print(person)
```

**Output:**
```
{'name': 'Bob', 'age': 25}
```

### 2.2 Using `dict()` Constructor

```python
person = dict(name="Bob", age=25)
print(person)
```

**Output:**
```
{'name': 'Bob', 'age': 25}
```

> Keys created this way must be valid Python identifiers (no spaces, can't start with a digit) since they're passed as keyword arguments.

### 2.3 dict() with Keyword Arguments

```python
config = dict(host="localhost", port=8080, debug=True)
print(config)
```

**Output:**
```
{'host': 'localhost', 'port': 8080, 'debug': True}
```

### 2.4 dict() from a List of Tuples

```python
pairs = [("a", 1), ("b", 2), ("c", 3)]
d = dict(pairs)
print(d)
```

**Output:**
```
{'a': 1, 'b': 2, 'c': 3}
```

### 2.5 dict() from Tuple of Tuples

```python
pairs = (("x", 10), ("y", 20))
d = dict(pairs)
print(d)
```

**Output:**
```
{'x': 10, 'y': 20}
```

### 2.6 Dictionary Comprehension

```python
squares = {x: x**2 for x in range(1, 6)}
print(squares)
```

**Output:**
```
{1: 1, 2: 4, 3: 9, 4: 16, 5: 25}
```

### 2.7 `fromkeys()`

Creates a dictionary from an iterable of keys, all sharing the same value.

```python
keys = ["a", "b", "c"]
d = dict.fromkeys(keys, 0)
print(d)
```

**Output:**
```
{'a': 0, 'b': 0, 'c': 0}
```

> **Warning:** If the default value is a mutable object (like a list), all keys will **share the same reference**.

```python
d = dict.fromkeys(["a", "b"], [])
d["a"].append(1)
print(d)
```

**Output:**
```
{'a': [1], 'b': [1]}
```

### 2.8 Using `zip()`

```python
keys = ["name", "age", "city"]
values = ["Charlie", 35, "Boston"]
d = dict(zip(keys, values))
print(d)
```

**Output:**
```
{'name': 'Charlie', 'age': 35, 'city': 'Boston'}
```

### 2.9 Using `copy()`

```python
original = {"a": 1, "b": 2}
duplicate = original.copy()
duplicate["c"] = 3
print(original)
print(duplicate)
```

**Output:**
```
{'a': 1, 'b': 2}
{'a': 1, 'b': 2, 'c': 3}
```

### 2.10 Nested Dictionary Creation

```python
company = {
    "name": "TechCorp",
    "address": {
        "street": "123 Main St",
        "city": "Springfield"
    },
    "employees": [
        {"name": "Alice", "role": "Engineer"},
        {"name": "Bob", "role": "Manager"}
    ]
}
print(company["address"]["city"])
```

**Output:**
```
Springfield
```

---

## 3. Dictionary Anatomy

### Keys

A **key** is the unique identifier used to access a value. Keys must be **hashable** — meaning they have a fixed hash value for their lifetime (strings, numbers, tuples of hashables).

### Values

A **value** is the data associated with a key. Values can be of **any type**, including lists, dictionaries, functions, or custom objects, and they don't need to be unique.

### Key-Value Pairs

Each entry in a dictionary is a key-value pair, sometimes called an **item**.

```python
d = {"key": "value"}
for item in d.items():
    print(item)
```

**Output:**
```
('key', 'value')
```

### Hashable Keys

```python
valid_keys = {1: "int", "a": "str", (1, 2): "tuple", 3.14: "float"}
print(valid_keys)

# Invalid: lists are NOT hashable
try:
    bad = {[1, 2]: "value"}
except TypeError as e:
    print("Error:", e)
```

**Output:**
```
{1: 'int', 'a': 'str', (1, 2): 'tuple', 3.14: 'float'}
Error: unhashable type: 'list'
```

### Mutable Values

Values, unlike keys, can be mutable.

```python
d = {"scores": [90, 85, 70]}
d["scores"].append(100)
print(d)
```

**Output:**
```
{'scores': [90, 85, 70, 100]}
```

### Unique Keys

A dictionary can never have two identical keys at the same time.

### Duplicate Keys Behavior

If you define a literal dictionary with a duplicate key, the **last one wins**.

```python
d = {"a": 1, "a": 2, "a": 3}
print(d)
```

**Output:**
```
{'a': 3}
```

---

## 4. Accessing Dictionary Data

### `[]` (Square Bracket Access)

```python
person = {"name": "Dana"}
print(person["name"])

try:
    print(person["age"])
except KeyError as e:
    print("KeyError:", e)
```

**Output:**
```
Dana
KeyError: 'age'
```

### `get()`

`get()` returns `None` (or a custom default) instead of raising an error.

```python
print(person.get("age"))           # None
print(person.get("age", "N/A"))    # N/A
```

**Output:**
```
None
N/A
```

### `setdefault()`

Returns the value if the key exists; otherwise inserts the key with a default value **and** returns that default.

```python
person = {"name": "Dana"}
age = person.setdefault("age", 18)
print(age)
print(person)
```

**Output:**
```
18
{'name': 'Dana', 'age': 18}
```

### Comparison Table

| Method | Missing Key Behavior | Modifies Dict? |
|---|---|---|
| `d[key]` | Raises `KeyError` | No |
| `d.get(key)` | Returns `None` | No |
| `d.get(key, default)` | Returns `default` | No |
| `d.setdefault(key, default)` | Inserts `default` | **Yes** |

---

## 5. Adding and Updating Data

### Assignment

```python
d = {"a": 1}
d["b"] = 2          # add new key
d["a"] = 100         # update existing key
print(d)
```

**Output:**
```
{'a': 100, 'b': 2}
```

### `update()`

```python
d = {"a": 1, "b": 2}
d.update({"b": 20, "c": 3})
print(d)

d.update(d=4, e=5)
print(d)
```

**Output:**
```
{'a': 1, 'b': 20, 'c': 3}
{'a': 1, 'b': 20, 'c': 3, 'd': 4, 'e': 5}
```

### `setdefault()` for Conditional Adding

```python
inventory = {}
for item in ["apple", "banana", "apple"]:
    inventory.setdefault(item, 0)
    inventory[item] += 1
print(inventory)
```

**Output:**
```
{'apple': 2, 'banana': 1}
```

### Merge Operators (Python 3.9+)

```python
a = {"x": 1, "y": 2}
b = {"y": 20, "z": 3}

merged = a | b      # new dict
print(merged)

a |= b               # in-place merge
print(a)
```

**Output:**
```
{'x': 1, 'y': 20, 'z': 3}
{'x': 1, 'y': 20, 'z': 3}
```

### Nested Updates

```python
config = {"db": {"host": "localhost", "port": 5432}}
config["db"]["port"] = 5433
print(config)
```

**Output:**
```
{'db': {'host': 'localhost', 'port': 5433}}
```

---

## 6. Removing Dictionary Items

### `del`

```python
d = {"a": 1, "b": 2}
del d["a"]
print(d)

try:
    del d["x"]
except KeyError as e:
    print("KeyError:", e)
```

**Output:**
```
{'b': 2}
KeyError: 'x'
```

### `pop()`

Removes a key and **returns its value**. Accepts an optional default to avoid errors.

```python
d = {"a": 1, "b": 2}
val = d.pop("a")
print(val, d)

val2 = d.pop("z", "not found")
print(val2)
```

**Output:**
```
1 {'b': 2}
not found
```

### `popitem()`

Removes and returns the **last inserted** key-value pair (LIFO order) as a tuple.

```python
d = {"a": 1, "b": 2, "c": 3}
item = d.popitem()
print(item, d)
```

**Output:**
```
('c', 3) {'a': 1, 'b': 2}
```

### `clear()`

Removes all items, leaving an empty dictionary.

```python
d = {"a": 1, "b": 2}
d.clear()
print(d)
```

**Output:**
```
{}
```

### Comparison

| Method | Removes | Returns | Raises on missing key |
|---|---|---|---|
| `del d[k]` | Specific key | Nothing | Yes |
| `d.pop(k)` | Specific key | The value | Yes (unless default given) |
| `d.popitem()` | Last inserted pair | `(key, value)` tuple | Yes if empty |
| `d.clear()` | Everything | Nothing | No |

---

## 7. Dictionary Views

### `keys()`, `values()`, `items()`

```python
d = {"a": 1, "b": 2}
print(d.keys())
print(d.values())
print(d.items())
```

**Output:**
```
dict_keys(['a', 'b'])
dict_values([1, 2])
dict_items([('a', 1), ('b', 2)])
```

### Dynamic Views

Views are **live** — they reflect changes made to the dictionary automatically.

```python
d = {"a": 1}
keys = d.keys()
d["b"] = 2
print(keys)
```

**Output:**
```
dict_keys(['a', 'b'])
```

### Iteration over Views

```python
for k in d.keys():
    print(k)
```

### Conversions

```python
d = {"a": 1, "b": 2}
key_list = list(d.keys())
value_list = list(d.values())
item_list = list(d.items())
print(key_list, value_list, item_list)
```

**Output:**
```
['a', 'b'] [1, 2] [('a', 1), ('b', 2)]
```

---

## 8. Dictionary Iteration

### Iterating Keys

```python
d = {"a": 1, "b": 2}
for key in d:           # default iteration is over keys
    print(key)
```

### Iterating Values

```python
for value in d.values():
    print(value)
```

### Iterating Items

```python
for key, value in d.items():
    print(key, "->", value)
```

**Output:**
```
a -> 1
b -> 2
```

### Using `enumerate()`

```python
for index, (key, value) in enumerate(d.items()):
    print(index, key, value)
```

**Output:**
```
0 a 1
1 b 2
```

### Nested Iteration

```python
students = {
    "Alice": {"math": 90, "science": 85},
    "Bob": {"math": 70, "science": 75}
}
for student, subjects in students.items():
    for subject, score in subjects.items():
        print(f"{student} - {subject}: {score}")
```

**Output:**
```
Alice - math: 90
Alice - science: 85
Bob - math: 70
Bob - science: 75
```

---

## 9. Dictionary Methods

### `clear()`
- **Syntax:** `d.clear()`
- **Parameters:** None
- **Returns:** `None`
- **Example:**
```python
d = {"a": 1}
d.clear()
print(d)  # {}
```
- **Common mistake:** Confusing `d.clear()` with `d = {}` — `clear()` mutates the existing object in place, which matters if other variables reference the same dictionary.

### `copy()`
- **Syntax:** `d.copy()`
- **Parameters:** None
- **Returns:** A shallow copy
- **Example:**
```python
a = {"x": 1}
b = a.copy()
```
- **Common mistake:** Assuming `copy()` deep-copies nested structures (it does not).

### `fromkeys(iterable, value=None)`
- **Syntax:** `dict.fromkeys(iterable, value)`
- **Parameters:** `iterable` (keys), `value` (optional default)
- **Returns:** New dictionary
- **Example:**
```python
dict.fromkeys(["a", "b"], 0)  # {'a': 0, 'b': 0}
```
- **Common mistake:** Using a mutable default that gets shared across all keys.

### `get(key, default=None)`
- **Syntax:** `d.get(key, default)`
- **Returns:** Value or default
- **Example:**
```python
{"a": 1}.get("b", 0)  # 0
```

### `items()`
- **Syntax:** `d.items()`
- **Returns:** A view of `(key, value)` tuples
- **Example:**
```python
list({"a": 1}.items())  # [('a', 1)]
```

### `keys()`
- **Syntax:** `d.keys()`
- **Returns:** A view of keys
- **Example:**
```python
list({"a": 1}.keys())  # ['a']
```

### `pop(key, default)`
- **Syntax:** `d.pop(key, default)`
- **Returns:** Removed value, or default
- **Common mistake:** Forgetting that without a default, a missing key raises `KeyError`.

### `popitem()`
- **Syntax:** `d.popitem()`
- **Returns:** Last `(key, value)` pair
- **Common mistake:** Assuming it removes a random item — since 3.7 it removes the **last inserted** item.

### `setdefault(key, default=None)`
- **Syntax:** `d.setdefault(key, default)`
- **Returns:** Existing or newly-set value
- **Common mistake:** Using it inside a loop when `defaultdict` would be cleaner.

### `update(other)`
- **Syntax:** `d.update(other)`
- **Parameters:** Dict, iterable of pairs, or keyword arguments
- **Returns:** `None` (mutates in place)
- **Common mistake:** Expecting `update()` to return the merged dictionary — it returns `None`.

### `values()`
- **Syntax:** `d.values()`
- **Returns:** A view of values
- **Example:**
```python
list({"a": 1}.values())  # [1]
```

---

## 10. Dictionary Comprehensions

### Basic Comprehension

```python
squares = {x: x*x for x in range(5)}
print(squares)
```

**Output:**
```
{0: 0, 1: 1, 2: 4, 3: 9, 4: 16}
```

### Conditional Comprehension

```python
evens = {x: x*x for x in range(10) if x % 2 == 0}
print(evens)
```

**Output:**
```
{0: 0, 2: 4, 4: 16, 6: 36, 8: 64}
```

### Nested Comprehension

```python
matrix = {i: {j: i*j for j in range(3)} for i in range(3)}
print(matrix)
```

**Output:**
```
{0: {0: 0, 1: 0, 2: 0}, 1: {0: 0, 1: 1, 2: 2}, 2: {0: 0, 1: 2, 2: 4}}
```

### Filtering

```python
prices = {"apple": 1.5, "banana": 0.5, "mango": 3.0}
cheap = {k: v for k, v in prices.items() if v < 2}
print(cheap)
```

**Output:**
```
{'apple': 1.5, 'banana': 0.5}
```

### Transformations

```python
names = ["alice", "bob"]
upper = {name: name.upper() for name in names}
print(upper)
```

**Output:**
```
{'alice': 'ALICE', 'bob': 'BOB'}
```

### Reversing Key/Value

```python
original = {"a": 1, "b": 2}
reversed_dict = {v: k for k, v in original.items()}
print(reversed_dict)
```

**Output:**
```
{1: 'a', 2: 'b'}
```

---

## 11. Nested Dictionaries

### Creation

```python
students = {
    "alice": {"math": 90, "science": 95},
    "bob": {"math": 80, "science": 70}
}
```

### Access

```python
print(students["alice"]["math"])
```

**Output:**
```
90
```

### Update

```python
students["alice"]["math"] = 99
print(students["alice"])
```

**Output:**
```
{'math': 99, 'science': 95}
```

### Deletion

```python
del students["bob"]["science"]
print(students["bob"])
```

**Output:**
```
{'math': 80}
```

### Iteration

```python
for student, grades in students.items():
    avg = sum(grades.values()) / len(grades)
    print(f"{student}: avg = {avg}")
```

### Real-world: Student Database

```python
student_db = {
    "S001": {"name": "Alice", "grades": {"math": 90, "science": 85}},
    "S002": {"name": "Bob", "grades": {"math": 75, "science": 80}}
}
```

### Real-world: Employee Records

```python
employees = {
    "E001": {"name": "Carol", "department": "Engineering", "salary": 95000},
    "E002": {"name": "Dave", "department": "Sales", "salary": 60000}
}
```

### Real-world: API Response Structure

```python
api_response = {
    "status": "success",
    "data": {
        "user": {"id": 1, "name": "Eve"},
        "posts": [{"id": 101, "title": "Hello World"}]
    }
}
print(api_response["data"]["user"]["name"])
```

**Output:**
```
Eve
```

---

## 12. Dictionary Merging

### `update()`

```python
a = {"x": 1}
b = {"y": 2}
a.update(b)
print(a)
```

**Output:**
```
{'x': 1, 'y': 2}
```

### Unpacking (`**`)

```python
a = {"x": 1}
b = {"y": 2}
merged = {**a, **b}
print(merged)
```

**Output:**
```
{'x': 1, 'y': 2}
```

### Union Operator `|` (Python 3.9+)

```python
merged = a | b
print(merged)
```

### `|=` In-place Merge

```python
a |= b
print(a)
```

### Comparison Table

| Method | Creates new dict | Mutates original | Min Python |
|---|---|---|---|
| `update()` | No | Yes | All |
| `{**a, **b}` | Yes | No | 3.5+ |
| `a | b` | Yes | No | 3.9+ |
| `a |= b` | No | Yes | 3.9+ |

---

## 13. Dictionary Copying

### Assignment (Not a Copy!)

```python
a = {"x": 1}
b = a
b["x"] = 100
print(a)
```

**Output:**
```
{'x': 100}
```

`b` is just another name for the same object — no copy occurred.

```
a ----\
       --> {'x': 100}
b ----/
```

### Shallow Copy

```python
import copy
a = {"x": [1, 2, 3]}
b = a.copy()
b["x"].append(4)
print(a)
```

**Output:**
```
{'x': [1, 2, 3, 4]}
```

A shallow copy duplicates the **top-level** dictionary but nested objects are still shared.

```
a --> {'x': ref} ----\
                       --> [1, 2, 3, 4]
b --> {'x': ref} ----/
```

### Deep Copy

```python
import copy
a = {"x": [1, 2, 3]}
b = copy.deepcopy(a)
b["x"].append(4)
print(a)
print(b)
```

**Output:**
```
{'x': [1, 2, 3]}
{'x': [1, 2, 3, 4]}
```

Deep copy recursively duplicates all nested objects, so changes to `b` no longer affect `a`.

---

## 14. Dictionary Sorting

### Sort by Key

```python
d = {"banana": 3, "apple": 1, "cherry": 2}
sorted_by_key = dict(sorted(d.items()))
print(sorted_by_key)
```

**Output:**
```
{'apple': 1, 'banana': 3, 'cherry': 2}
```

### Sort by Value

```python
sorted_by_value = dict(sorted(d.items(), key=lambda item: item[1]))
print(sorted_by_value)
```

**Output:**
```
{'apple': 1, 'cherry': 2, 'banana': 3}
```

### Descending Order

```python
desc = dict(sorted(d.items(), key=lambda item: item[1], reverse=True))
print(desc)
```

**Output:**
```
{'banana': 3, 'cherry': 2, 'apple': 1}
```

### Custom Sorting

```python
words = {"apple": 5, "fig": 1, "banana": 6}
by_length_then_value = dict(sorted(words.items(), key=lambda item: (len(item[0]), item[1])))
print(by_length_then_value)
```

**Output:**
```
{'fig': 1, 'apple': 5, 'banana': 6}
```

> **Note:** Dictionaries themselves cannot be "sorted in place" — sorting always produces an ordering you store in a new dict (which then preserves that insertion order).

---

## 15. Dictionary Searching

### Membership Testing (Keys)

```python
d = {"a": 1, "b": 2}
print("a" in d)
print("z" in d)
```

**Output:**
```
True
False
```

### Searching Values

```python
print(2 in d.values())
```

**Output:**
```
True
```

### Searching Key-Value Pairs

```python
print(("a", 1) in d.items())
```

**Output:**
```
True
```

### Finding a Key by Value

```python
d = {"a": 1, "b": 2, "c": 2}
keys_with_value_2 = [k for k, v in d.items() if v == 2]
print(keys_with_value_2)
```

**Output:**
```
['b', 'c']
```

---

## 16. Dictionary Performance

### Big-O Complexity

| Operation | Average Case | Worst Case |
|---|---|---|
| Lookup (`d[key]`) | O(1) | O(n) |
| Insertion | O(1) | O(n) |
| Deletion | O(1) | O(n) |
| Iteration | O(n) | O(n) |
| `key in d` | O(1) | O(n) |

> Dictionaries achieve near-constant time lookup because they use a hash-based structure internally. Worst case (O(n)) is extremely rare and typically only relevant with pathological hash collisions.

### Dict vs List Lookup Performance

```python
import time

n = 100_000
data_list = list(range(n))
data_dict = {i: True for i in range(n)}

start = time.time()
print(n - 1 in data_list)
print("List search:", time.time() - start)

start = time.time()
print(n - 1 in data_dict)
print("Dict search:", time.time() - start)
```

**Typical Output (illustrative):**
```
True
List search: 0.0021 seconds
True
Dict search: 0.0000 seconds
```

Dictionaries dramatically outperform lists for membership testing as data grows.

---

## 17. Advanced Dictionary Features

### Dictionary Unpacking

```python
def greet(name, age):
    print(f"Hello {name}, age {age}")

person = {"name": "Frank", "age": 40}
greet(**person)
```

**Output:**
```
Hello Frank, age 40
```

### Nested Unpacking

```python
defaults = {"a": 1, "b": 2}
overrides = {"b": 20, "c": 3}
final = {**defaults, **overrides}
print(final)
```

**Output:**
```
{'a': 1, 'b': 20, 'c': 3}
```

### Dynamic Creation

```python
fields = ["name", "age", "city"]
values = ["Grace", 28, "Austin"]
record = {field: value for field, value in zip(fields, values)}
print(record)
```

### Default Values with `defaultdict` (preview)

```python
from collections import defaultdict
d = defaultdict(int)
d["count"] += 1
print(d)
```

**Output:**
```
defaultdict(<class 'int'>, {'count': 1})
```

### Inversion (Swapping Keys and Values)

```python
original = {"a": 1, "b": 2, "c": 3}
inverted = {v: k for k, v in original.items()}
print(inverted)
```

**Output:**
```
{1: 'a', 2: 'b', 3: 'c'}
```

### Grouping

```python
words = ["apple", "ant", "banana", "bear", "cat"]
groups = {}
for word in words:
    first_letter = word[0]
    groups.setdefault(first_letter, []).append(word)
print(groups)
```

**Output:**
```
{'a': ['apple', 'ant'], 'b': ['banana', 'bear'], 'c': ['cat']}
```

---

## 18. collections Module Dictionaries

### `defaultdict`

`defaultdict` automatically creates a default value for missing keys, avoiding `KeyError`.

```python
from collections import defaultdict

# Counting
counts = defaultdict(int)
for letter in "mississippi":
    counts[letter] += 1
print(dict(counts))
```

**Output:**
```
{'m': 1, 'i': 4, 's': 4, 'p': 2}
```

```python
# Grouping
groups = defaultdict(list)
words = ["apple", "ant", "banana"]
for word in words:
    groups[word[0]].append(word)
print(dict(groups))
```

**Output:**
```
{'a': ['apple', 'ant'], 'b': ['banana']}
```

### `Counter`

`Counter` is a specialized dictionary subclass for counting hashable items.

```python
from collections import Counter

words = ["apple", "banana", "apple", "cherry", "banana", "apple"]
counter = Counter(words)
print(counter)
print(counter.most_common(2))
```

**Output:**
```
Counter({'apple': 3, 'banana': 2, 'cherry': 1})
[('apple', 3), ('banana', 2)]
```

### `OrderedDict`

Before Python 3.7, regular dictionaries did not guarantee order. `OrderedDict` explicitly preserves insertion order and offers extra methods like `move_to_end()`.

```python
from collections import OrderedDict

d = OrderedDict()
d["a"] = 1
d["b"] = 2
d.move_to_end("a")
print(d)
```

**Output:**
```
OrderedDict([('b', 2), ('a', 1)])
```

> Since Python 3.7+, regular `dict` also preserves insertion order, but `OrderedDict` is still useful for order-sensitive equality checks and `move_to_end()`.

### `ChainMap`

`ChainMap` groups multiple dictionaries into a single, updatable view — useful for layered configurations (e.g., defaults + user settings).

```python
from collections import ChainMap

defaults = {"theme": "light", "font_size": 12}
user_settings = {"theme": "dark"}

combined = ChainMap(user_settings, defaults)
print(combined["theme"])       # dark (user override)
print(combined["font_size"])   # 12 (falls back to defaults)
```

**Output:**
```
dark
12
```

---

## 19. Dictionaries in Real Applications

### Configuration Systems

```python
config = {
    "debug": False,
    "database": {"host": "localhost", "port": 5432},
    "allowed_hosts": ["example.com"]
}
```

### JSON Handling

Dictionaries map directly to JSON objects (see Section 20 for details).

### APIs

API responses are typically parsed into nested dictionaries for easy field access.

### Caching

```python
cache = {}

def expensive_computation(n):
    if n in cache:
        return cache[n]
    result = n ** 2  # pretend this is expensive
    cache[n] = result
    return result

print(expensive_computation(4))
print(expensive_computation(4))  # served from cache
```

### Counting Frequencies

```python
text = "the quick brown fox jumps over the lazy dog the fox runs"
frequency = {}
for word in text.split():
    frequency[word] = frequency.get(word, 0) + 1
print(frequency)
```

**Output:**
```
{'the': 3, 'quick': 1, 'brown': 1, 'fox': 2, 'jumps': 1, 'over': 1, 'lazy': 1, 'dog': 1, 'runs': 1}
```

### Data Aggregation

```python
sales = [
    {"region": "East", "amount": 100},
    {"region": "West", "amount": 150},
    {"region": "East", "amount": 200},
]
totals = {}
for sale in sales:
    totals[sale["region"]] = totals.get(sale["region"], 0) + sale["amount"]
print(totals)
```

**Output:**
```
{'East': 300, 'West': 150}
```

### Database-like Structures

```python
users_table = {
    1: {"name": "Alice", "email": "alice@example.com"},
    2: {"name": "Bob", "email": "bob@example.com"}
}
print(users_table[1]["email"])
```

**Output:**
```
alice@example.com
```

---

## 20. Dictionaries and JSON

### `json.loads()` — Parse JSON String into Dict

```python
import json

json_string = '{"name": "Helen", "age": 32, "active": true}'
data = json.loads(json_string)
print(data)
print(type(data))
```

**Output:**
```
{'name': 'Helen', 'age': 32, 'active': True}
<class 'dict'>
```

### `json.dumps()` — Convert Dict into JSON String

```python
person = {"name": "Helen", "age": 32, "active": True}
json_string = json.dumps(person, indent=2)
print(json_string)
```

**Output:**
```
{
  "name": "Helen",
  "age": 32,
  "active": true
}
```

### Reading JSON from a File

```python
with open("data.json", "r") as f:
    data = json.load(f)
```

### Writing JSON to a File

```python
with open("data.json", "w") as f:
    json.dump(person, f, indent=2)
```

### Handling API Responses

```python
# Simulating an API response payload
response_text = '{"status": "ok", "results": [{"id": 1}, {"id": 2}]}'
response = json.loads(response_text)
for result in response["results"]:
    print(result["id"])
```

**Output:**
```
1
2
```

> **Note:** JSON's `null` maps to Python's `None`, JSON `true`/`false` map to `True`/`False`, and JSON objects always map to Python `dict`.

---

## 21. Common Mistakes

### Mistake 1: Mutable Default Value Confusion

```python
# WRONG
def add_item(item, inventory={}):
    inventory[item] = inventory.get(item, 0) + 1
    return inventory

print(add_item("apple"))
print(add_item("banana"))  # Unexpectedly contains 'apple' too!
```

**Output:**
```
{'apple': 1}
{'apple': 1, 'banana': 1}
```

**Fix:**
```python
def add_item(item, inventory=None):
    if inventory is None:
        inventory = {}
    inventory[item] = inventory.get(item, 0) + 1
    return inventory
```

### Mistake 2: Accessing a Missing Key Directly

```python
# WRONG
d = {"a": 1}
print(d["b"])  # KeyError
```

**Fix:** Use `.get()` or check `in` first.

### Mistake 3: Overwriting Keys Accidentally

```python
d = {}
for i in [1, 1, 2]:
    d[i] = i * 10
print(d)  # {1: 10, 2: 20}, the second "1" overwrote the first
```

### Mistake 4: Shallow Copy Problems

```python
original = {"items": [1, 2, 3]}
copy1 = original.copy()
copy1["items"].append(4)
print(original)  # Also changed! {'items': [1, 2, 3, 4]}
```

**Fix:** Use `copy.deepcopy()` for nested mutable structures.

### Mistake 5: Modifying a Dictionary During Iteration

```python
d = {"a": 1, "b": 2, "c": 3}
try:
    for key in d:
        if key == "b":
            del d[key]  # RuntimeError
except RuntimeError as e:
    print("RuntimeError:", e)
```

**Output:**
```
RuntimeError: dictionary changed size during iteration
```

**Fix:** Iterate over a copy of the keys.
```python
for key in list(d.keys()):
    if key == "b":
        del d[key]
print(d)
```

**Output:**
```
{'a': 1, 'c': 3}
```

---

## 22. Best Practices

- **Naming conventions:** Use clear, descriptive variable names (`user_profile` instead of `d`); prefer singular names for dictionaries representing one record, and plural/`_by_` names for lookup tables (e.g. `users_by_id`).
- **Safe access:** Prefer `.get()` over direct indexing when a key might be missing.
- **Readability:** Use dictionary comprehensions for simple transformations, but fall back to explicit loops if the logic is complex — clarity beats cleverness.
- **Performance:** Use `in` for membership checks rather than scanning `.keys()` or `.values()` manually.
- **Maintainability:** Avoid deeply nested dictionaries (4+ levels) when a class or `dataclass` would be clearer.
- Use `defaultdict` or `setdefault()` instead of manually checking and initializing keys.
- Use `dict.fromkeys()` for quick initialization of dictionaries with a constant default — but never with a mutable default value.
- Prefer `**unpacking` or `|` for merging immutable, predictable configs instead of repeated `update()` calls.
- Always document the expected structure of complex nested dictionaries (or replace them with typed structures like `TypedDict` or `dataclass`).

---

## 23. Interview Preparation

### Beginner Questions (20)

1. **What is a dictionary in Python?**
   A built-in mutable data structure that stores data as unique key-value pairs.
2. **How do you create an empty dictionary?**
   `{}` or `dict()`.
3. **How do you access a value by key?**
   `d[key]` or `d.get(key)`.
4. **What happens if you access a missing key with `[]`?**
   A `KeyError` is raised.
5. **What does `.get()` return for a missing key by default?**
   `None`.
6. **Can dictionary keys be duplicated?**
   No, keys must be unique; duplicates overwrite earlier values.
7. **Can dictionary values be duplicated?**
   Yes, values can repeat freely.
8. **What types can be used as dictionary keys?**
   Any hashable, immutable type (strings, numbers, tuples of hashables).
9. **Are dictionaries ordered in modern Python?**
   Yes, since Python 3.7, insertion order is preserved.
10. **How do you add a new key-value pair?**
    `d[new_key] = value`.
11. **How do you remove a key from a dictionary?**
    `del d[key]`, `d.pop(key)`, or `d.popitem()`.
12. **What does `len(d)` return?**
    The number of key-value pairs.
13. **How do you check if a key exists?**
    `key in d`.
14. **What does `d.keys()` return?**
    A view object containing all keys.
15. **What does `d.values()` return?**
    A view object containing all values.
16. **What does `d.items()` return?**
    A view object of `(key, value)` tuples.
17. **How do you loop through key-value pairs?**
    `for k, v in d.items():`.
18. **Is a list a valid dictionary key?**
    No, because lists are mutable and unhashable.
19. **What is the difference between `[]` and `.get()`?**
    `[]` raises `KeyError` on missing keys; `.get()` returns `None` or a default.
20. **How do you merge two dictionaries (simple way)?**
    `d1.update(d2)` or `{**d1, **d2}`.

### Intermediate Questions (20)

1. **What is a dictionary comprehension? Give an example.**
   A concise way to build a dict: `{x: x**2 for x in range(5)}`.
2. **Explain the difference between shallow and deep copy.**
   Shallow copy duplicates the top-level dict only; nested objects remain shared. Deep copy recursively duplicates everything.
3. **What does `setdefault()` do?**
   Returns a key's value, inserting a default if the key is missing.
4. **How do you sort a dictionary by value?**
   `dict(sorted(d.items(), key=lambda x: x[1]))`.
5. **What is the time complexity of dictionary lookup?**
   O(1) on average.
6. **What is `defaultdict` used for?**
   Auto-initializing missing keys with a default factory (avoiding `KeyError`/manual checks).
7. **What is `Counter` used for?**
   Counting occurrences of hashable items efficiently.
8. **What's the difference between `update()` and the `|` operator?**
   `update()` mutates in place and returns `None`; `|` returns a new merged dictionary.
9. **How can you invert a dictionary (swap keys and values)?**
   `{v: k for k, v in d.items()}` (assuming values are unique and hashable).
10. **Why might modifying a dictionary while iterating raise an error?**
    Python detects the size change mid-iteration and raises `RuntimeError` to prevent undefined behavior.
11. **How do you safely modify a dictionary during iteration?**
    Iterate over a copy: `for k in list(d.keys()):`.
12. **What's the difference between `pop()` and `popitem()`?**
    `pop(key)` removes a specific key; `popitem()` removes the last inserted pair.
13. **How would you group a list of items by a property using a dictionary?**
    Use `setdefault()` or `defaultdict(list)` to append items under shared keys.
14. **What is `ChainMap` and when would you use it?**
    A view combining multiple dicts; useful for layered configs (defaults + overrides) without merging them.
15. **How do you convert a JSON string to a Python dictionary?**
    `json.loads(json_string)`.
16. **Can dictionary values be functions?**
    Yes — dictionaries can map keys to callables (useful for dispatch tables).
17. **What happens when you use `dict.fromkeys()` with a mutable default?**
    All keys share the same reference to that single mutable object.
18. **How do you merge nested dictionaries deeply?**
    There's no built-in deep merge; you typically write a recursive function or use a third-party library.
19. **What is the difference between `OrderedDict` and a regular `dict` today?**
    Both preserve order, but `OrderedDict` supports `move_to_end()` and treats order as significant in equality comparisons.
20. **How would you implement a simple cache using a dictionary?**
    Store computed results keyed by input; check `if key in cache` before recomputing.

### Advanced Questions (20)

1. **Why must dictionary keys be hashable, and what does that imply about mutability?**
   Hashing relies on a stable hash value; mutable objects could change their hash after insertion, breaking lookups, so Python disallows them as keys.
2. **How would you implement a multi-level (nested) default dictionary?**
   `defaultdict(lambda: defaultdict(int))` or nested `defaultdict(dict)`.
3. **What are the trade-offs of using deeply nested dictionaries vs classes/dataclasses?**
   Dictionaries are flexible and JSON-friendly but lack type safety, autocompletion, and validation; classes/dataclasses offer structure at the cost of flexibility.
4. **How does dictionary equality (`==`) work?**
   Two dictionaries are equal if they have the same keys mapped to equal values, regardless of insertion order.
5. **How would you efficiently merge a large number of dictionaries?**
   Use `ChainMap` for a lazy, non-copying view, or `functools.reduce` with `|` for a single merged copy.
6. **What's a real risk of using `dict.fromkeys()` for initializing counters?**
   If using a mutable default, all keys reference the same object, causing unintended shared state.
7. **How can you make a dictionary's keys order-independent for comparison or hashing?**
   Convert to a `frozenset` of items (if values are hashable) or sort items before comparison.
8. **Explain how `Counter` arithmetic works (e.g., `Counter` + `Counter`).**
   `Counter` supports `+`, `-`, `&`, `|` for combining counts, intersections, and unions of counts.
9. **What's the difference between `dict` and `types.MappingProxyType`?**
   `MappingProxyType` provides a read-only, immutable *view* over a dictionary, useful for exposing internal state safely.
10. **How would you group and aggregate data (e.g., sum by category) using only dictionaries?**
    Iterate records, use `setdefault()` or `defaultdict(float)` to accumulate sums keyed by category.
11. **What pitfalls exist when using tuples as dictionary keys?**
    All elements of the tuple must themselves be hashable; mixing types can complicate equality and lookup expectations.
12. **How can a dictionary be used to implement a simple state machine?**
    Map current states to dictionaries (or functions) of valid transitions, used as a dispatch table.
13. **Explain how dictionary view objects support set operations.**
    `keys()` and `items()` views support set operations like `&`, `|`, `-` because their elements are unique and hashable.
14. **How would you detect and handle hash collisions conceptually (without internals)?**
    You generally don't manage this yourself; trust Python's hashing, but be aware that poor `__hash__`/`__eq__` implementations on custom keys can degrade performance.
15. **What's the safest way to merge configuration dictionaries with nested structures?**
    Write a recursive merge function that merges nested dicts key-by-key rather than overwriting whole sub-dictionaries.
16. **How do you make a custom object usable as a dictionary key?**
    Implement `__hash__` and `__eq__` consistently on the class.
17. **What's the difference between `is` and `==` when comparing dictionary values?**
    `is` checks identity (same object in memory); `==` checks value equality — important when copying or caching.
18. **How would you implement memoization using a dictionary and decorators?**
    Wrap a function so its calls and results are cached in a dictionary keyed by arguments, returning cached results on repeat calls.
19. **Why is dictionary iteration order important for reproducibility in tests?**
    Since 3.7, order is guaranteed by insertion, so iterating produces consistent, reproducible results across runs.
20. **What strategies exist for thread-safe dictionary access?**
    Use locks (`threading.Lock`) around mutations, or use thread-safe alternatives/queues designed for concurrent access since plain dicts aren't guaranteed thread-safe for compound operations.

---

## 24. Coding Exercises

### Easy (10)

1. Create a dictionary mapping 5 countries to their capitals, then print each pair.
2. Write a function that returns `True` if a given key exists in a dictionary.
3. Given a dictionary of student scores, print the student with the highest score.
4. Merge two dictionaries without using `update()` or `|`.
5. Count how many keys in a dictionary have integer values greater than 10.
6. Write a function to safely get a nested value: `safe_get(d, "a", "b", "c")`.
7. Convert two parallel lists (names, ages) into a dictionary.
8. Remove all keys from a dictionary whose values are `None`.
9. Write a function that reverses a dictionary's keys and values.
10. Given a sentence, build a dictionary counting each unique word.

### Medium (10)

1. Implement a word-frequency counter using only a plain dictionary (no `Counter`).
2. Write a function to deep merge two nested dictionaries.
3. Implement a simple LRU-style cache using a dictionary (track insertion order manually).
4. Given a list of dictionaries (records), group them by a specified key.
5. Write a function to flatten a nested dictionary into dot-notation keys, e.g. `{"a": {"b": 1}}` → `{"a.b": 1}`.
6. Implement an "unflatten" function that reverses the above transformation.
7. Build a simple dispatcher: map command strings to functions, and call the right one based on input.
8. Given two dictionaries representing sets of inventory, compute items only in the first, only in the second, and in both.
9. Write a function that validates a dictionary against a required schema (a dict of required keys and expected types).
10. Implement a simple in-memory key-value store class with `set`, `get`, `delete`, and `exists` methods.

### Hard (10)

1. Implement a deep-equality checker for two arbitrarily nested dictionaries (handling lists and dicts inside).
2. Build a JSON-like pretty printer for nested dictionaries without using the `json` module.
3. Implement a trie-like structure using nested dictionaries for autocomplete suggestions.
4. Implement your own simplified `defaultdict` class from scratch using `__missing__`.
5. Write a function to detect circular references in a nested dictionary structure.
6. Implement a graph using an adjacency-list dictionary and write BFS/DFS traversal functions.
7. Build a basic in-memory relational "join" between two dictionaries representing tables (e.g., users and orders).
8. Implement a dictionary-based event system: register handlers per event name, and trigger them with arguments.
9. Write a function that diffs two nested dictionaries and reports added, removed, and changed keys (with paths).
10. Implement a simple memoization decorator using a dictionary keyed by function arguments, supporting unhashable argument fallback.

---

## 25. Mini Projects

### Project 1: Student Management System

```python
class StudentManager:
    def __init__(self):
        self.students = {}

    def add_student(self, student_id, name):
        self.students[student_id] = {"name": name, "grades": {}}

    def add_grade(self, student_id, subject, grade):
        if student_id in self.students:
            self.students[student_id]["grades"][subject] = grade
        else:
            print("Student not found.")

    def get_average(self, student_id):
        grades = self.students[student_id]["grades"]
        if not grades:
            return 0
        return sum(grades.values()) / len(grades)

    def display(self):
        for sid, info in self.students.items():
            avg = self.get_average(sid)
            print(f"{sid}: {info['name']} - Average: {avg:.2f}")


manager = StudentManager()
manager.add_student("S001", "Alice")
manager.add_grade("S001", "Math", 90)
manager.add_grade("S001", "Science", 85)
manager.display()
```

**Output:**
```
S001: Alice - Average: 87.50
```

### Project 2: Inventory Management System

```python
class Inventory:
    def __init__(self):
        self.items = {}

    def add_item(self, name, quantity, price):
        self.items[name] = {"quantity": quantity, "price": price}

    def sell_item(self, name, quantity):
        if name not in self.items:
            print("Item not found.")
            return
        if self.items[name]["quantity"] < quantity:
            print("Not enough stock.")
            return
        self.items[name]["quantity"] -= quantity

    def total_value(self):
        return sum(i["quantity"] * i["price"] for i in self.items.values())

    def report(self):
        for name, info in self.items.items():
            print(f"{name}: qty={info['quantity']}, price=${info['price']}")
        print(f"Total inventory value: ${self.total_value():.2f}")


inv = Inventory()
inv.add_item("Widget", 100, 2.5)
inv.add_item("Gadget", 50, 10.0)
inv.sell_item("Widget", 20)
inv.report()
```

**Output:**
```
Widget: qty=80, price=$2.5
Gadget: qty=50, price=$10.0
Total inventory value: $700.00
```

### Project 3: Contact Book

```python
class ContactBook:
    def __init__(self):
        self.contacts = {}

    def add(self, name, phone, email):
        self.contacts[name] = {"phone": phone, "email": email}

    def remove(self, name):
        self.contacts.pop(name, None)

    def search(self, name):
        return self.contacts.get(name, "Contact not found")

    def list_all(self):
        for name, info in self.contacts.items():
            print(f"{name}: {info['phone']} | {info['email']}")


book = ContactBook()
book.add("Alice", "555-1234", "alice@example.com")
book.add("Bob", "555-5678", "bob@example.com")
book.list_all()
print(book.search("Alice"))
```

**Output:**
```
Alice: 555-1234 | alice@example.com
Bob: 555-5678 | bob@example.com
{'phone': '555-1234', 'email': 'alice@example.com'}
```

### Project 4: Word Frequency Analyzer

```python
def word_frequency(text):
    frequency = {}
    for word in text.lower().split():
        word = word.strip(".,!?")
        frequency[word] = frequency.get(word, 0) + 1
    return frequency


text = "the quick brown fox. The fox jumps! THE fox runs."
freq = word_frequency(text)
for word, count in sorted(freq.items(), key=lambda x: -x[1]):
    print(f"{word}: {count}")
```

**Output:**
```
the: 3
fox: 3
quick: 1
brown: 1
jumps: 1
runs: 1
```

### Project 5: JSON Configuration Manager

```python
import json
import os

class ConfigManager:
    def __init__(self, filepath):
        self.filepath = filepath
        self.config = {}
        self.load()

    def load(self):
        if os.path.exists(self.filepath):
            with open(self.filepath, "r") as f:
                self.config = json.load(f)
        else:
            self.config = {}

    def set(self, key, value):
        self.config[key] = value
        self.save()

    def get(self, key, default=None):
        return self.config.get(key, default)

    def save(self):
        with open(self.filepath, "w") as f:
            json.dump(self.config, f, indent=2)


cfg = ConfigManager("app_config.json")
cfg.set("theme", "dark")
cfg.set("max_connections", 10)
print(cfg.get("theme"))
print(cfg.get("missing_key", "default_value"))
```

**Output:**
```
dark
default_value
```

---

## 26. Dictionary Cheat Sheet

```python
# ---------- CREATION ----------
d = {}
d = dict()
d = {"a": 1, "b": 2}
d = dict(a=1, b=2)
d = dict([("a", 1), ("b", 2)])
d = dict(zip(["a", "b"], [1, 2]))
d = {k: v for k, v in [("a", 1)]}
d = dict.fromkeys(["a", "b"], 0)

# ---------- ACCESS ----------
d["a"]                    # KeyError if missing
d.get("a")                 # None if missing
d.get("a", "default")
d.setdefault("a", 1)       # inserts if missing

# ---------- UPDATE ----------
d["c"] = 3
d.update({"d": 4})
d.update(e=5)
d |= {"f": 6}               # Python 3.9+
merged = d | {"g": 7}        # Python 3.9+

# ---------- DELETE ----------
del d["a"]
d.pop("b")
d.pop("z", "default")
d.popitem()
d.clear()

# ---------- VIEWS / ITERATION ----------
d.keys()
d.values()
d.items()
for k in d: ...
for k, v in d.items(): ...

# ---------- COMPREHENSIONS ----------
{x: x**2 for x in range(5)}
{k: v for k, v in d.items() if v > 1}
{v: k for k, v in d.items()}          # invert

# ---------- COPYING ----------
b = a.copy()                # shallow
import copy
b = copy.deepcopy(a)        # deep

# ---------- SORTING ----------
dict(sorted(d.items()))                          # by key
dict(sorted(d.items(), key=lambda x: x[1]))       # by value
dict(sorted(d.items(), key=lambda x: x[1], reverse=True))

# ---------- SEARCH ----------
"a" in d
1 in d.values()
("a", 1) in d.items()

# ---------- COLLECTIONS ----------
from collections import defaultdict, Counter, OrderedDict, ChainMap
defaultdict(int)
Counter(["a", "a", "b"])
OrderedDict()
ChainMap(dict1, dict2)

# ---------- JSON ----------
import json
json.loads(json_string)
json.dumps(d, indent=2)
```

---

## 27. Final Summary

- A **dictionary** stores unique, hashable **keys** mapped to **values** of any type, preserving insertion order since Python 3.7.
- Use `[]` for direct access (risk of `KeyError`), `.get()` for safe access, and `.setdefault()` to access-or-insert.
- Modify dictionaries with assignment, `.update()`, or merge operators `|` / `|=` (3.9+).
- Remove items with `del`, `.pop()`, `.popitem()`, or `.clear()` — each has distinct behavior.
- `.keys()`, `.values()`, and `.items()` return **dynamic views**, not static copies.
- Dictionary **comprehensions** offer concise creation, filtering, and transformation.
- **Shallow copies** duplicate only the top level; **deep copies** (`copy.deepcopy`) duplicate everything recursively.
- Dictionary lookup, insertion, and deletion are **O(1) on average**, making dictionaries ideal for fast access patterns.
- The **`collections`** module extends dictionaries with `defaultdict`, `Counter`, `OrderedDict`, and `ChainMap` for specialized needs.
- Dictionaries map naturally to **JSON**, making them central to API and configuration handling.
- Avoid common pitfalls: mutable default arguments, shallow-copy surprises, and modifying a dictionary while iterating over it.
- Master dictionaries thoroughly — they are one of the most used, most tested, and most practically essential data structures in all of Python.

**You now have a complete reference for Python Dictionaries — from your first `{}` to advanced real-world systems design. Practice the exercises, build the mini projects, and revisit the cheat sheet before interviews.**
