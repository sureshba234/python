<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Python Decorators Part 3: Async, Typing and Framework Applications

## Table of Contents

1. [Async Decorators](#section-1---async-decorators)
2. [Generator Decorators](#section-2---generator-decorators)
3. [Type-Safe Decorators](#section-3---type-safe-decorators)
4. [Framework Decorators](#section-4---framework-decorators)
5. [Decorator Stack Architectures](#section-5---decorator-stack-architectures)
6. [Performance Analysis](#section-6---performance-analysis)
7. [Memory Analysis](#section-7---memory-analysis)
8. [Production Patterns](#section-8---production-patterns)
9. [Exercises](#section-9---exercises)
10. [Interview Questions](#section-10---interview-questions)
11. [Production Best Practices \& Cheat Sheets](#final---production-best-practices-and-cheat-sheets)

***

## Section 1 — Async Decorators

### Understanding Async Fundamentals

**async functions** are functions declared with `async def` that return coroutine objects when called. They don't execute immediately—they must be awaited or scheduled on an event loop.

```python
import asyncio

async def fetch_data():
    await asyncio.sleep(1)
    return {"data": "important"}

# This creates a coroutine object, doesn't execute
coro = fetch_data()  # <coroutine object fetch_data at 0x...>

# This executes the coroutine
result = asyncio.run(fetch_data())  # {"data": "important"}
```

**Coroutine objects** are suspended execution contexts that can be resumed with `await`. They support the `__await__` protocol.

**Event loop interactions**: The event loop (managed by `asyncio`) schedules coroutines, handles I/O operations, and manages concurrent execution without blocking.

### Decorating Coroutines Correctly

The most critical rule: **async decorators must return async functions**. You cannot use a regular `def` wrapper around an `async def` function.

```python
import asyncio
import time

# ❌ WRONG: Regular wrapper around async function
def bad_decorator(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)  # Returns coroutine, doesn't await!
    return wrapper

# ❌ This breaks the async chain
@bad_decorator
async def bad_async():
    await asyncio.sleep(1)
    return "result"

# ✅ CORRECT: Async wrapper around async function
def good_decorator(func):
    async def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        return await func(*args, **kwargs)  # Properly awaits the coroutine
    return wrapper

# ✅ This preserves async behavior
@good_decorator
async def good_async():
    await asyncio.sleep(1)
    return "result"

# Test
async def main():
    result = await good_async()
    print(f"Result: {result}")

asyncio.run(main())
```


### Common Async Decorator Patterns

#### Timing Decorator for Async Functions

```python
import asyncio
import time
from functools import wraps

def async_timer(func):
    """Measure execution time of async functions."""
    @wraps(func)
    async def wrapper(*args, **kwargs):
        start = time.perf_counter()
        try:
            result = await func(*args, **kwargs)
            end = time.perf_counter()
            print(f"{func.__name__} took {end - start:.4f} seconds")
            return result
        except Exception as e:
            end = time.perf_counter()
            print(f"{func.__name__} failed after {end - start:.4f} seconds: {e}")
            raise
    return wrapper

@async_timer
async def slow_operation():
    await asyncio.sleep(2)
    return "done"

@async_timer
async def fast_operation():
    await asyncio.sleep(0.1)
    return "fast"

async def benchmark():
    await slow_operation()
    await fast_operation()
    await asyncio.gather(slow_operation(), fast_operation())

asyncio.run(benchmark())
```


#### Retry Decorator for Async Functions

```python
import asyncio
from functools import wraps

def async_retry(max_attempts=3, delay=1, backoff=2):
    """Retry async function on failure with exponential backoff."""
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            current_delay = delay
            for attempt in range(1, max_attempts + 1):
                try:
                    return await func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts:
                        raise
                    print(f"Attempt {attempt} failed: {e}. Waiting {current_delay}s...")
                    await asyncio.sleep(current_delay)
                    current_delay *= backoff
            return None
        return wrapper
    return decorator

@async_retry(max_attempts=3, delay=0.5)
async def flaky_api_call():
    if random.random() < 0.7:  # 70% failure rate
        raise ConnectionError("API unavailable")
    return {"status": "success"}

async def test_retry():
    try:
        result = await flaky_api_call()
        print(f"Success: {result}")
    except Exception as e:
        print(f"All attempts failed: {e}")

asyncio.run(test_retry())
```


#### Rate Limiter for Async Functions

```python
import asyncio
from functools import wraps
from collections import defaultdict

class AsyncRateLimiter:
    """Rate limiter using token bucket algorithm for async functions."""
    
    def __init__(self, rate: float, per: float = 1.0):
        self.rate = rate  # tokens per second
        self.per = per    # time window
        self.tokens = defaultdict(float)
        self.last_update = defaultdict(float)
    
    def __call__(self, func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            # Use function name + first arg as key (e.g., user_id)
            key = f"{func.__name__}:{args[0] if args else 'global'}"
            
            now = asyncio.get_event_loop().time()
            
            # Replenish tokens
            if key not in self.last_update:
                self.tokens[key] = self.rate * self.per
                self.last_update[key] = now
            else:
                elapsed = now - self.last_update[key]
                self.tokens[key] = min(
                    self.rate * self.per,
                    self.tokens[key] + elapsed * self.rate
                )
                self.last_update[key] = now
            
            # Check if we have tokens
            if self.tokens[key] >= 1:
                self.tokens[key] -= 1
                return await func(*args, **kwargs)
            else:
                # Wait for next token
                wait_time = (1 - self.tokens[key]) / self.rate
                await asyncio.sleep(wait_time)
                self.tokens[key] = 0
                self.last_update[key] = asyncio.get_event_loop().time()
                return await func(*args, **kwargs)
        
        return wrapper

rate_limiter = AsyncRateLimiter(rate=2, per=1)  # 2 requests per second

@rate_limiter
async def api_call(user_id):
    print(f"API call for user {user_id}")
    return {"user": user_id, "data": "response"}

async def demo_rate_limiter():
    # Simulate multiple users making requests
    tasks = [
        api_call(f"user{i}") for i in range(5)
    ]
    await asyncio.gather(*tasks)

asyncio.run(demo_rate_limiter())
```


### Async Context Manager Decorator

```python
import asyncio
from contextlib import asynccontextmanager
from functools import wraps

def async_with_context(context_factory):
    """Decorator to wrap async function with context manager."""
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            async with context_factory() as ctx:
                return await func(ctx, *args, **kwargs)
        return wrapper
    return decorator

@asynccontextmanager
async def db_connection():
    print("Connecting to database...")
    conn = {"connected": True}
    try:
        yield conn
    finally:
        print("Closing database connection...")

@async_with_context(db_connection)
async def query_database(conn, query):
    print(f"Executing query: {query}")
    return {"result": f"data from {query}"}

async def main():
    result = await query_database("SELECT * FROM users")
    print(result)

asyncio.run(main())
```


***

## Section 2 — Generator Decorators

### Generator Fundamentals

Generators use `yield` to produce values one-at-a-time, maintaining state between yields. `yield from` delegates to another generator.

```python
def simple_generator():
    yield 1
    yield 2
    yield 3

gen = simple_generator()
print(next(gen))  # 1
print(next(gen))  # 2
print(next(gen))  # 3
```


### Generator Decorator Patterns

#### Basic Generator Wrapper

```python
def generator_decorator(func):
    """Wrap a generator function with additional behavior."""
    def wrapper(*args, **kwargs):
        print(f"Starting {func.__name__}")
        gen = func(*args, **kwargs)
        
        for value in gen:
            print(f"Yielding: {value}")
            yield value
        
        print(f"Finished {func.__name__}")
    
    return wrapper

@generator_decorator
def number_generator(n):
    for i in range(n):
        yield i

gen = number_generator(3)
for num in gen:
    print(f"Received: {num}")
```


#### Generator Pipeline Decorator

```python
def pipeline(*transforms):
    """Apply multiple transformations to a generator pipeline."""
    def decorator(func):
        def wrapper(*args, **kwargs):
            gen = func(*args, **kwargs)
            
            # Apply each transform in sequence
            for transform in transforms:
                gen = transform(gen)
            
            return gen
        return wrapper
    return decorator

def filter_odd(gen):
    """Filter out odd numbers."""
    for value in gen:
        if value % 2 == 0:
            yield value

def multiply(gen, factor=2):
    """Multiply each value by factor."""
    for value in gen:
        yield value * factor

def add_suffix(gen, suffix=" processed"):
    """Add suffix to string values."""
    for value in gen:
        yield f"{value}{suffix}"

@pipeline(filter_odd, multiply, add_suffix)
def data_source():
    for i in range(10):
        yield i

gen = data_source()
for item in gen:
    print(item)
# Output: 0 processed, 4 processed, 8 processed, 12 processed, 16 processed
```


#### Generator Caching Decorator

```python
from functools import wraps

def generator_cache(func):
    """Cache generator results to avoid re-computation."""
    @wraps(func)
    def wrapper(*args, **kwargs):
        cache_key = f"{func.__name__}:{args}:{kwargs}"
        
        if not hasattr(wrapper, '_cache'):
            wrapper._cache = {}
        
        if cache_key not in wrapper._cache:
            wrapper._cache[cache_key] = list(func(*args, **kwargs))
        
        for value in wrapper._cache[cache_key]:
            yield value
    
    return wrapper

@generator_cache
def expensive_generator(n):
    print(f"Computing for {n}...")
    for i in range(n):
        yield i * i

# First call computes and caches
gen1 = expensive_generator(5)
print("First call:")
for item in gen1:
    print(item)

# Second call uses cache (no computation)
gen2 = expensive_generator(5)
print("Second call (from cache):")
for item in gen2:
    print(item)
```


#### Generator Transform with yield from

```python
def transform_with_yield_from(func):
    """Decorator that uses yield from to delegate to inner generator."""
    def wrapper(*args, **kwargs):
        print(f"Wrapper starting for {func.__name__}")
        
        # Pre-processing
        yield "pre-processing"
        
        # Delegate to inner generator using yield from
        result = yield from func(*args, **kwargs)
        
        # Post-processing
        yield f"post-processing, result: {result}"
        
        return result
    
    return wrapper

@transform_with_yield_from
def inner_generator(values):
    for value in values:
        yield f"processing {value}"
    return "done"

gen = inner_generator(["a", "b", "c"])
for item in gen:
    print(item)
```


### Advanced Generator Pipeline

```python
class GeneratorPipeline:
    """Advanced pipeline with error handling and metrics."""
    
    def __init__(self):
        self.transforms = []
        self.metrics = {"processed": 0, "errors": 0}
    
    def add_transform(self, transform):
        self.transforms.append(transform)
        return self
    
    def build(self, func):
        def wrapper(*args, **kwargs):
            gen = func(*args, **kwargs)
            
            for transform in self.transforms:
                gen = transform(gen)
            
            for value in gen:
                self.metrics["processed"] += 1
                try:
                    yield value
                except Exception as e:
                    self.metrics["errors"] += 1
                    print(f"Error: {e}")
        
        wrapper.metrics = self.metrics
        return wrapper

# Usage
pipeline = GeneratorPipeline()

def validate(gen):
    for value in gen:
        if value is not None:
            yield value

def sanitize(gen):
    for value in gen:
        yield str(value).strip()

def enrich(gen):
    for value in gen:
        yield {"value": value, "enriched": True}

@pipeline.add_transform(validate).add_transform(sanitize).add_transform(enrich).build
def data_reader():
    yield "  hello  "
    yield None
    yield "  world  "
    yield "test"

gen = data_reader()
for item in gen:
    print(item)

print(f"Metrics: {gen.metrics}")
```


***

## Section 3 — Type-Safe Decorators

### The Problem with Regular Decorators

Regular decorators often break type checking because they lose the original function's signature:

```python
from typing import Callable

def log(func: Callable) -> Callable:
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@log
def add(x: int, y: int) -> int:
    return x + y

# Type checker sees: add: Callable -> Callable (loses int, int -> int)
result = add("not int", "not int")  # Type checker won't catch this!
```


### typing.Callable with Precise Signatures

```python
from typing import Callable, TypeVar

# Basic approach: preserve exact signature
def log_v1(func: Callable[[int, int], int]) -> Callable[[int, int], int]:
    def wrapper(x: int, y: int) -> int:
        print(f"Calling with {x}, {y}")
        return func(x, y)
    return wrapper

# Limitation: Only works for specific signatures
```


### TypeVar for Generic Decorators

```python
from typing import TypeVar, Callable

F = TypeVar('F', bound=Callable)

def log_v2(func: F) -> F:
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

# Still loses type information in practice
```


### ParamSpec: The Modern Solution

`ParamSpec` (from `typing` or `typing_extensions`) captures callable parameters as a single type variable.

```python
from typing import Callable, TypeVar
from typing_extensions import ParamSpec

P = ParamSpec('P')
R = TypeVar('R')

def log(func: Callable[P, R]) -> Callable[P, R]:
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@log
def add(x: int, y: int) -> int:
    return x + y

@log
def greet(name: str, age: int) -> str:
    return f"Hello {name}, age {age}"

# Type checker now knows:
# add: (int, int) -> int
# greet: (str, int) -> str
result = add(1, 2)  # ✅ Type checked
result = add("1", "2")  # ❌ Type error!
```


### Concatenate: Adding Parameters

`Concatenate` allows decorators to add parameters while preserving the original signature.

```python
from typing import Callable, TypeVar
from typing_extensions import ParamSpec, Concatenate

P = ParamSpec('P')
R = TypeVar('R')

def add_prefix(prefix: str):
    def decorator(func: Callable[Concatenate[str, P], R]) -> Callable[P, R]:
        def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
            return func(prefix, *args, **kwargs)
        return wrapper
    return decorator

@add_prefix("ERROR: ")
def log_message(prefix: str, message: str, level: int) -> str:
    return f"{prefix}[level {level}] {message}"

# Original signature without prefix: (message: str, level: int) -> str
result = log_message("Something failed", 1)  # ✅ Type checked
```


### Complete Type-Safe Decorator Examples

#### Retry Decorator with Full Type Safety

```python
from typing import Callable, TypeVar
from typing_extensions import ParamSpec
import time

P = ParamSpec('P')
R = TypeVar('R')

def retry(max_attempts: int = 3):
    def decorator(func: Callable[P, R]) -> Callable[P, R]:
        def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    time.sleep(1)
            raise RuntimeError("Max attempts reached")
        return wrapper
    return decorator

@retry(max_attempts=5)
def fetch_data(url: str, timeout: int) -> dict:
    return {"url": url, "timeout": timeout}

@retry(max_attempts=3)
def calculate(x: int, y: int) -> int:
    return x + y

# Type checking works perfectly
data = fetch_data("https://api.example.com", 30)  # ✅ dict
result = calculate(5, 3)  # ✅ int
```


#### Cache Decorator with Type Safety

```python
from typing import Callable, TypeVar, Dict
from typing_extensions import ParamSpec
from functools import wraps

P = ParamSpec('P')
R = TypeVar('R')

def cache(func: Callable[P, R]) -> Callable[P, R]:
    cache_dict: Dict[tuple, R] = {}
    
    @wraps(func)
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        key = (args, tuple(kwargs.items()))
        if key not in cache_dict:
            cache_dict[key] = func(*args, **kwargs)
        return cache_dict[key]
    
    wrapper.cache = cache_dict  # Add cache attribute
    return wrapper

@cache
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

@cache
def greet(name: str) -> str:
    return f"Hello, {name}"

# Type checking preserved
fib = fibonacci(10)  # ✅ int
hello = greet("World")  # ✅ str
```


#### Throttle Decorator with Type Safety

```python
from typing import Callable, TypeVar
from typing_extensions import ParamSpec
import time

P = ParamSpec('P')
R = TypeVar('R')

def throttle(seconds: float):
    def decorator(func: Callable[P, R]) -> Callable[P, R]:
        last_call = 0.0
        
        def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
            current_time = time.time()
            if current_time - last_call < seconds:
                time.sleep(seconds - (current_time - last_call))
            last_call = current_time
            return func(*args, **kwargs)
        
        return wrapper
    return decorator

@throttle(1.0)
def send_email(to: str, subject: str, body: str) -> bool:
    return True

@throttle(0.5)
def log_message(level: str, message: str) -> None:
    print(f"{level}: {message}")

# Type checking works
email_sent = send_email("user@example.com", "Hello", "Body")  # ✅ bool
log_message("INFO", "Message")  # ✅ None
```


### Advanced ParamSpec Patterns

#### Decorator Chain with Type Safety

```python
from typing import Callable, TypeVar
from typing_extensions import ParamSpec

P = ParamSpec('P')
R = TypeVar('R')

def log(func: Callable[P, R]) -> Callable[P, R]:
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        print(f"Log: calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

def cache_result(func: Callable[P, R]) -> Callable[P, R]:
    cache: dict = {}
    
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        key = (args, tuple(kwargs.items()))
        if key not in cache:
            cache[key] = func(*args, **kwargs)
        return cache[key]
    
    return wrapper

def retry(max_attempts: int = 3):
    def decorator(func: Callable[P, R]) -> Callable[P, R]:
        def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
            for _ in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception:
                    pass
            raise RuntimeError("Failed")
        return wrapper
    return decorator

# Stack decorators - type safety preserved throughout
@log
@cache_result
@retry(max_attempts=5)
def expensive_computation(x: int, y: int) -> int:
    return x * y + x // y

# Type checker knows: (int, int) -> int
result = expensive_computation(10, 2)  # ✅ int
```


#### Generic Class Method Decorators

```python
from typing import Callable, TypeVar, Generic
from typing_extensions import ParamSpec

P = ParamSpec('P')
R = TypeVar('R')
T = TypeVar('T')

def instance_logger(func: Callable[P, R]) -> Callable[P, R]:
    def wrapper(self: T, *args: P.args, **kwargs: P.kwargs) -> R:
        print(f"[{self.__class__.__name__}] Calling {func.__name__}")
        return func(self, *args, **kwargs)
    return wrapper

class DataService:
    @instance_logger
    def fetch(self, url: str) -> dict:
        return {"url": url}
    
    @instance_logger
    def save(self, data: dict) -> bool:
        return True

service = DataService()
data = service.fetch("https://api.example.com")  # ✅ dict
saved = service.save({"key": "value"})  # ✅ bool
```


***

## Section 4 — Framework Decorators

### Flask Decorators

Flask uses decorators for route registration, view protection, and middleware.

```python
from flask import Flask, request, jsonify
from functools import wraps

app = Flask(__name__)

# Built-in route decorator
@app.route('/users/<int:user_id>')
@app.method(['GET', 'POST'])
def get_user(user_id):
    return jsonify({"user_id": user_id})

# Authentication decorator pattern
def require_auth(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        token = request.headers.get('Authorization')
        if not token or not validate_token(token):
            return jsonify({"error": "Authentication required"}), 401
        return f(*args, **kwargs)
    return decorated

@app.route('/authenticated')
@require_auth
def authenticated_view():
    return jsonify({"message": "Authenticated!"})

# CORS decorator
def cors(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        response = f(*args, **kwargs)
        response.headers['Access-Control-Allow-Origin'] = '*'
        return response
    return decorated

@app.route('/cors')
@cors
def cors_view():
    return jsonify({"cors": True})
```

**How Flask uses decorators internally:**

Flask's `@app.route` is a wrapper that registers the function in the URL map:

```python
class Flask:
    def route(self, rule, **options):
        def decorator(f):
            self.add_url_rule(rule, f, **options)
            return f
        return decorator
```


### FastAPI Decorators

FastAPI leverages decorators with type hints for automatic validation and documentation.

```python
from fastapi import FastAPI, HTTPException, Depends
from typing import Optional
from functools import wraps

app = FastAPI()

# Dependency injection (decorator-like)
def validate_token(token: str):
    if token != "valid":
        raise HTTPException(status_code=401, detail="Invalid token")
    return token

# Route decorator with type validation
@app.post("/items/")
async def create_item(
    name: str,
    price: float,
    token: str = Depends(validate_token)
):
    return {"name": name, "price": price}

# Custom dependency decorator
def rate_limit(max_requests: int = 100):
    def dependency(request: any):
        # Rate limiting logic
        pass
    return Depends(dependency)

@app.get("/limited")
@rate_limit(50)
async def limited_endpoint():
    return {"limited": True}

# Background tasks decorator pattern
@app.post("/process")
async def process_data(data: dict):
    # Process and add background task
    response = {"status": "processing"}
    response.background = BackgroundTask(process_background, data)
    return response
```

**FastAPI's internal decorator mechanism:**

```python
class FastAPI:
    def route(self, path: str):
        def decorator(func):
            self.router.add_route(path, func)
            return func
        return decorator
    
    def post(self, path: str):
        return self.route(path, methods=["POST"])
```


### Django Decorators

Django uses decorators for view protection, CSRF handling, and middleware.

```python
from django.http import JsonResponse
from django.views.decorators import csrf_exempt
from django.contrib.auth.decorators import login_required
from functools import wraps

# Built-in decorators
@login_required
def protected_view(request):
    return JsonResponse({"protected": True})

@csrf_exempt
def api_endpoint(request):
    return JsonResponse({"api": True})

# Custom permission decorator
def require_permission(permission):
    def decorator(view_func):
        @wraps(view_func)
        def wrapper(request, *args, **kwargs):
            if not request.user.has_permission(permission):
                return JsonResponse({"error": "Permission denied"}, status=403)
            return view_func(request, *args, **kwargs)
        return wrapper
    return decorator

@require_permission('edit_content')
def edit_content(request):
    return JsonResponse({"edited": True})

# Cache decorator
from django.core.cache import cache

def cache_page(seconds):
    def decorator(view_func):
        @wraps(view_func)
        def wrapper(request, *args, **kwargs):
            cache_key = f"page_{request.path}_{args}"
            cached = cache.get(cache_key)
            if cached:
                return cached
            response = view_func(request, *args, **kwargs)
            cache.set(cache_key, response, seconds)
            return response
        return wrapper
    return decorator

@cache_page(60 * 15)  # 15 minutes
def cached_view(request):
    return JsonResponse({"cached": True})
```

**Django's decorator internals:**

```python
def login_required(view_func):
    @wraps(view_func)
    def _wrapped_view(request, *args, **kwargs):
        if request.user.is_authenticated:
            return view_func(request, *args, **kwargs)
        return redirect('/login/')
    return _wrapped_view
```


### Click Decorators

Click uses decorators for CLI argument and option parsing.

```python
import click
from functools import wraps

# Built-in decorators
@click.command()
@click.option('--name', default='World', help='Name to greet')
@click.option('--count', default=1, help='Number of greetings')
def greet(name, count):
    for _ in range(count):
        click.echo(f'Hello {name}!')

# Custom decorator for validation
def validate_email(option):
    def decorator(f):
        @wraps(f)
        def wrapper(*args, **kwargs):
            email = kwargs.get(option)
            if email and '@' not in email:
                raise click.BadParameter('Invalid email')
            return f(*args, **kwargs)
        return wrapper
    return decorator

@click.command()
@click.option('--email', callback=validate_email('email'))
def send_email(email):
    click.echo(f'Sending to {email}')

# Timeout decorator
def timeout(seconds):
    def decorator(f):
        @wraps(f)
        def wrapper(*args, **kwargs):
            import signal
            def timeout_handler(signum, frame):
                raise click.Exit(f'Timed out after {seconds}s')
            signal.signal(signal.SIGALRM, timeout_handler)
            signal.alarm(seconds)
            try:
                return f(*args, **kwargs)
            finally:
                signal.alarm(0)
        return wrapper
    return decorator

@click.command()
@timeout(5)
def long_process():
    click.echo('Processing...')
    import time
    time.sleep(10)  # Will timeout
```

**Click's internal decorator pattern:**

```python
class Command:
    def __init__(self, name=None):
        self.params = []
    
    def option(self, *args, **attrs):
        def decorator(f):
            param = Option(args, **attrs)
            self.params.append(param)
            return f
        return decorator
```


### Typer Decorators

Typer (type-based CLI) combines Click with type hints.

```python
import typer
from typing import Optional

app = typer.Typer()

# Built-in decorators with type hints
@app.command()
def greet(name: str, count: int = 1):
    for _ in range(count):
        typer.echo(f'Hello {name}!')

@app.command()
def create(
    name: str,
    verbose: bool = typer.Option(False, help='Verbose output')
):
    if verbose:
        typer.echo(f'Creating {name}')
    typer.echo(f'Created {name}')

# Custom decorator for validation
def validate_positive():
    def callback(value: int):
        if value <= 0:
            raise typer.BadParameter('Must be positive')
        return value
    return callback

@app.command()
def process(
    iterations: int = typer.Option(
        1,
        callback=validate_positive(),
        help='Number of iterations'
    )
):
    typer.echo(f'Processing {iterations} times')
```


### pytest Decorators

pytest uses decorators for test configuration, fixtures, and marks.

```python
import pytest
from functools import wraps

# Built-in marks
@pytest.mark.parametrize("input,expected", [
    (1, 2),
    (2, 4),
    (3, 6),
])
def test_multiply(input, expected):
    assert input * 2 == expected

@pytest.mark.skip(reason="Not implemented yet")
def test_future_feature():
    pass

@pytest.mark.timeout(5)  # Requires pytest-timeout
def test_slow_operation():
    import time
    time.sleep(1)

# Custom fixture decorator pattern
def tracked_fixture(name=None):
    def decorator(func):
        fixture_name = name or func.__name__
        
        @pytest.fixture(name=fixture_name)
        @wraps(func)
        def wrapped(*args, **kwargs):
            print(f"Setting up {fixture_name}")
            result = func(*args, **kwargs)
            print(f"Tearing down {fixture_name}")
            return result
        
        return wrapped
    return decorator

@tracked_fixture()
def database_connection():
    return {"connected": True}

def test_with_db(database_connection):
    assert database_connection["connected"]

# Retry decorator for tests
def retry_test(max_attempts=3):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    print(f"Attempt {attempt} failed: {e}")
            return None
        return wrapper
    return decorator

@retry_test(max_attempts=5)
def test_flaky_api():
    import random
    if random.random() < 0.7:
        raise ConnectionError("API unavailable")
    assert True
```

**How pytest uses decorators internally:**

```python
class Mark:
    def __call__(self, obj):
        obj.pytestmark = self
        return obj

@pytest.mark
def parametrize(argnames, argvalues):
    def decorator(func):
        func.pytestparametrize = (argnames, argvalues)
        return func
    return decorator
```


### Unified Framework Decorator Pattern

```python
from typing import Callable, TypeVar
from typing_extensions import ParamSpec
from functools import wraps

P = ParamSpec('P')
R = TypeVar('R')

# Framework-agnostic decorator factory
def framework_decorator(
    name: str,
    before: Callable = None,
    after: Callable = None,
    error: Callable = None
):
    def decorator(func: Callable[P, R]) -> Callable[P, R]:
        @wraps(func)
        def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
            if before:
                before(name, args, kwargs)
            
            try:
                result = func(*args, **kwargs)
                if after:
                    after(name, result)
                return result
            except Exception as e:
                if error:
                    return error(name, e)
                raise
        
        wrapper._framework_decorator = name
        return wrapper
    return decorator

# Usage across frameworks
api_logging = framework_decorator(
    "api",
    before=lambda name, args, kwargs: print(f"API call: {name}"),
    after=lambda name, result: print(f"API result: {result}")
)

@api_logging
def flask_endpoint():
    return {"flask": True}

@api_logging
def fastapi_endpoint():
    return {"fastapi": True}
```


***

## Section 5 — Decorator Stack Architectures

### Middleware Systems

Decorator stacks form the backbone of modern middleware systems, particularly in web frameworks.

```python
from typing import Callable, TypeVar, List
from typing_extensions import ParamSpec
from functools import wraps
import time

P = ParamSpec('P')
R = TypeVar('R')

class MiddlewareStack:
    """Build middleware stacks from decorator composition."""
    
    def __init__(self):
        self.middleware: List[Callable] = []
    
    def add(self, middleware: Callable):
        self.middleware.append(middleware)
        return self
    
    def build(self, func: Callable[P, R]) -> Callable[P, R]:
        """Apply all middleware to function."""
        result = func
        for mw in reversed(self.middleware):
            result = mw(result)
        return result

# Example middleware
def logging_middleware(func: Callable[P, R]) -> Callable[P, R]:
    @wraps(func)
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        print(f"[LOG] Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

def auth_middleware(func: Callable[P, R]) -> Callable[P, R]:
    @wraps(func)
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        if 'auth' not in kwargs:
            raise PermissionError("Authentication required")
        return func(*args, **kwargs)
    return wrapper

def cache_middleware(func: Callable[P, R]) -> Callable[P, R]:
    cache: dict = {}
    
    @wraps(func)
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        key = (args, tuple(kwargs.items()))
        if key not in cache:
            cache[key] = func(*args, **kwargs)
        return cache[key]
    return wrapper

def rate_limit_middleware(func: Callable[P, R]) -> Callable[P, R]:
    request_count = 0
    
    @wraps(func)
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        request_count += 1
        if request_count > 100:
            raise RateLimitError("Too many requests")
        return func(*args, **kwargs)
    return wrapper

# Build stack
stack = MiddlewareStack()
stack.add(logging_middleware)
stack.add(auth_middleware)
stack.add(cache_middleware)
stack.add(rate_limit_middleware)

@stack.build
def api_handler(user_id: int, data: dict) -> dict:
    return {"user_id": user_id, "data": data}

# Execution flows through all middleware
result = api_handler(123, {"key": "value"}, auth="token")
```


### Request Pipelines

```python
from abc import ABC, abstractmethod
from typing import Any, Dict

class RequestMiddleware(ABC):
    """Abstract base for request middleware."""
    
    @abstractmethod
    def process_request(self, request: Dict) -> Dict:
        """Process incoming request."""
        pass
    
    @abstractmethod
    def process_response(self, response: Any) -> Any:
        """Process outgoing response."""
        pass

class ValidationMiddleware(RequestMiddleware):
    def process_request(self, request: Dict) -> Dict:
        if not request.get('data'):
            raise ValueError("Data required")
        request['validated'] = True
        return request
    
    def process_response(self, response: Any) -> Any:
        response['validated'] = True
        return response

class LoggingMiddleware(RequestMiddleware):
    def process_request(self, request: Dict) -> Dict:
        print(f"Processing request: {request}")
        request['logged'] = True
        return request
    
    def process_response(self, response: Any) -> Any:
        print(f"Response: {response}")
        response['logged'] = True
        return response

class Pipeline:
    """Request pipeline with middleware ordering."""
    
    def __init__(self):
        self.middleware: List[RequestMiddleware] = []
    
    def add(self, middleware: RequestMiddleware):
        self.middleware.append(middleware)
        return self
    
    def handle(self, request: Dict, handler: Callable) -> Any:
        # Process request through all middleware
        for mw in self.middleware:
            request = mw.process_request(request)
        
        # Execute handler
        response = handler(request)
        
        # Process response through all middleware (reverse)
        for mw in reversed(self.middleware):
            response = mw.process_response(response)
        
        return response

# Usage
pipeline = Pipeline()
pipeline.add(LoggingMiddleware())
pipeline.add(ValidationMiddleware())

def handle_request(request: Dict) -> Dict:
    return {"status": "success", "data": request.get('data')}

result = pipeline.handle({'data': 'test'}, handle_request)
```


### Plugin Systems

```python
from typing import Dict, Callable, List
from functools import wraps
import inspect

class PluginSystem:
    """Dynamic plugin system using decorator registration."""
    
    def __init__(self):
        self.plugins: Dict[str, Callable] = {}
        self.hooks: Dict[str, List[Callable]] = {}
    
    def register(self, name: str):
        """Decorator to register a plugin."""
        def decorator(func: Callable) -> Callable:
            self.plugins[name] = func
            
            # Auto-register hooks
            for hook_name in getattr(func, '_hooks', []):
                if hook_name not in self.hooks:
                    self.hooks[hook_name] = []
                self.hooks[hook_name].append(func)
            
            return func
        return decorator
    
    def hook(self, hook_name: str):
        """Decorator to mark a function as a hook."""
        def decorator(func: Callable) -> Callable:
            if not hasattr(func, '_hooks'):
                func._hooks = []
            func._hooks.append(hook_name)
            
            if hook_name not in self.hooks:
                self.hooks[hook_name] = []
            self.hooks[hook_name].append(func)
            
            return func
        return decorator
    
    def trigger(self, hook_name: str, *args, **kwargs):
        """Trigger all hooks for a given name."""
        results = []
        for hook in self.hooks.get(hook_name, []):
            results.append(hook(*args, **kwargs))
        return results

# Usage
plugins = PluginSystem()

@plugins.register("user_plugin")
@plugins.hook("on_create")
@plugins.hook("on_save")
def user_handler(data: dict) -> dict:
    data['user_processed'] = True
    return data

@plugins.register("audit_plugin")
@plugins.hook("on_save")
def audit_handler(data: dict) -> dict:
    data['audit_trail'] = 'saved'
    return data

# Trigger hooks
result = plugins.trigger("on_save", {"name": "John"})
print(result)  # [{'name': 'John', 'user_processed': True, 'audit_trail': 'saved'}, ...]

# Execute plugin
user_result = plugins.plugins["user_plugin"]({"name": "Jane"})
print(user_result)  # {'name': 'Jane', 'user_processed': True}
```


### Aspect-Oriented Programming

```python
from typing import Callable, TypeVar, List
from typing_extensions import ParamSpec
from functools import wraps
import inspect

P = ParamSpec('P')
R = TypeVar('R')

class Aspect:
    """Base class for aspects (AOP)."""
    
    def __call__(self, func: Callable[P, R]) -> Callable[P, R]:
        raise NotImplementedError

class TransactionAspect(Aspect):
    """Manage database transactions."""
    
    def __call__(self, func: Callable[P, R]) -> Callable[P, R]:
        @wraps(func)
        def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
            # Begin transaction
            print("BEGIN TRANSACTION")
            try:
                result = func(*args, **kwargs)
                # Commit transaction
                print("COMMIT TRANSACTION")
                return result
            except Exception as e:
                # Rollback transaction
                print(f"ROLLBACK TRANSACTION: {e}")
                raise
        return wrapper

class LoggingAspect(Aspect):
    """Log method calls and results."""
    
    def __call__(self, func: Callable[P, R]) -> Callable[P, R]:
        @wraps(func)
        def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
            print(f"CALL {func.__name__}({args}, {kwargs})")
            result = func(*args, **kwargs)
            print(f"RESULT {func.__name__}: {result}")
            return result
        return wrapper

class CacheAspect(Aspect):
    """Cache method results."""
    
    def __init__(self, key_func: Callable = None):
        self.cache: dict = {}
        self.key_func = key_func
    
    def __call__(self, func: Callable[P, R]) -> Callable[P, R]:
        @wraps(func)
        def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
            key = self.key_func(*args, **kwargs) if self.key_func else (args, kwargs)
            if key not in self.cache:
                self.cache[key] = func(*args, **kwargs)
            return self.cache[key]
        return wrapper

class MetricsAspect(Aspect):
    """Collect performance metrics."""
    
    def __init__(self):
        self.metrics: dict = {}
    
    def __call__(self, func: Callable[P, R]) -> Callable[P, R]:
        @wraps(func)
        def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
            import time
            start = time.perf_counter()
            result = func(*args, **kwargs)
            end = time.perf_counter()
            
            if func.__name__ not in self.metrics:
                self.metrics[func.__name__] = {'calls': 0, 'total_time': 0}
            
            self.metrics[func.__name__]['calls'] += 1
            self.metrics[func.__name__]['total_time'] += end - start
            
            return result
        return wrapper

# Compose aspects
def apply_aspects(*aspects: Aspect):
    def decorator(func: Callable[P, R]) -> Callable[P, R]:
        result = func
        for aspect in reversed(aspects):
            result = aspect(result)
        return result
    return decorator

# Usage
metrics = MetricsAspect()
cache = CacheAspect(key_func=lambda x: x)

@apply_aspects(
    TransactionAspect(),
    LoggingAspect(),
    cache,
    metrics
)
def process_user(user_id: int) -> dict:
    return {"user_id": user_id, "processed": True}

@apply_aspects(
    TransactionAspect(),
    LoggingAspect()
)
def save_data(data: dict) -> bool:
    return True

# Execute
result1 = process_user(123)
result2 = process_user(123)  # Cached
result3 = save_data({"key": "value"})

print(f"Metrics: {metrics.metrics}")
```


### Architecture Diagrams

```
 request  →  →  ──┐
                  │
    ┌─────────────────────────┐
    │   Middleware Stack      │
    │  ┌─────────────────┐    │
    │  │  Logging MW     │    │
    │  └─────────────────┘    │
    │  ┌─────────────────┐    │
    │  │  Auth MW        │    │
    │  └─────────────────┘    │
    │  ┌─────────────────┐    │
    │  │  Cache MW       │    │
    │  └─────────────────┘    │
    │  ┌─────────────────┐    │
    │  │  RateLimit MW   │    │
    │  └─────────────────┘    │
    └─────────────────────────┘
                  │
                  ↓
         ┌───────────────┐
         │   Handler     │
         │  (Your Code)  │
         └───────────────┘
                  │
                  ↓
    ┌─────────────────────────┐
    │   Response Pipeline     │
    │  (Reverse Middleware)   │
    └─────────────────────────┘
                  │
                  ↓
               response
```

```
Aspect Composition Flow:

    @TransactionAspect()
    @LoggingAspect()
    @CacheAspect()
    @MetricsAspect()
    def my_function():
        ...

    ↓ Applies in reverse order ↓

    my_function = TransactionAspect(
        LoggingAspect(
            CacheAspect(
                MetricsAspect(
                    original_function
                )
            )
        )
    )

    Execution Order:
    1. Metrics (start)
    2. Cache (check)
    3. Logging (call)
    4. Transaction (begin)
    5. original_function()
    6. Transaction (commit/rollback)
    7. Logging (result)
    8. Cache (store)
    9. Metrics (end)
```


***

## Section 6 — Performance Analysis

### Benchmarking with timeit

```python
import timeit
from functools import wraps

# Simple function
def add(x, y):
    return x + y

# With decorator
def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        end = time.perf_counter()
        return result
    return wrapper

@timer
def add_decorated(x, y):
    return x + y

# Benchmark single function
print("add:", timeit.timeit('add(1, 2)', globals={'add': add}, number=1000000))
print("add_decorated:", timeit.timeit('add_decorated(1, 2)', globals={'add_decorated': add_decorated}, number=1000000))

# Wrapper overhead benchmark
def benchmark_overhead():
    """Measure decorator wrapper overhead."""
    
    def no_decorator(x, y):
        return x + y
    
    def with_decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            return func(*args, **kwargs)
        return wrapper
    
    @with_decorator
    def decorated(x, y):
        return x + y
    
    # Benchmark 1M calls
    no_time = timeit.timeit('no_decorator(1, 2)', globals={'no_decorator': no_decorator}, number=1000000)
    dec_time = timeit.timeit('decorated(1, 2)', globals={'decorated': decorated}, number=1000000)
    
    overhead = dec_time - no_time
    overhead_per_call = overhead / 1000000
    
    print(f"No decorator: {no_time:.4f}s")
    print(f"With decorator: {dec_time:.4f}s")
    print(f"Overhead: {overhead:.4f}s ({overhead_per_call:.2e}s per call)")
    print(f"Overhead percentage: {(overhead/no_time)*100:.2f}%")

benchmark_overhead()
```


### Closure Overhead Analysis

```python
import timeit

# Without closure
def simple_add(x, y):
    return x + y

# With closure
def make_adder():
    constant = 10
    def add_with_closure(x, y):
        return x + y + constant
    return add_with_closure

add_with_closure = make_adder()

# Benchmark
simple_time = timeit.timeit('simple_add(1, 2)', globals={'simple_add': simple_add}, number=1000000)
closure_time = timeit.timeit('add_with_closure(1, 2)', globals={'add_with_closure': add_with_closure}, number=1000000)

print(f"Simple function: {simple_time:.4f}s")
print(f"Function with closure: {closure_time:.4f}s")
print(f"Closure overhead: {(closure_time - simple_time):.4f}s ({(closure_time/simple_time - 1)*100:.2f}%)")
```


### Cache Impact Benchmark

```python
import timeit
from functools import lru_cache

# Without cache
def factorial_no_cache(n):
    if n <= 1:
        return 1
    return n * factorial_no_cache(n - 1)

# With lru_cache
@lru_cache(maxsize=None)
def factorial_with_cache(n):
    if n <= 1:
        return 1
    return n * factorial_with_cache(n - 1)

# First call (no cache benefit)
print("First call (computing):")
print(f"  No cache: {timeit.timeit('factorial_no_cache(20)', globals={'factorial_no_cache': factorial_no_cache}, number=10000):.4f}s")
print(f"  With cache: {timeit.timeit('factorial_with_cache(20)', globals={'factorial_with_cache': factorial_with_cache}, number=10000):.4f}s")

# Second call (cache benefit)
print("Second call (cached):")
print(f"  No cache: {timeit.timeit('factorial_no_cache(20)', globals={'factorial_no_cache': factorial_no_cache}, number=10000):.4f}s")
print(f"  With cache: {timeit.timeit('factorial_with_cache(20)', globals={'factorial_with_cache': factorial_with_cache}, number=10000):.4f}s")

# Custom cache decorator overhead
def manual_cache(func):
    cache = {}
    def wrapper(*args):
        if args in cache:
            return cache[args]
        cache[args] = func(*args)
        return cache[args]
    return wrapper

@manual_cache
def factorial_manual_cache(n):
    if n <= 1:
        return 1
    return n * factorial_manual_cache(n - 1)

print("Custom cache overhead:")
print(f"  lru_cache: {timeit.timeit('factorial_with_cache(20)', globals={'factorial_with_cache': factorial_with_cache}, number=100000):.4f}s")
print(f"  manual: {timeit.timeit('factorial_manual_cache(20)', globals={'factorial_manual_cache': factorial_manual_cache}, number=100000):.4f}s")
```


### cProfile Analysis

```python
import cProfile
from functools import wraps

def profile_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        profiler = cProfile.Profile()
        result = profiler.runcall(func, *args, **kwargs)
        profiler.print_stats()
        return result
    return wrapper

@profile_decorator
def complex_operation(n):
    result = 0
    for i in range(n):
        for j in range(n):
            result += i * j
    return result

# Run with profiling
complex_operation(100)

# Alternative: Profile entire script
def profile_script():
    def fast_operation(n):
        return sum(i * j for i in range(n) for j in range(n))
    
    def slow_operation(n):
        result = 0
        for i in range(n):
            for j in range(n):
                result += i * j
        return result
    
    profiler = cProfile.Profile()
    
    profiler.runcall(fast_operation, 1000)
    profiler.runcall(slow_operation, 1000)
    
    profiler.print_stats(sort='cumulative')

profile_script()
```


### Complete Performance Benchmark Suite

```python
import timeit
import cProfile
from functools import wraps, lru_cache
from typing import Callable, TypeVar
from typing_extensions import ParamSpec

P = ParamSpec('P')
R = TypeVar('R')

class PerformanceBenchmark:
    """Comprehensive performance benchmarking for decorators."""
    
    def __init__(self):
        self.results = {}
    
    def benchmark(self, name: str, func: Callable, number: int = 100000):
        """Run timeit benchmark."""
        time = timeit.timeit(lambda: func(), number=number)
        self.results[name] = {'time': time, 'per_call': time / number}
        print(f"{name}: {time:.4f}s ({time/number:.2e}s per call)")
        return time
    
    def profile(self, name: str, func: Callable):
        """Run cProfile analysis."""
        profiler = cProfile.Profile()
        profiler.runcall(func)
        
        stats = profiler.getstats()
        print(f"\n{name} profiling:")
        for stat in stats[:5]:  # Top 5 functions
            print(f"  {stat.code_context[0]}: {stat.ncalls} calls, {stat.ttime:.4f}s")
    
    def compare(self, base_name: str, decorated_name: str):
        """Compare base vs decorated performance."""
        if base_name not in self.results or decorated_name not in self.results:
            raise ValueError("Both benchmarks must be run first")
        
        base = self.results[base_name]
        dec = self.results[decorated_name]
        
        overhead = dec['time'] - base['time']
        overhead_pct = (overhead / base['time']) * 100
        
        print(f"\n{'='*50}")
        print(f"Comparison: {base_name} vs {decorated_name}")
        print(f"{'='*50}")
        print(f"Base time: {base['time']:.4f}s")
        print(f"Decorated time: {dec['time']:.4f}s")
        print(f"Overhead: {overhead:.4f}s ({overhead_pct:.2f}%)")
        print(f"Overhead per call: {dec['per_call'] - base['per_call']:.2e}s")

# Benchmark suite
benchmark = PerformanceBenchmark()

# 1. Simple function
def simple_add():
    return 1 + 2

benchmark.benchmark("simple_add", simple_add)

# 2. With @wraps
def with_wraps(func):
    @wraps(func)
    def wrapper():
        return func()
    return wrapper

@with_wraps
def decorated_with_wraps():
    return 1 + 2

benchmark.benchmark("decorated_with_wraps", decorated_with_wraps)

# 3. With custom timing
def with_timing(func):
    def wrapper():
        start = time.perf_counter()
        result = func()
        end = time.perf_counter()
        return result
    return wrapper

@with_timing
def decorated_with_timing():
    return 1 + 2

benchmark.benchmark("decorated_with_timing", decorated_with_timing)

# 4. With lru_cache
@lru_cache(maxsize=None)
def cached_add():
    return 1 + 2

benchmark.benchmark("cached_add", cached_add)

# 5. With closure
def with_closure():
    constant = 10
    def wrapper():
        return 1 + 2 + constant
    return wrapper

decorated_closure = with_closure()
benchmark.benchmark("decorated_closure", decorated_closure)

# Compare results
benchmark.compare("simple_add", "decorated_with_wraps")
benchmark.compare("simple_add", "decorated_with_timing")
benchmark.compare("simple_add", "cached_add")

# Profile complex operation
def complex_op():
    return sum(i * j for i in range(100) for j in range(100))

benchmark.profile("complex_op", complex_op)
```


***

## Section 7 — Memory Analysis

### Using gc for Memory Analysis

```python
import gc
import sys
from functools import wraps

# Enable garbage collection debugging
gc.set_debug(gc.DEBUG_STATS)

def analyze_memory(func):
    """Decorator to analyze memory usage."""
    @wraps(func)
    def wrapper(*args, **kwargs):
        # Get object count before
        gc.collect()
        before_count = len(gc.get_objects())
        
        # Execute function
        result = func(*args, **kwargs)
        
        # Get object count after
        gc.collect()
        after_count = len(gc.get_objects())
        
        print(f"Objects before: {before_count}")
        print(f"Objects after: {after_count}")
        print(f"New objects: {after_count - before_count}")
        
        return result
    return wrapper

@analyze_memory
def create_objects(n):
    return [i for i in range(n)]

create_objects(1000)

# Find garbage
print(f"\nGarbage: {gc.garbage}")
```


### weakref for Reference Tracking

```python
import weakref
import sys
from functools import wraps

class TrackedObject:
    """Object with weak reference tracking."""
    
    def __init__(self, name):
        self.name = name
        self._weakref = weakref.ref(self)
    
    def is_alive(self):
        return self._weakref() is not None

def weakref_decorator(func):
    """Decorator that tracks object references with weakref."""
    live_references = []
    
    @wraps(func)
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        
        # Create weak reference if result is an object
        if hasattr(result, '__dict__'):
            try:
                weak = weakref.ref(result, lambda ref: print(f"{func.__name__} result deleted"))
                live_references.append(weak)
            except TypeError:
                pass  # Object doesn't support weakref
        
        return result
    
    wrapper.live_references = live_references
    return wrapper

@weakref_decorator
def createtracked():
    return TrackedObject("test")

obj = createtracked()
print(f"Object alive: {obj.is_alive()}")

# Delete object
del obj

# Check if garbage collected
import gc
gc.collect()
print(f"Live references: {len([ref for ref in createtracked.live_references if ref() is not None])}")
```


### sys.getsizeof for Size Analysis

```python
import sys
from functools import wraps

def size_decorator(func):
    """Decorator to measure object sizes."""
    @wraps(func)
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        
        def get_total_size(obj):
            """Get total size of object and its contents."""
            size = sys.getsizeof(obj)
            
            if isinstance(obj, dict):
                size += sum(get_total_size(v) for v in obj.values())
                size += sum(get_total_size(k) for k in obj.keys())
            elif isinstance(obj, (list, tuple, set)):
                size += sum(get_total_size(item) for item in obj)
            
            return size
        
        total_size = get_total_size(result)
        print(f"{func.__name__} result size: {sys.getsizeof(result)} bytes")
        print(f"{func.__name__} total size: {total_size} bytes")
        
        return result
    return wrapper

@size_decorator
def create_large_dict():
    return {f"key{i}": i for i in range(1000)}

@size_decorator
def create_large_list():
    return [i for i in range(1000)]

create_large_dict()
create_large_list()
```


### Closure Memory Analysis

```python
import sys
import gc
from functools import wraps

# Analyze closure memory
def analyze_closure():
    """Analyze memory captured by closures."""
    
    # Without closure
    def simple_func(x, y):
        return x + y
    
    # With closure
    def make_with_closure(constant):
        def func_with_closure(x, y):
            return x + y + constant
        return func_with_closure
    
    func_closure = make_with_closure(100)
    
    # Check closure variables
    print("Closure variables:")
    print(f"  simple_func closures: {simple_func.__closure__}")
    print(f"  func_closure closures: {func_closure.__closure__}")
    
    if func_closure.__closure__:
        for i, cell in enumerate(func_closure.__closure__):
            print(f"  Cell {i}: {cell.cell_contents} ({sys.getsizeof(cell.cell_contents)} bytes)")
    
    # Size comparison
    print(f"\nSize comparison:")
    print(f"  simple_func: {sys.getsizeof(simple_func)} bytes")
    print(f"  func_closure: {sys.getsizeof(func_closure)} bytes")

analyze_closure()
```


### Captured Objects Analysis

```python
import sys
from functools import wraps

def captured_objects_decorator(func):
    """Decorator to track captured objects in closures."""
    
    # Capture some objects
    captured_list = [i for i in range(100)]
    captured_dict = {f"key{i}": i for i in range(100)}
    captured_string = "large string " * 100
    
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    
    # Analyze captured objects
    print(f"\nCaptured objects in {func.__name__}:")
    print(f"  captured_list: {sys.getsizeof(captured_list)} bytes")
    print(f"  captured_dict: {sys.getsizeof(captured_dict)} bytes")
    print(f"  captured_string: {sys.getsizeof(captured_string)} bytes")
    print(f"  Total captured: {sys.getsizeof(captured_list) + sys.getsizeof(captured_dict) + sys.getsizeof(captured_string)} bytes")
    
    return wrapper

@captured_objects_decorator
def example_function():
    return "result"

example_function()
```


### Reference Cycle Detection

```python
import gc
import sys
from functools import wraps

def detect_reference_cycles():
    """Detect reference cycles created by decorators."""
    
    # Create reference cycle
    class Node:
        def __init__(self, name):
            self.name = name
            self.parent = None
            self.children = []
        
        def add_child(self, child):
            child.parent = self
            self.children.append(child)
    
    # Create cycle
    parent = Node("parent")
    child = Node("child")
    parent.add_child(child)
    
    # Create cycle
    child.add_child(parent)
    
    # Check for cycles
    print(f"Objects in garbage: {len(gc.garbage)}")
    
    # Force garbage collection
    gc.collect()
    print(f"After gc.collect(), objects in garbage: {len(gc.garbage)}")
    
    # Find reference cycles
    gc.set_debug(gc.DEBUG_SAVEALL)
    gc.collect()
    gc.set_debug(0)
    
    print(f"\nGarbage objects: {len(gc.garbage)}")
    for obj in gc.garbage[:5]:
        print(f"  {type(obj).__name__}: {obj}")

detect_reference_cycles()

# Decorator that creates reference cycles (BAD)
def bad_cycle_decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    
    # Create cycle: wrapper -> func -> wrapper
    wrapper.func = func
    func.wrapper = wrapper
    
    return wrapper

# Decorator without cycles (GOOD)
def good_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@bad_cycle_decorator
def bad_function():
    return "bad"

@good_decorator
def good_function():
    return "good"

# Check for cycles
gc.collect()
print(f"\nBad function cycles: {len([obj for obj in gc.get_objects() if hasattr(obj, 'wrapper') and hasattr(obj, 'func')])}")
```


### Memory Leak Detection

```python
import gc
import sys
from functools import wraps
import time

class MemoryLeakDetector:
    """Detect memory leaks in decorator usage."""
    
    def __init__(self):
        self.baseline = None
        self.measurements = []
    
    def baseline(self):
        """Set baseline memory measurement."""
        gc.collect()
        self.baseline = len(gc.get_objects())
        print(f"Baseline objects: {self.baseline}")
    
    def measure(self, name: str):
        """Measure object count after operation."""
        gc.collect()
        count = len(gc.get_objects())
        self.measurements.append((name, count))
        
        if self.baseline:
            delta = count - self.baseline
            print(f"{name}: {count} objects (+{delta} from baseline)")
        else:
            print(f"{name}: {count} objects")
    
    def report(self):
        """Report memory leak analysis."""
        print(f"\n{'='*50}")
        print("Memory Leak Report")
        print(f"{'='*50}")
        
        if not self.measurements:
            print("No measurements taken")
            return
        
        for name, count in self.measurements:
            delta = count - self.baseline if self.baseline else 0
            trend = "↑ LEAK" if delta > 1000 else "✓ OK"
            print(f"{name}: +{delta} objects {trend}")

detector = MemoryLeakDetector()
detector.baseline()

# Decorator that might leak
def leaky_decorator(func):
    cache = {}  # Never cleared!
    
    @wraps(func)
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    
    return wrapper

@leaky_decorator
def leaky_function(n):
    return [i for i in range(n)]

# Call many times
for i in range(100):
    leaky_function(100)

detector.measure("After 100 leaky calls")

# Clear leak
import gc
for obj in gc.get_objects():
    if isinstance(obj, dict) and len(obj) > 1000:
        obj.clear()

detector.measure("After clearing cache")
```


***

## Section 8 — Production Patterns

### API Rate Limiting

```python
import time
from typing import Callable, TypeVar, Dict
from typing_extensions import ParamSpec
from functools import wraps
import threading

P = ParamSpec('P')
R = TypeVar('R')

class RateLimiter:
    """Production-ready rate```

