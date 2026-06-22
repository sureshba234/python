<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" style="height:64px;margin-right:32px"/>

# Python File Handling (Part 1)

# Table of Contents

1. [Introduction](#introduction)
2. [File Handling Fundamentals](#file-handling-fundamentals)
    - [What is File Handling?](#what-is-file-handling)
    - [File Objects](#file-objects)
    - [open() and File Modes](#open-and-file-modes)
    - [Reading Files](#reading-files)
    - [Writing Files](#writing-files)
    - [File Cursor](#file-cursor)
    - [Closing Files](#closing-files)
    - [Buffering](#buffering)
    - [Error Handling](#error-handling)
3. [Text Files](#text-files)
4. [Binary Files](#binary-files)
5. [CSV Files](#csv-files)
6. [JSON Files](#json-files)
7. [Best Practices](#best-practices)
8. [Common Mistakes](#common-mistakes)
9. [Performance Tips](#performance-tips)
10. [Security Considerations](#security-considerations)
11. [Real-World Projects](#real-world-projects)
12. [Interview Preparation](#interview-preparation)
13. [Exercises](#exercises)
14. [Cheat Sheet](#cheat-sheet)
15. [Summary](#summary)

***

# Introduction

File handling is one of the most practical parts of Python programming because it connects your code to durable data on disk, external systems, logs, configurations, and exchange formats. Python’s I/O model centers around streams, which are file-like objects that can be read from, written to, or both, and it distinguishes between text I/O, binary I/O, and raw I/O.[^1_1][^1_2]

This handbook takes you from beginner concepts to advanced usage across text files, binary files, CSV, and JSON. It emphasizes real-world usage, safe patterns, and performance-aware practices, because file bugs often appear only in production after data has already been written incorrectly.[^1_2][^1_1]

***

# File Handling Fundamentals

## What is File Handling?

File handling means creating, opening, reading, writing, updating, and closing files stored on persistent storage. A file keeps data even after the program ends, which makes it **non-volatile** storage, unlike volatile memory such as RAM that loses contents when power is removed.[^1_1]

A file typically goes through a lifecycle: it is created, opened, read or written, flushed, closed, and later reopened or deleted. In day-to-day programming, the main job is to choose the right mode, manage resources correctly, and ensure the data format matches the task.[^1_1]

### Persistent vs Volatile Memory

| Type | Example | Data survives power off? | Typical use |
| :-- | :-- | --: | :-- |
| Volatile memory | RAM | No | Temporary program data |
| Non-volatile memory | SSD, HDD, flash | Yes | Files, databases, backups |

Persistent storage is what makes logs, reports, configuration files, and exported datasets useful across runs. Volatile memory is faster, but it is not a storage medium for long-term data.

### File Lifecycle

```text
Create -> Open -> Read/Write -> Flush -> Close -> Reopen -> Archive/Delete
```

A useful mental model is: open a file only when you need it, do the minimum necessary work, then close it as early as possible. This reduces resource leaks and makes failures easier to isolate.[^1_1]

## File Objects

Python treats files as file-like objects, often called streams. A stream is an abstraction over a sequence of data flowing in or out; it may be a regular disk file, in-memory data, or even a device/pipe. Python’s I/O layer explicitly describes text I/O, binary I/O, and raw I/O as the main categories.[^1_2][^1_1]

A file object may be readable, writable, seekable, or some combination of these. Some streams support random access with `seek()`, while others only support sequential access, which is common for pipes and sockets.[^1_1]

### Streams, descriptors, buffering

| Concept | Meaning | Python connection |
| :-- | :-- | :-- |
| Stream | A flow of data | File objects, `StringIO`, `BytesIO` |
| File descriptor | OS-level integer handle | `fileno()` exposes it |
| Buffering | Temporary memory between app and OS | Improves performance for many operations |

File descriptors are low-level operating system handles. Python usually hides them behind higher-level objects, but they matter when integrating with OS APIs or debugging advanced I/O behavior.[^1_1]

## open() and File Modes

`open()` creates a file object and is the main entry point for file handling in Python. In text mode, you typically work with strings; in binary mode, you work with bytes.[^1_1]

### Core syntax

```python
f = open("example.txt", mode="r", encoding="utf-8")
```

Common parameters:


| Parameter | Purpose |
| :-- | :-- |
| `file` | Path or file descriptor |
| `mode` | Read/write/append/binary behavior |
| `encoding` | Text decoding/encoding, usually set for text files |
| `newline` | Controls newline translation |
| `buffering` | Buffering policy |

Python recommends specifying encoding explicitly for text files, especially UTF-8, because relying on platform defaults can break portability.[^1_1]

### Text modes

| Mode | Meaning | File must exist? | Creates file? | Truncates? | Can read? | Can write? |
| :-- | :-- | --: | --: | --: | --: | --: |
| `r` | Read | Yes | No | No | Yes | No |
| `w` | Write | No | Yes | Yes | No | Yes |
| `a` | Append | No | Yes | No | No | Yes |
| `x` | Exclusive create | No | Yes | No | No | Yes |
| `r+` | Read/write | Yes | No | No | Yes | Yes |
| `w+` | Read/write | No | Yes | Yes | Yes | Yes |
| `a+` | Append/read | No | Yes | No | Yes | Yes |

`r` fails if the file does not exist; `w` overwrites if it does exist; `a` always writes at the end; `x` fails if the file already exists. `r+`, `w+`, and `a+` all allow both read and write, but they differ in creation and truncation behavior.[^1_1]

### Binary modes

| Mode | Meaning | Relationship to text mode |
| :-- | :-- | :-- |
| `rb` | Read binary | Same access pattern, but returns bytes |
| `wb` | Write binary | Writes bytes, no encoding |
| `ab` | Append binary | Adds bytes to end |
| `rb+` | Read/write binary | Update existing binary file |
| `wb+` | Read/write binary | Create/truncate binary file |
| `ab+` | Read/append binary | Append bytes and allow reading |

Binary mode disables text encoding/decoding and newline translation. That is essential for images, PDFs, audio, executables, compressed files, and any data that is not meant to be interpreted as text.[^1_2][^1_1]

### Mode comparison notes

| Mode | Typical use case | Risk |
| :-- | :-- | :-- |
| `r` | Reading reports or configs | File may be missing |
| `w` | Rebuilding output files | Accidentally destroying old data |
| `a` | Logs or event streams | File can grow indefinitely |
| `x` | Safe one-time file creation | Raises if file exists |
| `r+` | In-place updates | Must handle pointer carefully |
| `wb` | Images, PDFs, model artifacts | Corruption if misused as text |
| `a+` | Append plus occasional inspection | Cursor starts at end |

A practical rule: choose the narrowest mode that satisfies your task. If you only need read access, do not open the file in update mode.[^1_1]

## Reading Files

Python provides several ways to read file contents, and each has different memory and control tradeoffs. The main methods are `read()`, `readline()`, and `readlines()`.[^1_1]

### `read()`

`read()` returns the whole file or up to a specified size.

```python
with open("notes.txt", "r", encoding="utf-8") as f:
    data = f.read()
```

If you pass a size, you can read in chunks:

```python
with open("large.txt", "r", encoding="utf-8") as f:
    chunk = f.read(4096)
```

`read()` is simple and convenient, but on large files it can consume a lot of memory. If the file is huge, prefer chunked reading or iteration.[^1_1]

### `readline()`

`readline()` reads one line at a time.

```python
with open("notes.txt", "r", encoding="utf-8") as f:
    first = f.readline()
```

It is useful when you want to process file content incrementally, such as logs. It preserves the newline character in most cases, which helps distinguish blank lines from EOF.[^1_1]

### `readlines()`

`readlines()` returns a list of all lines.

```python
with open("notes.txt", "r", encoding="utf-8") as f:
    lines = f.readlines()
```

This is easy to use but not memory-friendly for large files. In many cases, iterating directly over the file object is better:

```python
with open("notes.txt", "r", encoding="utf-8") as f:
    for line in f:
        process(line)
```

That pattern is efficient and idiomatic.[^1_1]

### Reading method comparison

| Method | Returns | Memory use | Best for |
| :-- | :-- | --: | :-- |
| `read()` | String/bytes | High for large files | Small-to-medium files |
| `readline()` | One line | Low | Logs, line-by-line parsing |
| `readlines()` | List of lines | High | Small files, quick scripts |
| `for line in f` | Line iterator | Low | Most large-text workflows |

## Writing Files

Writing in Python is straightforward, but correctness matters because writes can overwrite data, append unexpectedly, or fail before buffers are flushed.[^1_1]

### `write()`

`write()` writes a string in text mode or bytes in binary mode.

```python
with open("output.txt", "w", encoding="utf-8") as f:
    f.write("Hello, world!\n")
```

The method returns the number of characters or bytes written, which can be useful when verifying output sizes.[^1_1]

### `writelines()`

`writelines()` writes a sequence of strings. It does **not** add newline characters automatically.

```python
lines = ["one\n", "two\n", "three\n"]

with open("output.txt", "w", encoding="utf-8") as f:
    f.writelines(lines)
```

A common mistake is expecting `writelines()` to separate lines for you. If your items do not already include newline characters, the file may end up as one long line.[^1_1]

### Writing patterns

| Task | Best method |
| :-- | :-- |
| Write one formatted message | `write()` |
| Write many prepared lines | `writelines()` |
| Append log entries | `write()` in append mode |
| Stream data in chunks | repeated `write()` |

## File Cursor

The file cursor is the current position inside the file. You can inspect it with `tell()` and move it with `seek()`. In binary mode, `tell()` is generally a byte position; in text mode, the value is more opaque and should be used mainly with `seek()`.[^1_1]

### `tell()`

```python
with open("sample.txt", "r", encoding="utf-8") as f:
    pos = f.tell()
```

`tell()` helps when you need checkpoints, resuming reads, or partial processing. In text mode, treat the result as an opaque token rather than a raw byte offset.[^1_1]

### `seek()`

```python
with open("sample.txt", "rb") as f:
    f.seek(10)
    data = f.read(5)
```

Common `whence` values:


| `whence` | Meaning |
| :-- | :-- |
| `0` | From beginning |
| `1` | From current position |
| `2` | From end |

### Cursor behavior diagram

```text
|----data in file----|
^                    ^
start                end

seek(0) -> start
seek(10) -> 10 bytes/positions from start
seek(-5, 2) -> 5 bytes before end
```

In text mode, seeking rules are more restricted than in binary mode. If you need precise byte-level navigation, open the file in binary mode.[^1_1]

## Closing Files

Files should be closed after use because open handles consume system resources and buffered output may not be fully written otherwise. Python’s I/O base classes provide `close()` and support context management with the `with` statement.[^1_1]

### `close()`

```python
f = open("temp.txt", "w", encoding="utf-8")
f.write("data")
f.close()
```

After closing, operations on the file object fail. This is correct behavior because the underlying resource is no longer valid.[^1_1]

### Context managers

```python
with open("temp.txt", "w", encoding="utf-8") as f:
    f.write("data")
```

This is the recommended approach because the file is closed automatically even if an exception occurs. It is shorter, safer, and easier to maintain.[^1_1]

### `with` vs manual close

| Approach | Pros | Cons |
| :-- | :-- | :-- |
| Manual `close()` | Explicit control | Easy to forget, more error-prone |
| `with` statement | Safe, concise, automatic cleanup | Slightly less explicit to beginners |

A good rule is to use `with` everywhere unless you have a special need to keep the file open across a custom lifecycle.[^1_1]

## Buffering

Buffering means Python stores data temporarily in memory before handing it to the operating system or before returning it to your code. Python’s I/O system has buffered binary I/O as well as text I/O built on top of buffering.[^1_2][^1_1]

### Buffered I/O

Buffered I/O improves performance by reducing system calls. Instead of asking the OS for tiny reads or writes repeatedly, Python reads or writes larger chunks internally. That usually makes file access faster and smoother.[^1_1]

### Line buffering

Line buffering flushes output when a newline is encountered, which is useful for interactive terminals and logs. It is often used when output is tied to a terminal rather than a file.[^1_1]

### Unbuffered I/O

Unbuffered I/O writes data immediately. It gives you less latency but can be much less efficient, and it is generally a low-level choice. Python’s I/O documentation notes that raw I/O is a low-level interface and should be used carefully.[^1_1]

### Buffering summary

| Type | Behavior | Typical use |
| :-- | :-- | :-- |
| Buffered | Data accumulates before OS write/read | Most file work |
| Line buffered | Flushes on newline | Interactive text output |
| Unbuffered | Immediate transfer | Special low-level scenarios |

## Error Handling

File operations can fail for many reasons: missing files, permission errors, disk issues, invalid encoding, and malformed content. Good code catches exceptions narrowly and responds appropriately.[^1_1]

### Example: safe reading

```python
try:
    with open("missing.txt", "r", encoding="utf-8") as f:
        content = f.read()
except FileNotFoundError:
    content = ""
except PermissionError:
    content = ""
except OSError as e:
    content = ""
```


### Example: safe writing

```python
try:
    with open("report.txt", "w", encoding="utf-8") as f:
        f.write("Report data\n")
except OSError as e:
    print(f"Write failed: {e}")
```


### Error handling principles

- Catch specific exceptions first.
- Avoid hiding unexpected failures.
- Log the filename and operation.
- Make the fallback behavior explicit.

***

# Text Files

Text files store human-readable characters, but internally those characters must be encoded into bytes. Python text I/O decodes and encodes automatically when you specify a text encoding, and it also manages newline translation depending on the platform and `newline` setting.[^1_1]

## Encodings

An encoding defines how characters map to bytes. If the encoding is wrong, the same bytes can display as gibberish or raise decode errors. Python’s documentation recommends specifying an encoding explicitly, commonly UTF-8, when opening text files.[^1_1]

### Common encodings

| Encoding | Characteristics | Typical use |
| :-- | :-- | :-- |
| ASCII | 7-bit, very limited character set | Legacy systems |
| UTF-8 | Variable-length, backward compatible with ASCII | Modern default |
| UTF-16 | Uses 2-byte code units, often includes BOM | Some Windows and APIs |
| UTF-32 | Fixed-width, large memory footprint | Rare, specialized |
| Locale default | Depends on system | Avoid for portable code |

### ASCII

ASCII includes basic English letters, digits, punctuation, and control characters. It cannot represent most world languages, so it is unsuitable for modern multilingual software unless you are dealing with a legacy system.

### UTF-8

UTF-8 is the standard choice for text files in modern Python code because it is compact for ASCII text and supports all Unicode characters. It is also widely used in JSON and web APIs.[^1_1]

### UTF-16 and UTF-32

UTF-16 and UTF-32 can be useful when interacting with specific systems or protocols, but they are less common in general file handling. They may include byte-order marks and can be more awkward for line-based workflows than UTF-8.

## Reading text files

```python
with open("story.txt", "r", encoding="utf-8") as f:
    text = f.read()
```

For large text files, read line by line:

```python
with open("story.txt", "r", encoding="utf-8") as f:
    for line in f:
        handle(line)
```


## Writing text files

```python
with open("story.txt", "w", encoding="utf-8") as f:
    f.write("First line\n")
    f.write("Second line\n")
```

If the file already exists, `w` replaces it. If you want to preserve prior content, use `a` for appending.[^1_1]

## Appending text files

```python
with open("app.log", "a", encoding="utf-8") as f:
    f.write("New event\n")
```

Appending is ideal for logs, audit trails, and simple event histories. It is usually safer than rewriting the entire file on each update.[^1_1]

## Large file processing

Large files should almost never be read all at once. Instead, stream them in chunks or line by line to keep memory usage stable.[^1_1]

### Chunked reading

```python
def iter_chunks(path, chunk_size=8192):
    with open(path, "r", encoding="utf-8") as f:
        while True:
            chunk = f.read(chunk_size)
            if not chunk:
                break
            yield chunk
```


### Stream processing diagram

```text
Disk -> buffer -> Python chunk -> process -> next chunk
```

This pattern is especially important for logs, transcripts, exports, and large text datasets.

## Log processing

Logs are usually text files with one event per line. The most common workflow is to scan line by line, filter by keywords, parse timestamps, and count matching events.

```python
with open("server.log", "r", encoding="utf-8") as f:
    for line in f:
        if "ERROR" in line:
            print(line.rstrip())
```


## Text projects

### Word Counter

Read a file, normalize case, split into words, and count frequencies. This teaches file reading, string processing, and dictionary usage.

### Log Analyzer

Scan large log files and count error types, top endpoints, or status-code frequencies. This teaches streaming and incremental aggregation.

### Search Utility

Build a simple `grep`-style tool that finds matching lines across one or many files. This teaches iteration, filtering, and command-line style thinking.

***

# Binary Files

Binary files store raw bytes, not human-readable text. They are used for images, PDFs, audio, archives, compiled files, and many serialized formats. Python’s binary I/O layer works with bytes and does no encoding or newline translation.[^1_1]

## Binary vs Text

| Aspect | Text file | Binary file |
| :-- | :-- | :-- |
| Unit | Characters | Bytes |
| Encoding | Required | Not used |
| Newline translation | May happen | No |
| Read result | `str` | `bytes` |
| Examples | `.txt`, `.csv`, `.json` | `.png`, `.pdf`, `.zip`, `.bin` |

A file is not “binary” just because it has an unusual extension; it is binary because its contents are intended to be interpreted as bytes rather than text.

## `bytes` and `bytearray`

`bytes` is an immutable sequence of byte values. `bytearray` is mutable, which makes it useful for incremental binary manipulation.

```python
b = b"hello"
ba = bytearray(b"hello")
ba[^1_0] = ord("H")
```


### When to use each

| Type | Mutable? | Use case |
| :-- | --: | :-- |
| `bytes` | No | Reading, hashing, protocol payloads |
| `bytearray` | Yes | In-place edits, buffers, transformations |

## Reading binary files

```python
with open("image.png", "rb") as f:
    data = f.read()
```

Binary reads return bytes, which you can inspect, slice, hash, or pass to other libraries. If you need to parse structure, use binary-aware tooling rather than text functions.[^1_1]

## Writing binary files

```python
data = b"\x89PNG\r\n\x1a\n"
with open("copy.png", "wb") as f:
    f.write(data)
```

Writing the wrong data type to a binary file raises a type-related error because binary streams expect bytes-like objects.[^1_1]

## Images and PDFs

Images and PDFs should generally be handled in binary mode. If you open them as text, newline conversion and character decoding can corrupt the file. Python’s documentation explicitly warns that text-mode transformations can corrupt binary data.[^1_1]

### Safe transfer pattern

```python
with open("input.pdf", "rb") as src, open("backup.pdf", "wb") as dst:
    dst.write(src.read())
```

This pattern is useful for backups, file uploads, and attachment handling. For large files, use chunked copying instead of loading everything into memory.

## `struct` module

The `struct` module converts between Python values and C-style packed binary data. It is especially useful for binary protocols, file headers, and compact on-disk formats. The documentation describes `pack()` and `unpack()` as the core functions for converting values to and from packed byte strings.[^1_3][^1_4]

### `pack()` and `unpack()`

```python
import struct

data = struct.pack("i f", 42, 3.14)
values = struct.unpack("i f", data)
```

`pack()` turns Python values into bytes according to a format string, and `unpack()` reconstructs the original values from bytes. The format string defines the layout, size, and type of each field.[^1_4][^1_3]

### Example: simple binary record

```python
import struct

record = struct.pack("I 10s", 7, b"Alice\x00\x00\x00\x00\x00")
id_value, name_bytes = struct.unpack("I 10s", record)
```


### Binary layout diagram

```text
[ field 1 ][ field 2 ][ field 3 ]
  4 bytes    4 bytes    8 bytes
```


## Binary data projects

- Build a file signature detector from magic bytes.
- Parse a custom binary sensor record format.
- Create a thumbnail copier for images.
- Inspect the first bytes of a PDF or ZIP archive.
- Implement a simple binary backup tool.

***

# CSV Files

CSV stands for Comma-Separated Values and is one of the most common formats for tabular data exchange. Python’s standard `csv` module provides readers and writers for CSV format.[^1_5]

## Why CSV matters

CSV is simple, readable, and compatible with spreadsheet tools, databases, and analytics workflows. Its weakness is that it has limited structure compared with JSON, and it cannot naturally represent deeply nested data.

## `import csv`

```python
import csv
```

The module includes `reader`, `writer`, `DictReader`, and `DictWriter`. These tools support the common cases of row-based processing and dictionary-based schema handling.[^1_5]

## `reader`

```python
import csv

with open("students.csv", "r", newline="", encoding="utf-8") as f:
    reader = csv.reader(f)
    for row in reader:
        print(row)
```

`newline=""` is recommended when using the CSV module so newline handling is left to the module itself rather than the text layer. This avoids line-ending surprises on different platforms.[^1_5][^1_1]

## `writer`

```python
import csv

rows = [
    ["name", "age"],
    ["Alice", 30],
    ["Bob", 25],
]

with open("students.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.writer(f)
    writer.writerows(rows)
```

The CSV writer automatically handles quoting and delimiter escaping according to CSV rules. This is much safer than manually joining strings with commas.

## `DictReader`

```python
import csv

with open("students.csv", "r", newline="", encoding="utf-8") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row["name"], row["age"])
```

`DictReader` turns each row into a dictionary keyed by the header names. This is excellent when column names matter more than fixed positions.[^1_6][^1_5]

## `DictWriter`

```python
import csv

fieldnames = ["name", "age"]

with open("students.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=fieldnames)
    writer.writeheader()
    writer.writerow({"name": "Alice", "age": 30})
```

`DictWriter` is useful for exporting structured records, especially from APIs, ORM objects, or cleaned datasets.

## Custom delimiters

CSV is often not actually comma-separated; it might be tab-separated, pipe-separated, or semicolon-separated. The `csv` module supports this through the `delimiter` parameter.

```python
with open("data.tsv", "r", newline="", encoding="utf-8") as f:
    reader = csv.reader(f, delimiter="\t")
```


## Large CSV processing

For large CSV files, stream rows instead of loading everything at once. This keeps memory usage stable and makes it easier to process millions of rows.

```python
import csv

total = 0
with open("big.csv", "r", newline="", encoding="utf-8") as f:
    reader = csv.DictReader(f)
    for row in reader:
        total += int(row["amount"])
```


## CSV cleaning

Common cleaning tasks include trimming whitespace, normalizing missing values, fixing encodings, validating numeric columns, and removing duplicates. Always validate imported columns before trusting them downstream.

## CSV vs JSON

| Feature | CSV | JSON |
| :-- | :-- | :-- |
| Shape | Flat tables | Nested structures |
| Human-readable | Yes | Yes |
| Spreadsheet-friendly | Excellent | Moderate |
| Nested data | Poor | Good |
| Standard module | `csv` | `json` |

CSV is ideal for rows and columns; JSON is ideal for hierarchical data. Choose CSV when interoperability with spreadsheets or analytics tools matters most.

## CSV projects

- Clean a sales export and generate summary totals.
- Convert a CSV dataset into JSON records.
- Build a duplicate row detector.
- Create a report from a student marksheet.
- Normalize messy delimiter-separated logs.

***

# JSON Files

JSON is a lightweight text format for data interchange. Python’s `json` module converts between Python objects and JSON text using `dump()`, `dumps()`, `load()`, and `loads()`.[^1_7][^1_1]

## `import json`

```python
import json
```


## Serialization and deserialization

Serialization means converting a Python object into a stored or transferable representation, usually a JSON string. Deserialization means reconstructing a Python object from that JSON representation.[^1_1]

### Core functions

| Function | Direction | Works with |
| :-- | :-- | :-- |
| `dump()` | Python object -> file | File object |
| `dumps()` | Python object -> string | In-memory text |
| `load()` | file -> Python object | File object |
| `loads()` | string -> Python object | JSON text |

## `dumps()`

```python
import json

data = {"name": "Alice", "age": 30}
text = json.dumps(data)
```

This returns a JSON string, which is useful for logging, messaging, or sending data over HTTP. The `json` module always produces `str`, not bytes.[^1_7]

## `dump()`

```python
with open("data.json", "w", encoding="utf-8") as f:
    json.dump({"name": "Alice", "age": 30}, f)
```

For JSON files, opening with `encoding="utf-8"` is recommended because JSON is UTF-8 based and text-mode encoding should be explicit.[^1_1]

## `loads()`

```python
import json

data = json.loads('{"name": "Alice", "age": 30}')
```

This parses a JSON string into Python objects such as dictionaries, lists, strings, numbers, booleans, and `None`.[^1_7]

## `load()`

```python
with open("data.json", "r", encoding="utf-8") as f:
    data = json.load(f)
```

This is the standard way to read a JSON file. It is more direct than reading the file manually and then calling `loads()`.[^1_7][^1_1]

## Pretty printing

```python
import json

print(json.dumps({"name": "Alice", "skills": ["Python", "SQL"]}, indent=2))
```

Pretty printing makes JSON easier to inspect, review, and debug. In production APIs, compact form is often preferred for network efficiency, while pretty form is useful for configuration and human maintenance.

## Custom encoders

Default JSON encoding handles standard Python types but not all custom objects. For custom classes, you can provide an encoder or convert the object into a serializable dictionary first.

```python
import json
from datetime import datetime

class CustomEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, datetime):
            return obj.isoformat()
        return super().default(obj)
```


## Complex objects

JSON does not directly support tuples as tuples, sets, bytes, or arbitrary Python objects. You must convert them into JSON-compatible forms such as lists, strings, or dictionaries before serializing.[^1_7][^1_1]

## JSON validation

Validation means checking that incoming JSON has the expected structure, fields, and types. The built-in `json` module parses syntax, but business-rule validation must be done separately.

```python
import json

try:
    data = json.loads(payload)
except json.JSONDecodeError:
    data = None
```


## API processing

JSON is the default data format for many web APIs. Common tasks include parsing responses, extracting fields, transforming structures, and writing normalized records to files or databases.

## JSON vs CSV

| Feature | JSON | CSV |
| :-- | :-- | :-- |
| Nested data | Excellent | Poor |
| Arrays and objects | Native | Not native |
| Human readability | Good | Good |
| API usage | Excellent | Limited |
| Spreadsheet compatibility | Moderate | Excellent |

## JSON vs Pickle

JSON is text-based, portable, and language-friendly. Pickle is Python-specific and can execute arbitrary code during deserialization if the data is untrusted, which makes it insecure for untrusted input.[^1_1]

## JSON projects

- Build a configuration loader for app settings.
- Parse API responses into clean tabular output.
- Export database rows as JSON.
- Merge and normalize nested JSON files.
- Build a CLI that transforms JSON into CSV.

***

# Internal Working

Python file handling sits on top of layered I/O abstractions. At the lowest level, raw I/O interacts with the operating system; on top of that, buffered I/O improves performance; and on top of that, text I/O decodes bytes into strings and encodes strings back into bytes.[^1_1]

## I/O layering diagram

```text
Your code
   |
Text I/O  -> strings, encoding/decoding, newline translation
   |
Buffered I/O -> chunking, buffering, flushing
   |
Raw I/O  -> bytes, OS-level read/write
   |
Operating system
   |
Disk / device / pipe
```

This layered design is why text files are easy to use but binary workflows demand more precision. Python’s standard library provides concrete objects such as `FileIO`, `BufferedReader`, `BufferedWriter`, `BytesIO`, `TextIOWrapper`, and `StringIO` to implement these layers.[^1_1]

## Data flow

When reading text, the file bytes are read from disk, buffered, decoded using an encoding, and returned as strings. When writing, the reverse happens: strings are encoded to bytes, buffered, and then flushed to storage.[^1_1]

## Why buffering exists

Without buffering, every tiny read or write would involve an expensive operating system call. Buffering reduces overhead, improves throughput, and gives Python a chance to batch work efficiently.[^1_1]

## Why text I/O is slower than binary I/O

Text I/O has extra work: encoding, decoding, newline translation, and Unicode processing. This is usually worth it for convenience, but for huge files and non-text payloads, binary I/O can be noticeably faster.[^1_1]

***

# Best Practices

Use `with open(...)` for all file operations unless you have a compelling reason not to. It guarantees cleanup and reduces the chance of leaving files open after exceptions.[^1_1]

## Production recommendations

- Always specify `encoding="utf-8"` for text files unless you need a different encoding.
- Use binary mode for non-text files.
- Process large files incrementally.
- Validate input before trusting it.
- Keep file paths configurable, not hard-coded.


## Security

- Never deserialize untrusted pickle data.
- Validate JSON schemas or required keys.
- Sanitize file paths to avoid directory traversal.
- Limit file sizes before reading uploads.
- Treat logs and exported files as potentially sensitive.


## Error handling

- Catch expected exceptions only.
- Log filename, mode, and operation.
- Make fallback behavior explicit.
- Distinguish missing file from permission failure.


## Logging

Add logs around file open, read, write, parse, and flush steps. This helps diagnose issues such as permission errors, encoding failures, incomplete writes, and malformed input.

## Performance

- Prefer iteration over `readlines()`.
- Read large files in chunks.
- Use buffered defaults instead of unbuffered I/O.
- Avoid repeated open/close in tight loops.
- For CSV and JSON pipelines, stream data row by row.

***

# Common Mistakes

1. Using `read()` on huge files and running out of memory.
2. Forgetting `encoding="utf-8"` for text files.
3. Opening binary files in text mode.
4. Opening text files in binary mode when strings are needed.
5. Using `w` when you meant `a`.
6. Overwriting data accidentally with `w`.
7. Forgetting to close files.
8. Not using `with`.
9. Calling `writelines()` without newline characters.
10. Assuming `readline()` removes newline characters.
11. Using `readlines()` for giant files.
12. Ignoring decode errors from mismatched encodings.
13. Treating `tell()` as a stable byte offset in text mode.
14. Seeking arbitrarily in text mode without understanding restrictions.
15. Writing strings to binary files.
16. Writing bytes to text files.
17. Parsing CSV by splitting on commas manually.
18. Forgetting `newline=""` with `csv`.
19. Forgetting CSV quoting rules.
20. Assuming JSON supports comments.
21. Assuming JSON supports tuples, sets, or bytes directly.
22. Loading JSON and skipping validation.
23. Deserializing untrusted pickle data.
24. Hard-coding file paths across environments.
25. Ignoring permissions and OS-level failures.
26. Reading entire API responses into memory when streaming would do.
27. Assuming append mode can safely replace records in place.
28. Reusing a file object after closing it.
29. Ignoring flush behavior in buffered writes.
30. Using the wrong delimiter when reading CSV exports.

### Wrong vs correct examples

| Mistake | Wrong | Correct |
| :-- | :-- | :-- |
| CSV writing | `f.write(",".join(row))` | `csv.writer(f).writerow(row)` |
| Text encoding | `open("file.txt")` | `open("file.txt", encoding="utf-8")` |
| Large file read | `f.read()` | `for line in f:` |
| JSON parse | `eval(text)` | `json.loads(text)` |
| Binary copy | text mode open | `rb` / `wb` |


***

# Performance Tips

Use the smallest data access pattern that solves the task. If you only need line-level processing, do not read the entire file into memory.[^1_1]

## Practical tips

- Iterate over files instead of loading all lines.
- Read or write in chunks for large files.
- Use buffering, not unbuffered I/O, for normal disk files.
- Prefer `csv`/`json` libraries over manual string parsing.
- Avoid frequent disk writes in loops when batching is possible.


## Chunked copy

```python
def copy_file(src_path, dst_path, chunk_size=1024 * 1024):
    with open(src_path, "rb") as src, open(dst_path, "wb") as dst:
        while True:
            chunk = src.read(chunk_size)
            if not chunk:
                break
            dst.write(chunk)
```


## Streaming CSV or JSON

For CSV, process one row at a time. For JSON, stream the input if possible or process the parsed object incrementally after loading. For deeply nested or very large JSON, consider a streaming parser library when appropriate.

***

# Security Considerations

File handling can expose your application to data corruption, data leakage, and unsafe code execution if you are not careful. Python’s docs explicitly warn that pickle can execute arbitrary code when loading untrusted data.[^1_1]

## Key risks

- Path traversal from untrusted filenames.
- Overwriting important data with careless write mode.
- Corruption from opening binary data as text.
- Untrusted JSON shape or size.
- Unsafe pickle deserialization.


## Defensive habits

- Whitelist directories.
- Validate file extensions and content type.
- Use temporary files and atomic renames for critical writes.
- Limit input size.
- Reject malformed encodings.
- Never trust external data without validation.

***

# Real-World Projects

## 1. Word Counter

Build a utility that reads a text file, normalizes text, removes punctuation, and counts words. Extend it with top-N output and stop-word filtering.

### Core idea

```python
from collections import Counter
import re

with open("book.txt", "r", encoding="utf-8") as f:
    words = re.findall(r"\b\w+\b", f.read().lower())
counts = Counter(words)
```


## 2. Log Analyzer

Read log files line by line, extract timestamps and severity levels, and produce a summary. Add filters for date ranges, error counts, or top endpoints.

## 3. Search Utility

Implement a mini `grep` that searches multiple files for a keyword. Return matching line numbers and filenames.

## 4. Configuration System

Store app settings in JSON, load them on startup, validate required fields, and allow overrides from environment variables.

## 5. Data Export Tool

Export records to CSV for Excel users and to JSON for API consumers. Support both flat and nested output structures.

***

# Interview Preparation

## Beginner Questions

1. What is file handling in Python?
2. What is the difference between text and binary files?
3. What does `open()` do?
4. What is the default mode of `open()`?
5. What is the purpose of `with open(...)`?
6. What does `read()` return?
7. What does `write()` return?
8. What is the difference between `readline()` and `readlines()`?
9. What is `seek()` used for?
10. What is `tell()` used for?
11. What is append mode?
12. What happens in write mode if the file exists?
13. Why should we specify encoding?
14. What is UTF-8?
15. What is buffering?
16. Why do we close files?
17. What is a file cursor?
18. What is a file descriptor?
19. What is a stream?
20. Why is `with` preferred?

### Beginner answers

1. File handling is reading, writing, updating, and closing files stored on disk.
2. Text files store characters; binary files store raw bytes.
3. `open()` creates a file object.
4. The default mode is `r`.
5. It automatically closes the file.
6. `read()` returns the file content as a string or bytes.
7. `write()` returns the number of characters or bytes written.
8. `readline()` reads one line; `readlines()` returns all lines in a list.
9. `seek()` moves the file pointer.
10. `tell()` returns the current file position.
11. Append mode adds new data at the end.
12. It truncates the file.
13. To avoid decoding errors and improve portability.
14. UTF-8 is a Unicode encoding widely used for text.
15. Buffering batches I/O for efficiency.
16. To release resources and flush data.
17. It is the current read/write position in the file.
18. It is the OS-level file handle.
19. A stream is a file-like flow of data.
20. Because it ensures cleanup automatically.

## Intermediate Questions

1. Difference between `w`, `a`, and `x`.
2. Why is `newline=""` used with CSV?
3. Why is text mode bad for binary files?
4. How does `DictReader` help?
5. What is the benefit of `DictWriter`?
6. Difference between `load()` and `loads()`.
7. Difference between `dump()` and `dumps()`.
8. What is serialization?
9. What is deserialization?
10. Why is `readlines()` often discouraged?
11. How do you process large files efficiently?
12. What is the role of buffering?
13. What is the difference between `bytes` and `bytearray`?
14. Why use `struct`?
15. What does `seek(0, 2)` mean?
16. What happens if you write a string to a binary stream?
17. Why is `json` preferred over `pickle` for exchange?
18. How do you append to a file safely?
19. How do you detect file existence before creating?
20. What are common causes of `FileNotFoundError`?

### Intermediate answers

1. `w` overwrites, `a` appends, `x` creates exclusively.
2. To prevent newline translation conflicts with the CSV module.
3. Because text mode may decode or transform bytes.
4. It returns each row as a dictionary keyed by headers.
5. It writes dictionaries using column names.
6. `load()` reads from file; `loads()` reads from string.
7. `dump()` writes to file; `dumps()` returns string.
8. Serialization converts objects into storable/transmittable form.
9. Deserialization reconstructs the object.
10. It loads everything into memory.
11. Use iteration or chunked reads.
12. It reduces system calls and improves throughput.
13. `bytes` is immutable; `bytearray` is mutable.
14. To pack/unpack compact binary records.
15. It seeks to the end of the file.
16. Python raises a type-related error.
17. It is portable and safer for untrusted data.
18. Use `a` mode or explicit file locking and atomic updates.
19. Use `os.path.exists()` or handle `FileExistsError` for `x`.
20. Missing path, wrong directory, or bad filename.

## Advanced Questions

1. What is the internal layering of Python I/O?
2. Why does text I/O return an opaque position from `tell()`?
3. How does buffered I/O improve performance?
4. When would you disable buffering?
5. How does newline translation work?
6. Why is UTF-8 recommended for text files?
7. What are the risks of using `eval()` on text data?
8. How do you implement atomic file replacement?
9. How do you safely process untrusted JSON?
10. How would you design a high-volume log processor?
11. What is the difference between raw, buffered, and text streams?
12. How do you handle partial writes in low-level I/O?
13. Why can `seek()` behavior differ in text mode?
14. How do you store complex Python objects in a file safely?
15. What is the best way to copy a huge binary file?
16. How can you detect corrupted CSV input?
17. How do you validate JSON structure?
18. How should a backend app store config data?
19. How can you make file handling code testable?
20. What file practices matter most in production?

### Advanced answers

1. The stack is raw I/O, buffered I/O, then text I/O.
2. Because text decoding makes byte offsets less directly meaningful.
3. It batches small operations into larger OS interactions.
4. Rarely, mostly for specialized low-level scenarios.
5. Python may convert system-specific line endings in text mode.
6. It is cross-platform, Unicode-complete, and standard for modern tools.
7. It can execute arbitrary code or produce unintended behavior.
8. Write to a temp file, flush, then rename atomically.
9. Parse with `json`, validate keys and types, and limit size.
10. Stream line by line, aggregate incrementally, and avoid full-file loads.
11. Raw I/O is low-level bytes; buffered I/O adds batching; text I/O adds encoding.
12. Check return values and loop until all data is written.
13. Encoding and newline handling make positions opaque.
14. Prefer JSON, CSV, or a safe custom format; avoid pickle for untrusted data.
15. Copy in chunks in binary mode.
16. Validate headers, field counts, types, quoting, and delimiters.
17. Parse JSON, then validate schema or required fields.
18. Use JSON or environment-based config, not ad hoc text formats.
19. Inject paths, use temp directories, and mock file access.
20. Correct modes, explicit encoding, safe cleanup, logging, and validation.

***

# Exercises

## Beginner

### Exercise 1

Read a text file and print the first 5 lines.

**Solution:**

```python
with open("input.txt", "r", encoding="utf-8") as f:
    for _ in range(5):
        line = f.readline()
        if not line:
            break
        print(line, end="")
```


### Exercise 2

Write a list of names to a file, one per line.

**Solution:**

```python
names = ["Asha", "Ravi", "Meera"]

with open("names.txt", "w", encoding="utf-8") as f:
    for name in names:
        f.write(name + "\n")
```


### Exercise 3

Append a timestamped message to a log file.

**Solution:**

```python
from datetime import datetime

msg = f"{datetime.now().isoformat()} Event occurred\n"
with open("app.log", "a", encoding="utf-8") as f:
    f.write(msg)
```


## Intermediate

### Exercise 1

Count the number of words in a file.

**Solution:**

```python
import re

count = 0
with open("text.txt", "r", encoding="utf-8") as f:
    for line in f:
        count += len(re.findall(r"\b\w+\b", line))
print(count)
```


### Exercise 2

Read a CSV file and compute the sum of a numeric column.

**Solution:**

```python
import csv

total = 0
with open("sales.csv", "r", newline="", encoding="utf-8") as f:
    reader = csv.DictReader(f)
    for row in reader:
        total += float(row["amount"])
print(total)
```


### Exercise 3

Read JSON from a file and print all top-level keys.

**Solution:**

```python
import json

with open("config.json", "r", encoding="utf-8") as f:
    data = json.load(f)
print(list(data.keys()))
```


## Advanced

### Exercise 1

Implement a chunked binary file copy.

**Solution:**

```python
def copy_large_file(src, dst, chunk_size=1024 * 1024):
    with open(src, "rb") as fsrc, open(dst, "wb") as fdst:
        while True:
            chunk = fsrc.read(chunk_size)
            if not chunk:
                break
            fdst.write(chunk)
```


### Exercise 2

Convert CSV rows to JSON records.

**Solution:**

```python
import csv
import json

with open("input.csv", "r", newline="", encoding="utf-8") as f:
    rows = list(csv.DictReader(f))

with open("output.json", "w", encoding="utf-8") as f:
    json.dump(rows, f, indent=2)
```


### Exercise 3

Write a safe JSON loader with validation.

**Solution:**

```python
import json

def load_config(path):
    with open(path, "r", encoding="utf-8") as f:
        data = json.load(f)
    if not isinstance(data, dict):
        raise ValueError("Config must be a JSON object")
    if "name" not in data:
        raise ValueError("Missing name")
    return data
```


***

# Cheat Sheet

## File mode table

| Mode | Read | Write | Create | Truncate | Append |
| :-- | --: | --: | --: | --: | --: |
| `r` | Yes | No | No | No | No |
| `w` | No | Yes | Yes | Yes | No |
| `a` | No | Yes | Yes | No | Yes |
| `x` | No | Yes | Yes | No | No |
| `r+` | Yes | Yes | No | No | No |
| `w+` | Yes | Yes | Yes | Yes | No |
| `a+` | Yes | Yes | Yes | No | Yes |
| `rb` | Yes | No | No | No | No |
| `wb` | No | Yes | Yes | Yes | No |
| `ab` | No | Yes | Yes | No | Yes |

## File method table

| Method | Purpose |
| :-- | :-- |
| `read()` | Read all or part of a file |
| `readline()` | Read one line |
| `readlines()` | Read all lines into a list |
| `write()` | Write text or bytes |
| `writelines()` | Write multiple strings |
| `tell()` | Current position |
| `seek()` | Move position |
| `close()` | Close file |

## CSV quick notes

- Use `csv.reader` for row lists.
- Use `csv.writer` for row lists.
- Use `csv.DictReader` for dictionary rows.
- Use `csv.DictWriter` for dictionary output.
- Use `newline=""` when opening CSV files.


## JSON quick notes

- Use `json.dump()` and `json.load()` for files.
- Use `json.dumps()` and `json.loads()` for strings.
- JSON is text, usually UTF-8.
- JSON supports objects, arrays, strings, numbers, booleans, and null.
- Convert unsupported Python objects before serialization.


## Binary quick notes

- Use `rb` and `wb` for binary work.
- Use `bytes` for immutable byte sequences.
- Use `bytearray` for mutable byte buffers.
- Use `struct` for packed records and headers.
- Never open images or PDFs in text mode.

***

# Summary

Python file handling is built around streams, buffering, and clear separation between text and binary data. Once you understand `open()`, file modes, cursor movement, closing patterns, and the standard `csv` and `json` modules, you can handle most real-world file workflows confidently.[^1_5][^1_7][^1_1]

The safest production habits are simple: use `with`, specify encodings, process large files incrementally, validate input, and avoid unsafe deserialization. Those habits prevent silent data corruption and make your code more portable, efficient, and secure.[^1_1]
<span style="display:none">[^1_10][^1_11][^1_12][^1_13][^1_14][^1_15][^1_16][^1_17][^1_18][^1_19][^1_20][^1_8][^1_9]</span>

<div align="center">⁂</div>

[^1_1]: https://coodeverse.com/reading-materials/python/python-filehandling.html

[^1_2]: https://docs.python.org/3/library/io.html

[^1_3]: https://www.geeksforgeeks.org/struct-module-python/

[^1_4]: https://docs.python.org/3/library/struct.html

[^1_5]: https://docs.python.org/3/library/csv.html

[^1_6]: https://python-adv-web-apps.readthedocs.io/en/latest/csv.html

[^1_7]: https://docs.python.org/3/library/json.html

[^1_8]: https://medium.com/@j.dodgson/mastering-file-handling-in-python-a-beginners-guide-d0f3f5d9b00a

[^1_9]: https://www.pythonmorsels.com/how-does-file-buffering-work/

[^1_10]: https://stackoverflow.com/questions/19697846/how-to-convert-csv-file-to-multiline-json

[^1_11]: https://www.geeksforgeeks.org/python/file-handling-python/

[^1_12]: https://github.com/wdk-docs/python-documentation/blob/master/docs/library/io.rst

[^1_13]: https://realpython.com/python-json/

[^1_14]: https://www.youtube.com/watch?v=4QHk-y_EQng

[^1_15]: https://www.geeksforgeeks.org/python/python-open-function/

[^1_16]: https://www.w3schools.com/python/python_file_handling.asp

[^1_17]: https://medium.com/@3valuedlogic/using-python-csv-3-dictreader-e4814ce2e44

[^1_18]: https://docs.micropython.org/en/latest/library/struct.html

[^1_19]: https://python-reference.readthedocs.io/en/latest/docs/functions/open.html

[^1_20]: https://docs.python.org/3/tutorial/inputoutput.html

