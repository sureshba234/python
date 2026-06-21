<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# \# ROLE

You are a Python Architect, OOP Expert, Technical Author, Computer Science Instructor, and Senior Software Engineer.

# TASK

Create a COMPLETE Markdown guide on:

1. Composition
2. Aggregation

# OUTPUT RULES

Generate ONLY valid Markdown.

Start with:

# Python Relationships and Reusability – Part 2

Include a detailed Table of Contents.

Explain everything from beginner to advanced level.

Use diagrams, tables, notes, warnings, code examples, enterprise examples, and design-focused explanations.

# COVER EVERYTHING

## Composition

Include:

- Definition
- Why composition exists
- Has-A relationship
- Strong ownership
- Object composition
- Nested objects


### Composition Architecture

Include:

- Object containment
- Lifecycle dependency
- Object creation
- Object destruction
- Ownership concepts


### Composition Patterns

Include:

- Component-based systems
- Service composition
- Modular design
- Domain-driven design examples


### Real World Examples

Examples from:

- Car and Engine
- House and Rooms
- Computer and CPU
- Smartphone and Components


### Enterprise Examples

Examples from:

- Microservices
- Django applications
- Game engines
- ERP systems
- SaaS products


### Memory Relationship Explanations

Include:

- Object references
- Memory diagrams
- Lifecycle diagrams


### Advantages

### Disadvantages

### Best Practices

### Anti-Patterns

### Common Mistakes

### Interview Questions

### Exercises


---

## Aggregation

Include:

- Definition
- Why aggregation exists
- Weak ownership
- Shared objects
- Independent lifecycle


### Aggregation Architecture

Include:

- Object association
- Shared references
- Object independence
- Memory relationships


### Aggregation Examples

Examples from:

- Department and Employee
- School and Teacher
- Team and Player
- Library and Book


### Enterprise Examples

Examples from:

- CRM systems
- ERP systems
- Healthcare software
- Educational platforms


### Memory Relationship Explanations

Include:

- Object references
- Shared ownership
- Lifecycle diagrams


### Advantages

### Disadvantages

### Best Practices

### Anti-Patterns

### Common Mistakes

### Interview Questions

### Exercises


---

## Composition vs Aggregation

Cover:

- Ownership
- Lifecycle
- Coupling
- Flexibility
- Maintainability
- Performance
- Scalability

Include:

- UML-style diagrams
- Side-by-side comparison tables
- Decision framework
- When to use each approach

---

## Design Perspective

Cover:

- Favor Composition Over Inheritance
- SOLID Principles
- Dependency Injection
- Clean Architecture
- Domain Driven Design
- Design Patterns using Composition

Include examples and diagrams.

# SPECIAL REQUIREMENTS

For every major topic include:

- Definition
- Concept explanation
- Diagrams
- Code examples
- Real-world analogy
- Enterprise usage
- Pitfalls
- Best practices

End with:

# Part 2 Summary

Include:

- Composition Cheat Sheet
- Aggregation Cheat Sheet
- Composition vs Aggregation Comparison Table
- Architecture Decision Guide
- Practice Problems
- Advanced Exercises

```python
# Python Relationships and Reusability – Part 2

# This script will generate the complete Markdown guide
markdown_content = """# Python Relationships and Reusability – Part 2

> **Author**: Python Architect & OOP Expert  
> **Level**: Beginner → Advanced → Enterprise  
> **Focus**: Composition, Aggregation, and Design Principles

---

## Table of Contents

1. [Introduction to Object Relationships](#introduction-to-object-relationships)
2. [Composition](#composition)
   - [Definition and Core Concepts](#definition-and-core-concepts)
   - [Why Composition Exists](#why-composition-exists)
   - [Has-A Relationship and Strong Ownership](#has-a-relationship-and-strong-ownership)
   - [Composition Architecture](#composition-architecture)
   - [Composition Patterns](#composition-patterns)
   - [Real World Examples](#real-world-examples)
   - [Enterprise Examples](#enterprise-examples)
   - [Memory Relationship Explanations](#memory-relationship-explanations)
   - [Advantages](#advantages-1)
   - [Disadvantages](#disadvantages-1)
   - [Best Practices](#best-practices-1)
   - [Anti-Patterns](#anti-patterns-1)
   - [Common Mistakes](#common-mistakes-1)
   - [Interview Questions](#interview-questions-1)
   - [Exercises](#exercises-1)
3. [Aggregation](#aggregation)
   - [Definition and Core Concepts](#definition-and-core-concepts-1)
   - [Why Aggregation Exists](#why-aggregation-exists)
   - [Weak Ownership and Independent Lifecycle](#weak-ownership-and-independent-lifecycle)
   - [Aggregation Architecture](#aggregation-architecture)
   - [Aggregation Examples](#aggregation-examples)
   - [Enterprise Examples](#enterprise-examples-1)
   - [Memory Relationship Explanations](#memory-relationship-explanations-1)
   - [Advantages](#advantages-2)
   - [Disadvantages](#disadvantages-2)
   - [Best Practices](#best-practices-2)
   - [Anti-Patterns](#anti-patterns-2)
   - [Common Mistakes](#common-mistakes-2)
   - [Interview Questions](#interview-questions-2)
   - [Exercises](#exercises-2)
4. [Composition vs Aggregation](#composition-vs-aggregation)
   - [Key Differences](#key-differences)
   - [UML-Style Diagrams](#uml-style-diagrams)
   - [Side-by-Side Comparison Tables](#side-by-side-comparison-tables)
   - [Decision Framework](#decision-framework)
5. [Design Perspective](#design-perspective)
   - [Favor Composition Over Inheritance](#favor-composition-over-inheritance)
   - [SOLID Principles](#solid-principles)
   - [Dependency Injection](#dependency-injection)
   - [Clean Architecture](#clean-architecture)
   - [Domain Driven Design](#domain-driven-design)
   - [Design Patterns using Composition](#design-patterns-using-composition)
6. [Part 2 Summary](#part-2-summary)
   - [Composition Cheat Sheet](#composition-cheat-sheet)
   - [Aggregation Cheat Sheet](#aggregation-cheat-sheet)
   - [Composition vs Aggregation Comparison Table](#composition-vs-aggregation-comparison-table)
   - [Architecture Decision Guide](#architecture-decision-guide)
   - [Practice Problems](#practice-problems)
   - [Advanced Exercises](#advanced-exercises)

---

## Introduction to Object Relationships

In object-oriented programming, classes don't exist in isolation. They interact through **relationships** that define how objects work together. The two most important relationships are:

| Relationship | Ownership | Lifecycle | Strength |
|-------------|-----------|-----------|----------|
| **Composition** | Strong | Dependent | Strong |
| **Aggregation** | Weak | Independent | Weak |

Understanding these relationships is crucial for:
- Building maintainable code
- Reducing coupling
- Designing scalable systems
- Following SOLID principles

---

## Composition

### Definition and Core Concepts

**Composition** is a design principle where a class is composed of one or more objects of other classes, creating a **strong ownership** relationship where the contained object cannot exist independently.

```python
# Basic composition example
class Engine:
    def __init__(self, horsepower):
        self.horsepower = horsepower
    
    def start(self):
        return f"Engine starting with {self.horsepower} HP"

class Car:
    def __init__(self, brand, model):
        self.brand = brand
        self.model = model
        # Engine is PART OF Car - strong ownership
        self.engine = Engine(200)
    
    def start(self):
        return f"{self.brand} {self.model}: {self.engine.start()}"

# Usage
my_car = Car("Toyota", "Camry")
print(my_car.start())
```

**Key characteristics:**

- **Has-A relationship**: Car *has an* Engine
- **Strong ownership**: Engine belongs to Car
- **Lifecycle dependency**: When Car is destroyed, Engine is destroyed
- **No independent existence**: Engine cannot exist without Car


### Why Composition Exists

Composition exists to address several fundamental problems in software design:

1. **Avoiding Inheritance Pitfalls**
    - Inheritance creates tight coupling
    - subclass depends on parent implementation
    - "The inheritance tree is rigid"
2. **Building Flexible Systems**
    - Objects can be combined dynamically
    - Easy to swap components
    - Follows "Favor composition over inheritance"
3. **Encapsulating Complexity**
    - Hide internal details
    - Expose only necessary interfaces
    - Create clear boundaries
4. **Modeling Real-World Relationships**
    - Some things are literally parts of others
    - Car *contains* Engine
    - House *contains* Rooms

### Has-A Relationship and Strong Ownership

The **Has-A relationship** is the hallmark of composition. When Class A has Class B through composition:

```python
class CPU:
    def __init__(self, cores):
        self.cores = cores
    
    def process(self):
        return f"Processing on {self.cores} cores"

class Computer:
    def __init__(self, brand):
        self.brand = brand
        # CPU is PART OF Computer
        self.cpu = CPU(8)  # Created inside Computer
    
    def run(self):
        return f"{self.brand}: {self.cpu.process()}"

# The CPU cannot exist without the Computer
computer = Computer("Dell")
# computer.cpu is owned by computer
```

**Strong Ownership Means:**

- The parent object **creates** the child object
- The parent object **owns** the child object
- The child object **cannot be shared** with other objects
- When parent is destroyed, child is **automatically destroyed**


### Composition Architecture

#### Object Containment

```python
class Room:
    def __init__(self, name, area):
        self.name = name
        self.area = area

class House:
    def __init__(self, address):
        self.address = address
        # Rooms are contained within House
        self.rooms = [
            Room("Living Room", 50),
            Room("Kitchen", 30),
            Room("Bedroom", 40)
        ]
    
    def total_area(self):
        return sum(room.area for room in self.rooms)

house = House("123 Main St")
print(f"Total area: {house.total_area()} sq ft")
```

**Containment principles:**

- Parent holds direct references to children
- Children are created within parent's scope
- Children are not accessible outside parent


#### Lifecycle Dependency

```python
class Battery:
    def __init__(self, capacity):
        self.capacity = capacity
        print(f"Battery {capacity}mAh created")
    
    def __del__(self):
        print(f"Battery {self.capacity}mAh destroyed")
    
    def power(self):
        return f"Providing {self.capacity}mAh power"

class Smartphone:
    def __init__(self, model):
        self.model = model
        # Battery lifecycle tied to Smartphone
        self.battery = Battery(4000)
        print(f"Smartphone {model} created with battery")
    
    def __del__(self):
        print(f"Smartphone {self.model} destroyed")
    
    def use(self):
        return f"{self.model}: {self.battery.power()}"

# Lifecycle test
print("=== Creating Smartphone ===")
phone = Smartphone("iPhone 15")
print(f"=== Using Phone: {phone.use()} ===")
print("=== Deleting Smartphone ===")
del phone
```

**Output:**

```
=== Creating Smartphone ===
Battery 4000mAh created
Smartphone iPhone 15 created with battery
=== Using Phone: iPhone 15: Providing 4000mAh power ===
=== Deleting Smartphone ===
Smartphone iPhone 15 destroyed
Battery 4000mAh destroyed
```


#### Object Creation

In composition, the parent object **creates** the child object:

```python
# WRONG: Child created outside (not composition)
engine = Engine(200)
car = Car("Toyota", "Camry", engine)  # Aggregation!

# CORRECT: Child created inside (composition)
class Car:
    def __init__(self, brand, model):
        self.brand = brand
        self.model = model
        self.engine = Engine(200)  # Created inside!
```


#### Object Destruction

```python
class Disk:
    def __init__(self, size):
        self.size = size
    
    def __del__(self):
        print(f"Disk {self.size}TB destroyed")

class Server:
    def __init__(self, name):
        self.name = name
        self.disk = Disk(2)  # Created inside
    
    def __del__(self):
        print(f"Server {self.name} destroyed")

server = Server("Production-1")
del server  # Disk automatically destroyed
```


#### Ownership Concepts

```python
class Wheel:
    def __init__(self, size):
        self.size = size

class Car:
    def __init__(self, brand):
        self.brand = brand
        # Car OWNS all 4 wheels
        self.wheels = [
            Wheel(18), Wheel(18), 
            Wheel(18), Wheel(18)
        ]
    
    def count_wheels(self):
        return len(self.wheels)

car = Car("BMW")
print(f"Car has {car.count_wheels()} wheels")
# Wheels cannot exist without car
```


### Composition Patterns

#### Component-Based Systems

```python
# Component-based architecture
class Renderer:
    def render(self, content):
        return f"Rendering: {content}"

class Database:
    def query(self, sql):
        return f"Query result: {sql}"

class Cache:
    def get(self, key):
        return f"Cache: {key}"

class WebApp:
    def __init__(self, name):
        self.name = name
        # Composed of components
        self.renderer = Renderer()
        self.database = Database()
        self.cache = Cache()
    
    def handle_request(self, sql):
        cached = self.cache.get(sql)
        result = self.database.query(sql)
        return self.renderer.render(result)

app = WebApp("MyShop")
print(app.handle_request("SELECT * FROM users"))
```


#### Service Composition

```python
# Microservices composition
class UserService:
    def get_user(self, user_id):
        return {"id": user_id, "name": "John"}

class OrderService:
    def get_orders(self, user_id):
        return [{"id": 1, "item": "Laptop"}]

class NotificationService:
    def send(self, user_id, message):
        return f"Sent to {user_id}: {message}"

class UserPortal:
    def __init__(self):
        # Composes multiple services
        self.user_service = UserService()
        self.order_service = OrderService()
        self.notification_service = NotificationService()
    
    def get_user_dashboard(self, user_id):
        user = self.user_service.get_user(user_id)
        orders = self.order_service.get_orders(user_id)
        self.notification_service.send(user_id, "Welcome!")
        return {"user": user, "orders": orders}

portal = UserPortal()
print(portal.get_user_dashboard(123))
```


#### Modular Design

```python
# Modular composition
class Logger:
    def log(self, message):
        print(f"[LOG] {message}")

class Authenticator:
    def authenticate(self, user, password):
        return user == "admin" and password == "secret"

class DatabaseConnector:
    def connect(self, url):
        return f"Connected to {url}"

class Application:
    def __init__(self, name):
        self.name = name
        # Modular components
        self.logger = Logger()
        self.authenticator = Authenticator()
        self.db = DatabaseConnector()
    
    def run(self, user, password, db_url):
        self.logger.log(f"Starting {self.name}")
        if self.authenticator.authenticate(user, password):
            self.logger.log("Authenticated")
            connection = self.db.connect(db_url)
            self.logger.log(connection)
            return True
        return False

app = Application("ERP System")
app.run("admin", "secret", "postgresql://localhost/erp")
```


#### Domain-Driven Design Examples

```python
# DDD: Order domain with composition
class Price:
    def __init__(self, amount, currency):
        self.amount = amount
        self.currency = currency
    
    def __str__(self):
        return f"{self.amount} {self.currency}"

class Quantity:
    def __init__(self, value):
        self.value = value
    
    def __str__(self):
        return f"{self.value} units"

class Product:
    def __init__(self, name, price):
        self.name = name
        self.price = price  # Price is PART OF Product

class OrderItem:
    def __init__(self, product, quantity):
        self.product = product
        self.quantity = quantity  # Quantity is PART OF OrderItem
    
    def total(self):
        return self.product.price.amount * self.quantity.value

class Order:
    def __init__(self, order_id):
        self.order_id = order_id
        self.items = []  # OrderItem is PART OF Order
    
    def add_item(self, product, quantity):
        item = OrderItem(product, quantity)
        self.items.append(item)
    
    def total_amount(self):
        return sum(item.total() for item in self.items)

# Usage
product = Product("Laptop", Price(1000, "USD"))
order = Order("ORD-001")
order.add_item(product, Quantity(2))
print(f"Order total: ${order.total_amount()}")
```


### Real World Examples

#### Car and Engine

```python
class Engine:
    def __init__(self, type, horsepower):
        self.type = type
        self.horsepower = horsepower
        self.is_running = False
    
    def start(self):
        self.is_running = True
        return f"{self.type} engine started ({self.horsepower} HP)"
    
    def stop(self):
        self.is_running = False
        return f"{self.type} engine stopped"

class Transmission:
    def __init__(self, gears):
        self.gears = gears
        self.current_gear = 1
    
    def shift(self, gear):
        self.current_gear = gear
        return f"Shifted to gear {gear}"

class Car:
    def __init__(self, brand, model, year):
        self.brand = brand
        self.model = model
        self.year = year
        # Engine and Transmission are PART OF Car
        self.engine = Engine("V8", 450)
        self.transmission = Transmission(6)
    
    def drive(self, gear):
        engine_status = self.engine.start()
        shift_status = self.transmission.shift(gear)
        return f"{self.brand} {self.model}: {engine_status}, {shift_status}"
    
    def park(self):
        self.engine.stop()
        self.transmission.shift(0)
        return f"{self.brand} {self.model} parked"

# Usage
car = Car("Ford", "Mustang", 2024)
print(car.drive(3))
print(car.park())
```


#### House and Rooms

```python
class Room:
    def __init__(self, name, length, width):
        self.name = name
        self.length = length
        self.width = width
    
    def area(self):
        return self.length * self.width
    
    def __str__(self):
        return f"{self.name} ({self.area()} sq ft)"

class House:
    def __init__(self, address, floors):
        self.address = address
        self.floors = floors
        # Rooms are PART OF House
        self.rooms = []
    
    def add_room(self, name, length, width):
        room = Room(name, length, width)
        self.rooms.append(room)
    
    def total_area(self):
        return sum(room.area() for room in self.rooms)
    
    def __str__(self):
        room_list = "\n  ".join(str(room) for room in self.rooms)
        return f"House at {self.address}\n  Floors: {self.floors}\n  Rooms:\n  {room_list}\n  Total: {self.total_area()} sq ft"

# Usage
house = House("123 Main St", 2)
house.add_room("Living Room", 20, 15)
house.add_room("Kitchen", 12, 10)
house.add_room("Bedroom", 15, 12)
print(house)
```


#### Computer and CPU

```python
class CPU:
    def __init__(self, cores, speed):
        self.cores = cores
        self.speed = speed  # GHz
        self.is_active = False
    
    def boot(self):
        self.is_active = True
        return f"CPU booted: {self.cores} cores, {self.speed} GHz"
    
    def process_task(self, task):
        if not self.is_active:
            return "CPU not active"
        return f"Processing '{task}' on {self.cores} cores"

class RAM:
    def __init__(self, capacity):
        self.capacity = capacity  # GB
        self.available = capacity
    
    def allocate(self, size):
        if size <= self.available:
            self.available -= size
            return f"Allocated {size}GB, {self.available}GB free"
        return "Insufficient RAM"

class Computer:
    def __init__(self, brand, model):
        self.brand = brand
        self.model = model
        # CPU and RAM are PART OF Computer
        self.cpu = CPU(8, 3.5)
        self.ram = RAM(32)
    
    def startup(self):
        cpu_status = self.cpu.boot()
        return f"{self.brand} {self.model}: {cpu_status}"
    
    def run_application(self, app_name, ram_needed):
        ram_status = self.ram.allocate(ram_needed)
        process_status = self.cpu.process_task(app_name)
        return f"{app_name}: {ram_status}, {process_status}"

# Usage
computer = Computer("Dell", "XPS 15")
print(computer.startup())
print(computer.run_application("Video Editor", 16))
```


#### Smartphone and Components

```python
class Battery:
    def __init__(self, capacity_mah):
        self.capacity_mah = capacity_mah
        self.current_mah = capacity_mah
    
    def percentage(self):
        return (self.current_mah / self.capacity_mah) * 100
    
    def consume(self, mah):
        self.current_mah -= mah
        return f"Consumed {mah}mAh, {self.percentage():.1f}% remaining"

class Camera:
    def __init__(self, megapixels):
        self.megapixels = megapixels
    
    def take_photo(self):
        return f"Photo captured: {self.megapixels}MP"

class Display:
    def __init__(self, resolution, inches):
        self.resolution = resolution
        self.inches = inches
        self.is_on = False
    
    def turn_on(self):
        self.is_on = True
        return f"Display on: {self.resolution} ({self.inches} inches)"
    
    def turn_off(self):
        self.is_on = False
        return "Display off"

class Smartphone:
    def __init__(self, model, brand):
        self.model = model
        self.brand = brand
        # Components are PART OF Smartphone
        self.battery = Battery(5000)
        self.camera = Camera(12)
        self.display = Display("1920x1080", 6.7)
    
    def power_on(self):
        display_status = self.display.turn_on()
        return f"{self.brand} {self.model}: {display_status}"
    
    def use_camera(self):
        photo = self.camera.take_photo()
        consumption = self.battery.consume(50)
        return f"{photo}, {consumption}"
    
    def power_off(self):
        display_status = self.display.turn_off()
        return f"{self.brand} {self.model}: {display_status}"

# Usage
phone = Smartphone("Galaxy S24", "Samsung")
print(phone.power_on())
print(phone.use_camera())
print(phone.power_off())
```


### Enterprise Examples

#### Microservices

```python
# Enterprise microservices composition
class AuthenticationService:
    def validate_token(self, token):
        return token == "valid_token_123"
    
    def get_user_id(self, token):
        return 1001 if token == "valid_token_123" else None

class ProductService:
    def get_product(self, product_id):
        return {
            "id": product_id,
            "name": "Laptop Pro",
            "price": 1299.99,
            "category": "Electronics"
        }
    
    def get_products_by_category(self, category):
        return [
            {"id": 1, "name": "Laptop Pro", "price": 1299.99},
            {"id": 2, "name": "Phone X", "price": 899.99}
        ]

class OrderService:
    def create_order(self, user_id, product_id, quantity):
        return {
            "order_id": f"ORD-{user_id}-{product_id}",
            "user_id": user_id,
            "product_id": product_id,
            "quantity": quantity,
            "status": "created"
        }

class PaymentService:
    def process_payment(self, order_id, amount):
        return {
            "transaction_id": f"TXN-{order_id}",
            "order_id": order_id,
            "amount": amount,
            "status": "success"
        }

class ECommercePlatform:
    def __init__(self):
        # Composes multiple microservices
        self.auth_service = AuthenticationService()
        self.product_service = ProductService()
        self.order_service = OrderService()
        self.payment_service = PaymentService()
    
    def purchase_product(self, token, product_id, quantity):
        # Validate authentication
        if not self.auth_service.validate_token(token):
            return {"error": "Invalid token"}
        
        user_id = self.auth_service.get_user_id(token)
        
        # Get product details
        product = self.product_service.get_product(product_id)
        
        # Create order
        order = self.order_service.create_order(user_id, product_id, quantity)
        
        # Process payment
        payment = self.payment_service.process_payment(
            order["order_id"], 
            product["price"] * quantity
        )
        
        return {
            "order": order,
            "payment": payment,
            "product": product
        }

# Usage
platform = ECommercePlatform()
result = platform.purchase_product("valid_token_123", 1, 2)
print(result)
```


#### Django Applications

```python
# Django-style application composition
from datetime import datetime

class DatabaseManager:
    def __init__(self, connection_string):
        self.connection_string = connection_string
        self.connected = False
    
    def connect(self):
        self.connected = True
        return f"Connected to {self.connection_string}"
    
    def query(self, sql):
        if not self.connected:
            raise Exception("Not connected")
        return f"Query result: {sql}"

class CacheManager:
    def __init__(self, backend):
        self.backend = backend
        self.cache = {}
    
    def set(self, key, value):
        self.cache[key] = value
        return f"Set {key} in {self.backend}"
    
    def get(self, key):
        return self.cache.get(key)

class Logger:
    def __init__(self, level):
        self.level = level
    
    def log(self, message, level=None):
        if level is None or level == self.level:
            print(f"[{self.level}] {message}")

class UserManager:
    def __init__(self, db, cache, logger):
        self.db = db
        self.cache = cache
        self.logger = logger
    
    def create_user(self, username, email):
        self.logger.log(f"Creating user: {username}")
        user_id = f"user_{username}"
        self.cache.set(user_id, {"username": username, "email": email})
        return {"id": user_id, "username": username}

class Application:
    def __init__(self, name):
        self.name = name
        # Composed infrastructure components
        self.db = DatabaseManager("postgresql://localhost/app")
        self.cache = CacheManager("redis://localhost")
        self.logger = Logger("INFO")
        
        # Composed business logic
        self.user_manager = UserManager(self.db, self.cache, self.logger)
    
    def startup(self):
        db_status = self.db.connect()
        cache_status = self.cache.set("startup", datetime.now())
        self.logger.log(f"{self.name} started")
        return {"db": db_status, "cache": cache_status}
    
    def register_user(self, username, email):
        return self.user_manager.create_user(username, email)

# Usage
app = Application("MyDjangoApp")
app.startup()
user = app.register_user("john_doe", "john@example.com")
print(user)
```


#### Game Engines

```python
# Game engine composition
class Renderer:
    def __init__(self, resolution):
        self.resolution = resolution
    
    def draw(self, object_type, position):
        return f"Drawing {object_type} at {position} ({self.resolution})"

class PhysicsEngine:
    def __init__(self, gravity):
        self.gravity = gravity
    
    def calculate_collision(self, obj1, obj2):
        return f"Collision: {obj1} vs {obj2}"
    
    def apply_gravity(self, position):
        return (position, position - self.gravity)

class AudioManager:
    def __init__(self, volume):
        self.volume = volume
    
    def play_sound(self, sound_name):
        return f"Playing {sound_name} at {self.volume}% volume"

class InputManager:
    def __init__(self):
        self.keys = {}
    
    def get_key(self, key_name):
        return self.keys.get(key_name, False)
    
    def set_key(self, key_name, pressed):
        self.keys[key_name] = pressed

class GameObject:
    def __init__(self, name, position):
        self.name = name
        self.position = position

class GameEngine:
    def __init__(self, game_name):
        self.game_name = game_name
        # Composed subsystems
        self.renderer = Renderer("1920x1080")
        self.physics = PhysicsEngine(9.8)
        self.audio = AudioManager(75)
        self.input = InputManager()
        
        # Game objects
        self.objects = []
    
    def start(self):
        self.audio.play_sound("startup_music")
        return f"{self.game_name} started"
    
    def add_object(self, name, position):
        obj = GameObject(name, position)
        self.objects.append(obj)
        return self.renderer.draw(name, position)
    
    def update(self):
        for obj in self.objects:
            obj.position = self.physics.apply_gravity(obj.position)
            self.renderer.draw(obj.name, obj.position)
        return "Game updated"
    
    def handle_input(self, key, pressed):
        self.input.set_key(key, pressed)
        if pressed:
            return self.audio.play_sound("key_press")
        return None

# Usage
engine = GameEngine("Super Adventure")
print(engine.start())
print(engine.add_object("Player", (100, 500)))
print(engine.add_object("Enemy", (200, 400)))
print(engine.update())
print(engine.handle_input("SPACE", True))
```


#### ERP Systems

```python
# Enterprise ERP composition
class InventoryModule:
    def __init__(self):
        self.products = {}
    
    def add_product(self, product_id, name, quantity):
        self.products[product_id] = {"name": name, "quantity": quantity}
    
    def get_quantity(self, product_id):
        return self.products.get(product_id, {}).get("quantity", 0)

class FinanceModule:
    def __init__(self):
        self.transactions = []
    
    def record_transaction(self, type, amount, description):
        transaction = {
            "type": type,
            "amount": amount,
            "description": description
        }
        self.transactions.append(transaction)
        return transaction

class HRModule:
    def __init__(self):
        self.employees = []
    
    def add_employee(self, emp_id, name, department):
        employee = {"id": emp_id, "name": name, "department": department}
        self.employees.append(employee)
        return employee

class ProcurementModule:
    def __init__(self):
        self.orders = []
    
    def create_order(self, vendor, products):
        order = {"vendor": vendor, "products": products, "status": "pending"}
        self.orders.append(order)
        return order

class ERPSystem:
    def __init__(self, company_name):
        self.company_name = company_name
        # Composed ERP modules
        self.inventory = InventoryModule()
        self.finance = FinanceModule()
        self.hr = HRModule()
        self.procurement = ProcurementModule()
    
    def onboard_employee(self, emp_id, name, department):
        employee = self.hr.add_employee(emp_id, name, department)
        self.finance.record_transaction("expense", 50000, f"Onboarding {name}")
        return employee
    
    def stock_product(self, product_id, name, quantity):
        self.inventory.add_product(product_id, name, quantity)
        self.finance.record_transaction("asset", quantity * 10, f"Stock {name}")
        return {"product_id": product_id, "quantity": quantity}
    
    def order_from_vendor(self, vendor, products):
        order = self.procurement.create_order(vendor, products)
        self.finance.record_transaction("expense", 10000, f"Order from {vendor}")
        return order

# Usage
erp = ERPSystem("TechCorp Inc")
print(erp.onboard_employee("E001", "Alice Johnson", "Engineering"))
print(erp.stock_product("P001", "Laptop", 50))
print(erp.order_from_vendor("VendorABC", [{"item": "Mouse", "qty": 100}]))
```


#### SaaS Products

```python
# SaaS platform composition
class UserManagement:
    def __init__(self):
        self.users = {}
    
    def create_user(self, user_id, email, role):
        self.users[user_id] = {"email": email, "role": role}
        return {"user_id": user_id, "email": email, "role": role}

class SubscriptionManager:
    def __init__(self):
        self.subscriptions = {}
    
    def create_subscription(self, user_id, plan, price):
        subscription = {
            "user_id": user_id,
            "plan": plan,
            "price": price,
            "status": "active"
        }
        self.subscriptions[user_id] = subscription
        return subscription

class BillingEngine:
    def __init__(self):
        self.bills = []
    
    def generate_bill(self, user_id, amount):
        bill = {
            "user_id": user_id,
            "amount": amount,
            "date": "2024-01-15",
            "status": "pending"
        }
        self.bills.append(bill)
        return bill

class AnalyticsService:
    def __init__(self):
        self.metrics = {}
    
    def track_event(self, user_id, event_type):
        if user_id not in self.metrics:
            self.metrics[user_id] = []
        self.metrics[user_id].append(event_type)
        return f"Tracked {event_type} for {user_id}"

class SaaSPlatform:
    def __init__(self, platform_name):
        self.platform_name = platform_name
        # Composed SaaS components
        self.user_management = UserManagement()
        self.subscription = SubscriptionManager()
        self.billing = BillingEngine()
        self.analytics = AnalyticsService()
    
    def register_customer(self, user_id, email, plan, price):
        user = self.user_management.create_user(user_id, email, "customer")
        subscription = self.subscription.create_subscription(user_id, plan, price)
        bill = self.billing.generate_bill(user_id, price)
        event = self.analytics.track_event(user_id, "registration")
        
        return {
            "user": user,
            "subscription": subscription,
            "bill": bill,
            "analytics": event
        }
    
    def process_usage(self, user_id, event_type):
        return self.analytics.track_event(user_id, event_type)

# Usage
saas = SaaSPlatform("CloudApp Pro")
customer = saas.register_customer("C001", "customer@example.com", "pro", 99.99)
print(customer)
print(saas.process_usage("C001", "login"))
```


### Memory Relationship Explanations

#### Object References

```python
# Memory diagram conceptualization
class Engine:
    def __init__(self, horsepower):
        self.horsepower = horsepower

class Car:
    def __init__(self, brand):
        self.brand = brand
        self.engine = Engine(200)  # Engine object created here

# Memory representation:
# Car object (address: 0x1000)
#   ├─ brand: "Toyota"
#   └─ engine: 0x2000 (Engine object)
#                ├─ horsepower: 200
# When Car is deleted, Engine at 0x2000 is also deleted

car = Car("Toyota")
print(f"Car at: {id(car)}")
print(f"Engine at: {id(car.engine)}")
```


#### Memory Diagrams

```
┌─────────────────┐
│    Car Object   │ (address: 0x1000)
│  ┌───────────┐  │
│  │ brand     │  │ → "Toyota"
│  │ engine    │  │ → ┌─────────────┐
│  └───────────┘  │   │ Engine Object │ (address: 0x2000)
└─────────────────┘   │ ┌─────────┐ │
                      │ │ HP: 200 │ │
                      │ └─────────┘ │
                      └─────────────┘

STRONG OWNERSHIP: Car OWNS Engine
LIFECYCLE: When Car dies, Engine dies
```


#### Lifecycle Diagrams

```
Time: 0          1          2          3
     │          │          │          │
Car  │───○──────┼──────────┼──────────○───
     │  Create  │   Use    │  Delete  │
     │          │          │          │
Engine│    ───○──┼──────────┼──────────○──
     │      Create        Delete       │
     │          │          │          │
     └──────────┴──────────┴──────────┘
     
     Engine created WHEN Car created
     Engine destroyed WHEN Car destroyed
```


### Advantages

| Advantage | Explanation | Example |
| :-- | :-- | :-- |
| **Strong Encapsulation** | Internal components hidden | Car hides Engine details |
| **Clear Ownership** | Explicit who owns what | Car owns Engine |
| **Lifecycle Management** | Automatic cleanup | Engine auto-deleted with Car |
| **Reduced Coupling** | Components know only parent | Engine knows only Car |
| **Easier Testing** | Test whole unit together | Test Car + Engine together |
| **Memory Safety** | No dangling references | Engine never exists alone |
| **Domain Accuracy** | Matches real world | House contains Rooms |

### Disadvantages

| Disadvantage | Explanation | When to Avoid |
| :-- | :-- | :-- |
| **Less Flexibility** | Cannot swap components easily | When components need sharing |
| **Tighter Coupling** | Parent-child tightly linked | When independence needed |
| **Memory Overhead** | All children created always | When lazy loading preferred |
| **Cannot Share** | Child cannot be shared | When multiple parents need child |
| **Complex Creation** | Parent must create all children | When children created dynamically |

### Best Practices

1. **Create Children Inside Parent**
```python
# GOOD
class Car:
    def __init__(self):
        self.engine = Engine()  # Created inside

# BAD
engine = Engine()
car = Car(engine)  # Aggregation, not composition
```

2. **Don't Expose Children Directly**
```python
# GOOD
class Car:
    def start_engine(self):
        return self.engine.start()

# BAD
class Car:
    @property
    def engine(self):
        return self.engine  # Exposes internal
```

3. **Use Composition for Required Parts**
```python
# GOOD - Engine is required for Car
class Car:
    def __init__(self):
        self.engine = Engine()

# BAD - Sab should not be composition
class Car:
    def __init__(self):
        self.radio = Radio()  # Car can exist without radio
```

4. **Keep Children Private**
```python
class Car:
    def __init__(self):
        self._engine = Engine()  # Private
    
    def start(self):
        return self._engine.start()
```


### Anti-Patterns

1. **Composition When Aggregation Needed**
```python
# WRONG - Database should be shared
class Application:
    def __init__(self):
        self.db = Database()  # Each app gets new DB

# CORRECT - Share database
db = Database()
app1 = Application(db)
app2 = Application(db)
```

2. **Over-Composition**
```python
# WRONG - Too much composition
class Car:
    def __init__(self):
        self.engine = Engine()
        self.wheel1 = Wheel()
        self.wheel2 = Wheel()
        self.wheel3 = Wheel()
        self.wheel4 = Wheel()
        self.battery = Battery()
        self.transmission = Transmission()

# BETTER - Group related components
class Car:
    def __init__(self):
        self.engine_system = EngineSystem()
        self.wheel_system = WheelSystem()
```

3. **Creating Children in Methods**
```python
# WRONG
class Car:
    def get_engine(self):
        return Engine()  # New engine each time

# CORRECT
class Car:
    def __init__(self):
        self.engine = Engine()  # Created once
```


### Common Mistakes

1. **Confusing Composition with Aggregation**
```python
# Mistake: Treating aggregation as composition
class Department:
    def __init__(self):
        self.employees = []  # Employees can exist without department

# This is aggregation, not composition
```

2. **Not Initializing Children**
```python
# Mistake
class Car:
    def __init__(self):
        self.engine = None  # Not initialized

# Fix
class Car:
    def __init__(self):
        self.engine = Engine()  # Always initialized
```

3. **Exposing Children for Modification**
```python
# Mistake
class Car:
    def __init__(self):
        self.engine = Engine()

car = Car()
car.engine = Engine(500)  # Can replace engine

# Fix - make immutable
class Car:
    def __init__(self):
        self._engine = Engine()
    
    @property
    def engine(self):
        return self._engine  # Read-only
```


### Interview Questions

1. **What is composition? Give a real-world example.**
```python
# Answer: Composition is strong ownership where child cannot exist without parent.
# Example: Car and Engine - Engine is PART OF Car
```

2. **How does composition differ from aggregation?**
```python
# Answer: 
# - Composition: Strong ownership, dependent lifecycle
# - Aggregation: Weak ownership, independent lifecycle
```

3. **When would you use composition over inheritance?**
```python
# Answer: When you need flexibility, want to avoid tight coupling,
# or when relationship is "has-a" not "is-a"
```

4. **Show code example of composition.**
```python
class House:
    def __init__(self):
        self.rooms = [Room("Living"), Room("Kitchen")]
```

5. **What happens to child object when parent is deleted in composition?**
```python
# Answer: Child is automatically deleted (garbage collected)
```


### Exercises

1. **Create a Library class with composition**
```python
# Required: Book objects must be part of Library
# Library creates Books, Books cannot exist without Library
class Library:
    pass  # Implement this

# Test
lib = Library()
lib.add_book("Harry Potter", "JK Rowling")
```

2. **Create a Team class with Player composition**
```python
# Players are PART OF Team
# Team creates Players
class Team:
    pass  # Implement this
```

3. **Implement Restaurant with Menu composition**
```python
# Menu items are PART OF Restaurant
class Restaurant:
    pass  # Implement this
```


---

## Aggregation

### Definition and Core Concepts

**Aggregation** is a relationship where a class contains objects of another class, but with **weak ownership** - the contained objects can exist independently and be shared across multiple objects.

```python
# Basic aggregation example
class Employee:
    def __init__(self, name, emp_id):
        self.name = name
        self.emp_id = emp_id
    
    def work(self):
        return f"{self.name} is working"

class Department:
    def __init__(self, name):
        self.name = name
        # Employees are NOT created here - passed in
        self.employees = []  # Weak ownership
    
    def add_employee(self, employee):
        self.employees.append(employee)
    
    def get_employees(self):
        return [e.name for e in self.employees]

# Employees exist independently
emp1 = Employee("John", 1)
emp2 = Employee("Jane", 2)

# Added to department (aggregation)
dept = Department("Engineering")
dept.add_employee(emp1)
dept.add_employee(emp2)

# Employee can exist without department
print(emp1.work())  # Still works!
```

**Key characteristics:**

- **Has-A relationship**: Department *has Employees*
- **Weak ownership**: Employees belong to themselves
- **Independent lifecycle**: Employee exists without Department
- **Shared objects**: Employee can be in multiple Departments


### Why Aggregation Exists

Aggregation exists to solve problems where:

1. **Objects Need to Be Shared**

```python
# Teacher can teach multiple classes
teacher = Teacher("John")
class1 = Class("Math")
class2 = Class("Physics")
class1.add_teacher(teacher)
class2.add_teacher(teacher)  # Shared!
```

2. **Independent Lifecycle Required**

```python
# Employee exists before and after Department
emp = Employee("John")
dept = Department("Engineering")
dept.add_employee(emp)
del dept  # Employee still exists!
```

3. **Dynamic Relationships**

```python
# Players can move between teams
player = Player("Messi")
team1 = Team("Barcelona")
team2 = Team("PSG")
team1.add_player(player)
team2.add_player(player)  # Moved!
```

4. **Avoiding Tight Coupling**

```python
# Database can be shared across applications
db = Database()
app1 = Application(db)
app2 = Application(db)  # Shared resource
```


### Weak Ownership and Independent Lifecycle

**Weak Ownership** means the container does not own the object:

```python
class Book:
    def __init__(self, title, author):
        self.title = title
        self.author = author
    
    def read(self):
        return f"Reading {self.title}"

class Library:
    def __init__(self, name):
        self.name = name
        self.books = []  # Does NOT own books
    
    def add_book(self, book):
        self.books.append(book)  # Book added, not created

# Book exists independently
book1 = Book("Harry Potter", "JK Rowling")
book2 = Book("1984", "George Orwell")

# Added to library
lib = Library("City Library")
lib.add_book(book1)
lib.add_book(book2)

# Book can exist without library
print(book1.read())  # Works!

# Book can be in multiple libraries
lib2 = Library("University Library")
lib2.add_book(book1)  # Same book in two libraries!
```

**Independent Lifecycle:**

```python
class Teacher:
    def __init__(self, name):
        self.name = name
        print(f"Teacher {name} created")
    
    def __del__(self):
        print(f"Teacher {self.name} destroyed")
    
    def teach(self):
        return f"{self.name} is teaching"

class Classroom:
    def __init__(self, subject):
        self.subject = subject
        self.teacher = None  # Teacher assigned later
    
    def assign_teacher(self, teacher):
        self.teacher = teacher
        print(f"{teacher.name} assigned to {self.subject}")

# Independent lifecycle
print("=== Creating Teacher ===")
teacher = Teacher("John Smith")
print("=== Creating Classroom ===")
classroom = Classroom("Mathematics")
print("=== Assigning Teacher ===")
classroom.assign_teacher(teacher)
print("=== Deleting Classroom ===")
del classroom  # Teacher still exists!
print("=== Teacher still works ===")
print(teacher.teach())  # Teacher still works!
print("=== Deleting Teacher ===")
del teacher
```


### Aggregation Architecture

#### Object Association

```python
# Object association without containment
class Customer:
    def __init__(self, name):
        self.name = name

class Order:
    def __init__(self, order_id):
        self.order_id = order_id
        self.customer = None  # Associated, not owned
    
    def set_customer(self, customer):
        self.customer = customer

# Association
customer = Customer("John")
order = Order("ORD-001")
order.set_customer(customer)  # Associated
```


#### Shared References

```python
# Shared database connection
class Database:
    def __init__(self, url):
        self.url = url
    
    def query(self, sql):
        return f"Query: {sql}"

class UserService:
    def __init__(self, db):
        self.db = db  # Shared reference
    
    def get_user(self, id):
        return self.db.query(f"SELECT * FROM users WHERE id={id}")

class OrderService:
    def __init__(self, db):
        self.db = db  # Same shared reference
    
    def get_order(self, id):
        return self.db.query(f"SELECT * FROM orders WHERE id={id}")

# Shared database
db = Database("postgresql://localhost/app")
user_service = UserService(db)
order_service = OrderService(db)  # Same db!
```


#### Object Independence

```python
class Vehicle:
    def __init__(self, brand, model):
        self.brand = brand
        self.model = model
    
    def drive(self):
        return f"Driving {self.brand} {self.model}"

class Driver:
    def __init__(self, name):
        self.name = name
        self.vehicle = None  # Vehicle assigned, not owned
    
    def assign_vehicle(self, vehicle):
        self.vehicle = vehicle
    
    def drive(self):
        if self.vehicle:
            return f"{self.name}: {self.vehicle.drive()}"
        return f"{self.name}: No vehicle"

# Vehicle exists independently
vehicle = Vehicle("Toyota", "Camry")

# Driver can be assigned vehicle
driver = Driver("John")
driver.assign_vehicle(vehicle)
print(driver.drive())

# Vehicle can be reassigned
driver2 = Driver("Jane")
driver2.assign_vehicle(vehicle)  # Same vehicle!
print(driver2.drive())
```


#### Memory Relationships

```
┌─────────────────┐       ┌─────────────────┐
│  Department     │       │   Employee      │
│  ┌───────────┐  │       │  ┌───────────┐  │
│  │ name      │  │       │  │ name      │  │
│  │ "Eng"     │  │       │  │ "John"    │  │
│  └───────────┘  │       │  └───────────┘  │
│  ┌───────────┐  │       │  ┌───────────┐  │
│  │ employees │  │───┬───│  │ emp_id    │  │
│  │ ┌─────┐   │  │   │   │  │ 123     │  │
│  │ │ ○───┼───┼───┘   │   │  └─────────┘  │
│  │ └─────┘   │       │                   │
│  └───────────┘       └─────────────────┘
└─────────────────┘       ↑
                          │
                    Employee exists
                    independently
```


### Aggregation Examples

#### Department and Employee

```python
class Employee:
    def __init__(self, name, emp_id, skill):
        self.name = name
        self.emp_id = emp_id
        self.skill = skill
        self.current_department = None
    
    def work(self):
        dept = self.current_department.name if self.current_department else "No department"
        return f"{self.name} ({self.skill}) working in {dept}"

class Department:
    def __init__(self, name):
        self.name = name
        self.employees = []  # Employees added, not created
    
    def add_employee(self, employee):
        self.employees.append(employee)
        employee.current_department = self
    
    def remove_employee(self, employee):
        self.employees.remove(employee)
        employee.current_department = None
    
    def get_employee_count(self):
        return len(self.employees)

# Usage
emp1 = Employee("John", 1, "Python")
emp2 = Employee("Jane", 2, "Java")
emp3 = Employee("Bob", 3, "SQL")

dept1 = Department("Engineering")
dept2 = Department("Marketing")

# Employee can be in multiple departments (over time)
dept1.add_employee(emp1)
dept1.add_employee(emp2)
dept2.add_employee(emp3)

print(f"Engineering: {dept1.get_employee_count()} employees")
print(f"Marketing: {dept2.get_employee_count()} employees")

print(emp1.work())  # Employee works independently

# Transfer employee
dept1.remove_employee(emp2)
dept2.add_employee(emp2)  # Same employee, different department
print(emp2.work())
```


#### School and Teacher

```python
class Teacher:
    def __init__(self, name, subject):
        self.name = name
        self.subject = subject
        self.schools = []  # Can teach at multiple schools
    
    def add_school(self, school):
        self.schools.append(school)
    
    def teach(self):
        return f"{self.name} teaching {self.subject}"

class School:
    def __init__(self, name):
        self.name = name
        self.teachers = []  # Teachers added, not created
    
    def add_teacher(self, teacher):
        self.teachers.append(teacher)
        teacher.add_school(self)
    
    def get_teacher_count(self):
        return len(self.teachers)

# Usage
teacher1 = Teacher("John", "Mathematics")
teacher2 = Teacher("Jane", "Physics")
teacher3 = Teacher("Bob", "Chemistry")

school1 = School("High School A")
school2 = School("High School B")

# Teacher can teach at multiple schools
school1.add_teacher(teacher1)
school1.add_teacher(teacher2)
school2.add_teacher(teacher2)  # Same teacher!
school2.add_teacher(teacher3)

print(f"School A: {school1.get_teacher_count()} teachers")
print(f"School B: {school2.get_teacher_count()} teachers")

print(teacher2.teach())  # Teacher works independently
```


#### Team and Player

```python
class Player:
    def __init__(self, name, position):
        self.name = name
        self.position = position
        self.current_team = None
        self.history = []  # Team history
    
    def join_team(self, team):
        self.history.append(self.current_team)
        self.current_team = team
        team.add_player(self)
    
    def play(self):
        team = self.current_team.name if self.current_team else "No team"
        return f"{self.name} ({self.position}) playing for {team}"

class Team:
    def __init__(self, name):
        self.name = name
        self.players = []  # Players added, not created
    
    def add_player(self, player):
        self.players.append(player)
    
    def get_player_count(self):
        return len(self.players)
    
    def get_team(self):
        return self.name

# Usage
player1 = Player("Messi", "Forward")
player2 = Player("Ronaldo", "Forward")
player3 = Player("Neymar", "Midfielder")

team1 = Team("Barcelona")
team2 = Team("PSG")

# Players join teams
player1.join_team(team1)
player2.join_team(team1)
player3.join_team(team2)

print(f"Barcelona: {team1.get_player_count()} players")
print(f"PSG: {team2.get_player_count()} players")

print(player1.play())

# Player transfers
player1.join_team(team2)  # Messi transfers to PSG
print(player1.play())
print(f"Barcelona: {team1.get_player_count()} players")
print(f"PSG: {team2.get_player_count()} players")
```


#### Library and Book

```python
class Book:
    def __init__(self, title, author, isbn):
        self.title = title
        self.author = author
        self.isbn = isbn
        self.libraries = []  # Can be in multiple libraries
    
    def add_library(self, library):
        self.libraries.append(library)
    
    def read(self):
        return f"Reading '{self.title}' by {self.author}"

class Library:
    def __init__(self, name, location):
        self.name = name
        self.location = location
        self.books = []  # Books added, not created
    
    def add_book(self, book):
        self.books.append(book)
        book.add_library(self)
    
    def search_book(self, title):
        return [b for b in self.books if b.title.lower() == title.lower()]
    
    def get_book_count(self):
        return len(self.books)

# Usage
book1 = Book("Harry Potter", "JK Rowling", "123456")
book2 = Book("1984", "George Orwell", "789012")
book3 = Book("To Kill a Mockingbird", "Harper Lee", "345678")

library1 = Library("City Library", "Downtown")
library2 = Library("University Library", "Campus")

# Book can be in multiple libraries
library1.add_book(book1)
library1.add_book(book2)
library2.add_book(book1)  # Same book!
library2.add_book(book3)

print(f"City Library: {library1.get_book_count()} books")
print(f"University Library: {library2.get_book_count()} books")

print(book1.read())  # Book exists independently
```


### Enterprise Examples

#### CRM Systems

```python
# CRM aggregation example
class Customer:
    def __init__(self, customer_id, name):
        self.customer_id = customer_id
        self.name = name
        self.agencies = []  # Can be in multiple agencies
    
    def add_agency(self, agency):
        self.agencies.append(agency)

class SalesAgent:
    def __init__(self, agent_id, name):
        self.agent_id = agent_id
        self.name = name
        self.customers = []  # Customers assigned, not owned
    
    def assign_customer(self, customer):
        self.customers.append(customer)
        customer.add_agency(self)

class CRMSystem:
    def __init__(self):
        self.agents = []
        self.customers = []
    
    def add_agent(self, agent):
        self.agents.append(agent)
    
    def add_customer(self, customer):
        self.customers.append(customer)
    
    def assign_agent_to_customer(self, agent, customer):
        agent.assign_customer(customer)

# Usage
crm = CRMSystem()

customer1 = Customer("C001", "John Corp")
customer2 = Customer("C002", "Jane Inc")

agent1 = SalesAgent("A001", "Agent Alice")
agent2 = SalesAgent("A002", "Agent Bob")

crm.add_customer(customer1)
crm.add_customer(customer2)
crm.add_agent(agent1)
crm.add_agent(agent2)

# Agent can handle multiple customers
crm.assign_agent_to_customer(agent1, customer1)
crm.assign_agent_to_customer(agent1, customer2)

# Customer can have multiple agents
crm.assign_agent_to_customer(agent2, customer1)

print(f"Agent Alice handles {len(agent1.customers)} customers")
print(f"Customer John Corp has {len(customer1.agencies)} agents")
```


#### ERP Systems

```python
# ERP aggregation - Supplier and Product
class Supplier:
    def __init__(self, supplier_id, name):
        self.supplier_id = supplier_id
        self.name = name
        self.products = []  # Products supplied, not owned
    
    def supply_product(self, product):
        self.products.append(product)

class Product:
    def __init__(self, product_id, name, price):
        self.product_id = product_id
        self.name = name
        self.price = price
        self.suppliers = []  # Can be supplied by multiple suppliers
    
    def add_supplier(self, supplier):
        self.suppliers.append(supplier)
        supplier.supply_product(self)

class ERPSystem:
    def __init__(self):
        self.suppliers = []
        self.products = []
    
    def add_supplier(self, supplier):
        self.suppliers.append(supplier)
    
    def add_product(self, product):
        self.products.append(product)
    
    def link_supplier_product(self, supplier, product):
        product.add_supplier(supplier)

# Usage
erp = ERPSystem()

supplier1 = Supplier("S001", "Tech Supplies")
supplier2 = Supplier("S002", "Global Parts")

product1 = Product("P001", "Laptop", 1299.99)
product2 = Product("P002", "Mouse", 29.99)

erp.add_supplier(supplier1)
erp.add_supplier(supplier2)
erp.add_product(product1)
erp.add_product(product2)

# Product can be supplied by multiple suppliers
erp.link_supplier_product(supplier1, product1)
erp.link_supplier_product(supplier2, product1)  # Same product!

erp.link_supplier_product(supplier1, product2)

print(f"Laptop has {len(product1.suppliers)} suppliers")
print(f"Tech Supplies supplies {len(supplier1.products)} products")
```


#### Healthcare Software

```python
# Healthcare - Doctor and Patient aggregation
class Patient:
    def __init__(self, patient_id, name):
        self.patient_id = patient_id
        self.name = name
        self.doctors = []  # Can have multiple doctors
    
    def add_doctor(self, doctor):
        self.doctors.append(doctor)

class Doctor:
    def __init__(self, doctor_id, name, specialty):
        self.doctor_id = doctor_id
        self.name = name
        self.specialty = specialty
        self.patients = []  # Patients assigned, not owned
    
    def add_patient(self, patient):
        self.patients.append(patient)
        patient.add_doctor(self)

class Hospital:
    def __init__(self, name):
        self.name = name
        self.doctors = []
        self.patients = []
    
    def register_doctor(self, doctor):
        self.doctors.append(doctor)
    
    def register_patient(self, patient):
        self.patients.append(patient)
    
    def assign_doctor_to_patient(self, doctor, patient):
        doctor.add_patient(patient)

# Usage
hospital = Hospital("St. Mary's")

patient1 = Patient("P001", "John Doe")
patient2 = Patient("P002", "Jane Smith")

doctor1 = Doctor("D001", "Dr. Alice", "Cardiology")
doctor2 = Doctor("D002", "Dr. Bob", "General")

hospital.register_patient(patient1)
hospital.register_patient(patient2)
hospital.register_doctor(doctor1)
hospital.register_doctor(doctor2)

# Patient can have multiple doctors
hospital.assign_doctor_to_patient(doctor1, patient1)
hospital.assign_doctor_to_patient(doctor2, patient1)  # Same patient!

hospital.assign_doctor_to_patient(doctor2, patient2)

print(f"John Doe has {len(patient1.doctors)} doctors")
print(f"Dr. Alice has {len(doctor1.patients)} patients")
```


#### Educational Platforms

```python
# Educational platform - Course and Student aggregation
class Student:
    def __init__(self, student_id, name):
        self.student_id = student_id
        self.name = name
        self.courses = []  # Can enroll in multiple courses
    
    def enroll(self, course):
        self.courses.append(course)
        course.add_student(self)

class Course:
    def __init__(self, course_id, title, instructor):
        self.course_id = course_id
        self.title = title
        self.instructor = instructor
        self.students = []  # Students enrolled, not owned
    
    def add_student(self, student):
        self.students.append(student)

class EducationalPlatform:
    def __init__(self):
        self.courses = []
        self.students = []
    
    def add_course(self, course):
        self.courses.append(course)
    
    def add_student(self, student):
        self.students.append(student)
    
    def enroll_student(self, student, course):
        student.enroll(course)

# Usage
platform = EducationalPlatform()

student1 = Student("S001", "Alice")
student2 = Student("S002", "Bob")

course1 = Course("C001", "Python 101", "Dr. Smith")
course2 = Course("C002", "Java 101", "Dr. Jones")
course3 = Course("C003", "Data Science", "Dr. Brown")

platform.add_student(student1)
platform.add_student(student2)
platform.add_course(course1)
platform.add_course(course2)
platform.add_course(course3)

# Student can enroll in multiple courses
platform.enroll_student(student1, course1)
platform.enroll_student(student1, course2)
platform.enroll_student(student1, course3)  # Same student!

platform.enroll_student(student2, course1)
platform.enroll_student(student2, course3)

print(f"Alice enrolled in {len(student1.courses)} courses")
print(f"Python 101 has {len(course1.students)} students")
```


### Memory Relationship Explanations

#### Object References

```python
# Memory representation of aggregation
class Employee:
    def __init__(self, name):
        self.name = name

class Department:
    def __init__(self, name):
        self.name = name
        self.employees = []

# Create independent employee
emp = Employee("John")
print(f"Employee at: {id(emp)}")

# Add to department
dept = Department("Engineering")
dept.employees.append(emp)

# Department holds reference, but doesn't own
print(f"Department at: {id(dept)}")
print(f"Employee in dept at: {id(dept.employees)}")

# Employee exists independently
del dept
print(f"Employee still exists at: {id(emp)}")  # Still valid!
```


#### Shared Ownership

```
┌─────────────────┐       ┌─────────────────┐
│  Department A   │       │   Employee      │
│  ┌───────────┐  │       │  ┌───────────┐  │
│  │ name      │  │       │  │ name      │  │
│  │ "Eng"     │  │       │  │ "John"    │  │
│  └───────────┘  │       │  └───────────┘  │
│  ┌───────────┐  │       │  ┌───────────┐  │
│  │ employees │  │───┬───│  │ emp_id    │  │
│  │ ┌─────┐   │  │   │   │  │ 123     │  │
│  │ │ ○───┼───┼───┘   │   │  └─────────┘  │
│  └───────────┘       │   └─────────────────┘
└─────────────────┘       │
                          │
┌─────────────────┐       │
│  Department B   │       │
│  ┌───────────┐  │       │
│  │ name      │  │       │
│  │ "Marketing"│ │       │
│  └───────────┘  │       │
│  ┌───────────┐  │       │
│  │ employees │  │───┘   │
│  │ ┌─────┐   │         │
│  │ │ ○───┼───┘         │
│  └───────────┘         │
└─────────────────┘       │
                    Employee shared
                    by both departments
```


#### Lifecycle Diagrams

```
Time: 0          1          2          3
     │          │          │          │
Emp  │───○──────┼──────────┼──────────○───
     │  Create  │          │  Delete  │
     │          │          │          │
Dept │    ──────○──────────○──────────
     │         Create    Delete       │
     │          │          │          │
     └──────────┴──────────┴──────────┘
     
     Employee created BEFORE Department
     Employee survives after Department deleted
```


### Advantages

| Advantage | Explanation | Example |
| :-- | :-- | :-- |
| **Flexibility** | Objects can be shared | Employee in multiple departments |
| **Independent Lifecycle** | Objects exist independently | Book exists without library |
| **Dynamic Relationships** | Can change associations | Player transfers between teams |
| **Loose Coupling** | Less dependency | Teacher not owned by school |
| **Resource Sharing** | Same object used multiple times | Database shared across apps |
| **Reusability** | Objects reused | Same product from multiple suppliers |

### Disadvantages

| Disadvantage | Explanation | Risk |
| :-- | :-- | :-- |
| **Lifecycle Management** | Must manage independently | Object might be deleted while still referenced |
| **Dangling References** | Reference to deleted object | Crash if not checked |
| **Complex State** | Object in multiple places | State inconsistency |
| **Less Encapsulation** | Internal objects exposed | Can modify from outside |
| **Memory Leaks** | Objects not cleaned up | If references not removed |

### Best Practices

1. **Always Check for None**
```python
class Department:
    def get_manager(self):
        if self.manager:  # Check first!
            return self.manager.name
        return None
```

2. **Remove References When Done
