# Python Study Guide — Foundations

A revision-friendly, interview-ready guide covering Python's history, philosophy, and core execution concepts.

---

# History

## Definition
Python is a high-level, general-purpose programming language created by **Guido van Rossum**. Its history traces the evolution of the language from a hobby project into one of the world's most widely used programming languages.

## Why It Exists
Guido van Rossum wanted a language that was:
- Easy to read and write (unlike C or Perl at the time)
- Good for scripting and rapid development
- A successor to the ABC language, fixing its limitations

## How It Works (Timeline)
| Year | Event |
|------|-------|
| 1989 | Guido van Rossum starts working on Python as a hobby project during Christmas holidays at CWI, Netherlands |
| 1991 | Python 0.9.0 released (had functions, exception handling, classes) |
| 1994 | Python 1.0 released |
| 2000 | Python 2.0 released (introduced list comprehensions, garbage collection) |
| 2008 | Python 3.0 released (not fully backward compatible with Python 2) |
| 2020 | Python 2 officially reaches End-Of-Life (Jan 1, 2020) |
| Present | Python 3.x is actively developed (e.g., 3.12, 3.13) |

```
1989 ──► 1991 ──► 1994 ──► 2000 ──► 2008 ──► 2020 ──► Present
(idea)   (0.9.0)  (1.0)    (2.0)    (3.0)   (2 EOL)   (3.x)
```

## Example
```python
# Python is named after "Monty Python's Flying Circus",
# a British comedy show Guido van Rossum enjoyed — not the snake!
print("Hello, Python!")
```

## Key Points
- Named after **Monty Python**, not the snake.
- Created by **Guido van Rossum** in 1989, released in 1991.
- Guido was Python's "Benevolent Dictator For Life" (BDFL) until he stepped down in 2018.
- Python is now governed by a **Steering Council**.
- Python 2 vs Python 3 is a common interview discussion point (print statement vs function, integer division, unicode handling).

## Common Mistakes
- Thinking Python is named after the snake (it's a pop-culture reference).
- Confusing Python 2 and Python 3 syntax (e.g., `print "hi"` vs `print("hi")`).
- Assuming Python 2 is still officially supported (it is not, since 2020).

## Interview Questions
1. **Who created Python and why is it named "Python"?**
   - Created by Guido van Rossum; named after Monty Python's Flying Circus.
2. **What is BDFL?**
   - "Benevolent Dictator For Life" — Guido's former title as the final decision-maker for Python's design.
3. **What major change happened in Python 3.0?**
   - Print became a function, integer division changed (`/` returns float), strings became Unicode by default.
4. **When did Python 2 stop being supported?**
   - January 1, 2020.
5. **Who governs Python development today?**
   - The Python Steering Council, elected by core developers.

## Summary
- Created by Guido van Rossum (1989), released 1991.
- Named after Monty Python, not the snake.
- Major versions: Python 1 → 2 → 3.
- Python 2 EOL: 2020.
- Now governed by a Steering Council, not a single BDFL.

---

# Philosophy

## Definition
Python's philosophy refers to the underlying design principles that guide how the language is built and how code should be written. It emphasizes **readability, simplicity, and explicitness** over cleverness.

## Why It Exists
A clear philosophy ensures:
- Consistency in language design over decades
- Code that is easy to read and maintain by multiple developers
- A welcoming language for beginners while still being powerful for experts

## How It Works
Python's philosophy is shaped by:
1. **Readability counts** — code is read more often than it's written.
2. **Simplicity over complexity** — prefer the straightforward solution.
3. **Explicit is better than implicit** — avoid hidden magic/behavior.
4. **One obvious way to do it** — reduces fragmentation of style.
5. This philosophy is formally captured in **PEP 20: The Zen of Python**.

## Example
```python
# Explicit (Pythonic)
total = 0
for number in numbers:
    total += number

# Implicit/clever (less Pythonic, harder to read for beginners)
total = sum(numbers)  # still fine in Python, but illustrates "obvious" preferred style
```

## Key Points
- Python favors **readability** over micro-optimized or "clever" code.
- The philosophy is documented informally via the **Zen of Python**.
- Guides naming conventions (PEP 8), language features, and even rejections of new syntax.
- "Pythonic" code means code that follows these idioms and conventions.

## Common Mistakes
- Writing overly clever one-liners that sacrifice readability.
- Ignoring PEP 8 style guidance, making code inconsistent.
- Assuming "Pythonic" just means "short" — it actually means "clear and idiomatic."

## Interview Questions
1. **What does "Pythonic" mean?**
   - Code that follows Python's idioms, conventions, and philosophy — readable, simple, explicit.
2. **Where is Python's philosophy documented?**
   - PEP 20, "The Zen of Python."
3. **Why does Python prefer explicit over implicit code?**
   - It reduces bugs and confusion by making behavior predictable and traceable.
4. **How does Python's philosophy affect language design decisions?**
   - New features are often rejected if they add complexity without a clear, single, obvious benefit.

## Summary
- Core values: readability, simplicity, explicitness.
- Formalized in PEP 20 (Zen of Python).
- "Pythonic" = idiomatic, clean, readable code.

---

# Zen of Python

## Definition
The Zen of Python is a collection of 19 guiding aphorisms for writing computer programs in Python, written by **Tim Peters**. It is officially **PEP 20**.

## Why It Exists
It exists to give developers a quick, memorable reference for Python's design philosophy, helping them write idiomatic and high-quality code.

## How It Works
- You can view it anytime by running `import this` in a Python shell.
- It's a literal Easter egg encoded in Python itself (originally obfuscated using a Caesar-cipher-like encoding called ROT13 in the `this` module).

## Example
```python
>>> import this
The Zen of Python, by Tim Peters

Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Readability counts.
...
```

## Key Points
- Official PEP: **PEP 20**.
- Author: **Tim Peters**.
- Accessed via `import this`.
- Contains 19 (not 20) aphorisms — one is intentionally left unwritten ("for Guido to fill in").
- Famous lines: "Beautiful is better than ugly", "Simple is better than complex", "There should be one—and preferably only one—obvious way to do it."

## Common Mistakes
- Thinking `this` is a regular importable module with functionality — it's just an Easter egg.
- Treating the Zen as strict rules rather than guiding principles (some are intentionally contradictory/philosophical).

## Interview Questions
1. **How do you view the Zen of Python?**
   - Run `import this` in a Python interpreter.
2. **Who wrote the Zen of Python and what PEP number is it?**
   - Tim Peters; PEP 20.
3. **Name two principles from the Zen of Python.**
   - "Simple is better than complex." / "Readability counts."
4. **How many aphorisms does the Zen of Python contain?**
   - 19 (the 20th is deliberately left blank).

## Summary
- PEP 20, written by Tim Peters.
- View via `import this`.
- Encodes Python's core values: simplicity, readability, explicitness.

---

# PEPs

## Definition
**PEP** stands for **Python Enhancement Proposal**. It is a design document that describes a new feature, process, or standard for the Python language.

## Why It Exists
PEPs exist to:
- Provide a formal, transparent process for proposing language changes
- Document design decisions and the reasoning behind them
- Allow community discussion before changes are accepted into Python

## How It Works
1. Someone drafts a PEP describing a proposed feature/change.
2. It is submitted to the Python community / Steering Council for discussion.
3. The PEP is reviewed, debated, and revised.
4. It is either **Accepted**, **Rejected**, **Deferred**, or **Withdrawn**.
5. If accepted, it gets implemented into CPython (the reference implementation).

```
Draft ──► Discussion ──► Review ──► Accepted/Rejected ──► Implemented
```

## Example
| PEP | Topic |
|-----|-------|
| PEP 8 | Style Guide for Python Code |
| PEP 20 | The Zen of Python |
| PEP 257 | Docstring conventions |
| PEP 484 | Type hints |
| PEP 572 | Assignment expressions (the `:=` walrus operator) |

```python
# PEP 572 example — walrus operator
if (n := len([1, 2, 3])) > 2:
    print(f"List has {n} items")
```

## Key Points
- PEP = Python Enhancement Proposal.
- PEP 8 = style guide (very commonly referenced).
- PEP 20 = Zen of Python.
- New syntax features (like `match` statements in PEP 634) go through this process.
- PEPs are publicly archived and readable at peps.python.org.

## Common Mistakes
- Confusing PEP 8 (style) with PEP 20 (philosophy) — a very common interview mix-up.
- Assuming all PEPs become part of the language (many are rejected or deferred).

## Interview Questions
1. **What does PEP stand for?**
   - Python Enhancement Proposal.
2. **What is PEP 8 about?**
   - The official style guide for writing readable Python code (naming, indentation, spacing).
3. **What is the purpose of a PEP?**
   - To formally propose, document, and discuss changes/additions to Python.
4. **Name one PEP that introduced a new operator.**
   - PEP 572 introduced the walrus operator (`:=`).
5. **What happens after a PEP is accepted?**
   - It gets implemented in CPython and becomes part of the official language/standard.

## Summary
- PEP = formal proposal process for Python changes.
- PEP 8: style guide. PEP 20: Zen of Python.
- Lifecycle: Draft → Discussion → Accepted/Rejected → Implemented.

---

# Interpreter

## Definition
The Python interpreter is the program that reads and executes Python source code, line by line (conceptually), without needing a separate compilation step into a standalone executable.

## Why It Exists
Interpreters exist to:
- Allow code to run immediately without a manual build/link step
- Enable platform independence (the same `.py` file runs anywhere Python is installed)
- Support interactive coding and rapid testing

## How It Works
1. The interpreter reads the source code (`.py` file).
2. It compiles the source into **bytecode** internally.
3. The **Python Virtual Machine (PVM)** executes that bytecode.
4. Output is produced immediately.

```
Source Code (.py) ──► Bytecode (.pyc) ──► PVM ──► Output
```

## Example
```bash
$ python3 hello.py
Hello, World!
```
```python
# hello.py
print("Hello, World!")
```

## Key Points
- **CPython** is the default/reference interpreter (written in C).
- Other interpreters: **PyPy** (JIT-compiled, faster), **Jython** (runs on JVM), **IronPython** (.NET).
- Python is often called an "interpreted language," though it actually compiles to bytecode first.
- The interpreter handles both compilation (to bytecode) and execution (via the VM).

## Common Mistakes
- Believing Python has *no* compilation step at all (it does — to bytecode, not machine code).
- Confusing "interpreter" with "compiler" — Python does both internally.
- Assuming all Python code runs through CPython only.

## Interview Questions
1. **What is the default Python interpreter called?**
   - CPython.
2. **Is Python a purely interpreted language?**
   - No — it compiles source to bytecode first, then the bytecode is interpreted/executed by the PVM.
3. **Name two alternative Python interpreters.**
   - PyPy and Jython (also IronPython).
4. **What is the main advantage of PyPy over CPython?**
   - PyPy uses Just-In-Time (JIT) compilation, often making it significantly faster for long-running programs.

## Summary
- Interpreter = reads & executes Python code.
- CPython = default implementation.
- Internally compiles to bytecode before execution.
- Alternatives: PyPy, Jython, IronPython.

---

# REPL

## Definition
REPL stands for **Read-Eval-Print Loop** — the interactive Python shell where you type code and immediately see the result.

## Why It Exists
The REPL exists to:
- Allow quick testing of small code snippets without creating a file
- Help beginners learn syntax interactively
- Support debugging and experimentation

## How It Works
1. **Read** — the shell reads your input (an expression or statement).
2. **Eval** — it evaluates/executes that input.
3. **Print** — it prints the result (if any).
4. **Loop** — it waits for the next input, repeating the cycle.

```
 ┌─────────┐
 │  Read   │ ← user types code
 └────┬────┘
      ▼
 ┌─────────┐
 │  Eval   │ ← interpreter executes it
 └────┬────┘
      ▼
 ┌─────────┐
 │  Print  │ ← result shown
 └────┬────┘
      ▼
 ┌─────────┐
 │  Loop   │ ← back to Read
 └─────────┘
```

## Example
```python
>>> 2 + 3
5
>>> name = "Claude"
>>> print(f"Hello, {name}")
Hello, Claude
```

## Key Points
- Started by typing `python` or `python3` in the terminal.
- Each expression's result is auto-printed (unlike in scripts, where you need `print()`).
- Useful tools that enhance the REPL: **IPython**, **Jupyter Notebook**, **bpython**.
- The REPL keeps state (variables) alive during the session.

## Common Mistakes
- Forgetting that REPL state resets every time you close and reopen it.
- Writing multi-line/complex logic directly in REPL instead of a script (hard to manage).
- Expecting `print()` behavior in scripts to auto-display values like REPL does (it won't — you must explicitly print).

## Interview Questions
1. **What does REPL stand for?**
   - Read-Eval-Print Loop.
2. **How is REPL different from running a `.py` script?**
   - REPL executes and prints each expression interactively, one at a time; scripts run top-to-bottom and only print what you explicitly tell them to.
3. **Name a popular enhanced REPL tool for Python.**
   - IPython or Jupyter Notebook.
4. **Does the Python REPL retain variables across the session?**
   - Yes, until the session is closed/restarted.

## Summary
- REPL = Read-Eval-Print Loop, the interactive Python shell.
- Great for quick testing and learning.
- Enhanced tools: IPython, Jupyter.

---

# Execution Model

## Definition
Python's execution model describes how a Python program is processed from start to finish — covering scopes, namespaces, and the runtime flow of execution.

## Why It Exists
Understanding the execution model helps developers:
- Predict variable scope and lifetime
- Understand how modules, functions, and classes are loaded and run
- Debug issues related to naming and scope (e.g., `NameError`, `UnboundLocalError`)

## How It Works
1. Python reads the source file top to bottom.
2. Each module/script execution creates a **namespace** (a mapping of names to objects).
3. Statements execute sequentially; function/class bodies are only executed when called/instantiated.
4. Scoping follows the **LEGB rule**: Local → Enclosing → Global → Built-in.

```
LEGB Lookup Order:
Local → Enclosing → Global → Built-in
```

## Example
```python
x = "global"

def outer():
    x = "enclosing"
    def inner():
        x = "local"
        print(x)  # Local
    inner()

outer()
print(x)  # global
```

## Key Points
- Python uses **dynamic typing** and **dynamic name resolution** at runtime.
- Each function call creates its own **local namespace**.
- Modules have their own **global namespace**.
- Execution is **top-down**; functions/classes are defined, not executed, until called.

## Common Mistakes
- Misunderstanding scope and causing `UnboundLocalError` by assigning to a variable inside a function without `global`/`nonlocal`.
- Assuming code inside a function definition runs immediately (it doesn't — only when called).
- Forgetting that import statements execute the entire imported module once.

## Interview Questions
1. **What is the LEGB rule?**
   - The order Python looks up variable names: Local, Enclosing, Global, Built-in.
2. **What is a namespace in Python?**
   - A mapping from names (identifiers) to objects, used to track variables/functions/classes.
3. **Why does modifying a global variable inside a function raise `UnboundLocalError` sometimes?**
   - Because assigning to a variable inside a function makes it local by default; you must declare `global varname` to modify the global one.
4. **When is a function's body executed?**
   - Only when the function is called, not when it's defined.

## Summary
- Execution model = how Python runs code: namespaces + scoping (LEGB) + sequential execution.
- Functions/classes are defined first, executed later (on call/instantiation).

---

# Source Code

## Definition
Source code is the human-readable text written by a programmer in `.py` files, following Python's syntax rules.

## Why It Exists
Source code exists so developers can express logic in a readable form that can later be translated (compiled) into something a machine/VM can execute.

## How It Works
1. You write code in a text editor/IDE and save it with a `.py` extension.
2. When run, the Python interpreter reads this source code.
3. It is tokenized, parsed, compiled into bytecode, and executed.

```
.py file (source code) → Tokenizer → Parser → AST → Compiler → Bytecode
```

## Example
```python
# greeting.py — this whole file is "source code"
def greet(name):
    return f"Hello, {name}!"

print(greet("World"))
```

## Key Points
- File extension: **`.py`**
- Source code must follow Python's **syntax** and **indentation rules**.
- Comments (`#`) and docstrings are part of source code but ignored/used differently during execution.
- Source code is portable across platforms — same `.py` file can run anywhere Python is installed.

## Common Mistakes
- Inconsistent indentation (mixing tabs and spaces) causing `IndentationError`.
- Forgetting Python is case-sensitive (`Name` ≠ `name`).
- Saving files with the wrong encoding, causing issues with special characters.

## Interview Questions
1. **What file extension is used for Python source code?**
   - `.py`
2. **What is the first step the interpreter takes with source code?**
   - Tokenizing/lexing it into tokens, then parsing into an Abstract Syntax Tree (AST).
3. **Why is indentation important in Python source code?**
   - Python uses indentation (not braces) to define code blocks; incorrect indentation causes errors or logic bugs.
4. **Can the same Python source file run on Windows, macOS, and Linux without changes?**
   - Generally yes, as long as the code doesn't use OS-specific features and Python is installed.

## Summary
- Source code = human-written `.py` files.
- Goes through tokenizing → parsing → compiling → execution.
- Must follow Python's strict indentation and syntax rules.

---

# Bytecode

## Definition
Bytecode is a low-level, platform-independent set of instructions that the Python interpreter generates from source code. It is not machine code, but an intermediate form executed by the Python Virtual Machine (PVM).

## Why It Exists
Bytecode exists to:
- Speed up repeated execution (avoids re-parsing source code every time)
- Provide a platform-independent intermediate representation
- Allow the PVM to execute instructions efficiently in a simple loop

## How It Works
1. Source code is compiled into bytecode by the Python compiler (part of CPython).
2. Bytecode is stored in `.pyc` files (inside a `__pycache__` folder) for modules.
3. The PVM reads bytecode instructions one at a time and executes them.

```
source.py ──compile──► bytecode (.pyc) ──► PVM executes
```

## Example
```python
import dis

def add(a, b):
    return a + b

dis.dis(add)
```
Output (simplified):
```
  2           0 LOAD_FAST                0 (a)
              2 LOAD_FAST                1 (b)
              4 BINARY_ADD
              6 RETURN_VALUE
```

## Key Points
- Bytecode files have the `.pyc` extension, stored in `__pycache__/`.
- Bytecode is **not** portable across different Python major versions.
- You can inspect bytecode using the built-in `dis` module.
- Bytecode is executed by the **Python Virtual Machine (PVM)**, not the CPU directly.
- Bytecode caching avoids recompiling unchanged modules, improving import speed.

## Common Mistakes
- Confusing bytecode with machine code (bytecode still needs the PVM to run; it can't run directly on hardware).
- Assuming `.pyc` files work across different Python versions (they don't — they're version-specific).
- Manually editing `.pyc` files expecting predictable results.

## Interview Questions
1. **What is Python bytecode?**
   - An intermediate, low-level set of instructions generated from source code, executed by the PVM.
2. **Where is compiled bytecode cached?**
   - In `.pyc` files inside the `__pycache__` directory.
3. **How can you view the bytecode of a function?**
   - Using the `dis` module: `dis.dis(function_name)`.
4. **Is Python bytecode the same as machine code?**
   - No — machine code runs directly on hardware; bytecode requires the PVM to interpret it.
5. **Is bytecode portable across Python versions?**
   - No, bytecode format can change between major/minor Python versions.

## Summary
- Bytecode = intermediate instructions between source code and execution.
- Stored in `.pyc` files, viewed via `dis` module.
- Executed by the PVM, not directly by hardware.

---

# Virtual Machine

## Definition
The Python Virtual Machine (PVM) is the runtime engine that actually executes Python bytecode instructions, one at a time, in a loop.

## Why It Exists
The PVM exists to:
- Provide a consistent execution environment regardless of the underlying OS/hardware
- Abstract away machine-specific details so bytecode can run anywhere CPython is installed
- Manage the core execution loop (fetch instruction → execute → repeat)

## How It Works
1. The PVM receives compiled bytecode.
2. It runs a loop (often called the "eval loop") that:
   - Fetches the next bytecode instruction
   - Decodes what operation it represents
   - Executes the operation (e.g., push/pop from the stack, arithmetic, function calls)
3. This continues until the program ends or an exception propagates out.

```
 ┌────────────┐
 │  Bytecode  │
 └─────┬──────┘
       ▼
 ┌─────────────────────────┐
 │   PVM (eval loop)        │
 │  fetch → decode → execute│
 └─────┬─────────────────────┘
       ▼
 ┌────────────┐
 │   Output   │
 └────────────┘
```

## Example
```python
# This entire process (compiling + PVM execution) happens automatically:
x = 5
y = 10
print(x + y)  # PVM executes LOAD, BINARY_ADD, CALL instructions internally
```

## Key Points
- The PVM is **part of** the Python interpreter (not a separate program you install).
- It is a **stack-based** virtual machine.
- The PVM is what makes Python "interpreted" — it interprets bytecode rather than running native machine code directly.
- Different Python implementations (CPython, PyPy) have their own VM internals.

## Common Mistakes
- Thinking the PVM is a separate installable tool (it's built into CPython).
- Confusing PVM with JVM (Java Virtual Machine) — they are different, though conceptually similar.
- Assuming the PVM compiles to machine code (CPython's PVM does not; PyPy's JIT does, but that's a different mechanism).

## Interview Questions
1. **What is the Python Virtual Machine (PVM)?**
   - The component of the Python interpreter that executes compiled bytecode instructions.
2. **Is the PVM a separate installation from Python?**
   - No, it's built into the Python interpreter (e.g., part of CPython).
3. **Is the PVM stack-based or register-based?**
   - Stack-based.
4. **How does the PVM relate to bytecode?**
   - The PVM reads and executes bytecode instructions in a loop until the program completes.

## Summary
- PVM = the engine that executes Python bytecode.
- Stack-based, built into CPython.
- Final stage of the compilation pipeline before output.

---

# Compilation Pipeline

## Definition
The compilation pipeline is the sequence of steps Python takes to transform human-written source code into executable bytecode that the PVM can run.

## Why It Exists
This pipeline exists to:
- Catch syntax errors early (before execution)
- Convert readable code into an efficient form for repeated execution
- Structure the process so each stage has a clear, single responsibility

## How It Works
```
Source Code (.py)
      │
      ▼
 Tokenizer / Lexer      → breaks code into tokens (keywords, names, operators)
      │
      ▼
 Parser                 → builds an Abstract Syntax Tree (AST) from tokens
      │
      ▼
 Compiler                → converts AST into bytecode
      │
      ▼
 Bytecode (.pyc cache)
      │
      ▼
 Python Virtual Machine (PVM) → executes bytecode
      │
      ▼
 Output / Program behavior
```

## Example
```python
import ast

code = "x = 1 + 2"
tree = ast.parse(code)
print(ast.dump(tree))
# Shows the Abstract Syntax Tree structure of the code
```

## Key Points
- Stages: **Tokenizing → Parsing (AST) → Compiling (bytecode) → Execution (PVM)**.
- Syntax errors are caught at the parsing stage, before any code runs.
- Bytecode caching (`__pycache__`) skips re-tokenizing/parsing/compiling unchanged modules.
- This pipeline is why Python is described as both "compiled" (to bytecode) and "interpreted" (bytecode execution).

## Common Mistakes
- Believing Python skips compilation entirely — it doesn't; it just doesn't compile to native machine code.
- Assuming syntax errors are runtime errors — they're actually caught during parsing, before execution begins.
- Not realizing `__pycache__` exists to cache this pipeline's output for faster subsequent runs.

## Interview Questions
1. **What are the main stages of Python's compilation pipeline?**
   - Tokenizing, parsing (AST), compiling to bytecode, and execution by the PVM.
2. **At which stage are syntax errors detected?**
   - During parsing (before the program starts executing).
3. **What is an AST?**
   - Abstract Syntax Tree — a tree representation of the source code's grammatical structure.
4. **Why does Python use a multi-stage pipeline instead of direct execution of source code?**
   - It allows error checking, optimization, and reuse (via cached bytecode) before/during execution.

## Summary
- Pipeline: Source → Tokens → AST → Bytecode → PVM execution.
- Syntax errors caught at parsing stage.
- Bytecode is cached to speed up future runs.

---

# Program Structure

## Definition
Program structure refers to how a Python program is organized — including modules, statements, blocks, functions, classes, and the overall layout of code in files.

## Why It Exists
A clear structure exists to:
- Make programs easier to read, maintain, and reuse
- Allow code organization into reusable units (functions, classes, modules, packages)
- Enable Python's indentation-based block structure to define scope clearly

## How It Works
A typical Python program is structured as:
1. **Imports** — bring in external modules/libraries.
2. **Constants/Global variables** — defined at module level.
3. **Function/Class definitions** — reusable logic.
4. **Main execution block** — often guarded by `if __name__ == "__main__":`.

```
my_program.py
├── imports
├── constants
├── function definitions
├── class definitions
└── if __name__ == "__main__": ...
```

## Example
```python
# main.py
import math  # 1. import

PI_APPROX = 3.14159  # 2. constant

def area_of_circle(radius):  # 3. function definition
    return PI_APPROX * radius ** 2

class Circle:  # 4. class definition
    def __init__(self, radius):
        self.radius = radius

if __name__ == "__main__":  # 5. main execution block
    c = Circle(5)
    print(area_of_circle(c.radius))
```

## Key Points
- Python uses **indentation** (not curly braces `{}`) to define blocks.
- A **module** is simply a `.py` file; a **package** is a folder of modules with `__init__.py`.
- `if __name__ == "__main__":` ensures code only runs when the file is executed directly, not when imported.
- Statements are typically one per line (though `;` can separate multiple, which is discouraged).

## Common Mistakes
- Mixing tabs and spaces for indentation, causing errors.
- Forgetting the `if __name__ == "__main__":` guard, causing code to run unexpectedly when a module is imported elsewhere.
- Writing all logic at the top level instead of organizing into functions/classes.

## Interview Questions
1. **How does Python define code blocks?**
   - Through indentation, not braces.
2. **What does `if __name__ == "__main__":` do?**
   - Ensures the block only runs when the script is executed directly, not when imported as a module.
3. **What is the difference between a module and a package?**
   - A module is a single `.py` file; a package is a directory containing multiple modules and an `__init__.py` file.
4. **Why is consistent program structure important in larger projects?**
   - It improves readability, maintainability, and reusability across a codebase.

## Summary
- Structure: imports → constants → functions/classes → main block.
- Indentation defines blocks (typically 4 spaces).
- `if __name__ == "__main__":` controls direct-execution-only code.

---

# Keywords

## Definition
Keywords are reserved words in Python that have special meaning to the interpreter and cannot be used as identifiers (variable, function, or class names).

## Why It Exists
Keywords exist to:
- Define the core syntax and control structures of the language (loops, conditionals, function/class definitions, etc.)
- Prevent ambiguity by reserving these words exclusively for language syntax

## How It Works
- The Python interpreter recognizes keywords during tokenizing/parsing.
- Using a keyword as a variable name causes a `SyntaxError`.
- You can list all current keywords using the `keyword` module.

## Example
```python
import keyword
print(keyword.kwlist)
# ['False', 'None', 'True', 'and', 'as', 'assert', 'async', 'await',
#  'break', 'class', 'continue', 'def', 'del', 'elif', 'else', 'except',
#  'finally', 'for', 'from', 'global', 'if', 'import', 'in', 'is',
#  'lambda', 'nonlocal', 'not', 'or', 'pass', 'raise', 'return', 'try',
#  'while', 'with', 'yield']

# Invalid usage:
# class = 5   --> SyntaxError: invalid syntax (class is a keyword)
```

## Key Points
- There are **35 keywords** in modern Python (this count can vary slightly by version; `async`/`await` became keywords in Python 3.7).
- Keywords are **case-sensitive** (`True` is a keyword, `true` is not).
- **Soft keywords** like `match`, `case`, `_` (introduced in Python 3.10) behave as keywords only in specific contexts.
- `keyword.iskeyword("word")` checks if a string is a keyword.

## Common Mistakes
- Trying to use a keyword as a variable name (e.g., `for = 5`) → `SyntaxError`.
- Confusing **keywords** with **built-in functions** (e.g., `print`, `len` are NOT keywords — they're built-ins and CAN be reassigned, though it's bad practice).
- Forgetting that soft keywords like `match`/`case` are context-dependent, not globally reserved.

## Interview Questions
1. **What is a keyword in Python?**
   - A reserved word with special syntactic meaning that cannot be used as an identifier.
2. **How can you list all Python keywords programmatically?**
   - `import keyword; print(keyword.kwlist)`
3. **Is `print` a keyword in Python 3?**
   - No, `print` is a built-in function, not a keyword (unlike Python 2, where it was a statement).
4. **What are "soft keywords"? Give an example.**
   - Words that act as keywords only in specific contexts; e.g., `match` and `case` in match statements (Python 3.10+).
5. **Can you use `True` or `None` as a variable name?**
   - No — they are keywords and reserved.

## Summary
- Keywords = reserved words with special meaning; cannot be used as identifiers.
- ~35 keywords; check via `keyword.kwlist`.
- Soft keywords (`match`, `case`, `_`) are context-sensitive (Python 3.10+).

---

# Identifiers

## Definition
Identifiers are names used to identify variables, functions, classes, modules, or other objects in Python code.

## Why It Exists
Identifiers exist to:
- Give meaningful, human-readable names to data and logic
- Allow programs to reference and reuse variables/functions/classes by name

## How It Works
Python enforces specific rules for valid identifiers:
1. Must start with a letter (a–z, A–Z) or an underscore (`_`).
2. Can be followed by letters, digits (0–9), or underscores.
3. Cannot be a Python keyword.
4. Are **case-sensitive** (`Name` and `name` are different identifiers).
5. Cannot contain spaces or special characters like `@`, `-`, `%`, etc.

## Example
```python
# Valid identifiers
age = 25
_name = "Claude"
total_score2 = 100
MAX_LIMIT = 1000

# Invalid identifiers (would raise SyntaxError)
# 2total = 5        --> starts with a digit
# my-var = 10        --> contains a hyphen
# class = "test"     --> uses a reserved keyword
```

## Key Points
- Naming conventions (per PEP 8):
  - `snake_case` for variables and functions (`user_name`)
  - `PascalCase`/`CamelCase` for classes (`UserAccount`)
  - `UPPER_CASE` for constants (`MAX_RETRIES`)
  - Leading underscore (`_private`) suggests "internal use" by convention.
  - Double leading underscore (`__name`) triggers name mangling in classes.
- Identifiers are case-sensitive.
- Python 3 supports Unicode identifiers (e.g., variable names with accented characters), though ASCII is conventional.

## Common Mistakes
- Starting an identifier with a digit (`1value` → `SyntaxError`).
- Using hyphens instead of underscores (`user-name` is invalid; use `user_name`).
- Using reserved keywords as identifiers (`def = 5` → `SyntaxError`).
- Inconsistent casing conventions across a codebase, hurting readability.

## Interview Questions
1. **What are the rules for naming a valid identifier in Python?**
   - Must start with a letter or underscore, followed by letters/digits/underscores; no keywords; case-sensitive; no special characters or spaces.
2. **Can an identifier start with a digit?**
   - No, that raises a `SyntaxError`.
3. **Is `Name` the same identifier as `name` in Python?**
   - No, Python identifiers are case-sensitive, so they are different.
4. **What does a leading double underscore in a class attribute name do (e.g., `__data`)?**
   - It triggers name mangling, making the attribute harder to access accidentally from outside the class (renamed internally to `_ClassName__data`).
5. **What naming convention does PEP 8 recommend for class names?**
   - PascalCase/CamelCase (e.g., `MyClass`).

## Summary
- Identifiers = names for variables, functions, classes, etc.
- Rules: start with letter/underscore, no keywords, case-sensitive, no special chars/spaces.
- PEP 8 conventions: `snake_case` (variables/functions), `PascalCase` (classes), `UPPER_CASE` (constants).

---

*End of Study Guide — covers Python History, Philosophy, Zen of Python, PEPs, Interpreter, REPL, Execution Model, Source Code, Bytecode, Virtual Machine, Compilation Pipeline, Program Structure, Keywords, and Identifiers.*
