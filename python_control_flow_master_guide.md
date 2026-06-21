# Python Control Flow Master Guide

## Table of Contents
- [How to Use This Guide](#how-to-use-this-guide)
- [Control Flow Fundamentals](#control-flow-fundamentals)
- [Truthiness and Falsy Values](#truthiness-and-falsy-values)
- [Boolean Logic](#boolean-logic)
- [if Statement](#if-statement)
- [elif Statement](#elif-statement)
- [else Statement](#else-statement)
- [Nested Conditionals](#nested-conditionals)
- [Conditional Expressions](#conditional-expressions)
- [match-case Statement](#match-case-statement)
- [Structural Pattern Matching](#structural-pattern-matching)
- [Advanced Pattern Matching Techniques](#advanced-pattern-matching-techniques)
- [Comparison Tables](#comparison-tables)
- [Real-World Projects](#real-world-projects)
- [Control Flow Design Patterns](#control-flow-design-patterns)
- [Cheat Sheets](#cheat-sheets)
- [Interview Master Section](#interview-master-section)
- [Exercise Section](#exercise-section)
- [Final Section](#final-section)

## How to Use This Guide
This guide is written as a reference chapter and teaching guide for beginners, practicing developers, automation engineers, backend developers, data scientists, and interview candidates. It starts with core ideas, then moves from simple branch logic to advanced structural pattern matching.

Read the sections in order if the topic is new. Use the table of contents as a lookup reference when reviewing interview topics, solving bugs, or refactoring real production code.

---

## Control Flow Fundamentals
Control flow is the order in which a Python program executes statements. Conditional control flow allows code to make decisions and choose one path over another based on data, state, or external events.

At a conceptual level, every conditional answers a question:
- Is the condition true?
- If yes, which code should run?
- If no, what should happen next?

This is the basis of:
- Input validation.
- Authentication and authorization.
- Workflow routing.
- Business rule evaluation.
- Error handling.
- State transitions.

### Execution Flow Model
A simple mental model looks like this:

```text
Start
  |
Evaluate condition
  |
  +--> True  ---> Execute branch A ---> Continue
  |
  +--> False ---> Execute branch B or skip ---> Continue
```

### Why Control Flow Matters
Control flow helps programs react instead of behaving like static scripts. Without conditionals, software could not safely process user input, enforce rules, branch on file types, classify data, or respond differently to different API responses.

---

## Truthiness and Falsy Values
Python does not require every condition to be an explicit `True` or `False`. Any object can be evaluated in a Boolean context.

### What Truthiness Means
A value is **truthy** if `bool(value)` returns `True`. A value is **falsy** if `bool(value)` returns `False`.

### Common Truthy Values
| Value | Example | `bool(value)` | Why It Is Truthy |
|---|---|---:|---|
| Boolean true | `True` | `True` | Already true |
| Non-zero integer | `1` | `True` | Non-zero numbers are truthy |
| Non-empty string | `"hello"` | `True` | Contains characters |
| Non-empty list | `[1]` | `True` | Contains items |
| Non-empty dict | `{"a": 1}` | `True` | Contains keys |
| Non-empty set | `{1}` | `True` | Contains elements |
| Non-zero float | `3.14` | `True` | Non-zero numbers are truthy |

### Common Falsy Values
| Value | Example | `bool(value)` | Why It Is Falsy |
|---|---|---:|---|
| Boolean false | `False` | `False` | Already false |
| Null value | `None` | `False` | Represents absence of value |
| Zero integer | `0` | `False` | Zero is falsy |
| Zero float | `0.0` | `False` | Zero is falsy |
| Empty string | `""` | `False` | No characters |
| Empty list | `[]` | `False` | No items |
| Empty dict | `{}` | `False` | No keys |
| Empty set | `set()` | `False` | No elements |

### `bool(value)` Examples
```python
print(bool(True))      # True
print(bool(1))         # True
print(bool("hello"))   # True
print(bool([]))        # False
print(bool(0))         # False
print(bool(None))      # False
```

### Why This Matters in Control Flow
Truthiness makes code compact and expressive:

```python
items = ["apple", "banana"]

if items:
    print("Items exist")
else:
    print("The list is empty")
```

Instead of writing `if len(items) > 0:`, Python lets the collection itself determine the branch.

### Best Practice
Prefer direct truthiness checks for emptiness when intent is obvious:

```python
if name:
    ...
```

Prefer explicit comparisons when business rules require clarity:

```python
if age >= 18:
    ...
```

---

## Boolean Logic
Boolean operators combine and transform conditions.

### Operators
- `and`: true only if both sides are true.
- `or`: true if at least one side is true.
- `not`: reverses truth value.

### Truth Tables
#### `and`
| A | B | A and B |
|---|---|---|
| False | False | False |
| False | True | False |
| True | False | False |
| True | True | True |

#### `or`
| A | B | A or B |
|---|---|---|
| False | False | False |
| False | True | True |
| True | False | True |
| True | True | True |

#### `not`
| A | not A |
|---|---|
| False | True |
| True | False |

### Evaluation Diagrams
```text
if is_logged_in and has_permission:
    access granted
```

```text
Check is_logged_in
  |
  +--> False -> whole expression is False
  |
  +--> True  -> check has_permission
                 |
                 +--> True  -> expression is True
                 +--> False -> expression is False
```

### Real-World Examples
```python
age = 22
has_id = True

if age >= 18 and has_id:
    print("Entry allowed")
```

```python
is_admin = False
is_owner = True

if is_admin or is_owner:
    print("Can modify resource")
```

```python
is_banned = False

if not is_banned:
    print("User is active")
```

### Interaction with Control Flow
Boolean logic is how complex decision rules are expressed. It allows one branch to represent combinations of requirements, exceptions, and fallback conditions.

---

## if Statement

### Introduction
The `if` statement executes a block only when its condition evaluates to true. It exists because programs often need to do something only under specific circumstances.

It solves the problem of selective execution. Real-world uses include checking login status, validating form input, verifying file existence, and ensuring data is safe before processing.

### Conceptual Understanding
The `if` statement is the simplest branching tool in Python. It asks one question and either runs the block or skips it.

```text
Start
  |
Evaluate condition
  |
  +--> True  ---> Run indented block ---> Continue
  |
  +--> False ---> Skip block ----------> Continue
```

Important ideas:
- Decision making: evaluates a Boolean condition.
- Branching logic: chooses whether a block runs.
- Execution flow: changes normal top-to-bottom execution.
- Program control: makes behavior data-dependent.

### Syntax
```python
if condition:
    statement
```

### Syntax Components
- `if`: starts the conditional statement.
- `condition`: expression evaluated as truthy or falsy.
- `:`: marks the beginning of the block.
- Indented block: code that runs only if the condition is true.

### Boolean Expressions and Indentation
A condition may be:
- A comparison, such as `score > 90`.
- A direct truthiness check, such as `if items:`.
- A compound expression, such as `age >= 18 and has_ticket`.

Indentation defines the body. Python uses indentation as syntax, so inconsistent spacing changes meaning or raises errors.

### Beginner Examples
#### Number check
```python
number = 10

if number > 0:
    print("Positive number")
```

Output:
```text
Positive number
```

#### Input handling
```python
name = "Asha"

if name:
    print("Name received")
```

Output:
```text
Name received
```

#### Validation
```python
password = "secret123"

if len(password) >= 8:
    print("Password length is valid")
```

Output:
```text
Password length is valid
```

### Intermediate Examples
#### User authentication pre-check
```python
username = "admin"
is_active = True

if username == "admin" and is_active:
    print("Admin access screen")
```

#### Safe file processing
```python
file_path = "report.csv"

if file_path.endswith(".csv"):
    print("Process CSV file")
```

#### Data science filter gate
```python
records = [1, 2, 3]

if records:
    average = sum(records) / len(records)
    print(average)
```

### Advanced Examples
#### Production-style request validation
```python
def create_user(payload: dict) -> dict:
    if not payload:
        return {"status": "error", "message": "Payload is required"}

    if "email" not in payload:
        return {"status": "error", "message": "Email is required"}

    if "@" not in payload["email"]:
        return {"status": "error", "message": "Invalid email format"}

    return {"status": "success", "message": "User created"}
```

#### Automation guard
```python
def run_backup(is_weekend: bool, backup_enabled: bool) -> None:
    if is_weekend and backup_enabled:
        print("Running scheduled backup")
```

### Common Mistakes
#### Mistake 1: Missing colon
Wrong:
```python
if age >= 18
    print("Adult")
```

Correct:
```python
if age >= 18:
    print("Adult")
```

Explanation: A colon is required after the condition.

#### Mistake 2: Assignment instead of comparison idea
Wrong concept:
```python
# Invalid Python syntax
if age = 18:
    print("Adult")
```

Correct:
```python
if age == 18:
    print("Exactly 18")
```

Explanation: `=` assigns a value, `==` compares values.

#### Mistake 3: Bad indentation
Wrong:
```python
if True:
print("Hello")
```

Correct:
```python
if True:
    print("Hello")
```

Explanation: The block must be indented.

### Best Practices
- Keep conditions simple and readable.
- Prefer descriptive variable names such as `is_authenticated`.
- Use helper functions when conditions become long.
- Prefer guard clauses for invalid input.
- Avoid deeply nesting single `if` blocks when an early return is clearer.

### Performance Considerations
For most applications, the biggest concern is readability, not raw speed. A clear condition is easier to test, debug, and maintain than a highly compressed expression.

Optimization matters when conditions are expensive, such as repeated database calls or heavy computations. In those cases, cache values, short-circuit expressions sensibly, and separate validation from processing.

### Interview Questions
#### Beginner
1. What does an `if` statement do?
   - It runs a block only when its condition is true.
2. What happens if the condition is false?
   - The block is skipped.
3. Why is indentation important?
   - It defines the code block that belongs to the `if` statement.

#### Intermediate
4. What is truthiness in Python?
   - It is Python's rule for converting objects into `True` or `False` in Boolean contexts.
5. Why might `if items:` be preferred over `if len(items) > 0:`?
   - It is more idiomatic and often clearer.
6. What is short-circuit behavior?
   - In Boolean expressions, Python may stop early when the result is already known.

#### Advanced
7. When should a complex `if` condition be refactored?
   - When it becomes hard to read, test, or reuse.
8. What is a guard clause?
   - An early conditional return used to reject invalid cases quickly.
9. How do conditionals influence API design?
   - They shape validation, branching, authorization, and error responses.

### Practice Exercises
#### Easy
1. Write a program that prints `"Positive"` if a number is greater than zero.
2. Check whether a username string is non-empty.
3. Print `"Adult"` if age is at least 18.

Solutions:
```python
number = 8
if number > 0:
    print("Positive")

username = "ravi"
if username:
    print("Username provided")

age = 21
if age >= 18:
    print("Adult")
```

#### Medium
1. Validate whether an uploaded filename ends with `.json`.
2. Print a warning if a list of errors is non-empty.
3. Allow report generation only if `is_logged_in and has_subscription`.

Solutions:
```python
filename = "data.json"
if filename.endswith(".json"):
    print("Valid JSON file")

errors = ["missing field"]
if errors:
    print("Warnings present")

is_logged_in = True
has_subscription = True
if is_logged_in and has_subscription:
    print("Generate report")
```

#### Hard
1. Write a function that rejects payloads missing required keys.
2. Run preprocessing only if raw data exists and contains at least 10 records.
3. Allow deployment only in approved environments.

Solutions:
```python
def validate_payload(payload):
    required = {"id", "email"}
    if payload and required.issubset(payload):
        return True
    return False

raw_data = list(range(12))
if raw_data and len(raw_data) >= 10:
    print("Preprocess data")

environment = "staging"
if environment in {"staging", "production"}:
    print("Deployment allowed")
```

---

## elif Statement

### Introduction
The `elif` statement adds multiple decision paths to a conditional chain. It exists because many business problems require more than a yes-or-no choice.

It solves the problem of sequential branching. Real-world applications include grade calculation, pricing tiers, menu routing, category classification, and business rules.

### Conceptual Understanding
`elif` means “else if.” Python evaluates conditions from top to bottom and runs the first matching branch.

```text
Start
  |
Check condition 1
  |
  +--> True  ---> Run branch 1 ---> End chain
  |
  +--> False ---> Check condition 2
                    |
                    +--> True  ---> Run branch 2 ---> End chain
                    |
                    +--> False ---> Check next branch or end
```

Key ideas:
- Multiple decision paths.
- Sequential evaluation.
- First-match behavior.
- One branch in the chain executes.

### Syntax
```python
if condition1:
    statement1
elif condition2:
    statement2
elif condition3:
    statement3
```

### Syntax Components
- `if`: starts the chain.
- `elif`: adds another condition if previous ones failed.
- Conditions are checked in order.
- The first true branch runs.

### Beginner Examples
#### Grade calculator
```python
score = 82

if score >= 90:
    print("A")
elif score >= 80:
    print("B")
elif score >= 70:
    print("C")
```

Output:
```text
B
```

#### Temperature category
```python
temp = 30

if temp < 10:
    print("Cold")
elif temp < 25:
    print("Warm")
elif temp >= 25:
    print("Hot")
```

Output:
```text
Hot
```

### Intermediate Examples
#### Menu system
```python
choice = "2"

if choice == "1":
    print("Create user")
elif choice == "2":
    print("Delete user")
elif choice == "3":
    print("Exit")
```

#### Classification system
```python
file_type = "csv"

if file_type == "json":
    print("Parse JSON")
elif file_type == "csv":
    print("Parse CSV")
elif file_type == "xml":
    print("Parse XML")
```

#### Business rule
```python
purchase_amount = 7500

if purchase_amount >= 10000:
    discount = 0.20
elif purchase_amount >= 5000:
    discount = 0.10
elif purchase_amount >= 1000:
    discount = 0.05
```

### Advanced Examples
#### Production-style severity mapping
```python
def classify_incident(severity_score: int) -> str:
    if severity_score >= 90:
        return "critical"
    elif severity_score >= 70:
        return "high"
    elif severity_score >= 40:
        return "medium"
    elif severity_score >= 1:
        return "low"
    return "none"
```

#### Data pipeline routing
```python
def route_dataset(size_mb: int) -> str:
    if size_mb < 100:
        return "local_processing"
    elif size_mb < 500:
        return "batch_worker"
    elif size_mb < 5000:
        return "distributed_cluster"
    return "archive_and_schedule"
```

### Comparison Table
| Aspect | `if` | `elif` |
|---|---|---|
| Purpose | Single condition | Additional conditions in chain |
| Position | First branch | Must follow `if` or another `elif` |
| Execution | May run if true | Runs only if previous branches failed |
| Typical use | Simple checks | Multi-way branching |

### Common Mistakes
#### Mistake 1: Unreachable branch order
Wrong:
```python
score = 95

if score >= 60:
    print("Pass")
elif score >= 90:
    print("Excellent")
```

Correct:
```python
score = 95

if score >= 90:
    print("Excellent")
elif score >= 60:
    print("Pass")
```

Explanation: The broader condition first makes the later branch unreachable.

#### Mistake 2: Using separate `if` instead of `elif`
Wrong:
```python
score = 85

if score >= 80:
    print("B")
if score >= 70:
    print("C")
```

Correct:
```python
score = 85

if score >= 80:
    print("B")
elif score >= 70:
    print("C")
```

Explanation: Separate `if` statements can trigger multiple outputs.

### Best Practices
- Order conditions from most specific to least specific.
- Avoid very long chains when a mapping or dispatch table would be clearer.
- Use named constants for business thresholds.
- Prefer `match-case` for literal structural branching in Python 3.10+ when it improves readability.

### Performance Considerations
`elif` chains are evaluated sequentially. If likely conditions are placed earlier, some checks can be skipped, but readability should still come first.

Long chains often indicate that data-driven design may be better, such as dictionaries, strategy objects, or state machines.

### Interview Questions
#### Beginner
1. What does `elif` stand for?
   - Else if.
2. How many branches run in an `if`-`elif` chain?
   - At most one.
3. What is first-match behavior?
   - The first true condition stops the chain.

#### Intermediate
4. Why does condition order matter?
   - Earlier branches can make later ones unreachable.
5. When would you replace an `elif` chain with a dictionary?
   - When matching fixed keys to actions.
6. What is the difference between `if` plus `if` and `if` plus `elif`?
   - Separate `if` statements can both run; `elif` is mutually exclusive within the chain.

#### Advanced
7. How can long `elif` chains become a design smell?
   - They may signal missing abstractions or rule tables.
8. What is a maintainable alternative for policy evaluation?
   - Strategy selection, decision tables, or rule engines.
9. When is `match-case` more expressive than `elif`?
   - When matching structured data and patterns.

### Practice Exercises
#### Easy
1. Categorize a number as negative, zero, or positive.
2. Print a letter grade for a numeric score.
3. Classify a day as weekday or weekend using multiple conditions.

Solutions:
```python
number = -1
if number < 0:
    print("Negative")
elif number == 0:
    print("Zero")
else:
    print("Positive")

score = 91
if score >= 90:
    print("A")
elif score >= 80:
    print("B")
else:
    print("C or below")

day = "Sunday"
if day == "Saturday":
    print("Weekend")
elif day == "Sunday":
    print("Weekend")
else:
    print("Weekday")
```

#### Medium
1. Route a support ticket by severity.
2. Classify a file as image, document, or unknown.
3. Apply shipping cost based on order amount.

Solutions:
```python
severity = 72
if severity >= 90:
    print("Critical queue")
elif severity >= 70:
    print("High queue")
elif severity >= 40:
    print("Medium queue")
else:
    print("Low queue")

ext = "png"
if ext in {"png", "jpg"}:
    print("Image")
elif ext in {"pdf", "docx"}:
    print("Document")
else:
    print("Unknown")

amount = 1800
if amount >= 5000:
    print("Free shipping")
elif amount >= 2000:
    print("Discounted shipping")
else:
    print("Standard shipping")
```

#### Hard
1. Build a function that classifies transaction fraud risk.
2. Route machine-learning jobs by dataset size.
3. Choose backup policy by environment and criticality.

Solutions:
```python
def fraud_risk(score: int) -> str:
    if score >= 90:
        return "block"
    elif score >= 70:
        return "manual_review"
    elif score >= 40:
        return "monitor"
    return "allow"


def ml_route(rows: int) -> str:
    if rows < 10_000:
        return "notebook"
    elif rows < 1_000_000:
        return "scheduled_job"
    return "distributed_training"


def backup_policy(env: str, critical: bool) -> str:
    if env == "production" and critical:
        return "hourly"
    elif env == "production":
        return "daily"
    elif env == "staging":
        return "weekly"
    return "manual"
```

---

## else Statement

### Introduction
The `else` statement defines the default execution path when preceding conditions are false. It exists so programs can handle the “everything else” case explicitly.

It solves fallback logic problems. Real-world uses include access denial, invalid input handling, default categorization, and safe error responses.

### Conceptual Understanding
An `else` block runs only when none of the earlier branches matched.

```text
Evaluate prior conditions
  |
  +--> Any earlier branch true? --> Run that branch
  |
  +--> No matches -------------> Run else block
```

This is useful for:
- Default execution paths.
- Fallback logic.
- Validation failure responses.
- Error handling patterns.

### Syntax
```python
if condition:
    statement_if_true
else:
    statement_if_false
```

### Beginner Examples
#### Eligibility check
```python
age = 16

if age >= 18:
    print("Eligible")
else:
    print("Not eligible")
```

Output:
```text
Not eligible
```

#### Validation
```python
email = ""

if email:
    print("Email provided")
else:
    print("Email is required")
```

### Intermediate Examples
#### Authentication
```python
username = "admin"
password = "wrong"

if username == "admin" and password == "secret":
    print("Login successful")
else:
    print("Invalid credentials")
```

#### Input handling
```python
user_input = "42"

if user_input.isdigit():
    print("Numeric input")
else:
    print("Input must be numeric")
```

### Advanced Examples
#### Service response fallback
```python
def get_discount(customer_type: str) -> float:
    if customer_type == "premium":
        return 0.20
    else:
        return 0.00
```

#### Safe record validation
```python
def validate_record(record: dict) -> tuple[bool, str]:
    if record.get("id") and record.get("name"):
        return True, "valid"
    else:
        return False, "missing required fields"
```

### Common Mistakes
#### Mistake 1: Forgetting that `else` has no condition
Wrong:
```python
if age >= 18:
    print("Adult")
else age < 18:
    print("Minor")
```

Correct:
```python
if age >= 18:
    print("Adult")
else:
    print("Minor")
```

Explanation: `else` does not take a condition.

#### Mistake 2: Too much logic in `else`
Wrong:
```python
if status == "success":
    handle_success()
else:
    if status == "pending":
        handle_pending()
    else:
        handle_failure()
```

Better:
```python
if status == "success":
    handle_success()
elif status == "pending":
    handle_pending()
else:
    handle_failure()
```

Explanation: Flatten the structure when branches are siblings.

### Best Practices
- Use `else` when a default branch improves clarity.
- Keep the `else` block intentional, not accidental.
- Avoid hiding complex branching inside `else` when `elif` is a better fit.
- Return explicit error messages or safe defaults in backend code.

### Performance Considerations
`else` adds no meaningful overhead by itself. Its main value is explicitness, which reduces bugs from unhandled cases.

Maintainability improves when every possible path is covered. This is especially valuable in validation and security-sensitive code.

### Interview Questions
#### Beginner
1. When does `else` run?
   - When the preceding condition or conditions are false.
2. Can `else` have its own condition?
   - No.
3. Why use `else`?
   - To handle the default case clearly.

#### Intermediate
4. When is `else` better than a second `if`?
   - When the two outcomes are mutually exclusive.
5. Why is explicit fallback logic important?
   - It prevents silent unhandled cases.
6. How can `else` improve validation code?
   - It provides a clear failure path.

#### Advanced
7. When might avoiding `else` improve readability?
   - When early returns reduce nesting.
8. How does `else` help API robustness?
   - It ensures every decision path returns a defined result.
9. What is the risk of a vague `else` branch?
   - It may group unrelated failure cases together.

### Practice Exercises
#### Easy
1. Print `"Pass"` or `"Fail"` based on a score threshold.
2. Check whether a list is empty.
3. Print `"Even"` or `"Odd"` for a number.

Solutions:
```python
score = 40
if score >= 35:
    print("Pass")
else:
    print("Fail")

items = []
if items:
    print("Not empty")
else:
    print("Empty")

number = 7
if number % 2 == 0:
    print("Even")
else:
    print("Odd")
```

#### Medium
1. Validate username and show fallback error.
2. Check whether a file is supported.
3. Build a yes/no access message.

Solutions:
```python
username = ""
if username:
    print("Username accepted")
else:
    print("Username required")

file_name = "notes.txt"
if file_name.endswith(".csv"):
    print("Supported")
else:
    print("Unsupported file")

is_member = False
if is_member:
    print("Access granted")
else:
    print("Access denied")
```

#### Hard
1. Build a function that returns a fallback configuration.
2. Return a structured error when required API keys are missing.
3. Choose primary or fallback data source.

Solutions:
```python
def get_timeout(custom_timeout: int | None) -> int:
    if custom_timeout:
        return custom_timeout
    else:
        return 30


def validate_api_keys(config: dict) -> dict:
    if config.get("api_key"):
        return {"ok": True}
    else:
        return {"ok": False, "error": "Missing api_key"}


def data_source(primary_available: bool) -> str:
    if primary_available:
        return "primary"
    else:
        return "fallback"
```

---

## Nested Conditionals

### Introduction
Nested conditionals place one conditional inside another. They exist because some decisions depend on earlier decisions.

They solve multi-level branching problems. Real-world examples include permission systems, order workflows, environment-aware deployments, and multi-stage validation.

### Conceptual Understanding
Nested conditionals create decision trees.

```text
Check outer condition
  |
  +--> False ---> Outer fallback
  |
  +--> True ----> Check inner condition
                    |
                    +--> True  ---> Inner branch A
                    +--> False ---> Inner branch B
```

This supports:
- Nested branching.
- Multi-level decision trees.
- Context-sensitive rules.

### Advantages
- Natural for hierarchical decisions.
- Makes dependency between conditions explicit.
- Useful when inner checks should run only after outer validation.

### Drawbacks
- Deep nesting reduces readability.
- Indentation grows quickly.
- Harder to test and maintain.
- Can signal missing abstractions.

### Syntax
```python
if outer_condition:
    if inner_condition:
        statement
```

### Beginner Examples
#### Simple nesting
```python
age = 20
has_id = True

if age >= 18:
    if has_id:
        print("Entry allowed")
```

Output:
```text
Entry allowed
```

#### Nested validation
```python
username = "sita"
password = "secret123"

if username:
    if len(password) >= 8:
        print("Basic validation passed")
```

### Intermediate Examples
#### Permission check
```python
def can_edit(is_logged_in: bool, role: str) -> bool:
    if is_logged_in:
        if role in {"admin", "editor"}:
            return True
    return False
```

#### Order processing decision
```python
def process_order(is_paid: bool, stock_available: bool) -> str:
    if is_paid:
        if stock_available:
            return "ship_order"
        return "backorder"
    return "await_payment"
```

### Advanced Examples
#### Deployment gate
```python
def deployment_action(env: str, tests_passed: bool, approved: bool) -> str:
    if env == "production":
        if tests_passed:
            if approved:
                return "deploy"
            return "wait_for_approval"
        return "block_failed_tests"
    return "deploy_non_production"
```

#### Data pipeline nesting
```python
def handle_dataset(dataset: list, schema_valid: bool) -> str:
    if dataset:
        if schema_valid:
            if len(dataset) > 1000:
                return "batch_process"
            return "inline_process"
        return "reject_invalid_schema"
    return "no_data"
```

### Decision-Tree Diagram
```text
Is environment production?
  |
  +--> No  ---> Deploy normally
  |
  +--> Yes ---> Did tests pass?
                |
                +--> No  ---> Block deployment
                |
                +--> Yes ---> Approved?
                              |
                              +--> Yes ---> Deploy
                              +--> No  ---> Wait
```

### Refactoring Strategies
- Use guard clauses.
- Return early for invalid cases.
- Extract helper functions.
- Replace deep nesting with `elif`, dispatch tables, or strategy objects.
- Use `match-case` for structured branching where appropriate.

### Code Smell Indicators
Deep nesting may be a smell when:
- Indentation reaches 3 or more nested levels often.
- Conditions become hard to read aloud.
- Branches mix validation, business logic, and side effects.
- The same nested shape appears in multiple functions.

### When to Avoid Deep Nesting
Avoid it when:
- A flat `if`/`elif` chain communicates better.
- Early returns can simplify the function.
- Rules can be modeled as data.
- State machines or strategy patterns are more natural.

### Common Mistakes
#### Mistake 1: Nesting instead of combining
Less clear:
```python
if is_logged_in:
    if has_permission:
        print("Allowed")
```

Often clearer:
```python
if is_logged_in and has_permission:
    print("Allowed")
```

Explanation: If the conditions are peers, combining them may be clearer.

#### Mistake 2: Excessive indentation
Wrong style:
```python
if a:
    if b:
        if c:
            if d:
                do_work()
```

Refactored:
```python
def do_workflow(a, b, c, d):
    if not a:
        return
    if not b:
        return
    if not c:
        return
    if not d:
        return
    do_work()
```

Explanation: Guard clauses flatten the function.

### Best Practices
- Nest only when conditions are truly hierarchical.
- Stop nesting early with returns or raises.
- Separate validation from action.
- Extract complex decisions into named functions.
- Consider decision tables when branches represent business policy.

### Performance Considerations
Performance differences are usually negligible compared with readability costs. The real issue is maintenance complexity.

Flattening nested conditionals often reduces bugs, improves testability, and makes future changes safer.

### Interview Questions
#### Beginner
1. What is a nested conditional?
   - A conditional inside another conditional.
2. When is nesting useful?
   - When one decision depends on another.
3. What is the main readability risk?
   - Too much indentation and complexity.

#### Intermediate
4. How can nested conditionals be refactored?
   - With guard clauses, helper functions, or flattened logic.
5. When should two nested conditions be combined?
   - When they are independent peer checks.
6. What is a hierarchical decision?
   - A decision where inner checks make sense only if outer checks pass.

#### Advanced
7. Why are deep nested branches a design smell?
   - They often indicate mixed responsibilities or missing abstractions.
8. What pattern can replace complex nested branching?
   - State machines, strategy selection, or rule engines.
9. How does refactoring affect test design?
   - Smaller functions and flat logic are easier to test exhaustively.

### Practice Exercises
#### Easy
1. Check whether a number is positive, then whether it is even.
2. Validate that a name exists, then check its length.
3. Check if a user is logged in, then if they are an admin.

Solutions:
```python
number = 8
if number > 0:
    if number % 2 == 0:
        print("Positive even")

name = "Ria"
if name:
    if len(name) >= 3:
        print("Valid name")

is_logged_in = True
role = "admin"
if is_logged_in:
    if role == "admin":
        print("Admin panel")
```

#### Medium
1. Process payment only if cart exists and card is valid.
2. Approve leave only if balance exists and manager approved.
3. Validate dataset only if file exists and schema is correct.

Solutions:
```python
cart = ["item1"]
card_valid = True
if cart:
    if card_valid:
        print("Process payment")

leave_balance = 4
manager_approved = True
if leave_balance > 0:
    if manager_approved:
        print("Leave approved")

file_exists = True
schema_ok = True
if file_exists:
    if schema_ok:
        print("Dataset valid")
```

#### Hard
1. Build a 3-level deployment decision.
2. Validate API payload, user role, and quota.
3. Refactor a deep nested branch using guard clauses.

Solutions:
```python
def deploy(env, tests_ok, approved):
    if env == "production":
        if tests_ok:
            if approved:
                return "deploy"
    return "do_not_deploy"


def authorize(payload, role, quota):
    if payload:
        if role == "admin":
            if quota > 0:
                return True
    return False


def authorize_flat(payload, role, quota):
    if not payload:
        return False
    if role != "admin":
        return False
    if quota <= 0:
        return False
    return True
```

---

## Conditional Expressions

### Introduction
A conditional expression is Python's inline form of `if-else`. It is often called the ternary operator.

It exists because some decisions produce a value directly and can be expressed more compactly than a full statement block. It solves the problem of choosing one of two values in a single expression.

### Conceptual Understanding
A conditional expression is an expression, not a statement. That means it evaluates to a value.

```text
Evaluate condition
  |
  +--> True  ---> result = value_if_true
  |
  +--> False ---> result = value_if_false
```

### Syntax
```python
value_if_true if condition else value_if_false
```

### Syntax Components
- `value_if_true`: returned if the condition is truthy.
- `condition`: checked in the middle.
- `else`: mandatory part of the expression.
- `value_if_false`: returned if the condition is falsy.

### Beginner Examples
```python
age = 20
status = "Adult" if age >= 18 else "Minor"
print(status)
```

Output:
```text
Adult
```

```python
items = []
message = "Has items" if items else "Empty"
print(message)
```

Output:
```text
Empty
```

### Intermediate Examples
```python
score = 88
result = "Pass" if score >= 35 else "Fail"
```

```python
user = {"name": "Anil"}
display_name = user["name"] if "name" in user else "Guest"
```

```python
temperature = 32
advice = "Carry water" if temperature > 30 else "Weather is comfortable"
```

### Advanced Examples
```python
def normalize_timeout(timeout: int | None) -> int:
    return timeout if timeout is not None else 30
```

```python
def get_cache_strategy(is_production: bool) -> str:
    return "redis" if is_production else "local-memory"
```

```python
def response_label(status_code: int) -> str:
    return "success" if 200 <= status_code < 300 else "error"
```

### Traditional `if-else` vs Conditional Expression
| Feature | Conditional Expression | `if-else` Statement |
|---|---|---|
| Form | Expression | Statement |
| Produces value directly | Yes | Not directly |
| Best for | Simple value selection | Larger branching blocks |
| Readability | Good for short cases | Better for complex logic |
| Multi-line side effects | Poor fit | Strong fit |

### Common Mistakes
#### Mistake 1: Reversing the order
Wrong:
```python
# Not valid syntax
status = if age >= 18 "Adult" else "Minor"
```

Correct:
```python
status = "Adult" if age >= 18 else "Minor"
```

#### Mistake 2: Overly complex nested ternary
Hard to read:
```python
label = "A" if score >= 90 else "B" if score >= 80 else "C" if score >= 70 else "D"
```

Better:
```python
if score >= 90:
    label = "A"
elif score >= 80:
    label = "B"
elif score >= 70:
    label = "C"
else:
    label = "D"
```

Explanation: Compact syntax should not sacrifice clarity.

### Best Practices
- Use conditional expressions for short, obvious value selection.
- Avoid nested ternaries in production code unless they remain very readable.
- Prefer a normal `if-elif-else` block for multi-step logic.
- Use parentheses when line breaks improve readability.

### Performance Considerations
Performance differences between a conditional expression and `if-else` are usually insignificant. Choose based on readability and maintainability.

The shorter form can reduce visual noise for assignments and returns, but only when the condition is simple.

### Interview Questions
#### Beginner
1. What is a ternary operator in Python?
   - The inline conditional expression `a if condition else b`.
2. Is it a statement or expression?
   - An expression.
3. When should it be used?
   - For simple two-way value selection.

#### Intermediate
4. Why is `else` mandatory in Python's conditional expression?
   - Because the expression must always evaluate to some value.
5. When is a regular `if-else` more readable?
   - When branches contain multiple statements or complex logic.
6. Can a conditional expression appear in a return statement?
   - Yes.

#### Advanced
7. Why are nested ternaries controversial?
   - They reduce clarity and make debugging harder.
8. How do conditional expressions support functional-style code?
   - They make inline transformations concise.
9. What is the design trade-off?
   - Brevity versus readability.

### Practice Exercises
#### Easy
1. Store `"Yes"` if a number is even, otherwise `"No"`.
2. Store `"Valid"` if a string is non-empty, otherwise `"Invalid"`.
3. Return `0` if a value is `None`, otherwise return the value.

Solutions:
```python
number = 6
answer = "Yes" if number % 2 == 0 else "No"

text = "python"
status = "Valid" if text else "Invalid"

value = None
result = 0 if value is None else value
```

#### Medium
1. Choose environment label by debug mode.
2. Select output format based on a file extension.
3. Set a shipping charge based on order value.

Solutions:
```python
debug = True
environment = "development" if debug else "production"

ext = "csv"
parser = "spreadsheet" if ext == "csv" else "generic"

amount = 2500
shipping = 0 if amount >= 2000 else 99
```

#### Hard
1. Return a fallback token if the main token is missing.
2. Choose strategy by feature flag.
3. Normalize optional retry count.

Solutions:
```python
def token(main_token, backup_token):
    return main_token if main_token else backup_token


def choose_strategy(new_engine_enabled: bool) -> str:
    return "new-engine" if new_engine_enabled else "legacy-engine"


def normalize_retry_count(value: int | None) -> int:
    return value if value is not None else 3
```

---

## match-case Statement

### Introduction
The `match-case` statement, introduced in Python 3.10, provides a powerful way to branch based on patterns. It exists because some branching tasks are awkward with long `if-elif-else` chains.

It solves the problem of matching values and structures more declaratively. Real-world uses include command processors, menu systems, protocol handlers, event dispatchers, and structured data routing.

### Conceptual Understanding
`match-case` compares a subject against patterns from top to bottom. The first matching case runs.

```text
match subject
  |
Try case 1 pattern
  |
  +--> Match? Yes -> run case 1
  |
  +--> No -> try case 2
              |
              +--> Match? Yes -> run case 2
              +--> No -> continue
```

### Syntax
```python
match value:
    case pattern:
        statement
```

### Basic Matching
```python
command = "start"

match command:
    case "start":
        print("Starting service")
    case "stop":
        print("Stopping service")
    case _:
        print("Unknown command")
```

### Literal Patterns
Literal patterns match exact values.

```python
status_code = 404

match status_code:
    case 200:
        print("OK")
    case 404:
        print("Not Found")
    case 500:
        print("Server Error")
```

### Variable Patterns
A bare name captures a value.

```python
value = 42

match value:
    case number:
        print(f"Captured: {number}")
```

Important note: a bare name captures almost anything, so use it carefully.

### Wildcard Pattern
The underscore `_` means “match anything, but do not capture it.”

```python
match action:
    case "create":
        ...
    case _:
        print("Unsupported action")
```

### OR Patterns
Use `|` to match one of multiple patterns.

```python
method = "PUT"

match method:
    case "GET" | "HEAD":
        print("Read-only request")
    case "POST" | "PUT":
        print("Write request")
```

### AS Patterns
Use `as` to capture a matched pattern.

```python
response = 404

match response:
    case 400 | 401 | 403 | 404 as client_error:
        print(f"Client error: {client_error}")
```

### Guards
A guard adds an extra condition after the pattern.

```python
point = (3, 4)

match point:
    case (x, y) if x == y:
        print("Diagonal")
    case (x, y):
        print(f"Point: {x}, {y}")
```

### Beginner Examples
```python
menu_choice = "2"

match menu_choice:
    case "1":
        print("Open profile")
    case "2":
        print("Open settings")
    case _:
        print("Invalid selection")
```

### Intermediate Examples
#### Command processor
```python
def handle_command(command: str) -> str:
    match command:
        case "start":
            return "Service started"
        case "stop":
            return "Service stopped"
        case "status":
            return "Service running"
        case _:
            return "Unknown command"
```

#### Protocol handler
```python
def protocol_message(code: int) -> str:
    match code:
        case 100 | 101:
            return "Informational"
        case 200 | 201:
            return "Success"
        case 400 | 404:
            return "Client error"
        case 500 | 503:
            return "Server error"
        case _:
            return "Unhandled code"
```

### Advanced Examples
#### Event dispatcher
```python
def dispatch_event(event_type: str, payload: dict) -> str:
    match event_type:
        case "user.created":
            return f"Create profile for {payload['id']}"
        case "user.deleted":
            return f"Archive data for {payload['id']}"
        case "order.completed":
            return f"Generate invoice for {payload['id']}"
        case _:
            return "Ignore unknown event"
```

#### Menu router with guards
```python
def access_action(role: str, action: str) -> str:
    match (role, action):
        case ("admin", _):
            return "allowed"
        case ("editor", "read" | "write"):
            return "allowed"
        case ("viewer", "read"):
            return "allowed"
        case _:
            return "denied"
```

### `match-case` vs `if-elif-else`
| Feature | `if-elif-else` | `match-case` |
|---|---|---|
| Best for | General Boolean conditions | Value and structure matching |
| Reads like | Sequential condition checks | Declarative pattern matching |
| Handles nested structures | Verbose | Strong fit |
| Guards | Manual Boolean checks | Built-in with `if` guard |
| Command routing | Good | Often cleaner |
| Decomposition | Manual indexing and key checks | Built-in patterns |

### Common Mistakes
#### Mistake 1: Accidental capture pattern
Confusing:
```python
value = "ok"

match value:
    case status:
        print(status)
```

Better when exact literal intended:
```python
match value:
    case "ok":
        print("Success")
```

Explanation: `status` captures any value instead of matching the literal word `status`.

#### Mistake 2: Forgetting wildcard fallback
Risky:
```python
match choice:
    case "yes":
        print("Proceed")
```

Better:
```python
match choice:
    case "yes":
        print("Proceed")
    case _:
        print("Unknown choice")
```

Explanation: A fallback branch handles unexpected input more safely.

### Best Practices
- Use `match-case` when patterns make branching clearer than `if-elif`.
- Keep cases focused and ordered from most specific to broad fallback.
- Add `_` for unknown or unsupported inputs.
- Use guards for extra conditions only when they improve clarity.
- Avoid turning every simple two-way condition into `match-case`.

### Performance Considerations
The major benefit is readability for structured branching, not micro-optimization. In many real codebases, better clarity produces fewer bugs than any tiny performance gain could.

Use `match-case` when the problem is structurally pattern-based. Use `if-elif-else` when the problem is fundamentally about arbitrary Boolean logic.

### Interview Questions
#### Beginner
1. What is `match-case` used for?
   - Pattern-based branching.
2. Which Python version introduced it?
   - Python 3.10.
3. What does `_` mean in a `case`?
   - Wildcard fallback.

#### Intermediate
4. What is an OR pattern?
   - A pattern using `|` to match multiple alternatives.
5. What is a guard?
   - An additional condition after a pattern.
6. Why can a bare name be dangerous?
   - It captures values instead of checking a literal.

#### Advanced
7. When is `match-case` better than `if-elif`?
   - When matching data shapes, tuples, mappings, or multiple literal forms.
8. What is the difference between matching and comparing?
   - Matching can decompose structure, not just test equality.
9. What design style does `match-case` encourage?
   - Declarative structural routing.

### Practice Exercises
#### Easy
1. Match a traffic light color.
2. Match menu choice values.
3. Add a wildcard fallback.

Solutions:
```python
color = "red"
match color:
    case "red":
        print("Stop")
    case "green":
        print("Go")
    case _:
        print("Wait")

choice = "1"
match choice:
    case "1":
        print("Home")
    case "2":
        print("Settings")
    case _:
        print("Invalid")
```

#### Medium
1. Match HTTP method groups.
2. Match tuple of role and action.
3. Match status codes.

Solutions:
```python
method = "GET"
match method:
    case "GET" | "HEAD":
        print("Read")
    case "POST" | "PUT" | "PATCH":
        print("Write")
    case _:
        print("Other")

pair = ("viewer", "read")
match pair:
    case ("admin", _):
        print("Full access")
    case ("viewer", "read"):
        print("Read access")
    case _:
        print("Denied")
```

#### Hard
1. Match command tuples with guards.
2. Dispatch event names.
3. Parse protocol codes into categories.

Solutions:
```python
def handle(pair):
    match pair:
        case ("delete", user_id) if user_id > 0:
            return "delete user"
        case ("create", _):
            return "create user"
        case _:
            return "invalid"


def dispatch(name):
    match name:
        case "invoice.created":
            return "billing"
        case "user.updated":
            return "identity"
        case _:
            return "misc"
```

---

## Structural Pattern Matching

### Introduction
Structural pattern matching extends `match-case` beyond simple literal comparisons. It allows code to inspect and decompose the structure of data.

It exists because modern programs often process lists, tuples, dictionaries, nested JSON, and typed objects. It solves the problem of writing many manual checks and unpacking steps.

### Conceptual Understanding
Structural matching is pattern-based control flow over shape and content. Instead of asking only “is this equal to X?”, code can ask “does this data have this form?”

This supports:
- Data decomposition.
- Pattern-based control flow.
- Safer parsing of structured input.

### Sequence Patterns
Sequence patterns match lists and tuples by shape.

```python
point = [10, 20]

match point:
    case [x, y]:
        print(f"x={x}, y={y}")
```

#### More sequence examples
```python
values = [1, 2, 3]

match values:
    case [a, b, c]:
        print(a, b, c)
    case [single]:
        print("One item")
```

### Mapping Patterns
Mapping patterns match dictionaries by key shape.

```python
data = {"name": "Maya", "age": 30}

match data:
    case {"name": name}:
        print(f"Name: {name}")
```

#### More mapping examples
```python
payload = {"status": "ok", "result": [1, 2, 3]}

match payload:
    case {"status": "ok", "result": result}:
        print(result)
    case {"status": "error", "message": msg}:
        print(msg)
```

### Class Patterns
Class patterns match objects by class and attributes.

```python
from dataclasses import dataclass

@dataclass
class Point:
    x: int
    y: int

p = Point(4, 7)

match p:
    case Point(x, y):
        print(f"Point at {x}, {y}")
```

### Nested Patterns
Patterns can be nested deeply.

```python
data = {"user": {"name": "Ira", "role": "admin"}}

match data:
    case {"user": {"name": name}}:
        print(name)
```

### Star Patterns
Star patterns capture remaining elements.

```python
numbers = [1, 2, 3, 4, 5]

match numbers:
    case [first, *middle, last]:
        print(first, middle, last)
```

### Beginner Examples
```python
record = ["python", 1991]

match record:
    case [name, year]:
        print(f"{name} started in {year}")
```

```python
profile = {"name": "Neha"}

match profile:
    case {"name": name}:
        print(f"Hello, {name}")
```

### Intermediate Examples
#### JSON-like response parsing
```python
def parse_response(data: dict) -> str:
    match data:
        case {"status": "success", "data": result}:
            return f"Received {len(result)} records"
        case {"status": "error", "message": msg}:
            return f"Error: {msg}"
        case _:
            return "Unknown response"
```

#### Event payload decomposition
```python
def process_event(event: dict) -> str:
    match event:
        case {"type": "user.created", "payload": {"id": user_id}}:
            return f"Create user {user_id}"
        case {"type": "order.paid", "payload": {"id": order_id}}:
            return f"Fulfill order {order_id}"
        case _:
            return "Ignore"
```

### Advanced Examples
#### Class pattern routing
```python
from dataclasses import dataclass

@dataclass
class EmailTask:
    recipient: str
    subject: str

@dataclass
class SmsTask:
    phone: str
    message: str


def route_task(task):
    match task:
        case EmailTask(recipient, subject):
            return f"Send email to {recipient}: {subject}"
        case SmsTask(phone, message):
            return f"Send sms to {phone}: {message}"
        case _:
            return "Unknown task"
```

#### Nested API payload parser
```python
def handle_api_payload(payload: dict) -> str:
    match payload:
        case {"meta": {"code": 200}, "data": {"user": {"name": name, "id": user_id}}}:
            return f"User {name} ({user_id}) loaded"
        case {"meta": {"code": code}, "error": {"message": msg}}:
            return f"API error {code}: {msg}"
        case _:
            return "Malformed payload"
```

### Common Mistakes
#### Mistake 1: Expecting exact dictionary equality
Confusing assumption:
```python
match {"name": "Ana", "age": 20}:
    case {"name": name}:
        print(name)
```

Explanation: This matches because required keys are present; extra keys do not prevent a mapping pattern match.

#### Mistake 2: Forgetting sequence length matters
```python
values = [1, 2, 3]

match values:
    case [x, y]:
        print("Two values")
```

Explanation: This does not match because the pattern expects exactly two elements.

### Best Practices
- Use structural matching when shape matters more than arbitrary Boolean logic.
- Prefer explicit patterns over manual indexing and key checks.
- Keep nested patterns readable; split complex cases into helper functions if needed.
- Pair patterns with guards for validation rules.

### Performance Considerations
Structural pattern matching can simplify code dramatically, especially for parsing nested data. The biggest win is reduced branching noise and fewer manual extraction bugs.

However, overusing deeply complex patterns can hurt readability. Clear structure is more important than pattern cleverness.

### Interview Questions
#### Beginner
1. What is structural pattern matching?
   - Matching based on data structure and content.
2. Can it match lists and dictionaries?
   - Yes.
3. What is a star pattern?
   - A pattern using `*name` to capture remaining items.

#### Intermediate
4. What is the difference between mapping and sequence patterns?
   - Mapping patterns match key-based structures; sequence patterns match ordered elements.
5. Why is structural matching useful for JSON?
   - It can validate and extract nested data in one step.
6. How do class patterns help object-oriented code?
   - They let code branch by object shape and attributes.

#### Advanced
7. What makes structural matching more expressive than `if-elif`?
   - It can decompose data while branching.
8. Why should patterns remain readable?
   - Dense nested patterns can become hard to maintain.
9. When should a guard be used with a pattern?
   - When the structure matches but additional conditions must also hold.

### Practice Exercises
#### Easy
1. Match a two-item list.
2. Match a dictionary containing a name.
3. Match a list with first and last item.

Solutions:
```python
value = [10, 20]
match value:
    case [x, y]:
        print(x, y)

person = {"name": "Leo"}
match person:
    case {"name": name}:
        print(name)

nums = [1, 2, 3, 4]
match nums:
    case [first, *middle, last]:
        print(first, middle, last)
```

#### Medium
1. Parse success and error payloads.
2. Match a tuple command with arguments.
3. Match a nested user dictionary.

Solutions:
```python
def parse(data):
    match data:
        case {"status": "ok", "data": items}:
            return items
        case {"status": "error", "message": msg}:
            return msg
        case _:
            return None

cmd = ("delete", 101)
match cmd:
    case ("delete", item_id):
        print(item_id)

user = {"user": {"name": "Sara"}}
match user:
    case {"user": {"name": name}}:
        print(name)
```

#### Hard
1. Match a dataclass object.
2. Parse nested API response with metadata.
3. Route tasks by object type and fields.

Solutions:
```python
from dataclasses import dataclass

@dataclass
class Point:
    x: int
    y: int

p = Point(1, 2)
match p:
    case Point(x, y):
        print(x, y)

response = {"meta": {"code": 200}, "data": {"items": [1, 2]}}
match response:
    case {"meta": {"code": 200}, "data": {"items": items}}:
        print(items)
```

---

## Advanced Pattern Matching Techniques

### Introduction
Advanced pattern matching combines structure, guards, nested shapes, and domain-specific routing. It is especially valuable in backend systems, event-driven software, automation pipelines, and API integrations.

It solves the problem of making structured branching both expressive and maintainable.

### Guards
Guards refine a successful pattern match.

```python
def classify_point(point):
    match point:
        case (x, y) if x == 0 and y == 0:
            return "origin"
        case (x, y) if x == 0:
            return "y-axis"
        case (x, y) if y == 0:
            return "x-axis"
        case (x, y):
            return f"point({x}, {y})"
```

### Complex Nested Patterns
```python
def handle_message(message: dict) -> str:
    match message:
        case {
            "type": "notification",
            "payload": {"channel": "email", "recipient": recipient, "subject": subject}
        }:
            return f"Email {recipient}: {subject}"
        case {
            "type": "notification",
            "payload": {"channel": "sms", "phone": phone, "message": text}
        }:
            return f"SMS {phone}: {text}"
        case _:
            return "Unsupported message"
```

### Data Validation
```python
def validate_temperature_reading(data: dict) -> str:
    match data:
        case {"sensor": str(name), "value": value} if isinstance(value, (int, float)) and -50 <= value <= 150:
            return f"valid reading from {name}"
        case {"sensor": _, "value": _}:
            return "reading out of range"
        case _:
            return "invalid payload"
```

### API Response Parsing
```python
def parse_api_response(payload: dict) -> str:
    match payload:
        case {"status": "success", "data": {"items": items, "count": count}} if count == len(items):
            return f"received {count} items"
        case {"status": "error", "error": {"code": code, "message": msg}}:
            return f"error {code}: {msg}"
        case _:
            return "unexpected response"
```

### JSON Processing
```python
def extract_user_city(doc: dict) -> str:
    match doc:
        case {"user": {"profile": {"address": {"city": city}}}}:
            return city
        case _:
            return "city unavailable"
```

### Configuration Processing
```python
def process_config(config: dict) -> str:
    match config:
        case {"mode": "development", "debug": True}:
            return "verbose logging enabled"
        case {"mode": "production", "replicas": replicas} if replicas >= 2:
            return "high-availability production config"
        case {"mode": mode}:
            return f"basic config for {mode}"
        case _:
            return "invalid config"
```

### Event-Driven Systems
```python
def handle_event(event: dict) -> str:
    match event:
        case {"type": "order.created", "payload": {"order_id": order_id}}:
            return f"reserve inventory for {order_id}"
        case {"type": "payment.failed", "payload": {"order_id": order_id, "reason": reason}}:
            return f"notify payment failure for {order_id}: {reason}"
        case {"type": "shipment.sent", "payload": {"tracking_id": tracking_id}}:
            return f"track shipment {tracking_id}"
        case _:
            return "unhandled event"
```

### Production-Quality Example
```python
def process_webhook(payload: dict) -> dict:
    match payload:
        case {
            "event": "user.signup",
            "data": {"user": {"id": user_id, "email": email}}
        }:
            return {"action": "create_profile", "user_id": user_id, "email": email}
        case {
            "event": "subscription.renewed",
            "data": {"subscription": {"id": sub_id, "plan": plan}}
        }:
            return {"action": "extend_subscription", "subscription_id": sub_id, "plan": plan}
        case {
            "event": event_name
        }:
            return {"action": "log_unknown_event", "event": event_name}
        case _:
            return {"action": "reject_invalid_payload"}
```

### Common Mistakes
#### Mistake 1: Too much logic inside one case
Problematic:
```python
match payload:
    case {"type": "x", "data": data} if many_checks(data) and more_checks(data) and another_check(data):
        ...
```

Better:
```python
def is_valid_x(data):
    return many_checks(data) and more_checks(data) and another_check(data)

match payload:
    case {"type": "x", "data": data} if is_valid_x(data):
        ...
```

Explanation: Extract complicated guards into named functions.

#### Mistake 2: Pattern cleverness over readability
A shorter pattern is not always a better pattern. Production code should optimize for comprehension.

### Best Practices
- Match on structure when structure is the real decision factor.
- Use guards for clear extra constraints.
- Name helper validation functions for complex rules.
- Keep cases cohesive and domain-oriented.
- Provide explicit fallback handling for malformed input.

### Performance Considerations
Advanced pattern matching often reduces duplicated parsing code. That can improve maintainability and lower bug risk in systems that consume varied payloads.

The main performance concern is organizational complexity, not raw CPU time. A readable parser or dispatcher is usually the correct optimization target.

### Interview Questions
#### Beginner
1. What is a guard in pattern matching?
   - An extra `if` condition attached to a pattern.
2. Why use pattern matching for JSON?
   - It can both validate shape and extract values.
3. What is a wildcard fallback for malformed payloads?
   - `case _:`.

#### Intermediate
4. When should complex guard logic be extracted?
   - When the guard becomes hard to read or reuse.
5. How does pattern matching help event-driven systems?
   - It routes events by type and payload shape clearly.
6. Why is structural decomposition useful in APIs?
   - It avoids repetitive key existence checks.

#### Advanced
7. What is the trade-off in advanced patterns?
   - Expressiveness versus readability.
8. When might a rule engine be better than a huge `match` block?
   - When policy changes frequently and should be data-driven.
9. How do guards differ from nested `if` statements conceptually?
   - Guards refine the match, while nested `if` continues branching after the match.

### Practice Exercises
#### Easy
1. Match a point and classify origin with a guard.
2. Match a dict with a `name` key.
3. Match a list with first and last values.

Solutions:
```python
point = (0, 0)
match point:
    case (x, y) if x == 0 and y == 0:
        print("origin")

person = {"name": "Dev"}
match person:
    case {"name": name}:
        print(name)

numbers = [1, 2, 3]
match numbers:
    case [first, *middle, last]:
        print(first, middle, last)
```

#### Medium
1. Parse success and error API payloads.
2. Route events by nested payload structure.
3. Match production config with replicas.

Solutions:
```python
def parse_payload(payload):
    match payload:
        case {"status": "success", "data": data}:
            return data
        case {"status": "error", "message": msg}:
            return msg
        case _:
            return None


def event_route(event):
    match event:
        case {"type": "user.created", "payload": {"id": user_id}}:
            return user_id
        case _:
            return None
```

#### Hard
1. Build a webhook processor with guards.
2. Parse nested JSON and validate counts.
3. Refactor API branching into pattern matching.

Solutions:
```python
def webhook(payload):
    match payload:
        case {"event": "batch", "items": items, "count": count} if count == len(items):
            return "valid batch"
        case _:
            return "invalid"


def parse_nested(doc):
    match doc:
        case {"meta": {"ok": True}, "data": {"items": items}}:
            return items
        case _:
            return []
```

---

## Comparison Tables

### Branching Forms
| Feature | `if` | `if-else` | `if-elif-else` |
|---|---|---|---|
| Number of branches | 1 possible branch | 2 paths | Multiple paths |
| Default fallback | No | Yes | Yes |
| Best for | Single check | Two-way decisions | Multi-way decisions |
| Readability | Very high for simple cases | High | Depends on chain length |
| Typical use | Validation gate | Eligibility or fallback | Classification or routing |

### `if-elif` and `match-case`
| Feature | `if-elif` | `match-case` |
|---|---|---|
| General Boolean logic | Excellent | Limited by pattern style |
| Literal value matching | Good | Excellent |
| Structured data decomposition | Manual | Excellent |
| Guard support | Separate conditions | Built-in guards |
| JSON and tuple parsing | Verbose | Cleaner |
| Version support | All modern Python | Python 3.10+ |

### Traditional Branching and Pattern Matching
| Feature | Traditional Branching | Pattern Matching |
|---|---|---|
| Primary style | Conditions compare values | Patterns match shapes and values |
| Data extraction | Manual indexing and key access | Built into case patterns |
| Deep nested data | Verbose | Strong fit |
| Business rules | Strong | Strong when structural |
| Learning curve | Lower | Higher |

### Conditional Expression and `if-else`
| Feature | Conditional Expression | `if-else` Statement |
|---|---|---|
| Returns a value inline | Yes | No |
| Supports multiple statements | No | Yes |
| Best use | Small assignments, returns | Full branch blocks |
| Risk | Hard-to-read nesting | More vertical code |

---

## Real-World Projects

### Login System
```python
def login_system(username: str, password: str, active: bool) -> str:
    if not username or not password:
        return "Username and password are required"

    if username == "admin" and password == "secret":
        if active:
            return "Login successful"
        return "Account inactive"

    return "Invalid credentials"
```

### ATM System
```python
def atm_action(balance: float, pin_ok: bool, amount: float) -> str:
    if not pin_ok:
        return "Invalid PIN"
    if amount <= 0:
        return "Invalid withdrawal amount"
    if amount > balance:
        return "Insufficient funds"
    return f"Withdrawn {amount}, remaining balance {balance - amount}"
```

### Student Grading System
```python
def grade_student(score: int) -> str:
    if score >= 90:
        return "A"
    elif score >= 80:
        return "B"
    elif score >= 70:
        return "C"
    elif score >= 60:
        return "D"
    return "F"
```

### E-commerce Discount Engine
```python
def discount_engine(amount: float, customer_type: str, coupon: str | None) -> float:
    discount = 0.0

    if customer_type == "premium":
        discount += 0.10
    elif customer_type == "vip":
        discount += 0.15

    if amount >= 5000:
        discount += 0.10
    elif amount >= 2000:
        discount += 0.05

    if coupon == "SAVE50":
        discount += 0.05

    return min(discount, 0.30)
```

### Order Processing System
```python
def process_order(order: dict) -> str:
    if not order.get("items"):
        return "Empty order"
    if not order.get("paid"):
        return "Awaiting payment"
    if not order.get("stock_available"):
        return "Backorder"
    return "Ship order"
```

### API Response Handler
```python
def api_response_handler(response: dict) -> str:
    match response:
        case {"status": "success", "data": data}:
            return f"Process {len(data)} records"
        case {"status": "error", "message": msg}:
            return f"API error: {msg}"
        case _:
            return "Unexpected response format"
```

### Command-Line Menu System
```python
def menu_system(choice: str) -> str:
    match choice:
        case "1":
            return "Create report"
        case "2":
            return "View logs"
        case "3":
            return "Exit"
        case _:
            return "Invalid option"
```

### JSON Data Processor
```python
def json_data_processor(payload: dict) -> str:
    match payload:
        case {"user": {"name": name, "email": email}}:
            return f"User {name} with email {email}"
        case {"errors": [first, *rest]}:
            return f"First error: {first}; remaining: {len(rest)}"
        case _:
            return "Unsupported JSON structure"
```

---

## Control Flow Design Patterns

### Guard Clauses
Guard clauses return early for invalid or exceptional cases.

```python
def register_user(payload: dict) -> str:
    if not payload:
        return "payload missing"
    if "email" not in payload:
        return "email missing"
    if "@" not in payload["email"]:
        return "invalid email"
    return "user registered"
```

Why it works:
- Flattens code.
- Reduces nesting.
- Makes failure paths explicit.

### Early Returns
Early returns simplify functions that otherwise build deep pyramids of indentation.

```python
def issue_refund(order: dict) -> str:
    if not order.get("exists"):
        return "order not found"
    if not order.get("paid"):
        return "cannot refund unpaid order"
    if order.get("refunded"):
        return "already refunded"
    return "refund issued"
```

### Decision Tables
Decision tables model business rules as data.

| Customer Type | Order Amount | Coupon | Discount |
|---|---|---|---|
| Premium | >= 5000 | Yes | 25% |
| Premium | < 5000 | No | 10% |
| Regular | >= 5000 | Yes | 15% |
| Regular | < 5000 | No | 0% |

Use decision tables when rules multiply faster than readable conditionals.

### State Machines
State machines are useful when behavior depends on current state and event.

```python
def next_state(state: str, event: str) -> str:
    match (state, event):
        case ("draft", "submit"):
            return "pending_review"
        case ("pending_review", "approve"):
            return "approved"
        case ("pending_review", "reject"):
            return "rejected"
        case _:
            return "invalid_transition"
```

### Strategy Selection
Choose an algorithm or workflow based on input.

```python
def choose_parser(file_type: str):
    if file_type == "csv":
        return "csv_parser"
    elif file_type == "json":
        return "json_parser"
    return "default_parser"
```

### Rule Engines
A rule engine is a data-driven or modular way to evaluate many policies. Use it when conditional logic becomes large, frequently changing, or business-owned.

### Pattern-Based Dispatching
Dispatch tasks by data shape and type.

```python
def dispatch(job: dict) -> str:
    match job:
        case {"kind": "email", "to": recipient}:
            return f"email:{recipient}"
        case {"kind": "sms", "to": phone}:
            return f"sms:{phone}"
        case _:
            return "unknown"
```

---

## Cheat Sheets

### `if`
```python
if condition:
    action()
```

Use when:
- One condition controls one optional block.

### `elif`
```python
if cond1:
    action1()
elif cond2:
    action2()
```

Use when:
- Exactly one of many branches should run.

### `else`
```python
if condition:
    action_if_true()
else:
    action_if_false()
```

Use when:
- A default fallback is required.

### Nested Conditionals
```python
if outer:
    if inner:
        action()
```

Use when:
- Inner checks depend on outer validation.

Avoid when:
- A flatter design is clearer.

### Conditional Expressions
```python
result = a if condition else b
```

Use when:
- Choosing between two values inline.

Avoid when:
- The logic is long or nested.

### `match-case`
```python
match value:
    case pattern1:
        action1()
    case _:
        fallback()
```

Use when:
- Matching values or structures.

### Pattern Matching
Useful patterns:
```python
case [x, y]
case {"name": name}
case Point(x, y)
case [first, *middle, last]
case (x, y) if x == y
```

---

## Interview Master Section

Below are more than 50 interview questions with concise but detailed answers.

### Beginner Questions
1. What is control flow in Python?
   - It is the order in which statements execute and the mechanisms that change that order.
2. What does an `if` statement do?
   - It conditionally executes a block when its condition is truthy.
3. What is the role of indentation in Python conditionals?
   - It defines the block that belongs to the branch.
4. What are truthy values?
   - Values that evaluate to `True` in Boolean contexts.
5. What are falsy values?
   - Values that evaluate to `False`, such as `0`, `None`, `""`, and empty collections.
6. What does `elif` mean?
   - Else if.
7. What is the purpose of `else`?
   - It provides a default branch when no earlier condition matches.
8. Can multiple `elif` branches run in one chain?
   - No, only the first matching branch runs.
9. What is a Boolean expression?
   - An expression that evaluates to true or false.
10. What does `not` do?
    - It flips a truth value.
11. What does `and` do?
    - It requires both operands to be truthy.
12. What does `or` do?
    - It requires at least one operand to be truthy.
13. What is a nested conditional?
    - A conditional placed inside another conditional.
14. What is a conditional expression?
    - An inline two-way expression like `a if cond else b`.
15. Is a conditional expression a statement?
    - No, it is an expression that produces a value.
16. Why is `else` required in a conditional expression?
    - Because the expression must always return one value or the other.
17. What is `match-case`?
    - Python's pattern matching statement introduced in 3.10.
18. What does `_` mean in `case _:`?
    - It is a wildcard that matches anything.
19. What is a guard in `match-case`?
    - An additional condition after the pattern.
20. When would `if x:` be used instead of `if len(x) > 0:`?
    - When testing whether a collection is non-empty in idiomatic Python.

### Intermediate Questions
21. What is first-match behavior?
    - Python evaluates conditional branches top-down and runs the first one that matches.
22. Why does branch ordering matter in `elif` chains?
    - Early broad conditions can make later specific branches unreachable.
23. What is short-circuit evaluation?
    - Python stops evaluating `and` or `or` once the final result is already known.
24. When should nested conditionals be refactored?
    - When readability, testability, or maintainability suffers.
25. What is a guard clause?
    - An early return or rejection branch that keeps the main path flat.
26. Why can separate `if` statements behave differently from `if` plus `elif`?
    - Separate `if` statements can each execute independently.
27. When is a conditional expression appropriate?
    - When choosing one of two simple values inline.
28. Why are nested ternaries discouraged?
    - They often reduce readability and increase mental load.
29. When is `match-case` better than `if-elif-else`?
    - When matching multiple literals, tuples, mappings, or nested structures.
30. What is a literal pattern?
    - A pattern that matches a specific fixed value.
31. What is a capture pattern?
    - A bare variable name that binds the matched value.
32. Why can a capture pattern cause bugs?
    - It matches broadly, so it may capture values when a literal match was intended.
33. What is a sequence pattern?
    - A pattern that matches ordered collections like lists or tuples.
34. What is a mapping pattern?
    - A pattern that matches dictionaries by required keys and subpatterns.
35. What is a class pattern?
    - A pattern that matches object instances and extracts attributes.
36. What is a star pattern?
    - A pattern like `*rest` that captures remaining sequence elements.
37. How does pattern matching help JSON parsing?
    - It combines shape validation and extraction into one branching form.
38. When are early returns better than nested `else` blocks?
    - When invalid cases can be rejected immediately.
39. What is a decision table?
    - A tabular representation of rule combinations and outcomes.
40. When should a dictionary dispatch be considered?
    - For direct fixed-key action mapping without complex conditions.

### Advanced Questions
41. What are signs that a conditional block is a code smell?
    - Deep nesting, repeated logic, long conditions, and mixed responsibilities.
42. Why are helper functions useful for complex conditions?
    - They name intent, improve reuse, and simplify tests.
43. How do guards differ from nested `if` inside a case block?
    - Guards refine the match itself before the branch body runs.
44. What is structural pattern matching?
    - Pattern-based branching that can inspect and decompose data structure.
45. How can conditionals affect API reliability?
    - They determine validation, fallback behavior, and consistent error handling.
46. When should pattern matching not be used?
    - When simple Boolean logic is clearer than structural patterns.
47. Why are default branches important in backend systems?
    - They protect against malformed input and unsupported states.
48. How can rule engines replace large conditional trees?
    - By moving policy into data or modular rule components.
49. What is strategy selection in control flow design?
    - Choosing among alternative algorithms or handlers based on context.
50. How can state machines simplify condition-heavy workflows?
    - They model valid transitions explicitly instead of scattering checks.
51. Why is readability considered a performance concern in teams?
    - Hard-to-read logic increases debugging cost and defect risk.
52. When should a condition be explicit instead of relying on truthiness?
    - When business meaning matters, such as `age >= 18` or `value is None`.
53. Why is `is None` preferred over `== None`?
    - It is the correct identity check for the singleton `None`.
54. How would you refactor a 20-branch `elif` chain?
    - Consider dispatch tables, state machines, strategy objects, decision tables, or `match-case`.
55. What is the main architectural advantage of pattern-based dispatching?
    - It aligns control flow with data shape and reduces manual extraction code.
56. When is a fallback `case _:` essential?
    - When untrusted or evolving inputs may introduce unexpected forms.
57. Why might complex branching belong outside a controller or API endpoint?
    - Business rules are often cleaner in service-layer abstractions.
58. How do conditionals influence test design?
    - Each branch usually requires targeted test coverage.
59. What is branch coverage?
    - A testing metric that checks whether each decision path has been executed.
60. What is the difference between validation branching and workflow branching?
    - Validation rejects bad input; workflow branching selects among valid business paths.

### Senior Python Developer Questions
61. How do you decide between `if-elif`, dispatch dictionaries, and `match-case`?
    - Use `if-elif` for general Boolean logic, dispatch dictionaries for direct key-to-function mapping, and `match-case` for structural or pattern-rich branching.
62. How do you keep policy logic maintainable in large applications?
    - Separate rules into named functions, tables, strategies, or domain services.
63. What conditional refactors most improve team velocity?
    - Guard clauses, extracted predicates, and declarative rule representations.
64. How do you review a pull request with complex branching?
    - Check readability, unreachable branches, branch order, fallback handling, and whether logic should move to a better abstraction.
65. What anti-pattern appears when every feature adds another conditional branch?
    - Condition explosion, often signaling that polymorphism or rule-driven design is needed.

---

## Exercise Section

### 20 Beginner Exercises
1. Print `"Positive"` if a number is greater than zero.
2. Print `"Negative"` if a number is less than zero.
3. Print `"Zero"` if a number is zero.
4. Check whether a string is empty.
5. Check whether a list has items.
6. Print `"Even"` or `"Odd"`.
7. Check whether age is at least 18.
8. Print `"Pass"` or `"Fail"` based on a score.
9. Validate that a password is at least 8 characters.
10. Check whether a file ends with `.csv`.
11. Use `elif` to classify temperature as cold, warm, or hot.
12. Use `else` to handle invalid login.
13. Write a simple nested conditional for age and ID check.
14. Use `and` to require two conditions.
15. Use `or` to allow either admin or owner access.
16. Use `not` to reject banned users.
17. Write a conditional expression to label adult or minor.
18. Use `match-case` to route menu option `1`, `2`, or fallback.
19. Match a dictionary with a `name` key.
20. Match a list with two values.

### Beginner Solutions
```python
number = 5
if number > 0:
    print("Positive")

number = -3
if number < 0:
    print("Negative")

number = 0
if number == 0:
    print("Zero")

text = ""
if text:
    print("Not empty")
else:
    print("Empty")

items = [1]
if items:
    print("Has items")

n = 4
if n % 2 == 0:
    print("Even")
else:
    print("Odd")

age = 19
if age >= 18:
    print("Adult")

score = 31
if score >= 35:
    print("Pass")
else:
    print("Fail")

password = "abc12345"
if len(password) >= 8:
    print("Valid")

filename = "report.csv"
if filename.endswith(".csv"):
    print("CSV")

temp = 15
if temp < 10:
    print("Cold")
elif temp < 25:
    print("Warm")
else:
    print("Hot")

logged_in = False
if logged_in:
    print("Welcome")
else:
    print("Invalid login")

age = 20
has_id = True
if age >= 18:
    if has_id:
        print("Allowed")

username = "u"
password = "p"
if username and password:
    print("Both present")

is_admin = False
is_owner = True
if is_admin or is_owner:
    print("Access")

is_banned = False
if not is_banned:
    print("Active")

age = 17
label = "Adult" if age >= 18 else "Minor"
print(label)

choice = "2"
match choice:
    case "1":
        print("Home")
    case "2":
        print("Settings")
    case _:
        print("Invalid")

person = {"name": "Ava"}
match person:
    case {"name": name}:
        print(name)

pair = [10, 20]
match pair:
    case [x, y]:
        print(x, y)
```

### 20 Intermediate Exercises
1. Classify marks into A, B, C, D, F.
2. Build a shipping calculator using `if-elif-else`.
3. Validate API payload keys.
4. Check login plus role authorization.
5. Build a menu router using `match-case`.
6. Parse success or error API response.
7. Match a tuple command and argument.
8. Use a guard to classify points.
9. Refactor nested conditions with guard clauses.
10. Build a file-type processor.
11. Route support tickets by severity.
12. Apply customer discounts by tier.
13. Check environment and deployment approval.
14. Match a nested user dictionary.
15. Match a list with first, middle, last.
16. Implement fallback config logic.
17. Check whether optional value is `None`.
18. Use Boolean logic for access control.
19. Build a simple state transition function.
20. Parse event payloads by type.

### Intermediate Solutions
```python
def grade(score):
    if score >= 90:
        return "A"
    elif score >= 80:
        return "B"
    elif score >= 70:
        return "C"
    elif score >= 60:
        return "D"
    return "F"


def shipping(amount):
    if amount >= 5000:
        return 0
    elif amount >= 2000:
        return 49
    return 99


def validate(payload):
    if not payload:
        return False
    if "id" not in payload or "email" not in payload:
        return False
    return True


def authorize(username, password, role):
    if username == "admin" and password == "secret":
        if role == "manager":
            return True
    return False


def menu(choice):
    match choice:
        case "1":
            return "create"
        case "2":
            return "delete"
        case _:
            return "invalid"


def parse_api(data):
    match data:
        case {"status": "success", "data": result}:
            return result
        case {"status": "error", "message": msg}:
            return msg
        case _:
            return None

cmd = ("delete", 101)
match cmd:
    case ("delete", item_id):
        print(item_id)

point = (0, 5)
match point:
    case (x, y) if x == 0:
        print("y-axis")


def process(user, quota):
    if not user:
        return "missing user"
    if quota <= 0:
        return "quota exceeded"
    return "ok"


def process_file(name):
    if name.endswith(".csv"):
        return "csv"
    elif name.endswith(".json"):
        return "json"
    return "other"


def route_ticket(score):
    if score >= 90:
        return "critical"
    elif score >= 70:
        return "high"
    return "normal"


def discount(customer):
    if customer == "vip":
        return 0.15
    elif customer == "premium":
        return 0.10
    return 0


def deploy(env, approved):
    if env == "production":
        if approved:
            return "deploy"
        return "wait"
    return "deploy"

user = {"user": {"name": "Tia"}}
match user:
    case {"user": {"name": name}}:
        print(name)

nums = [1, 2, 3, 4]
match nums:
    case [first, *middle, last]:
        print(first, middle, last)


def timeout(value):
    return value if value is not None else 30


def is_missing(value):
    return value is None


def can_access(is_admin, is_owner, active):
    return active and (is_admin or is_owner)


def next_state(state, event):
    match (state, event):
        case ("draft", "submit"):
            return "pending"
        case _:
            return "invalid"


def event_handler(event):
    match event:
        case {"type": "user.created", "payload": payload}:
            return payload
        case _:
            return None
```

### 20 Advanced Exercises
1. Design a discount rule engine.
2. Parse nested webhook payloads.
3. Build a state machine for order lifecycle.
4. Refactor a deep nested deployment workflow.
5. Build a pattern-based API validator.
6. Match dataclass tasks and dispatch actions.
7. Validate nested JSON with guards.
8. Route commands by tuple patterns.
9. Build a fraud-risk classifier.
10. Create a fallback chain for service discovery.
11. Implement feature-flag strategy selection.
12. Route data pipelines by file metadata.
13. Parse protocol messages by structure.
14. Design a notification dispatcher.
15. Build access control using multiple dimensions.
16. Normalize optional config values safely.
17. Replace large `elif` chain with `match-case`.
18. Create a rule table for eligibility logic.
19. Build a workflow engine with early returns.
20. Parse mixed-type event streams.

### Advanced Solutions
```python
def discount_rule_engine(order):
    rate = 0
    if order.get("customer") == "vip":
        rate += 0.15
    if order.get("amount", 0) >= 5000:
        rate += 0.10
    if order.get("coupon") == "SAVE":
        rate += 0.05
    return min(rate, 0.30)


def webhook_parser(payload):
    match payload:
        case {"event": "user.created", "data": {"user": {"id": user_id}}}:
            return user_id
        case _:
            return None


def order_state_machine(state, event):
    match (state, event):
        case ("new", "pay"):
            return "paid"
        case ("paid", "ship"):
            return "shipped"
        case ("shipped", "deliver"):
            return "delivered"
        case _:
            return "invalid"


def deploy_workflow(env, tests_ok, approved):
    if env != "production":
        return "deploy"
    if not tests_ok:
        return "block"
    if not approved:
        return "wait"
    return "deploy"


def validate_api(payload):
    match payload:
        case {"meta": {"version": version}, "data": data} if version >= 2:
            return data
        case _:
            return None

from dataclasses import dataclass

@dataclass
class EmailJob:
    to: str
    subject: str

@dataclass
class SmsJob:
    to: str
    message: str


def dispatch_job(job):
    match job:
        case EmailJob(to, subject):
            return f"email:{to}:{subject}"
        case SmsJob(to, message):
            return f"sms:{to}:{message}"
        case _:
            return "unknown"


def validate_json(data):
    match data:
        case {"items": items, "count": count} if count == len(items):
            return True
        case _:
            return False


def command_router(command):
    match command:
        case ("delete", item_id) if item_id > 0:
            return "delete"
        case ("create", _):
            return "create"
        case _:
            return "invalid"


def fraud(score, amount):
    if score >= 90 or amount > 100000:
        return "block"
    elif score >= 70:
        return "review"
    return "allow"


def service_discovery(primary, secondary, tertiary):
    if primary:
        return primary
    if secondary:
        return secondary
    return tertiary


def choose_strategy(feature_flag):
    return "new" if feature_flag else "legacy"


def route_pipeline(meta):
    match meta:
        case {"type": "csv", "size_mb": size} if size < 100:
            return "local"
        case {"type": "csv", "size_mb": size}:
            return "batch"
        case {"type": "parquet"}:
            return "warehouse"
        case _:
            return "unknown"


def protocol_parser(msg):
    match msg:
        case ["PING"]:
            return "pong"
        case ["AUTH", token]:
            return token
        case _:
            return "invalid"


def notification_dispatcher(event):
    match event:
        case {"channel": "email", "to": to}:
            return f"email:{to}"
        case {"channel": "sms", "to": to}:
            return f"sms:{to}"
        case _:
            return "unsupported"


def access_control(role, active, quota):
    if not active:
        return False
    if role == "admin":
        return True
    return quota > 0


def normalize_config(config):
    return {
        "timeout": config.get("timeout") if config.get("timeout") is not None else 30,
        "retries": config.get("retries") if config.get("retries") is not None else 3,
    }


def action(code):
    match code:
        case 1:
            return "create"
        case 2:
            return "update"
        case 3:
            return "delete"
        case _:
            return "unknown"

eligibility_rules = [
    {"age": 18, "income": 0, "eligible": True},
    {"age": 0, "income": 0, "eligible": False},
]


def workflow(payload):
    if not payload:
        return "missing"
    if not payload.get("valid"):
        return "invalid"
    if not payload.get("approved"):
        return "pending"
    return "complete"


def stream_parser(event):
    match event:
        case {"type": "metric", "value": value}:
            return ("metric", value)
        case {"type": "log", "message": message}:
            return ("log", message)
        case _:
            return ("unknown", None)
```

---

## Final Section

### Summary
Python control flow is the foundation of decision making in real programs. `if`, `elif`, and `else` handle classic branching, while conditional expressions provide compact value selection and `match-case` enables powerful structural pattern matching.

For beginners, the main goals are understanding truthiness, indentation, and branch order. For advanced developers, the focus shifts to maintainability, design patterns, data-driven rules, and readable structural dispatch.

### Key Takeaways
- Use `if` for single conditional execution.
- Use `elif` for multi-way mutually exclusive branching.
- Use `else` for explicit fallback behavior.
- Use nested conditionals only when decisions are truly hierarchical.
- Use conditional expressions for short inline value selection.
- Use `match-case` when matching values, tuples, mappings, and structured data.
- Use guards to refine patterns.
- Optimize first for clarity, then for maintainability, then for performance.

### Best Practices Checklist
- Keep conditions readable.
- Order branches carefully.
- Use descriptive Boolean variable names.
- Prefer explicit comparisons for business-critical logic.
- Use truthiness for simple emptiness checks.
- Add fallback branches for untrusted input.
- Refactor deep nesting with guard clauses.
- Extract complex conditions into helper functions.
- Use pattern matching only when it improves clarity.
- Test every meaningful branch.

### Common Mistakes Checklist
- Missing colons.
- Bad indentation.
- Using `=` instead of `==`.
- Writing unreachable `elif` branches.
- Using separate `if` statements when exclusivity is needed.
- Nesting too deeply.
- Overusing ternary expressions.
- Confusing capture patterns with literal matches.
- Forgetting wildcard fallbacks in `match-case`.
- Mixing validation, workflow logic, and side effects in one branch tree.

### Refactoring Guidelines
1. Flatten deep nesting with guard clauses.
2. Extract repeated conditions into helper functions.
3. Replace long literal-routing chains with `match-case` or dispatch tables.
4. Replace sprawling business logic with decision tables or rule objects.
5. Move domain rules out of controllers and scripts into reusable services.
6. Add tests before refactoring complicated branch logic.

### Learning Roadmap
1. Master Boolean expressions and truthiness.
2. Practice `if`, `elif`, and `else` with validation examples.
3. Learn nesting and then refactor it into flatter forms.
4. Use conditional expressions for concise assignments and returns.
5. Learn `match-case` for literal and tuple matching.
6. Study structural pattern matching for lists, dicts, and objects.
7. Apply control flow patterns in real projects.
8. Practice branch-heavy interview questions.

### Further Study Recommendations
- Exception handling with `try`, `except`, `else`, and `finally`.
- Loop control with `for`, `while`, `break`, and `continue`.
- Functional alternatives such as dictionary dispatch and higher-order functions.
- Object-oriented alternatives such as polymorphism and strategy classes.
- Testing techniques such as branch coverage and property-based testing.
