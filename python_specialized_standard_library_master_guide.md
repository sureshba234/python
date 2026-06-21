# Python Specialized Standard Library Master Guide

> This guide teaches `enum`, `pathlib`, and `collections` from beginner level to advanced usage, using verified behavior from the Python standard library documentation.[web:1][page:1][page:2]

## Table of Contents

1. [How to Use This Guide](#how-to-use-this-guide)
2. [Enum](#enum)
   1. [Introduction](#enum-introduction)
   2. [Why It Exists](#enum-why-it-exists)
   3. [Real-World Use Cases](#enum-real-world-use-cases)
   4. [Conceptual Model](#enum-conceptual-model)
   5. [Creation Methods](#enum-creation-methods)
   6. [Operations and Methods](#enum-operations-and-methods)
   7. [Examples](#enum-examples)
   8. [Advanced Patterns](#enum-advanced-patterns)
   9. [Common Mistakes](#enum-common-mistakes)
   10. [Best Practices](#enum-best-practices)
   11. [Performance Considerations](#enum-performance-considerations)
   12. [Interview Questions](#enum-interview-questions)
   13. [Practice Exercises with Solutions](#enum-practice-exercises-with-solutions)
   14. [Cheat Sheet](#enum-cheat-sheet)
3. [pathlib](#pathlib)
   1. [Introduction](#pathlib-introduction)
   2. [Why It Exists](#pathlib-why-it-exists)
   3. [Real-World Use Cases](#pathlib-real-world-use-cases)
   4. [Conceptual Model](#pathlib-conceptual-model)
   5. [Creation Methods](#pathlib-creation-methods)
   6. [Operations and Methods](#pathlib-operations-and-methods)
   7. [Examples](#pathlib-examples)
   8. [Advanced Patterns](#pathlib-advanced-patterns)
   9. [Common Mistakes](#pathlib-common-mistakes)
   10. [Best Practices](#pathlib-best-practices)
   11. [Performance Considerations](#pathlib-performance-considerations)
   12. [Interview Questions](#pathlib-interview-questions)
   13. [Practice Exercises with Solutions](#pathlib-practice-exercises-with-solutions)
   14. [Cheat Sheet](#pathlib-cheat-sheet)
4. [collections](#collections)
   1. [Introduction](#collections-introduction)
   2. [Why It Exists](#collections-why-it-exists)
   3. [Type Overview](#collections-type-overview)
   4. [Counter](#counter)
   5. [defaultdict](#defaultdict)
   6. [OrderedDict](#ordereddict)
   7. [deque](#deque)
   8. [ChainMap](#chainmap)
   9. [namedtuple](#namedtuple)
   10. [UserDict](#userdict)
   11. [UserList](#userlist)
   12. [UserString](#userstring)
   13. [Advanced Patterns](#collections-advanced-patterns)
   14. [Common Mistakes](#collections-common-mistakes)
   15. [Best Practices](#collections-best-practices)
   16. [Performance Considerations](#collections-performance-considerations)
   17. [Interview Questions](#collections-interview-questions)
   18. [Practice Exercises with Solutions](#collections-practice-exercises-with-solutions)
   19. [Cheat Sheets](#collections-cheat-sheets)
5. [Advanced Section](#advanced-section)
6. [Interview Section](#interview-section)
7. [Final Section](#final-section)

## How to Use This Guide

This guide is structured so a beginner can read top-to-bottom, while an experienced developer can jump to comparison tables, advanced patterns, and cheat sheets.[page:1][page:2] Every major topic includes examples, expected output, practical notes, common pitfalls, and production-oriented patterns.[page:1][page:2]

---

## Enum

### Enum Introduction

The `enum` module provides symbolic names bound to constant values, making code more readable and safer than using scattered raw numbers or strings.[web:1] Python’s enum support includes `Enum`, `IntEnum`, `Flag`, `IntFlag`, and helpers such as `auto()`.[web:1]

Beginners can think of an enum as a named set of valid choices. Advanced developers can use enums to model states, permissions, protocol values, and configuration options cleanly.[web:1][web:7]

### Enum Why It Exists

Enums exist because plain constants like `1`, `2`, or `'RUNNING'` are easy to mistype, duplicate, or misuse across a codebase.[web:10] A named enum gives structure, discoverability, and safer comparisons than loose constants.[web:10][web:7]

Without enums, code often becomes filled with magic values such as `status == 3` or `perm & 4`. With enums, those become `Status.DONE` and `Permission.WRITE`, which communicate intent immediately.[web:10]

### Enum Real-World Use Cases

- Application states: `STARTING`, `RUNNING`, `FAILED`, `STOPPED`.
- Status codes: HTTP-style categories, job result states, API outcomes.
- Permissions: read, write, execute combinations using `Flag` or `IntFlag`.
- Configuration systems: environment mode, logging mode, deployment region, output format.

These uses match the strength of enums: a finite, explicit set of allowed values.[web:10][web:7]

### Enum Conceptual Model

An enum member has two important parts:

- A **name**, such as `Color.RED`.
- A **value**, such as `1` or `'red'`.

The name is what developers usually care about semantically, while the value is what the program may store or exchange.[web:7] Enum classes are typically created with class syntax or function syntax, and calling an enum class performs value-based lookup rather than ordinary construction.[web:7]

#### Enumeration concepts

An enumeration is a closed set of choices. Closed means the program should use the declared members instead of inventing new ones at runtime.

```python
from enum import Enum

class Status(Enum):
    PENDING = "pending"
    RUNNING = "running"
    DONE = "done"
```

Output:

```python
>>> Status.RUNNING
<Status.RUNNING: 'running'>
>>> Status.RUNNING.name
'RUNNING'
>>> Status.RUNNING.value
'running'
```

#### Constants

Enums are a structured alternative to loose constants.

```python
# Weak style
PENDING = "pending"
RUNNING = "running"
DONE = "done"

# Better style
from enum import Enum

class Status(Enum):
    PENDING = "pending"
    RUNNING = "running"
    DONE = "done"
```

The enum version groups related constants together and makes the allowed values obvious.

#### Comparisons

Enum members are compared by enum identity and value lookup rules, not by arbitrary string equality patterns.[web:10][web:7]

```python
from enum import Enum

class Color(Enum):
    RED = 1
    BLUE = 2

print(Color.RED == Color.RED)
print(Color.RED == Color.BLUE)
```

Output:

```python
True
False
```

#### Iteration

Enums are iterable by member definition order.

```python
from enum import Enum

class Priority(Enum):
    LOW = 1
    MEDIUM = 2
    HIGH = 3

for item in Priority:
    print(item.name, item.value)
```

Output:

```python
LOW 1
MEDIUM 2
HIGH 3
```

#### Membership

Membership is usually expressed by trying value lookup or by iterating members.

```python
from enum import Enum

class Status(Enum):
    PENDING = "pending"
    RUNNING = "running"
    DONE = "done"

value = "running"
print(value in [member.value for member in Status])
print(Status(value))
```

Output:

```python
True
Status.RUNNING
```

### Enum Creation Methods

Python supports class syntax and function syntax for enums.[web:7]

#### 1. Class syntax

```python
from enum import Enum

class Role(Enum):
    ADMIN = "admin"
    USER = "user"
    GUEST = "guest"
```

#### 2. Function syntax

```python
from enum import Enum

Role = Enum("Role", ["ADMIN", "USER", "GUEST"])
print(Role.ADMIN)
```

Output:

```python
Role.ADMIN
```

#### 3. Using `auto()`

The `auto()` helper assigns values automatically when you do not care about the exact literal value.[web:1]

```python
from enum import Enum, auto

class State(Enum):
    CREATED = auto()
    STARTED = auto()
    FINISHED = auto()

for item in State:
    print(item, item.value)
```

Output:

```python
State.CREATED 1
State.STARTED 2
State.FINISHED 3
```

### Enum Operations and Methods

#### `Enum`

Use `Enum` when you want symbolic values that should not act like integers.

```python
from enum import Enum

class BuildResult(Enum):
    SUCCESS = "success"
    FAILURE = "failure"
```

#### `IntEnum`

Use `IntEnum` when enum members must behave like integers, such as legacy status codes.

```python
from enum import IntEnum

class ExitCode(IntEnum):
    OK = 0
    WARNING = 1
    ERROR = 2

print(ExitCode.OK == 0)
print(ExitCode.ERROR > ExitCode.WARNING)
```

Output:

```python
True
True
```

#### `Flag`

Use `Flag` for bitwise combinations where named flags can be combined with `|`.

```python
from enum import Flag, auto

class Permission(Flag):
    READ = auto()
    WRITE = auto()
    EXECUTE = auto()

user_perm = Permission.READ | Permission.WRITE
print(user_perm)
print(Permission.READ in user_perm)
```

Output:

```python
Permission.READ|WRITE
True
```

#### `IntFlag`

Use `IntFlag` when you need bitwise flags that also interoperate with integers.

```python
from enum import IntFlag, auto

class Feature(IntFlag):
    SEARCH = auto()
    EXPORT = auto()
    SYNC = auto()

mask = Feature.SEARCH | Feature.SYNC
print(mask)
print(int(mask))
```

Output:

```python
5
5
```

#### Common operations

```python
from enum import Enum

class Level(Enum):
    LOW = 1
    MEDIUM = 2
    HIGH = 3

print(Level(2))
print(Level.HIGH.name)
print(Level.HIGH.value)
print(list(Level))
```

Output:

```python
Level.MEDIUM
HIGH
3
[<Level.LOW: 1>, <Level.MEDIUM: 2>, <Level.HIGH: 3>]
```

### Enum Examples

#### Beginner example: user account status

```python
from enum import Enum

class AccountStatus(Enum):
    ACTIVE = "active"
    SUSPENDED = "suspended"
    DELETED = "deleted"

status = AccountStatus.ACTIVE

if status is AccountStatus.ACTIVE:
    print("Login allowed")
```

Output:

```python
Login allowed
```

#### Intermediate example: HTTP-like status family with `IntEnum`

```python
from enum import IntEnum

class JobCode(IntEnum):
    OK = 200
    NOT_FOUND = 404
    ERROR = 500

code = JobCode.NOT_FOUND
print(code == 404)
print(code.name)
```

Output:

```python
True
NOT_FOUND
```

#### Production-style example: permissions system with `IntFlag`

```python
from enum import IntFlag, auto

class Permission(IntFlag):
    READ = auto()
    WRITE = auto()
    DELETE = auto()
    ADMIN = auto()

class User:
    def __init__(self, name: str, permissions: Permission):
        self.name = name
        self.permissions = permissions

    def can(self, permission: Permission) -> bool:
        return bool(self.permissions & permission)

editor = User("Asha", Permission.READ | Permission.WRITE)
print(editor.can(Permission.READ))
print(editor.can(Permission.DELETE))
```

Output:

```python
True
False
```

#### Configuration system example

```python
from enum import Enum

class Environment(Enum):
    DEV = "dev"
    STAGING = "staging"
    PROD = "prod"

def get_db_name(env: Environment) -> str:
    mapping = {
        Environment.DEV: "app_dev",
        Environment.STAGING: "app_stage",
        Environment.PROD: "app_prod",
    }
    return mapping[env]

print(get_db_name(Environment.PROD))
```

Output:

```python
app_prod
```

### Enum Advanced Patterns

#### Enum-driven state machines

Enums work well for state machines because the state space is finite and explicit.

```python
from enum import Enum, auto

class OrderState(Enum):
    CREATED = auto()
    PAID = auto()
    SHIPPED = auto()
    DELIVERED = auto()
    CANCELLED = auto()

ALLOWED_TRANSITIONS = {
    OrderState.CREATED: {OrderState.PAID, OrderState.CANCELLED},
    OrderState.PAID: {OrderState.SHIPPED, OrderState.CANCELLED},
    OrderState.SHIPPED: {OrderState.DELIVERED},
    OrderState.DELIVERED: set(),
    OrderState.CANCELLED: set(),
}

state = OrderState.CREATED
next_state = OrderState.PAID
print(next_state in ALLOWED_TRANSITIONS[state])
```

Output:

```python
True
```

#### Advanced `Flag` example: layered access control

```python
from enum import IntFlag, auto

class Access(IntFlag):
    READ = auto()
    WRITE = auto()
    EXECUTE = auto()
    AUDIT = auto()

ROLE_PERMISSIONS = {
    "viewer": Access.READ,
    "editor": Access.READ | Access.WRITE,
    "ops": Access.READ | Access.WRITE | Access.EXECUTE | Access.AUDIT,
}

ops = ROLE_PERMISSIONS["ops"]
print(bool(ops & Access.AUDIT))
print(bool(ops & Access.EXECUTE))
```

Output:

```python
True
True
```

#### Configuration management with Enum

Enums help validate configuration options loaded from text files or environment variables.

```python
from enum import Enum

class LogMode(Enum):
    JSON = "json"
    PLAIN = "plain"

def parse_log_mode(raw: str) -> LogMode:
    return LogMode(raw.lower())

print(parse_log_mode("JSON"))
```

Output:

```python
LogMode.JSON
```

### Enum Comparison Table

| Feature | Enum | IntEnum | Flag | IntFlag |
|---|---|---|---|---|
| Primary use | Named constants | Integer-compatible constants | Bitwise flags | Integer-compatible bitwise flags |
| Behaves like int | No | Yes | No | Yes |
| Supports bitwise combinations | Not intended | Limited by int behavior, not the main goal | Yes | Yes |
| Best for statuses and labels | Yes | Yes | Sometimes | Sometimes |
| Best for permissions masks | No | Rarely | Yes | Yes |
| Safer against accidental integer mixing | High | Lower | High | Lower |

### Enum Common Mistakes

- Comparing plain strings to enum members instead of converting first.
- Using `IntEnum` when integer compatibility is not needed.
- Using regular `Enum` for permission masks instead of `Flag` or `IntFlag`.
- Persisting enum names or values without deciding which is the stable contract.
- Treating `auto()` values as meaningful external API values.

Bad example:

```python
if user_status == "ACTIVE":
    ...
```

Better:

```python
if user_status is AccountStatus.ACTIVE:
    ...
```

### Enum Best Practices

- Use `Enum` for labels and domain states.
- Use `IntEnum` only for compatibility with integer-based APIs or protocols.
- Use `Flag` or `IntFlag` for combinable permissions.
- Prefer `is` when comparing enum members in normal code.
- Keep external serialization rules explicit, especially for databases and APIs.
- Use `auto()` when exact values do not matter.

### Enum Performance Considerations

Enums improve correctness more than raw speed. They add a small abstraction layer, so they are usually slightly heavier than raw literals in hot loops, but that overhead is often insignificant compared with the readability gain.

`Flag` and `IntFlag` are effective for compact permission checks because bitwise operations are efficient. `IntEnum` can be useful where integer interoperability matters, but it also makes accidental comparisons with plain integers easier.

### Enum Interview Questions

1. What problem does `Enum` solve compared with constants?
2. When should `IntEnum` be used instead of `Enum`?
3. What is the difference between `Flag` and `IntFlag`?
4. What does `auto()` do?
5. Why are enums useful in state machines?

#### Enum interview answers

1. `Enum` groups related constants into a finite, named set and improves readability and safety.
2. `IntEnum` is useful when values must behave like integers for legacy systems, protocols, or numeric comparisons.
3. Both support bitwise combinations, but `IntFlag` also interoperates with integers.
4. `auto()` automatically assigns member values.
5. They make legal states and transitions explicit.

### Enum Practice Exercises with Solutions

#### Exercise 1

Create an enum for traffic lights and print all names.

Solution:

```python
from enum import Enum

class TrafficLight(Enum):
    RED = "stop"
    YELLOW = "wait"
    GREEN = "go"

for item in TrafficLight:
    print(item.name)
```

Output:

```python
RED
YELLOW
GREEN
```

#### Exercise 2

Create an `IntFlag` permission system and test whether a user has `WRITE`.

Solution:

```python
from enum import IntFlag, auto

class Perm(IntFlag):
    READ = auto()
    WRITE = auto()
    DELETE = auto()

user_perm = Perm.READ | Perm.WRITE
print(bool(user_perm & Perm.WRITE))
```

Output:

```python
True
```

### Enum Cheat Sheet

```python
from enum import Enum, IntEnum, Flag, IntFlag, auto

class Color(Enum):
    RED = 1
    BLUE = 2

class Code(IntEnum):
    OK = 200
    ERROR = 500

class Perm(Flag):
    READ = auto()
    WRITE = auto()

class Mask(IntFlag):
    A = auto()
    B = auto()

Color.RED.name
Color.RED.value
list(Color)
Code(200)
Perm.READ | Perm.WRITE
bool((Mask.A | Mask.B) & Mask.A)
```

---

## pathlib

### pathlib Introduction

`pathlib` provides object-oriented filesystem paths.[page:1] The module separates pure paths, which only perform path computations, from concrete paths, which also perform filesystem I/O.[page:1]

If a developer is unsure which class to use, `Path` is usually the right concrete choice for day-to-day work on the current operating system.[page:1]

### pathlib Why It Exists

`pathlib` exists to provide a clearer and more expressive alternative to string-based path manipulation and lower-level `os.path` usage.[page:1] It also exposes file reading, writing, traversal, and globbing directly from path objects.[page:1]

Instead of manually joining strings and remembering platform separators, code can compose paths with `/`, call methods like `read_text()`, and traverse directories with `iterdir()`, `glob()`, and `rglob()`.[page:1]

### pathlib Real-World Use Cases

- Log analyzer that scans `.log` files recursively.
- Backup utility that copies selected files from one directory tree to another.
- File organizer that groups files by extension.
- Build tools that read and write configuration files.
- Data pipelines that walk large directory trees and process files incrementally.

These are a natural fit because `pathlib` unifies path construction, metadata checks, and filesystem operations in one API.[page:1]

### pathlib Conceptual Model

`pathlib` has two big path families:[page:1]

- `PurePath` and its variants, for path logic without filesystem access.
- `Path` and platform-specific concrete subclasses, for real filesystem operations.[page:1]

#### `PurePath`

`PurePath` performs lexical path manipulation only and does not access the filesystem.[page:1]

```python
from pathlib import PurePath

p = PurePath("folder", "subfolder", "file.txt")
print(p)
print(p.parts)
```

Output:

```python
folder/subfolder/file.txt
('folder', 'subfolder', 'file.txt')
```

#### `PurePosixPath`

Use `PurePosixPath` to reason about Unix-style paths even on non-Unix systems.[page:1]

```python
from pathlib import PurePosixPath

p = PurePosixPath("/var", "log", "app.log")
print(p)
```

Output:

```python
/var/log/app.log
```

#### `PureWindowsPath`

Use `PureWindowsPath` to reason about Windows paths even on non-Windows systems.[page:1]

```python
from pathlib import PureWindowsPath

p = PureWindowsPath("C:/", "Users", "Asha", "report.txt")
print(p)
print(p.drive)
```

Output:

```python
C:\Users\Asha\report.txt
C:
```

#### `Path`

`Path` is a concrete path class for the current platform and supports real I/O such as `exists()`, `read_text()`, `write_text()`, `glob()`, and `walk()`.[page:1]

```python
from pathlib import Path

p = Path("notes.txt")
print(type(p).__name__)
```

Output:

```python
PosixPath
```

### pathlib Creation Methods

#### Creating paths

```python
from pathlib import Path

p1 = Path("data")
p2 = Path("data", "reports", "sales.csv")
print(p1)
print(p2)
```

Output:

```python
data
data/reports/sales.csv
```

#### Joining paths

The `/` operator and `joinpath()` are both supported.[page:1]

```python
from pathlib import Path

base = Path("/tmp")
file_path = base / "logs" / "app.log"
print(file_path)
print(base.joinpath("logs", "app.log"))
```

Output:

```python
/tmp/logs/app.log
/tmp/logs/app.log
```

#### Resolving paths

`resolve()` makes a path absolute and resolves symlinks where possible.[page:1]

```python
from pathlib import Path

p = Path("docs/../README.md")
print(p.resolve())
```

Output:

```python
# Absolute path on your system, such as:
/home/user/README.md
```

### pathlib Operations and Methods

#### Existence checks

`exists()`, `is_file()`, and `is_dir()` are core status methods.[page:1]

```python
from pathlib import Path

p = Path("example.txt")
print(p.exists())
print(p.is_file())
print(p.is_dir())
```

Possible output:

```python
False
False
False
```

#### Creating files

```python
from pathlib import Path

p = Path("demo.txt")
p.write_text("hello")
print(p.read_text())
```

Output:

```python
hello
```

#### Creating directories

```python
from pathlib import Path

folder = Path("output/reports")
folder.mkdir(parents=True, exist_ok=True)
print(folder.exists())
print(folder.is_dir())
```

Output:

```python
True
True
```

#### Reading files

```python
from pathlib import Path

p = Path("demo.txt")
print(p.read_text())
```

Output:

```python
hello
```

#### Writing files

```python
from pathlib import Path

p = Path("config.json")
p.write_text('{"debug": true}')
print(p.read_text())
```

Output:

```python
{"debug": true}
```

#### Renaming

```python
from pathlib import Path

old = Path("config.json")
new = Path("settings.json")
old.rename(new)
print(new.exists())
```

Output:

```python
True
```

#### Deleting

Use `unlink()` for files and `rmdir()` for empty directories.[web:2]

```python
from pathlib import Path

p = Path("temp.txt")
p.write_text("temporary")
p.unlink()
print(p.exists())
```

Output:

```python
False
```

#### Traversing directories

Use `iterdir()` for direct children and `walk()` for full directory trees.[page:1]

```python
from pathlib import Path

for child in Path(".").iterdir():
    print(child.name)
```

Possible output:

```python
main.py
README.md
src
```

#### Globbing

Use `glob()` for pattern matching.[page:1]

```python
from pathlib import Path

for p in Path(".").glob("*.py"):
    print(p.name)
```

Possible output:

```python
main.py
utils.py
```

#### Recursive search

Use `rglob()` for recursive matching.[page:1]

```python
from pathlib import Path

for p in Path("project").rglob("*.py"):
    print(p)
```

Possible output:

```python
project/app.py
project/lib/helpers.py
```

### pathlib Comparison Table

| Feature | pathlib | os.path |
|---|---|---|
| Style | Object-oriented | Function-based |
| Path joining | `/` or `joinpath()` | `os.path.join()` |
| Read/write helpers | Built in (`read_text()`, `write_text()`) | Separate file handling needed |
| Globbing | `glob()`, `rglob()` | Usually combined with `glob` module |
| Cross-platform path flavors | `PurePosixPath`, `PureWindowsPath` | Mostly string utilities |
| Readability | Usually higher | Often more verbose |

### pathlib Examples

#### Beginner example: read a file

```python
from pathlib import Path

notes = Path("notes.txt")
notes.write_text("Learn pathlib")
print(notes.read_text())
```

Output:

```python
Learn pathlib
```

#### Log analyzer project

```python
from pathlib import Path

log_dir = Path("logs")
error_count = 0

for log_file in log_dir.rglob("*.log"):
    text = log_file.read_text(encoding="utf-8")
    error_count += text.count("ERROR")

print(error_count)
```

Output:

```python
# Example:
12
```

#### Backup utility project

```python
from pathlib import Path
import shutil

source = Path("documents")
destination = Path("backup_documents")
destination.mkdir(exist_ok=True)

for file_path in source.glob("*.txt"):
    shutil.copy2(file_path, destination / file_path.name)

print("Backup complete")
```

Output:

```python
Backup complete
```

#### File organizer project

```python
from pathlib import Path

downloads = Path("downloads")

for file_path in downloads.iterdir():
    if file_path.is_file():
        target_dir = downloads / file_path.suffix.lstrip(".")
        target_dir.mkdir(exist_ok=True)
        file_path.rename(target_dir / file_path.name)

print("Files organized")
```

Output:

```python
Files organized
```

### pathlib Advanced Patterns

#### File-system abstraction layer

A small abstraction layer can isolate application code from direct filesystem calls.

```python
from pathlib import Path

class FileStore:
    def __init__(self, root: Path):
        self.root = root
        self.root.mkdir(parents=True, exist_ok=True)

    def write(self, relative_path: str, content: str) -> Path:
        path = self.root / relative_path
        path.parent.mkdir(parents=True, exist_ok=True)
        path.write_text(content, encoding="utf-8")
        return path

    def read(self, relative_path: str) -> str:
        return (self.root / relative_path).read_text(encoding="utf-8")

store = FileStore(Path("data_store"))
store.write("users/1/profile.txt", "active")
print(store.read("users/1/profile.txt"))
```

Output:

```python
active
```

#### Large-scale file processing with pathlib

`walk()`, `glob()`, and `rglob()` allow efficient tree traversal without hand-written recursion.[page:1]

```python
from pathlib import Path

total_bytes = 0
for root, dirs, files in Path("dataset").walk():
    if "__pycache__" in dirs:
        dirs.remove("__pycache__")
    for name in files:
        total_bytes += (root / name).stat().st_size

print(total_bytes)
```

Output:

```python
# Example:
8451021
```

### pathlib Common Mistakes

- Mixing raw strings and `Path` objects inconsistently.
- Using `PurePath` when actual filesystem I/O is required.
- Forgetting `parents=True` when creating nested directories.
- Assuming `glob()` returns sorted results; documentation notes no guaranteed order.[page:1]
- Calling `rmdir()` on non-empty directories.
- Ignoring encoding when reading text files.

### pathlib Best Practices

- Use `Path` for application code and `PurePath` variants for path logic only.
- Prefer `/` for joining paths because it is concise and readable.[page:1]
- Use `read_text()` and `write_text()` for simple text files.[page:1]
- Use `mkdir(parents=True, exist_ok=True)` for robust directory creation.
- Sort glob results if processing order matters because returned order is not guaranteed.[page:1]
- Keep path creation and I/O close to each other to reduce confusion.

### pathlib Performance Considerations

`pathlib` usually prioritizes clarity and maintainability over tiny micro-optimizations. In most application code, that tradeoff is worthwhile.

For large directory scans, avoid repeatedly re-reading the same files or calling metadata methods more than necessary. Batch work per directory when possible, and avoid converting paths to strings unless an API requires it.

### pathlib Interview Questions

1. What is the difference between `Path` and `PurePath`?
2. Why is `/` preferred for joining paths?
3. What is the difference between `glob()` and `rglob()`?
4. When would `PureWindowsPath` be useful on Linux?
5. Why should `glob()` results be sorted manually when order matters?

#### pathlib interview answers

1. `PurePath` performs path computations only, while `Path` also performs real filesystem operations.[page:1]
2. It makes path joining concise and readable and is built into the API.[page:1]
3. `glob()` searches relative patterns in one path context, while `rglob()` performs recursive matching.[page:1]
4. It is useful for generating or validating Windows-style paths without doing Windows I/O.[page:1]
5. Because `pathlib` does not guarantee a particular result order for glob operations.[page:1]

### pathlib Practice Exercises with Solutions

#### Exercise 1

Create a directory `reports/2026` and write a file `summary.txt` inside it.

Solution:

```python
from pathlib import Path

folder = Path("reports/2026")
folder.mkdir(parents=True, exist_ok=True)
file_path = folder / "summary.txt"
file_path.write_text("Done")
print(file_path.read_text())
```

Output:

```python
Done
```

#### Exercise 2

Find all `.py` files recursively under `src`.

Solution:

```python
from pathlib import Path

for path in Path("src").rglob("*.py"):
    print(path)
```

Output:

```python
# Example:
src/main.py
src/utils/helpers.py
```

### pathlib Cheat Sheet

```python
from pathlib import Path, PurePath, PurePosixPath, PureWindowsPath

p = Path("data") / "reports" / "sales.csv"
p.exists()
p.is_file()
p.is_dir()
p.parent
p.name
p.stem
p.suffix
p.read_text()
p.write_text("hello")
list(Path(".").glob("*.py"))
list(Path(".").rglob("*.py"))
Path("logs").mkdir(parents=True, exist_ok=True)
PureWindowsPath("C:/Users/Asha/report.txt")
PurePosixPath("/var/log/app.log")
```

---

## collections

### collections Introduction

The `collections` module provides specialized container datatypes that extend or complement Python’s built-in `dict`, `list`, `tuple`, and `str` types.[page:2] These include `Counter`, `defaultdict`, `OrderedDict`, `deque`, `ChainMap`, `namedtuple`, `UserDict`, `UserList`, and `UserString`.[page:2]

These tools exist because general-purpose containers are excellent defaults, but common patterns such as counting, queueing, layered mappings, and named records deserve cleaner or more efficient representations.[page:2]

### collections Why It Exists

The module exists to provide convenient and efficient alternatives to common manual patterns.[page:2] For example, `Counter` avoids hand-written counting loops, `defaultdict` avoids repeated missing-key checks, `deque` supports efficient appends and pops on both ends, and `ChainMap` creates a single view over multiple mappings.[page:2]

### collections Type Overview

| Type | Built-in alternative | Main advantage |
|---|---|---|
| Counter | dict | Built-in counting and frequency math |
| defaultdict | dict | Automatic default values for missing keys |
| OrderedDict | dict | Reordering helpers and order-sensitive equality |
| deque | list | Fast append/pop on both ends |
| ChainMap | dict merge/copies | Unified layered view over multiple mappings |
| namedtuple | tuple | Named fields with tuple efficiency |
| UserDict | dict subclass | Easier dict customization |
| UserList | list subclass | Easier list customization |
| UserString | str subclass | Easier string customization |

### Counter

#### Purpose

`Counter` is a dictionary subclass for counting hashable objects.[page:2] Missing keys return `0` instead of raising `KeyError`.[page:2]

#### Creation

```python
from collections import Counter

c1 = Counter()
c2 = Counter("banana")
c3 = Counter({"red": 4, "blue": 2})
print(c2)
```

Output:

```python
Counter({'a': 3, 'n': 2, 'b': 1})
```

#### Methods

Key methods include `most_common()`, `elements()`, `update()`, `subtract()`, and `total()`.[page:2]

```python
from collections import Counter

c = Counter("mississippi")
print(c.most_common(2))
print(c.total())
```

Output:

```python
[('i', 4), ('s', 4)]
11
```

#### Use cases

- Log level frequency analysis.
- Word counts in text processing.
- Product popularity metrics.
- Event aggregation.

#### Performance

`Counter` is ideal when counts are central to the logic. It also supports multiset-style operations such as `+`, `-`, `&`, and `|`.[page:2]

#### Example

```python
from collections import Counter

words = ["api", "db", "api", "cache", "api", "db"]
counts = Counter(words)
print(counts["api"])
print(counts.most_common(1))
```

Output:

```python
3
[('api', 3)]
```

### defaultdict

#### Purpose

`defaultdict` is a `dict` subclass that calls a factory function to supply missing values.[page:2] Its `__missing__()` behavior is triggered by `__getitem__()` when a key does not exist.[page:2]

#### Creation

```python
from collections import defaultdict

groups = defaultdict(list)
counts = defaultdict(int)
seen = defaultdict(set)
print(groups)
```

Output:

```python
defaultdict(<class 'list'>, {})
```

#### Methods

Its main special feature is the `default_factory` attribute and missing-key behavior.[page:2]

```python
from collections import defaultdict

d = defaultdict(int)
d["visits"] += 1
print(d["visits"])
print(d.default_factory)
```

Output:

```python
1
<class 'int'>
```

#### Use cases

- Grouping rows by category.
- Counting values without `if key not in d`.
- Building adjacency lists for graphs.
- Nested dictionaries.

#### Performance

The docs show it is simpler and faster than a comparable `setdefault()` pattern for grouping examples.[page:2]

#### Example

```python
from collections import defaultdict

pairs = [("blue", 1), ("blue", 2), ("red", 3)]
result = defaultdict(list)
for k, v in pairs:
    result[k].append(v)
print(dict(result))
```

Output:

```python
{'blue': [1, 2], 'red': [3]}
```

### OrderedDict

#### Purpose

`OrderedDict` remembers insertion order and adds methods specialized for reordering operations.[page:2] Its importance decreased after regular `dict` guaranteed insertion order in Python 3.7, but it still has specialized behavior such as `move_to_end()` and order-sensitive equality between ordered dictionaries.[page:2]

#### Creation

```python
from collections import OrderedDict

od = OrderedDict()
od["a"] = 1
od["b"] = 2
print(od)
```

Output:

```python
OrderedDict([('a', 1), ('b', 2)])
```

#### Methods

Key methods are `move_to_end()` and `popitem(last=True)`.[page:2]

```python
from collections import OrderedDict

od = OrderedDict.fromkeys("abcde")
od.move_to_end("b")
print("".join(od))
od.move_to_end("b", last=False)
print("".join(od))
```

Output:

```python
acdeb
bacde
```

#### Use cases

- LRU caches.
- Order-sensitive comparisons.
- Systems with frequent reordering.

#### Performance

The docs note `OrderedDict` was designed to be good at reordering operations, while regular `dict` was optimized primarily for mapping operations.[page:2]

#### Example

```python
from collections import OrderedDict

cache = OrderedDict()
cache["u1"] = "Alice"
cache["u2"] = "Bob"
cache.move_to_end("u1")
print(list(cache.keys()))
```

Output:

```python
['u2', 'u1']
```

### deque

#### Purpose

A `deque` is a double-ended queue that supports thread-safe, memory-efficient appends and pops from either side with approximately `O(1)` performance in both directions.[page:2]

#### Creation

```python
from collections import deque

d = deque([1, 2, 3])
print(d)
```

Output:

```python
deque([1, 2, 3])
```

#### Methods

Common methods include `append()`, `appendleft()`, `pop()`, `popleft()`, `extend()`, `extendleft()`, `rotate()`, and `maxlen`.[page:2]

```python
from collections import deque

d = deque("ghi")
d.append("j")
d.appendleft("f")
print(d)
d.rotate(1)
print(d)
```

Output:

```python
deque(['f', 'g', 'h', 'i', 'j'])
deque(['j', 'f', 'g', 'h', 'i'])
```

#### Use cases

- Task queues.
- Sliding windows.
- Recent-history buffers.
- Breadth-first search.

#### Performance

The docs explicitly note that lists incur `O(n)` movement costs for `pop(0)` and `insert(0, v)`, while deques are optimized for efficient end operations.[page:2]

#### Example

```python
from collections import deque

queue = deque()
queue.append("job1")
queue.append("job2")
print(queue.popleft())
print(queue.popleft())
```

Output:

```python
job1
job2
```

### ChainMap

#### Purpose

`ChainMap` groups multiple mappings into a single updateable view.[page:2] Lookups search mappings in order, while writes and deletions affect only the first mapping.[page:2]

#### Creation

```python
from collections import ChainMap

defaults = {"theme": "light", "page_size": 20}
user = {"theme": "dark"}
settings = ChainMap(user, defaults)
print(settings["theme"])
print(settings["page_size"])
```

Output:

```python
dark
20
```

#### Methods

Important features are `.maps`, `.new_child()`, and `.parents`.[page:2]

```python
from collections import ChainMap

base = ChainMap({"x": 1}, {"y": 2})
child = base.new_child({"z": 3})
print(child.maps)
print(child.parents)
```

Output:

```python
[{'z': 3}, {'x': 1}, {'y': 2}]
ChainMap({'x': 1}, {'y': 2})
```

#### Use cases

- Configuration layering.
- Templating contexts.
- Nested scope simulation.

#### Performance

The docs note it is often much faster than creating a new dictionary and running multiple `update()` calls for linked mapping views.[page:2]

#### Example

```python
from collections import ChainMap
import os

defaults = {"color": "red", "user": "guest"}
env = {"user": "deploy"}
cli = {"color": "blue"}
combined = ChainMap(cli, env, defaults)
print(combined["color"])
print(combined["user"])
```

Output:

```python
blue
deploy
```

### namedtuple

#### Purpose

`namedtuple()` creates tuple subclasses with named fields.[page:2] Instances are lightweight and support tuple behavior plus attribute access.[page:2]

#### Creation

```python
from collections import namedtuple

Point = namedtuple("Point", ["x", "y"])
p = Point(10, 20)
print(p)
```

Output:

```python
Point(x=10, y=20)
```

#### Methods

Important helpers include `_make()`, `_asdict()`, `_replace()`, `_fields`, and `_field_defaults`.[page:2]

```python
from collections import namedtuple

Point = namedtuple("Point", ["x", "y"])
p = Point(1, 2)
print(p._asdict())
print(p._replace(x=99))
```

Output:

```python
{'x': 1, 'y': 2}
Point(x=99, y=2)
```

#### Use cases

- Immutable records.
- Database or CSV row models.
- Geometry points.
- Lightweight data transfer objects.

#### Performance

Named tuple instances do not have per-instance dictionaries and require no more memory than regular tuples, according to the docs.[page:2]

#### Example

```python
from collections import namedtuple

Employee = namedtuple("Employee", ["name", "dept", "level"])
emp = Employee("Ravi", "Platform", 3)
print(emp.name)
print(emp[1])
```

Output:

```python
Ravi
Platform
```

### UserDict

#### Purpose

`UserDict` is a wrapper around dictionary objects intended to make dict subclassing easier.[page:2]

#### Creation

```python
from collections import UserDict

class LowerKeyDict(UserDict):
    def __setitem__(self, key, value):
        super().__setitem__(str(key).lower(), value)

obj = LowerKeyDict()
obj["Name"] = "Asha"
print(obj)
```

Output:

```python
{'name': 'Asha'}
```

#### Use cases

- Validated dictionaries.
- Key-normalizing mappings.
- Instrumented containers for logging or metrics.

#### Performance

Use it for customization simplicity, not raw speed.

### UserList

#### Purpose

`UserList` wraps list objects to simplify list subclassing.[page:2]

#### Creation

```python
from collections import UserList

class PositiveList(UserList):
    def append(self, item):
        if item < 0:
            raise ValueError("Only positive numbers allowed")
        super().append(item)

nums = PositiveList([1, 2])
nums.append(3)
print(nums)
```

Output:

```python
[1, 2, 3]
```

#### Use cases

- Validation-aware list containers.
- Domain collections with guarded mutations.
- Teaching custom sequence behavior.

#### Performance

Like `UserDict`, it favors easier customization over minimal overhead.

### UserString

#### Purpose

`UserString` wraps string objects to simplify string subclassing.[page:2]

#### Creation

```python
from collections import UserString

class SlugString(UserString):
    def slug(self):
        return self.data.lower().replace(" ", "-")

s = SlugString("Hello World")
print(s.slug())
```

Output:

```python
hello-world
```

#### Use cases

- Text normalization.
- String utility wrappers.
- Domain-specific text objects.

#### Performance

Useful for clean APIs, not for tight loops where raw strings are enough.

### collections Comparison Tables

| Type | dict | defaultdict | OrderedDict | Counter |
|---|---|---|---|---|
| Main purpose | General mapping | Missing-key defaults | Ordered reordering operations | Counting |
| Missing key behavior | `KeyError` | Creates value via factory on `__getitem__()` | `KeyError` | Returns `0` |
| Best for grouping | Manual | Excellent | Possible | Not primary |
| Best for counting | Manual | Good with `int` | Possible | Excellent |
| Order-sensitive equality | No | No | Yes, with other `OrderedDict` objects | No |

| Structure | list | deque |
|---|---|---|
| Fast append right | Yes | Yes |
| Fast pop right | Yes | Yes |
| Fast append left | No | Yes |
| Fast pop left | No, `pop(0)` is costly | Yes |
| Best for random indexing | Better | Good at ends, slower in middle |
| Best for queues | Fair | Excellent |

### collections Advanced Patterns

#### Frequency analysis using Counter

```python
from collections import Counter

text = "error warn info error error warn"
counts = Counter(text.split())
print(counts)
print(counts.most_common(2))
```

Output:

```python
Counter({'error': 3, 'warn': 2, 'info': 1})
[('error', 3), ('warn', 2)]
```

#### Nested data handling using defaultdict

```python
from collections import defaultdict

sales = defaultdict(lambda: defaultdict(int))
sales["south"]["laptops"] += 3
sales["south"]["phones"] += 5
sales["north"]["phones"] += 2
print(dict(sales["south"]))
print(dict(sales["north"]))
```

Output:

```python
{'laptops': 3, 'phones': 5}
{'phones': 2}
```

#### High-performance queues using deque

```python
from collections import deque

jobs = deque(maxlen=3)
for item in ["a", "b", "c", "d"]:
    jobs.append(item)
print(jobs)
```

Output:

```python
deque(['b', 'c', 'd'], maxlen=3)
```

### collections Common Mistakes

- Using `defaultdict` and expecting `.get()` to trigger `default_factory`; it does not.[page:2]
- Using `deque` for heavy random access in the middle.
- Using `OrderedDict` when a normal `dict` is enough.
- Forgetting that `ChainMap` writes only to the first mapping by default.[page:2]
- Assuming `Counter` removes zero-count items automatically; zero counts may remain until deleted.[page:2]
- Using `namedtuple` where mutability is required.

### collections Best Practices

- Use `Counter` when counting is the main job.
- Use `defaultdict(list)` for grouping and `defaultdict(int)` for counting.[page:2]
- Use `deque` for queues and sliding windows.[page:2]
- Use `ChainMap` for layered configuration without copying data.[page:2]
- Use `OrderedDict` only when its special ordering operations are needed.[page:2]
- Use `namedtuple` for lightweight immutable records.[page:2]

### collections Performance Considerations

The specialized containers usually help when your access pattern matches their design. `deque` excels at both-end operations, `Counter` excels at frequency work, and `defaultdict` reduces branching around missing keys.

The reverse is also true: using the wrong container can hurt clarity or performance. For example, a `deque` is a poor substitute for random-access-heavy list workloads, and `OrderedDict` adds complexity when plain `dict` already solves the problem.

### collections Interview Questions

1. What makes `Counter` different from a normal `dict`?
2. What does `default_factory` do in `defaultdict`?
3. When is `OrderedDict` still useful after ordered `dict`?
4. Why is `deque` better than `list` for queue behavior?
5. How does `ChainMap` handle writes?
6. Why use `namedtuple` instead of a plain tuple?
7. When should `UserDict` be preferred over direct `dict` subclassing?

#### collections interview answers

1. `Counter` is specialized for counts and returns `0` for missing items.[page:2]
2. It creates default values for missing keys during `__getitem__()` access.[page:2]
3. It is useful for reordering operations and order-sensitive equality.[page:2]
4. Because both-end appends and pops are efficient, while left-end list operations are costly.[page:2]
5. It writes only to the first mapping in the chain.[page:2]
6. It gives attribute names while keeping tuple-like lightweight behavior.[page:2]
7. When easier customization through a wrapper is preferable.[page:2]

### collections Practice Exercises with Solutions

#### Exercise 1

Count words in a sentence using `Counter`.

Solution:

```python
from collections import Counter

sentence = "red blue red green blue blue"
counts = Counter(sentence.split())
print(counts)
```

Output:

```python
Counter({'blue': 3, 'red': 2, 'green': 1})
```

#### Exercise 2

Group student names by grade using `defaultdict(list)`.

Solution:

```python
from collections import defaultdict

rows = [("A", "Nina"), ("B", "Omar"), ("A", "Kiran")]
by_grade = defaultdict(list)
for grade, name in rows:
    by_grade[grade].append(name)
print(dict(by_grade))
```

Output:

```python
{'A': ['Nina', 'Kiran'], 'B': ['Omar']}
```

#### Exercise 3

Implement a simple queue with `deque`.

Solution:

```python
from collections import deque

queue = deque([1, 2, 3])
queue.append(4)
print(queue.popleft())
print(queue)
```

Output:

```python
1
deque([2, 3, 4])
```

### collections Cheat Sheets

#### Counter

```python
from collections import Counter

c = Counter("banana")
c["a"]
c.most_common(2)
c.total()
+c
c1 + c2
c1 - c2
c1 & c2
c1 | c2
```

#### defaultdict

```python
from collections import defaultdict

d1 = defaultdict(list)
d2 = defaultdict(int)
d3 = defaultdict(set)
d1["k"].append(1)
d2["k"] += 1
d3["k"].add(1)
```

#### OrderedDict

```python
from collections import OrderedDict

od = OrderedDict()
od["a"] = 1
od.move_to_end("a")
od.popitem(last=False)
```

#### deque

```python
from collections import deque

d = deque(maxlen=5)
d.append(1)
d.appendleft(0)
d.pop()
d.popleft()
d.rotate(1)
```

#### ChainMap

```python
from collections import ChainMap

cm = ChainMap(user_config, env_config, defaults)
cm["timeout"]
cm.maps
cm.new_child({"debug": True})
cm.parents
```

#### namedtuple

```python
from collections import namedtuple

Point = namedtuple("Point", ["x", "y"])
p = Point(1, 2)
p.x
p._asdict()
p._replace(x=10)
Point._fields
```

---

## Advanced Section

### Enum-driven state machines

Enum-based state machines reduce invalid state transitions by representing legal states explicitly. This is especially useful in workflows such as payment processing, order handling, and background jobs.

```python
from enum import Enum, auto

class JobState(Enum):
    QUEUED = auto()
    RUNNING = auto()
    SUCCEEDED = auto()
    FAILED = auto()

TRANSITIONS = {
    JobState.QUEUED: {JobState.RUNNING},
    JobState.RUNNING: {JobState.SUCCEEDED, JobState.FAILED},
    JobState.SUCCEEDED: set(),
    JobState.FAILED: set(),
}

current = JobState.QUEUED
next_state = JobState.RUNNING
print(next_state in TRANSITIONS[current])
```

Output:

```python
True
```

### File-system abstraction layer

`pathlib` can power application storage layers with small, testable wrappers that hide directory creation and file encoding details. That keeps business logic separate from path manipulation.

### Configuration management with Enum

Enums are useful for config validation because they convert uncontrolled text into a closed set of accepted choices. That approach makes startup failures fast and obvious rather than causing vague runtime behavior.

### Large-scale file processing with pathlib

For large trees, combine `Path.walk()` or `rglob()` with streaming file reads and selective pruning. This keeps traversal readable while still supporting production-scale directory processing.[page:1]

### High-performance queues using deque

`deque` is a strong choice for producer-consumer style queues, fixed-length recent-history buffers, and sliding windows because both-end operations remain efficient.[page:2]

### Frequency analysis using Counter

`Counter` naturally fits analytics such as log classification, search term counts, event types, top-N reports, and category breakdowns. It can also combine counts across batches using math operations.[page:2]

### Nested data handling using defaultdict

`defaultdict` simplifies nested accumulators and grouped aggregations, especially when each missing key should create a new list, set, or integer counter automatically.[page:2]

---

## Interview Section

### 35 Interview Questions with Answers

1. **What is an enum?** A type that represents a finite set of named constant values.[web:10]
2. **Why use enums instead of strings?** They reduce magic values and improve readability and safety.[web:10]
3. **What is `IntEnum` for?** Integer-compatible enumerations.
4. **What is the difference between `Flag` and `Enum`?** `Flag` supports bitwise combinations.
5. **What does `auto()` do?** It automatically assigns member values.[web:1]
6. **What is `Path`?** A concrete filesystem path object for the current platform.[page:1]
7. **What is `PurePath`?** A path object for computations without filesystem I/O.[page:1]
8. **Why is `/` used with `Path`?** It joins child path segments readably.[page:1]
9. **What is `resolve()` used for?** It makes a path absolute and resolves symlinks where possible.[page:1]
10. **Difference between `glob()` and `rglob()`?** `rglob()` is recursive.[page:1]
11. **What is `Counter`?** A dict subclass specialized for counting.[page:2]
12. **How are missing keys handled in `Counter`?** They return `0`.[page:2]
13. **What does `most_common()` do?** Returns the highest-frequency items.[page:2]
14. **What is `defaultdict`?** A dict subclass that creates defaults for missing keys.[page:2]
15. **What is `default_factory`?** The callable that produces missing values.[page:2]
16. **Does `defaultdict.get()` call `default_factory`?** No.[page:2]
17. **What is `OrderedDict` best at?** Reordering operations and order-sensitive equality.[page:2]
18. **Why still use `OrderedDict` today?** For `move_to_end()`, FIFO/LIFO `popitem()`, and specialized ordering behavior.[page:2]
19. **What is a `deque`?** A double-ended queue.[page:2]
20. **Why is `deque` better than `list` for queues?** Left-side operations are efficient.[page:2]
21. **What does `rotate()` do in `deque`?** Rotates elements left or right.[page:2]
22. **What is `ChainMap`?** A single view over multiple mappings.[page:2]
23. **Where do writes go in `ChainMap`?** Only to the first mapping.[page:2]
24. **What is `namedtuple`?** A tuple subclass with named fields.[page:2]
25. **Why use `namedtuple`?** It gives readability with tuple-like lightweight storage.[page:2]
26. **What does `_asdict()` do?** Converts a named tuple to a dictionary.[page:2]
27. **What does `_replace()` do?** Returns a new named tuple with selected fields changed.[page:2]
28. **What is `UserDict`?** A wrapper class that simplifies dict subclassing.[page:2]
29. **What is `UserList`?** A wrapper class that simplifies list subclassing.[page:2]
30. **What is `UserString`?** A wrapper class that simplifies string subclassing.[page:2]
31. **When is `Flag` a better choice than `Enum`?** For combinable bit-mask style permissions.
32. **When should `PureWindowsPath` be used?** When modeling Windows paths without needing Windows filesystem calls.[page:1]
33. **When should `Counter` be preferred over `defaultdict(int)`?** When counting is central and methods like `most_common()` or multiset math are useful.[page:2]
34. **What is a common mistake with `deque`?** Using it for middle-heavy random indexing workloads.[page:2]
35. **What is a common mistake with enums in APIs?** Failing to define whether names or values are the stable serialized contract.

---

## Final Section

### Summary

`Enum` models finite named choices, `pathlib` models paths and filesystem work with an object-oriented API, and `collections` provides specialized containers for common patterns such as counting, grouping, ordering, queueing, layered mappings, and lightweight records.[web:10][page:1][page:2]

### Best Practices Checklist

- [ ] Use `Enum` for fixed domain choices.
- [ ] Use `IntFlag` or `Flag` for permission masks.
- [ ] Use `Path` for real filesystem work and `PurePath` for path logic only.[page:1]
- [ ] Use `read_text()` and `write_text()` for simple text file workflows.[page:1]
- [ ] Sort glob results when processing order matters.[page:1]
- [ ] Use `Counter` for counting problems.[page:2]
- [ ] Use `defaultdict` for grouping and nested accumulation.[page:2]
- [ ] Use `deque` for queues and bounded recent-history buffers.[page:2]
- [ ] Use `ChainMap` for layered configuration without copying.[page:2]
- [ ] Use `namedtuple` for immutable lightweight records.[page:2]

### Common Pitfalls Checklist

- [ ] Comparing enums to raw strings or integers without intentional conversion.
- [ ] Using `IntEnum` when integer interoperability is unnecessary.
- [ ] Using `PurePath` for actual file reads or writes.
- [ ] Assuming `glob()` output order is stable.[page:1]
- [ ] Forgetting that `defaultdict.get()` does not use `default_factory`.[page:2]
- [ ] Expecting `ChainMap` writes to affect all underlying mappings.[page:2]
- [ ] Using `deque` as a drop-in replacement for all list workloads.
- [ ] Choosing `OrderedDict` where plain `dict` is sufficient.

### Learning Roadmap

1. Start with `Enum`, `Path`, `Counter`, and `defaultdict`.
2. Practice replacing magic constants, string paths, and manual counting code.
3. Learn `Flag`/`IntFlag`, `rglob()`, `walk()`, `deque`, and `ChainMap`.
4. Build three mini-projects: permission system, log analyzer, and queue-based worker.
5. Move to production patterns: validation, configuration layering, and scalable file traversal.
