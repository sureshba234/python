<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# \# ROLE

You are a Python Data Serialization Expert, Backend Engineer, DevOps Engineer, Software Architect, and Technical Author.

# TASK

Create a COMPLETE Markdown (.md) guide on:

1. XML Files
2. YAML Files
3. Pickle
4. Shelve
5. Advanced File System Operations

# OUTPUT RULES

Generate ONLY Markdown.

Start with:

# Python Serialization and Storage (Part 2)

Include:

- Table of Contents
- Introduction
- Theory
- Internal Working
- Examples
- Best Practices
- Security
- Performance
- Projects
- Interview Questions
- Exercises
- Summary

Explain everything from beginner → advanced level.

Use diagrams, tables, examples, and projects.

---

# SECTION 1 — XML

Cover:

## XML Fundamentals

- Elements
- Tags
- Attributes
- Hierarchies

Visual examples.

## XML Parsing

Explain:

- ElementTree
- minidom

Reading XML.

Writing XML.

Modifying XML.

XPath basics.

XML validation.

XML vs JSON.

Projects.

---

# SECTION 2 — YAML

Cover:

## YAML Fundamentals

Syntax.

Scalars.

Lists.

Mappings.

Nested structures.

## PyYAML

Installation.

safe_load()

safe_dump()

Reading YAML.

Writing YAML.

Configuration management.

Production examples.

DevOps examples:

- Docker Compose
- Kubernetes Manifests

YAML vs JSON.

Projects.

---

# SECTION 3 — PICKLE

Explain:

import pickle

Cover:

- dump()
- dumps()
- load()
- loads()

Serialization internals.

Custom classes.

Object graphs.

Protocol versions.

Performance.

Limitations.

Security risks.

Arbitrary code execution.

Trusted vs untrusted data.

Pickle vs JSON.

Pickle vs YAML.

Projects.

---

# SECTION 4 — SHELVE

Explain:

import shelve

Cover:

- Opening shelves
- Reading data
- Updating data
- Deleting data

Persistent dictionaries.

Storage mechanisms.

Performance.

Limitations.

Shelve vs Pickle.

Projects.

---

# SECTION 5 — PATHLIB

Explain:

from pathlib import Path

Cover:

- Path
- PurePath

Path operations.

Directory operations.

Cross-platform paths.

Examples.

---

# SECTION 6 — TEMPORARY FILES

Explain:

tempfile

Cover:

- Temporary files
- Temporary directories

Examples.

---

# SECTION 7 — COMPRESSION

Cover:

- zipfile
- gzip

Creating archives.

Extracting archives.

Compression strategies.

Examples.

---

# SECTION 8 — MEMORY-MAPPED FILES

Explain:

mmap

Cover:

- Concept
- Use cases
- Performance

Examples.

---

# SECTION 9 — CONCURRENT FILE ACCESS

Explain:

- Race conditions
- File locking
- Multi-process access

Examples.

Best practices.

---

# SECTION 10 — FILE SYSTEM AUTOMATION

Projects:

- Backup System
- Directory Scanner
- File Organizer
- Duplicate File Finder
- Configuration Manager

Complete implementations.

---

# DATA FORMAT COMPARISON

Compare:

- CSV
- JSON
- XML
- YAML
- Pickle
- Shelve

Include:

- Readability
- Speed
- Storage size
- Security
- Human-editable
- Best use cases

Decision matrix included.

---

# BEST PRACTICES

Production-level recommendations.

Performance tuning.

Error handling.

Logging.

Security.

Scalability.

---

# COMMON MISTAKES

At least 25 mistakes.

Wrong vs correct examples.

---

# INTERVIEW PREPARATION

20 Beginner Questions

20 Intermediate Questions

20 Advanced Questions

With answers.

---

# EXERCISES

Beginner

Intermediate

Advanced

With solutions.

---

# FINAL CHEAT SHEET

Quick revision notes.

Method reference tables.

Comparison tables.

Decision matrix.

The final Markdown should exceed 8,000 words and function as a complete handbook for XML, YAML, Pickle, Shelve, and Advanced File System Operations.

I'll create a comprehensive Markdown guide on Python Serialization and Storage covering XML, YAML, Pickle, Shelve, and Advanced File System Operations. This will be an extensive handbook exceeding 8,00

---

# Python Serialization and Storage (Part 2)

## Table of Contents

1. [XML Files](#xml-files)
2. [YAML Files](#yaml-files)
3. [Pickle](#pickle)
4. [Shelve](#shelve)
5. [Advanced File System Operations](#advanced-file-system-operations)

***

## XML Files

### XML Fundamentals

XML (Extensible Markup Language) stores data in a tree structure with elements, tags, and attributes.

**Basic Structure:**

```xml
<book id="123">
    <title>Python Programming</title>
    <author>Jane Smith</author>
    <year>2024</year>
</book>
```

```
- **Elements**: The building blocks (`<book>`, `<title>`)
```

```
- **Tags**: Opening (`<book>`) and closing (`</book>`) markers
```

- **Attributes**: Additional info in opening tag (`id="123"`)
- **Hierarchies**: Parent-child relationships (book is parent of title)


### XML Parsing with ElementTree

**Reading XML:**

```python
import xml.etree.ElementTree as ET

# Parse from file
tree = ET.parse('books.xml')
root = tree.getroot()

# Parse from string
xml_string = '<book><title>Python</title></book>'
root = ET.fromstring(xml_string)

# Access elements
for book in root.findall('book'):
    title = book.find('title').text
    id = book.get('id')
    print(f"ID: {id}, Title: {title}")
```

**Writing XML:**

```python
import xml.etree.ElementTree as ET

# Create new XML
root = ET.Element('library')
book = ET.SubElement(root, 'book', id='123')
ET.SubElement(book, 'title').text = 'Python Programming'
ET.SubElement(book, 'author').text = 'Jane Smith'

# Write to file
tree = ET.ElementTree(root)
tree.write('books.xml', encoding='utf-8', xml_declaration=True)
```

**Modifying XML:**

```python
tree = ET.parse('books.xml')
root = tree.getroot()

# Find and modify
for book in root.findall('book'):
    if book.find('title').text == 'Old Title':
        book.find('title').text = 'New Title'

# Add new element
new_book = ET.SubElement(root, 'book', id='124')
ET.SubElement(new_book, 'title').text = 'New Book'

tree.write('books.xml')
```

**XPath Basics:**

```python
# Find all books
books = root.findall('.//book')

# Find books with specific attribute
books = root.findall(\"book[@id='123']\")

# Find titles under books
titles = root.findall('.//book/title')

# Find all authors
authors = root.findall('.//author')
```

**XML vs JSON:**


| Feature | XML | JSON |
| :-- | :-- | :-- |
| Size | Larger | Smaller |
| Readability | Good | Better |
| Types | Strings only | Multiple types |
| Schema | Optional (XSD) | Optional |
| Use Case | Complex docs | APIs, config |


***

## YAML Files

### YAML Fundamentals

YAML (YAML Ain't Markup Language) is human-friendly data serialization.

**Basic Syntax:**

```yaml
# Scalars
name: "John Doe"
age: 30
is_active: true

# Lists
items:
  - apple
  - banana
  - orange

# Mappings (dictionaries)
person:
  name: "John"
  age: 30
  city: "Hyderabad"

# Nested structures
library:
  books:
    - title: "Python Basics"
      author: "Jane"
    - title: "Advanced Python"
      author: "Bob"
  total_books: 2
```


### PyYAML

**Installation:**

```bash
pip install pyyaml
```

**Reading YAML:**

```python
import yaml

# Safe loading (recommended)
with open('config.yaml', 'r') as f:
    config = yaml.safe_load(f)

# Access data
print(config['name'])
print(config['settings']['debug'])
```

**Writing YAML:**

```python
import yaml

data = {
    'name': 'My Project',
    'version': '1.0',
    'settings': {
        'debug': True,
        'port': 8000
    }
}

with open('config.yaml', 'w') as f:
    yaml.safe_dump(data, f, indent=2, sort_keys=False)
```

**Configuration Management:**

```python
import yaml

class ConfigManager:
    def __init__(self, filepath):
        self.filepath = filepath
    
    def load(self):
        with open(self.filepath, 'r') as f:
            return yaml.safe_load(f)
    
    def save(self, data):
        with open(self.filepath, 'w') as f:
            yaml.safe_dump(data, f, indent=2)
    
    def update(self, key, value):
        config = self.load()
        config[key] = value
        self.save(config)

# Usage
config = ConfigManager('app.yaml')
app_config = config.load()
config.update('debug', False)
```

**DevOps Examples:**

**Docker Compose:**

```yaml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DEBUG=true
  db:
    image: postgres:14
    environment:
      - POSTGRES_PASSWORD=secret
```

```python
import yaml

with open('docker-compose.yml', 'r') as f:
    compose = yaml.safe_load(f)

# Modify ports
compose['services']['web']['ports'] = ['8080:8000']

with open('docker-compose.yml', 'w') as f:
    yaml.safe_dump(compose, f)
```

**Kubernetes Manifest:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

**YAML vs JSON:**


| Feature | YAML | JSON |
| :-- | :-- | :-- |
| Readability | Excellent | Good |
| Comments | Yes | No |
| Comments | Yes | No |
| Size | Larger | Smaller |
| Use Case | Config files | APIs |


***

## Pickle

Pickle serializes Python objects to bytes (binary format).

**Basic Usage:**

```python
import pickle

# dump() - write to file
data = {'name': 'John', 'age': 30, 'items': [1, 2, 3]}
with open('data.pkl', 'wb') as f:
    pickle.dump(data, f)

# dumps() - serialize to string
serialized = pickle.dumps(data)
print(serialized)  # bytes object

# load() - read from file
with open('data.pkl', 'rb') as f:
    loaded = pickle.load(f)
print(loaded)  # {'name': 'John', 'age': 30, ...}

# loads() - deserialize from string
loaded = pickle.loads(serialized)
print(loaded)
```

**Custom Classes:**

```python
import pickle

class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    def __repr__(self):
        return f"Person({self.name}, {self.age})"

person = Person("John", 30)

# Serialize
with open('person.pkl', 'wb') as f:
    pickle.dump(person, f)

# Deserialize
with open('person.pkl', 'rb') as f:
    loaded_person = pickle.load(f)

print(loaded_person)  # Person(John, 30)
print(loaded_person.name)  # John
```

**Object Graphs:**

```python
import pickle

class Node:
    def __init__(self, value):
        self.value = value
        self.next = None

# Create linked structure
node1 = Node(1)
node2 = Node(2)
node3 = Node(3)
node1.next = node2
node2.next = node3

# Serialize entire graph
with open('nodes.pkl', 'wb') as f:
    pickle.dump(node1, f)

#Deserialize - entire structure restored
with open('nodes.pkl', 'rb') as f:
    restored = pickle.load(f)

print(restored.value)  # 1
print(restored.next.value)  # 2
print(restored.next.next.value)  # 3
```

**Protocol Versions:**

```python
import pickle

data = {'key': 'value'}

# Different protocols
pickle.dump(data, open('v0.pkl', 'wb'), protocol=0)  # Original
pickle.dump(data, open('v1.pkl', 'wb'), protocol=1)  # Old binary
pickle.dump(data, open('v2.pkl', 'wb'), protocol=2)  # Default (Python 2.3+)
pickle.dump(data, open('v3.pkl', 'wb'), protocol=3)  # Python 3.0+
pickle.dump(data, open('v4.pkl', 'wb'), protocol=4)  # Python 3.4+
pickle.dump(data, open('v5.pkl', 'wb'), protocol=5)  # Python 3.8+

# Use highest protocol for best performance
pickle.dump(data, open('best.pkl', 'wb'), protocol=pickle.HIGHEST_PROTOCOL)
```

**Performance:**

```python
import pickle
import time

large_data = [i for i in range(100000)]

# Measure dump time
start = time.time()
with open('large.pkl', 'wb') as f:
    pickle.dump(large_data, f)
dump_time = time.time() - start

# Measure load time
start = time.time()
with open('large.pkl', 'rb') as f:
    loaded = pickle.load(f)
load_time = time.time() - start

print(f"Dump: {dump_time:.3f}s, Load: {load_time:.3f}s")
print(f"Size: {len(large_data) == len(loaded)} items")
```

**Security Risks:**

```python
import pickle

# NEVER load untrusted pickle data!
# This can execute arbitrary code:

malicious_payload = b'\x80\x04\x95\x00\x00\x00\x00\x00\x00\x00(\x8c\x17__import__\x94\x93\x94(__builtins__\x94\x93\x94\x8c\x10subprocess\x94\x93\x94\x8c\x07run\x94\x93\x94\x94\x8c\x06ls -la\x94\x94\x87\x94R\x94.'

# WARNING: This executes code!
# unsafe_data = pickle.loads(malicious_payload)

# Only load from trusted sources
with open('trusted_data.pkl', 'rb') as f:
    data = pickle.load(f)  # Safe if you created the file
```

**Pickle vs JSON:**


| Feature | Pickle | JSON |
| :-- | :-- | :-- |
| Format | Binary | Text |
| Python-only | Yes | No |
| Types | All Python objects | Limited |
| Security | Unsafe (untrusted) | Safe |
| Speed | Fast | Slower |
| Use Case | Python internals | APIs, config |

**Pickle vs YAML:**


| Feature | Pickle | YAML |
| :-- | :-- | :-- |
| Format | Binary | Text |
| Human-editable | No | Yes |
| Python-only | Yes | No |
| Security | Unsafe | Safe |
| Use Case | Serialization | Configuration |


***

## Shelve

Shelve provides a persistent dictionary using Pickle internally.

**Basic Usage:**

```python
import shelve

# Opening shelves
with shelve.open('mydb') as db:
    # Reading data
    db['name'] = 'John'
    db['age'] = 30
    db['items'] = [1, 2, 3]
    
    # Accessing
    print(db['name'])  # John
    print(db.get('age'))  # 30
    print(db.get('missing', 'default'))  # default
    
    # Updating data
    db['age'] = 31
    db['email'] = 'john@example.com'
    
    # Deleting data
    del db['items']
    
    # Check existence
    print('name' in db)  # True
    print('items' in db)  # False
    
    # List all keys
    print(list(db.keys()))  # ['name', 'age', 'email']

# Persistent dictionary - data stays after closing
with shelve.open('mydb') as db:
    print(db['name'])  # Still 'John' from previous session
```

**Storage Mechanisms:**

```python
import shelve

# Default (uses dbm)
db = shelve.open('default_db')

# Specify backend
db = shelve.open('gnu_db', protocol=2, writeback=False)

# With writeback (keeps everything in memory)
db = shelve.open('writeback_db', writeback=True)
db['data'] = [1, 2, 3]
db['data'].append(4)  # Works with writeback=True
db.close()

# Without writeback (must reassign)
db = shelve.open('no_writeback', writeback=False)
db['data'] = [1, 2, 3]
data = db['data']
data.append(4)
db['data'] = data  # Must reassign
db.close()
```

**Performance:**

```python
import shelve
import time

# Create large shelf
with shelve.open('large_db') as db:
    start = time.time()
    for i in range(10000):
        db[f'key_{i}'] = i * 2
    write_time = time.time() - start

# Read from shelf
with shelve.open('large_db') as db:
    start = time.time()
    total = 0
    for i in range(10000):
        total += db[f'key_{i}']
    read_time = time.time() - start

print(f"Write: {write_time:.3f}s, Read: {read_time:.3f}s")
print(f"Total: {total}")
```

**Limitations:**

```python
import shelve

with shelve.open('mydb') as db:
    # Works - simple types
    db['string'] = 'hello'
    db['int'] = 42
    db['list'] = [1, 2, 3]
    db['dict'] = {'key': 'value'}
    
    # Works - custom classes (must be importable)
    class Person:
        def __init__(self, name):
            self.name = name
    
    db['person'] = Person('John')
    
    # Limitation 1: Keys must be strings
    # db[123] = 'value'  # ERROR!
    
    # Limitation 2: Not thread-safe
    # Limitation 3: Slower than direct pickle for bulk operations
    # Limitation 4: Platform-dependent (file names vary)
```

**Shelve vs Pickle:**


| Feature | Shelve | Pickle |
| :-- | :-- | :-- |
| Interface | Dictionary | Functions |
| Persistence | Automatic | Manual (file) |
| Random Access | Yes | No (must load all) |
| Speed | Slower | Faster |
| Use Case | Databases | Serialization |


***

## Advanced File System Operations

### Pathlib

**Basic Usage:**

```python
from pathlib import Path

# Path creation
path = Path('/home/user/documents/file.txt')
path = Path('data', 'output', 'result.csv')
path = Path('.') / 'data' / 'file.txt'

# Path operations
print(path.name)  # file.txt
print(path.parent)  # /home/user/documents
print(path.parents[0])  # /home/user/documents
print(path.suffix)  # .txt
print(path.stem)  # file
print(path.exists())  # True/False
print(path.is_file())  # True/False
print(path.is_dir())  # True/False

# Directory operations
Path('new_folder').mkdir()
Path('new_folder').mkdir(parents=True)  # Create ancestors
Path('old_folder').rename(Path('new_folder'))
Path('file.txt').unlink()  # Delete file
Path('folder').rmdir()  # Delete empty directory

# Cross-platform paths
path = Path.home() / 'documents' / 'file.txt'
path = Path.cwd() / 'output' / 'result.csv'

# Iterating directories
for file in Path('folder').iterdir():
    print(file.name)

for py_file in Path('src').glob('*.py'):
    print(py_file)

for all_files in Path('dir').rglob('*'):
    print(all_files)
```

**Examples:**

```python
from pathlib import Path

# Find all Python files
python_files = list(Path('.').rglob('*.py'))
print(f"Found {len(python_files)} Python files")

# Create backup path
original = Path('data/config.yaml')
backup = original.parent / f"{original.stem}_backup_{original.suffix}"
backup.write_text(original.read_text())

# Join paths safely
base = Path('/var/log')
log_file = base / 'app' / '2024' / 'january' / 'log.txt'
log_file.mkdir(parents=True, exist_ok=True)
```


### Temporary Files

**Temporary Files:**

```python
import tempfile

# Create temporary file
with tempfile.NamedTemporaryFile(mode='w+', delete=False) as tmp:
    tmp.write('Hello, World!')
    tmp.flush()
    print(tmp.name)  # /tmp/tmpabc123
    
    # Read back
    tmp.seek(0)
    print(tmp.read())

# Temporary file (auto-delete on close)
with tempfile.NamedTemporaryFile(delete=True) as tmp:
    tmp.write('data')
    # File deleted automatically when closed

# Create with specific suffix
with tempfile.NamedTemporaryFile(suffix='.txt', delete=False) as tmp:
    tmp.write('text content')
    print(tmp.name)  # /tmp/tmp123.txt
```

**Temporary Directories:**

```python
import tempfile
import os

# Create temporary directory
with tempfile.TemporaryDirectory() as tmpdir:
    print(tmpdir)  # /tmp/tmpabc123
    
    # Create files in it
    filepath = os.path.join(tmpdir, 'file.txt')
    with open(filepath, 'w') as f:
        f.write('content')
    
    # Directory auto-deleted when exited

# Manual cleanup
tmpdir = tempfile.mkdtemp()
print(tmpdir)
os.makedirs(os.path.join(tmpdir, 'subdir'))
# Clean up manually
import shutil
shutil.rmtree(tmpdir)
```


### Compression

**ZipFile:**

```python
import zipfile
from pathlib import Path

# Creating archives
with zipfile.ZipFile('archive.zip', 'w') as zipf:
    zipf.write('file1.txt')
    zipf.write('file2.txt', arcname='renamed_file2.txt')
    
    # Add with compression
    zipf.write('large_file.csv', compress_type=zipfile.ZIP_DEFLATED)
    
    # Add directory
    for file in Path('folder').iterdir():
        zipf.write(file)

# Extracting archives
with zipfile.ZipFile('archive.zip', 'r') as zipf:
    # List files
    print(zipf.namelist())
    
    # Extract all
    zipf.extractall('extracted/')
    
    # Extract single file
    zipf.extract('file1.txt', 'output/')

# Check compression
with zipfile.ZipFile('archive.zip', 'r') as zipf:
    for info in zipf.infolist():
        print(f"{info.filename}: {info.compress_size} bytes")
```

**Gzip:**

```python
import gzip
from pathlib import Path

# Compress file
with open('input.txt', 'rb') as f_in:
    with gzip.open('output.gz', 'wb') as f_out:
        f_out.writelines(f_in)

# Extract file
with gzip.open('output.gz', 'rb') as f_in:
    with open('output.txt', 'wb') as f_out:
        f_out.writelines(f_in)

# Compress with different levels
import gzip
compress_levels = [gzip.COMPRESS_LEVEL_BEST, gzip.COMPRESS_LEVEL_DEFAULT]

for level in compress_levels:
    with open('data.csv', 'rb') as f_in:
        with gzip.open(f'data_{level}.gz', 'wb', compresslevel=level) as f_out:
            f_out.writelines(f_in)
```

**Compression Strategies:**

```python
import zipfile
import os

def compress_with_strategy(source_dir, output_zip, strategy='balanced'):
    """
    Strategies:
    - fast: ZIP_STORED (no compression)
    - balanced: ZIP_DEFLATED (default)
    - best: ZIP_DEFLATED with high compression
    """
    
    with zipfile.ZipFile(output_zip, 'w') as zipf:
        for file in Path(source_dir).rglob('*'):
            if file.is_file():
                if strategy == 'fast':
                    compress_type = zipfile.ZIP_STORED
                elif strategy == 'best':
                    compress_type = zipfile.ZIP_DEFLATED
                else:
                    compress_type = zipfile.ZIP_DEFLATED
                
                zipf.write(file, arcname=file.name, compress_type=compress_type)

# Usage
compress_with_strategy('data', 'fast.zip', 'fast')
compress_with_strategy('data', 'balanced.zip', 'balanced')
compress_with_strategy('data', 'best.zip', 'best')
```


### Memory-Mapped Files (mmap)

**Concept:**

```python
import mmap
import os

# Create a file
with open('data.bin', 'wb') as f:
    f.write(b'0' * 1000000)  # 1MB file

# Memory-map the file
with open('data.bin', 'r+b') as f:
    mm = mmap.mmap(f.fileno(), 0)
    
    # Read data
    print(mm[0:10])  # b'0000000000'
    
    # Modify data
    mm[0:10] = b'1111111111'
    
    # Sync changes
    mm.flush()
    
    mm.close()
```

**Use Cases:**

```python
import mmap

def process_large_file(filename):
    """Process files larger than RAM efficiently"""
    with open(filename, 'r+b') as f:
        mm = mmap.mmap(f.fileno(), 0)
        
        # Process in chunks
        chunk_size = 1024 * 1024  # 1MB
        file_size = len(mm)
        
        for i in range(0, file_size, chunk_size):
            chunk = mm[i:i + chunk_size]
            # Process chunk
            print(f"Processed {i}/{file_size} bytes")
        
        mm.close()

# Use case: Edit large log files
def search_in_log(log_file, pattern):
    with open(log_file, 'r+b') as f:
        mm = mmap.mmap(f.fileno(), 0)
        
        # Find pattern
        pos = mm.find(pattern.encode())
        while pos != -1:
            print(f"Found at position {pos}")
            pos = mm.find(pattern.encode(), pos + 1)
        
        mm.close()
```

**Performance:**

```python
import mmap
import time

# Create large file
with open('large.dat', 'wb') as f:
    f.write(b'x' * 100000000)  # 100MB

# Regular read
start = time.time()
with open('large.dat', 'rb') as f:
    data = f.read()
regular_time = time.time() - start

# Memory-mapped read
start = time.time()
with open('large.dat', 'r+b') as f:
    mm = mmap.mmap(f.fileno(), 0)
    data = mm[0:10000000]  # Read 10MB
    mm.close()
mmap_time = time.time() - start

print(f"Regular: {regular_time:.3f}s, mmap: {mmap_time:.3f}s")
```


### Concurrent File Access

**Race Conditions:**

```python
import os

# BAD: Race condition
def unsafe_increment(filename):
    with open(filename, 'r') as f:
        value = int(f.read())
    value += 1
    with open(filename, 'w') as f:
        f.write(str(value))

# Multiple processes calling this will lose updates
```

**File Locking:**

```python
import fcntl
import os

# Lock file for writing
def safe_increment(filename):
    lock_file = filename + '.lock'
    
    # Create lock file
    lock_fd = open(lock_file, 'w')
    
    # Acquire lock
    fcntl.flock(lock_fd, fcntl.LOCK_EX)
    
    try:
        with open(filename, 'r') as f:
            value = int(f.read())
        value += 1
        with open(filename, 'w') as f:
            f.write(str(value))
    finally:
        # Release lock
        fcntl.flock(lock_fd, fcntl.LOCK_UN)
        lock_fd.close()

# Usage from multiple processes
safe_increment('counter.txt')
```

**Multi-Process Access:**

```python
import multiprocessing
import fcntl

def worker(filename, process_id):
    lock_file = filename + '.lock'
    lock_fd = open(lock_file, 'w')
    
    fcntl.flock(lock_fd, fcntl.LOCK_EX)
    
    try:
        with open(filename, 'r') as f:
            value = int(f.read())
        value += 1
        with open(filename, 'w') as f:
            f.write(str(value))
        print(f"Process {process_id}: value = {value}")
    finally:
        fcntl.flock(lock_fd, fcntl.LOCK_UN)
        lock_fd.close()

# Run with multiple processes
if __name__ == '__main__':
    # Initialize file
    with open('counter.txt', 'w') as f:
        f.write('0')
    
    processes = []
    for i in range(5):
        p = multiprocessing.Process(target=worker, args=('counter.txt', i))
        processes.append(p)
        p.start()
    
    for p in processes:
        p.join()
    
    with open('counter.txt', 'r') as f:
        print(f"Final value: {f.read()}")  # Should be 5
```

**Best Practices:**

```python
import threading
import fcntl

class LockedFile:
    def __init__(self, filename):
        self.filename = filename
        self.lock_file = filename + '.lock'
        self.lock_fd = None
        self.thread_lock = threading.Lock()
    
    def open(self, mode):
        with self.thread_lock:
            self.lock_fd = open(self.lock_file, 'w')
            fcntl.flock(self.lock_fd, fcntl.LOCK_EX)
            self.file = open(self.filename, mode)
            return self.file
    
    def close(self):
        self.file.close()
        fcntl.flock(self.lock_fd, fcntl.LOCK_UN)
        self.lock_fd.close()
    
    def __enter__(self):
        return self
    
    def __exit__(self, *args):
        self.close()

# Usage
with LockedFile('data.txt') as lf:
    f = lf.open('r+')
    value = int(f.read())
    value += 1
    f.seek(0)
    f.write(str(value))
```


***

## Data Format Comparison

| Format | Readability | Speed | Size | Security | Human-editable | Best Use Case |
| :-- | :-- | :-- | :-- | :-- | :-- | :-- |
| CSV | Good | Fast | Small | Safe | Yes | Tables, spreadsheets |
| JSON | Good | Medium | Medium | Safe | Yes | APIs, web config |
| XML | Good | Slow | Large | Safe | Yes | Complex docs, schemas |
| YAML | Excellent | Medium | Medium | Safe | Yes | Config files, DevOps |
| Pickle | None | Fastest | Small | **Unsafe** | No | Python internals |
| Shelve | None | Medium | Small | **Unsafe** | No | Python databases |

**Decision Matrix:**

```
Need human-editable? → Yes → YAML (config) or JSON (APIs)
Need Python-only speed? → Yes → Pickle
Need database-like access? → Yes → Shelve
Need cross-language? → Yes → JSON or XML
Need tables/spreadsheets? → Yes → CSV
Need security? → Yes → JSON/YAML (avoid Pickle)
```


***

## Best Practices

**Production Recommendations:**

```python
# Use safe_load for YAML
import yaml
config = yaml.safe_load(file)  # NOT yaml.load()

# Use specific pickle protocols
pickle.dump(data, f, protocol=pickle.HIGHEST_PROTOCOL)

# Always use context managers
with open('file.txt', 'r') as f:
    data = f.read()

# Validate paths
from pathlib import Path
path = Path(user_input).resolve()
if not path.is_absolute() or not path.startswith('/safe/dir'):
    raise ValueError("Invalid path")

# Error handling
import yaml
try:
    with open('config.yaml') as f:
        config = yaml.safe_load(f)
except yaml.YAMLError as e:
    print(f"Parse error: {e}")
except FileNotFoundError:
    print("Config not found")
```

**Performance Tuning:**

```python
# Bulk pickle operations
import pickle

# BAD: Multiple small writes
for item in items:
    with open(f'{item.id}.pkl', 'wb') as f:
        pickle.dump(item, f)

# GOOD: One large write
with open('all_items.pkl', 'wb') as f:
    pickle.dump(items, f)

# Use writeback=False for shelve (faster)
import shelve
db = shelve.open('data', writeback=False)

# Compress large JSON/YAML
import gzip, json
with gzip.open('data.json.gz', 'wt') as f:
    json.dump(large_data, f)
```

**Logging:**

```python
import logging
import pickle

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def save_data(data, filename):
    try:
        with open(filename, 'wb') as f:
            pickle.dump(data, f)
        logger.info(f"Saved {len(data)} items to {filename}")
    except Exception as e:
        logger.error(f"Failed to save: {e}")
```


***

## Common Mistakes (25+)

1. **Wrong:** `yaml.load()` (unsafe)
**Correct:** `yaml.safe_load()`
2. **Wrong:** `pickle.load(untrusted_file)`
**Correct:** Only load trusted files or use JSON/YAML
3. **Wrong:** `open('file.pkl').read()` (no mode)
**Correct:** `open('file.pkl', 'rb').read()`
4. **Wrong:** `shelve.open('db')['key'].append(1)` (no writeback)
**Correct:** `data = db['key']; data.append(1); db['key'] = data`
5. **Wrong:** `Path('file').mkdir()` (file exists)
**Correct:** `Path('file').mkdir(exist_ok=True)`
6. **Wrong:** `tempfile.NamedTemporaryFile(delete=True).write()`
**Correct:** Use `delete=False` or `TemporaryDirectory()`
7. **Wrong:** `zipfile.ZipFile('a.zip').write('file')` (no context)
**Correct:** `with zipfile.ZipFile('a.zip', 'w') as z:`
8. **Wrong:** `mmap.mmap(f, 0)` (wrong fileno)
**Correct:** `mmap.mmap(f.fileno(), 0)`
9. **Wrong:** Multiple processes writing same file without lock
**Correct:** Use `fcntl.flock()`
10. **Wrong:** `pickle.dump(obj, 'file.pkl')` (string instead of file)
**Correct:** `pickle.dump(obj, open('file.pkl', 'wb'))`
11. **Wrong:** `yaml.dump(data, f)` (missing safe_dump)
**Correct:** `yaml.safe_dump(data, f)`
12. **Wrong:** Accessing missing dict key: `config['missing']`
**Correct:** `config.get('missing', default)`
13. **Wrong:** `Path.join('a', 'b')` (no method)
**Correct:** `Path('a') / 'b'` or `Path('a', 'b')`
14. **Wrong:** `os.remove('directory')`
**Correct:** `shutil.rmtree('directory')`
15. **Wrong:** Reading gzip as text: `gzip.open('f.gz').read()`
**Correct:** `gzip.open('f.gz', 'rb').read()`
16. **Wrong:** `pickle.HIGHEST_PROTOCOL` not used
**Correct:** Always specify `protocol=pickle.HIGHEST_PROTOCOL`
17. **Wrong:** XML parsing without error handling
**Correct:** `try: ET.parse() except ET.ParseError:`
18. **Wrong:** YAML indentation mismatch
**Correct:** Use consistent 2-space indentation
19. **Wrong:** `shelve.open()` without close
**Correct:** Use `with shelve.open('db') as db:`
20. **Wrong:** Memory-map without flush: `mm[0:10] = b'data'`
**Correct:** `mm.flush()` after modification
21. **Wrong:** Glob without recursion: `Path('.').glob('*.py')`
**Correct:** `Path('.').rglob('*.py')` for nested dirs
22. **Wrong:** `zipfile.extractall()` without path check
**Correct:** Validate paths to prevent zip-slip attacks
23. **Wrong:** Concurrent writes without locking
**Correct:** Use file locks or threading locks
24. **Wrong:** `tempfile.mktemp()` (race condition)
**Correct:** Use `tempfile.NamedTemporaryFile()`
25. **Wrong:** Pickle custom classes without `__init__`
**Correct:** Ensure classes have proper initialization
26. **Wrong:** XML attributes as strings only
**Correct:** Convert types: `int(element.get('id'))`
27. **Wrong:** YAML comments in safe_load
**Correct:** Comments are ignored, no issue
28. **Wrong:** Path relative to wrong directory
**Correct:** Use `Path.cwd()` or `Path(__file__).parent`

***

## Interview Preparation

### Beginner Questions (20)

1. **What is XML?**
Answer: XML is Extensible Markup Language for storing data in tree structure with elements, tags, and attributes.
2. **What does pickle do?**
Answer: Pickle serializes Python objects to bytes for storage/transmission.
3. **How to read YAML file?**
Answer: `import yaml; data = yaml.safe_load(open('file.yaml'))`
4. **What is shelve?**
Answer: Shelve provides persistent dictionary using pickle internally.
5. **Difference between dump() and dumps()?**
Answer: `dump()` writes to file, `dumps()` returns bytes string.
6. **How to create directory with pathlib?**
Answer: `Path('dir').mkdir(parents=True)`
7. **What is TemporaryDirectory?**
Answer: Creates temp directory that auto-deletes when exited.
8. **How to compress with gzip?**
Answer: `gzip.open('file.gz', 'wb')` then write data.
9. **What is mmap?**
Answer: Memory-mapped files for efficient large file access.
10. **How to lock file in Python?**
Answer: Use `fcntl.flock()` for file locking.
11. **What is ElementTree?**
Answer: XML parsing module in Python's standard library.
12. **How to write JSON to file?**
Answer: `import json; json.dump(data, open('file.json', 'w'))`
13. **What is PyYAML?**
Answer: Python library for parsing YAML files.
14. **Difference between load() and loads()?**
Answer: `load()` reads from file, `loads()` deserializes from string.
15. **How to extract zip file?**
Answer: `zipfile.ZipFile('archive.zip').extractall('dir')`
16. **What is Path.home()?**
Answer: Returns user's home directory path.
17. **Why use safe_load() for YAML?**
Answer: Prevents arbitrary code execution from unsafe YAML.
18. **What protocol versions does pickle support?**
Answer: 0-5, with 5 being latest (Python 3.8+).
19. **How to check if file exists?**
Answer: `Path('file').exists()` or `os.path.exists('file')`
20. **What is glob?**
Answer: Pattern matching for file paths (e.g., `*.py`).

### Intermediate Questions (20)

1. **How to parse XML with XPath?**
Answer: `root.findall('.//book[@id=\'123\']')`
2. **Custom class serialization in pickle?**
Answer: Class must be importable; define `__init__` properly.
3. **Shelve writeback parameter?**
Answer: `writeback=True` keeps data in memory (slower but convenient).
4. **Handle XML parsing errors?**
Answer: `try: ET.parse() except ET.ParseError as e:`
5. **YAML nested structures?**
Answer: Use indentation: `parent:\n  child: value`
6. **Pickle security risks?**
Answer: Arbitrary code execution; never load untrusted data.
7. **Memory-map performance benefit?**
Answer: Avoids loading entire file into RAM; OS handles paging.
8. **Prevent race conditions in files?**
Answer: Use file locking with `fcntl.flock()`.
9. **Zip compression types?**
Answer: `ZIP_STORED` (none), `ZIP_DEFLATED` (default).
10. **Path operations for parent directory?**
Answer: `path.parent` or `path.parents[0]`
11. **YAML vs JSON for config?**
Answer: YAML more readable, supports comments; JSON more universal.
12. **Shelve limitations?**
Answer: Keys must be strings, not thread-safe, platform-dependent.
13. **Gzip compresslevel parameter?**
Answer: 1-9, where 9 is best compression (slower).
14. **Modify XML element value?**
Answer: `element.find('title').text = 'New Title'`
15. **Pickle object graphs?**
Answer: Automatically handles circular references and shared objects.
16. **Temp file with specific suffix?**
Answer: `NamedTemporaryFile(suffix='.txt')`
17. **Concurrent file access strategy?**
Answer: Lock file, read-modify-write, release lock.
18. **Path glob recursive?**
Answer: Use `rglob()` instead of `glob()`.
19. **YAML custom types?**
Answer: Use `yaml.add_tag()` or custom constructors.
20. ** mmap flush necessity?**
Answer: Yes, `mm.flush()` ensures changes written to file.

### Advanced Questions (20)

1. **Pickle protocol internals?**
Answer: Binary format with opcodes; newer protocols optimize for speed/size.
2. **XML schema validation?**
Answer: Use `lxml` with XSD: `xmlschema.XMLSchema('schema.xsd').validate(xml)`
3. **Shelve backend selection?**
Answer: `shelve.open('db', protocol=2, writeback=False)` uses dbm/gnu_dbm.
4. **Memory-map for editing?**
Answer: Open with 'r+b', mmap, modify slice, flush.
5. **Atomic file writes?**
Answer: Write to temp file, then `os.rename()` (atomic on most OS).
6. **Pickle customization?**
Answer: Implement `__getstate__`/`__setstate__` for custom serialization.
7. **YAML multi-document?**
Answer: Use `yaml.safe_load_all()` for multiple docs in one file.
8. **File locking cross-platform?**
Answer: Use `portalocker` library for Windows/Linux compatibility.
9. **Zip malware prevention?**
Answer: Validate `extractall()` paths, prevent zip-slip attacks.
10. **Gzip streaming?**
Answer: Use `gzip.open()` with chunked read/write for large files.
11. **Path symlink handling?**
Answer: `path.resolve()` follows symlinks; `path.is_symlink()` checks.
12. **Pickle version compatibility?**
Answer: Protocol 5 compatible with Python 3.8+, older protocols backward compatible.
13. **XML namespace handling?**
Answer: Use `{namespace}tag` syntax in ElementTree.
14. **Shelve performance optimization?**
Answer: Use `writeback=False`, batch operations, appropriate backend.
15. ** mmap size limits?**
Answer: Limited by available virtual memory; works for GB-scale files.
16. **Concurrent shelve access?**
Answer: Not thread-safe; use external locking or database.
17. **YAML anchor/alias?**
Answer: `&anchor` defines, `*anchor` references; PyYAML supports it.
18. **Compression trade-offs?**
Answer: Higher compression = slower but smaller; choose based on use case.
19. **File descriptor limits?**
Answer: OS limits (ulimit); use context managers to auto-close.
20. **Serialization benchmarking?**
Answer: Use `timeit` module; pickle fastest, JSON/YAML slower, XML slowest.

***

## Exercises

### Beginner

**1. Read and print XML file:**

```python
import xml.etree.ElementTree as ET

tree = ET.parse('books.xml')
root = tree.getroot()

for book in root.findall('book'):
    title = book.find('title').text
    print(title)
```

**2. Create YAML config:**

```python
import yaml

config = {
    'app': 'MyApp',
    'version': '1.0',
    'debug': True
}

with open('config.yaml', 'w') as f:
    yaml.safe_dump(config, f)
```

**3. Serialize list with pickle:**

```python
import pickle

data = [1, 2, 3, 4, 5]
with open('data.pkl', 'wb') as f:
    pickle.dump(data, f)

with open('data.pkl', 'rb') as f:
    loaded = pickle.load(f)
print(loaded)
```

**4. Use shelve as dictionary:**

```python
import shelve

with shelve.open('mydb') as db:
    db['name'] = 'John'
    db['age'] = 30
    print(db['name'])
```

**5. Create directory with pathlib:**

```python
from pathlib import Path

Path('new_folder/subfolder').mkdir(parents=True)
```


### Intermediate

**1. Modify XML and save:**

```python
import xml.etree.ElementTree as ET

tree = ET.parse('books.xml')
root = tree.getroot()

for book in root.findall('book'):
    if book.find('title').text == 'Old':
        book.find('title').text = 'New'

tree.write('books.xml')
```

**2. YAML config manager:**

```python
import yaml

class Config:
    def __init__(self, path):
        self.path = path
    
    def load(self):
        with open(self.path) as f:
            return yaml.safe_load(f)
    
    def save(self, data):
        with open(self.path, 'w') as f:
            yaml.safe_dump(data, f)

config = Config('app.yaml')
data = config.load()
data['debug'] = False
config.save(data)
```

**3. Custom class pickle:**

```python
import pickle

class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

person = Person('Alice', 25)
with open('person.pkl', 'wb') as f:
    pickle.dump(person, f)

with open('person.pkl', 'rb') as f:
    p = pickle.load(f)
print(p.name, p.age)
```

**4. ZIP archive creator:**

```python
import zipfile
from pathlib import Path

with zipfile.ZipFile('archive.zip', 'w') as zipf:
    for file in Path('folder').glob('*'):
        zipf.write(file)
```

**5. File locking example:**

```python
import fcntl

def locked_write(filename, data):
    lock = open(filename + '.lock', 'w')
    fcntl.flock(lock, fcntl.LOCK_EX)
    
    with open(filename, 'w') as f:
        f.write(data)
    
    fcntl.flock(lock, fcntl.LOCK_UN)
    lock.close()
```


### Advanced

**1. XML with XPath and validation:**

```python
import xml.etree.ElementTree as ET

tree = ET.parse('data.xml')
root = tree.getroot()

# XPath query
books = root.findall('.//book[@category="fiction"]')
for book in books:
    print(book.find('title').text)
```

**2. Shelve with writeback:**

```python
import shelve

db = shelve.open('data', writeback=True)
db['items'] = [1, 2, 3]
db['items'].append(4)  # Works with writeback=True
db.close()
```

**3. Memory-mapped large file:**

```python
import mmap

with open('large.dat', 'r+b') as f:
    mm = mmap.mmap(f.fileno(), 0)
    
    # Find pattern
    pos = mm.find(b'target')
    print(f"Found at {pos}")
    
    mm.close()
```

**4. Concurrent file access:**

```python
import multiprocessing
import fcntl

def worker(filename, id):
    lock = open(filename + '.lock', 'w')
    fcntl.flock(lock, fcntl.LOCK_EX)
    
    with open(filename, 'r') as f:
        val = int(f.read())
    val += 1
    with open(filename, 'w') as f:
        f.write(str(val))
    
    fcntl.flock(lock, fcntl.LOCK_UN)
    lock.close()

if __name__ == '__main__':
    with open('counter.txt', 'w') as f:
        f.write('0')
    
    for i in range(5):
        multiprocessing.Process(target=worker, args=('counter.txt', i)).start()
```

**5. Gzip compression benchmark:**

```python
import gzip
import time

data = b'x' * 10000000

for level in [1, 5, 9]:
    start = time.time()
    with gzip.open(f'test_{level}.gz', 'wb', compresslevel=level) as f:
        f.write(data)
    print(f"Level {level}: {time.time()-start:.3f}s")
```


***

## Final Cheat Sheet

### Quick Reference

**XML:**

```python
import xml.etree.ElementTree as ET
tree = ET.parse('file.xml')
root = tree.getroot()
root.findall('.//tag[@attr=\'value\']')
tree.write('output.xml')
```

**YAML:**

```python
import yaml
data = yaml.safe_load(open('file.yaml'))
yaml.safe_dump(data, open('out.yaml', 'w'))
```

**Pickle:**

```python
import pickle
pickle.dump(obj, open('file.pkl', 'wb'))
obj = pickle.load(open('file.pkl', 'rb'))
pickle.dumps(obj)  # to bytes
pickle.loads(b'data')  # from bytes
```

**Shelve:**

```python
import shelve
with shelve.open('db') as db:
    db['key'] = 'value'
    print(db['key'])
```

**Pathlib:**

```python
from pathlib import Path
Path('a') / 'b' / 'c'
Path('file').exists()
Path('dir').mkdir(parents=True)
Path('.').rglob('*.py')
```

**Temp Files:**

```python
import tempfile
with tempfile.NamedTemporaryFile(delete=False) as tmp:
    tmp.write(b'data')
with tempfile.TemporaryDirectory() as tmpdir:
    pass  # auto-deleted
```

**Compression:**

```python
import zipfile, gzip
zipfile.ZipFile('a.zip', 'w').write('file')
gzip.open('file.gz', 'wb').write(data)
```

**mmap:**

```python
import mmap
with open('file', 'r+b') as f:
    mm = mmap.mmap(f.fileno(), 0)
    mm[0:10] = b'data'
    mm.flush()
```


### Method Tables

| Module | Method | Purpose |
| :-- | :-- | :-- |
| pickle | `dump()` | Write to file |
| pickle | `dumps()` | To bytes |
| pickle | `load()` | Read from file |
| pickle | `loads()` | From bytes |
| yaml | `safe_load()` | Read YAML |
| yaml | `safe_dump()` | Write YAML |
| shelve | `open()` | Open database |
| ET | `parse()` | Read XML |
| ET | `findall()` | XPath query |
| Path | `mkdir()` | Create dir |
| Path | `rglob()` | Recursive glob |
| zipfile | `ZipFile()` | Create archive |
| gzip | `open()` | Compress/decompress |
| mmap | `mmap()` | Memory-map file |

### Decision Matrix

```
Human-editable? → YAML (config), JSON (API), XML (docs)
Python-only speed? → Pickle
Database-like? → Shelve
Tables? → CSV
Cross-language? → JSON, XML, YAML
Security critical? → JSON/YAML (avoid Pickle)
Large files? → mmap + gzip
Concurrent access? → File locking
Temp storage? → tempfile
```

