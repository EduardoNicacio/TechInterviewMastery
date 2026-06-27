# Python – Interview Questions & Answers

**Question**: What is the Global Interpreter Lock (GIL) in Python and how does it impact multithreading?

**Answer**: The GIL is a mutex that protects CPython's internal state, allowing only one thread to execute Python bytecode at a time. This means CPU-bound multithreaded programs see no performance gain (and may even regress) because threads contend for the same lock. I/O-bound programs are less affected because the GIL is released during blocking I/O operations.

---

**Question**: How can you work around the GIL to achieve true parallelism in Python?

**Answer**: Use the `multiprocessing` module to spawn separate processes, each with its own GIL, enabling CPU-bound parallelism. For I/O-bound workloads, `asyncio` provides cooperative concurrency without threads. C extensions (e.g., via Cython or writing a C module) can release the GIL explicitly using `Py_BEGIN_ALLOW_THREADS` / `Py_END_ALLOW_THREADS`.

---

**Question**: What are decorators in Python and how do you write a simple function decorator?

**Answer**: A decorator is a callable that takes a function (or class) and returns a modified version of it, typically extending its behavior. The `@decorator` syntax is syntactic sugar for `func = decorator(func)`.

```python
def log_calls(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@log_calls
def greet(name):
    return f"Hello, {name}"
```

---

**Question**: What is a class decorator and how is it different from a function decorator?

**Answer**: A class decorator takes a class and returns a modified class (or a callable that replaces it). Unlike function decorators, which wrap individual callables, class decorators can modify the entire class' attributes, methods, or behavior at definition time.

```python
def add_repr(cls):
    cls.__repr__ = lambda self: f"{cls.__name__}({self.__dict__})"
    return cls

@add_repr
class Point:
    def __init__(self, x, y):
        self.x, self.y = x, y
```

---

**Question**: How do generators work in Python and what is the `yield` keyword?

**Answer**: A generator is a function that uses `yield` instead of `return`, producing a sequence of values lazily. Each call to `next()` resumes execution from the last `yield`, making generators memory-efficient for large or infinite sequences.

```python
def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b
```

---

**Question**: What does `yield from` do and when would you use it?

**Answer**: `yield from` delegates part of a generator's iteration to another iterable or generator, flattening nested generator code. It replaces the boilerplate of manually iterating and yielding from a sub-generator.

```python
def flatten(nested):
    for item in nested:
        if isinstance(item, (list, tuple)):
            yield from flatten(item)
        else:
            yield item
```

---

**Question**: How do you implement a context manager using a class (`__enter__` / `__exit__`)?

**Answer**: A class with `__enter__` (acquires resource, returns it) and `__exit__` (releases resource, handles exceptions) enables usage with the `with` statement. The `__exit__` method receives exception type, value, and traceback; returning `True` suppresses exceptions.

```python
class ManagedFile:
    def __init__(self, path):
        self.path = path
    def __enter__(self):
        self.file = open(self.path, 'w')
        return self.file
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.file.close()
```

---

**Question**: What is `@contextmanager` from the `contextlib` module?

**Answer**: It turns a generator function into a context manager using `yield` to split `__enter__` (code before `yield`) from `__exit__` (code after `yield`). This avoids writing a full class when a simple resource lifecycle is needed.

```python
from contextlib import contextmanager

@contextmanager
def managed_file(path):
    f = open(path, 'w')
    try:
        yield f
    finally:
        f.close()
```

---

**Question**: Explain asyncio's event loop, coroutines, tasks, and futures.

**Answer**: The event loop is the core scheduler that runs and manages asynchronous tasks. Coroutines (declared with `async def`) are awaitable functions. A `Task` wraps a coroutine for scheduling on the loop. A `Future` is a low-level awaitable representing a result that will be available later; `create_task()` returns a `Task` (a subclass of `Future`).

```python
import asyncio

async def fetch_data():
    await asyncio.sleep(1)
    return "data"

async def main():
    task = asyncio.create_task(fetch_data())
    result = await task
    print(result)

asyncio.run(main())
```

---

**Question**: How does `async`/`await` differ from `yield from` in asynchronous programming?

**Answer**: `async`/`await` is designed for asynchronous programming with coroutines, providing clearer semantics and better error messages. `yield from` is a generator delegation tool that predates async/await; it can be used to build simple coroutines but lacks the dedicated event loop integration and debugging support of native coroutines.

---

**Question**: What are `__slots__` and how do they improve memory usage?

**Answer**: `__slots__` declares a fixed set of allowed attributes, preventing the creation of a per-instance `__dict__`. This significantly reduces memory overhead for classes with many instances (e.g., data records) because attribute storage becomes a compact tuple-like structure.

```python
class Point:
    __slots__ = ('x', 'y')
    def __init__(self, x, y):
        self.x, self.y = x, y
```

---

**Question**: What is the difference between mutable and immutable objects in Python?

**Answer**: Mutable objects (list, dict, set) can be modified in-place after creation. Immutable objects (int, str, tuple, frozenset) cannot be changed; any "modification" creates a new object. This distinction affects hashing (immutable types can be dictionary keys), aliasing behavior, and thread safety.

---

**Question**: What is the difference between `is` and `==` in Python?

**Answer**: `is` checks identity (whether two references point to the same object in memory), while `==` checks equality (whether the objects have the same value). Use `is` for comparing to `None`, `True`, or `False`; use `==` for value comparison.

```python
a = [1, 2, 3]
b = [1, 2, 3]
print(a == b)  # True
print(a is b)  # False
```

---

**Question**: What is the difference between shallow copy and deep copy?

**Answer**: A shallow copy creates a new object but inserts references to the original elements, so nested mutable objects are shared. A deep copy recursively duplicates all objects, creating fully independent copies. Use `copy.copy()` for shallow and `copy.deepcopy()` for deep.

```python
import copy
original = [[1, 2], [3, 4]]
shallow = copy.copy(original)
deep = copy.deepcopy(original)
shallow[0][0] = 99  # affects original
deep[0][0] = 99     # does not affect original
```

---

**Question**: What are `*args` and `**kwargs` used for?

**Answer**: `*args` captures extra positional arguments as a tuple, while `**kwargs` captures extra keyword arguments as a dict. They are commonly used in decorators, function wrappers, and when delegating arguments to another function.

```python
def log_call(func):
    def wrapper(*args, **kwargs):
        print(f"Calling with {args}, {kwargs}")
        return func(*args, **kwargs)
    return wrapper
```

---

**Question**: What is a closure and what does the `nonlocal` keyword do?

**Answer**: A closure is a nested function that remembers variables from its enclosing scope even after that scope has exited. The `nonlocal` keyword allows assignment to variables in the enclosing (non-global) scope, enabling stateful closures like counters or memoizers.

```python
def make_counter():
    count = 0
    def counter():
        nonlocal count
        count += 1
        return count
    return counter
```

---

**Question**: How do you write a decorator that accepts its own arguments?

**Answer**: Create a three-level nested function: the outermost accepts the decorator arguments, the middle accepts the function, and the innermost (wrapper) accepts the call arguments. This allows parameterized decorators like `@repeat(n=3)`.

```python
def repeat(n):
    def decorator(func):
        def wrapper(*args, **kwargs):
            result = None
            for _ in range(n):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(n=3)
def say_hello():
    print("Hello!")
```

---

**Question**: What are metaclasses and when should you use them?

**Answer**: A metaclass is the class of a class, defining how classes are constructed (default: `type`). Metaclasses intercept class creation to modify attributes, register classes, or enforce conventions. Use them sparingly - frameworks (Django, SQLAlchemy) use them for ORM models, but simpler alternatives like decorators or class inheritance usually suffice.

```python
class SingletonMeta(type):
    _instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]
```

---

**Question**: What are descriptors and how do `__get__`, `__set__`, and `__delete__` work?

**Answer**: Descriptors are objects that define `__get__`, `__set__`, or `__delete__` to override attribute access on another class. They power `@property`, `@staticmethod`, `@classmethod`, and `__slots__`. When an attribute is accessed, Python checks if the attribute object is a descriptor and calls the appropriate method.

```python
class PositiveNumber:
    def __set_name__(self, owner, name):
        self.name = f"_{name}"
    def __get__(self, obj, objtype=None):
        return getattr(obj, self.name)
    def __set__(self, obj, value):
        if value <= 0:
            raise ValueError("Must be positive")
        setattr(obj, self.name, value)
```

---

**Question**: What are Abstract Base Classes (ABCs) and how do you use them?

**Answer**: ABCs define an interface that subclasses must implement, enforced by the `@abstractmethod` decorator. They prevent instantiation and raise `TypeError` if not all abstract methods are implemented. The `abc` module provides `ABC` as the base class and `abstractmethod` as the decorator.

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self): ...

class Circle(Shape):
    def __init__(self, r):
        self.r = r
    def area(self):
        return 3.14 * self.r ** 2
```

---

**Question**: What is the difference between `@staticmethod`, `@classmethod`, and regular methods?

**Answer**: A regular method receives `self` (instance) as the first argument. A `@classmethod` receives `cls` (class) instead, allowing access to class-level state and factory patterns. A `@staticmethod` receives neither, behaving like a plain function namespaced under the class.

```python
class MyClass:
    def instance_method(self): return "instance"
    @classmethod
    def class_method(cls): return "class"
    @staticmethod
    def static_method(): return "static"
```

---

**Question**: What is the difference between `__new__` and `__init__`?

**Answer**: `__new__` is a static method that creates and returns a new instance (called before `__init__`). `__init__` initializes the already-created instance. Override `__new__` for immutable types (e.g., tuples, singletons) or custom instance creation; override `__init__` for standard initialization.

```python
class Singleton:
    _instance = None
    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
```

---

**Question**: How does Python's MRO (Method Resolution Order) work?

**Answer**: Python uses C3 Linearization to determine the order in which base classes are searched for attribute resolution. The MRO ensures that a class always appears before its parents and that the order is monotonic (consistent across the hierarchy). Access `ClassName.__mro__` to inspect it.

```python
class A: pass
class B(A): pass
class C(A): pass
class D(B, C): pass
print(D.__mro__)
# (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)
```

---

**Question**: How does Python resolve the diamond problem in multiple inheritance?

**Answer**: Python's C3 Linearization ensures each class is visited once, in a consistent order that respects the local precedence list and monotonicity. The `super()` function follows the MRO, so each class's `super()` call delegates to the next class in the chain, avoiding duplicate calls.

```python
class A: pass
class B(A): pass
class C(A): pass
class D(B, C): pass
# D -> B -> C -> A -> object
```

---

**Question**: What is duck typing and how do EAFP and LBYL differ?

**Answer**: Duck typing means "if it walks like a duck and quacks like a duck, it's a duck" - an object's suitability is determined by its methods, not its type. EAFP (Easier to Ask Forgiveness than Permission) attempts an operation and catches exceptions, while LBYL (Look Before You Leap) checks preconditions first. Python favors EAFP.

```python
# EAFP
try:
    obj.quack()
except AttributeError:
    ...

# LBYL
if hasattr(obj, 'quack'):
    obj.quack()
```

---

**Question**: What is the difference between `__str__` and `__repr__`?

**Answer**: `__repr__` should return an unambiguous representation of the object, ideally one that can recreate it (evaluable by `eval()`). `__str__` returns a human-readable, informal string. `__repr__` is used by `repr()` and in the REPL fallback; `__str__` is used by `print()` and `str()`.

```python
class Point:
    def __init__(self, x, y):
        self.x, self.y = x, y
    def __repr__(self):
        return f"Point({self.x}, {self.y})"
    def __str__(self):
        return f"({self.x}, {self.y})"
```

---

**Question**: What is the difference between `__getattr__` and `__getattribute__`?

**Answer**: `__getattribute__` is called unconditionally for every attribute access. `__getattr__` is only invoked when normal attribute lookup fails (i.e., `__getattribute__` raises `AttributeError`). Override `__getattribute__` with caution due to recursion risks; `__getattr__` is safer for fallback behavior.

```python
class DefaultDict:
    def __init__(self, default):
        self._default = default
    def __getattr__(self, name):
        return self._default
```

---

**Question**: How do properties work with the `@property` decorator?

**Answer**: `@property` turns a method into a read-only attribute accessor. Combined with `@name.setter` and `@name.deleter`, it provides computed attributes with controlled get/set/delete behavior, while maintaining attribute-access syntax.

```python
class Circle:
    def __init__(self, radius):
        self._radius = radius
    @property
    def area(self):
        return 3.14 * self._radius ** 2
    @property
    def radius(self):
        return self._radius
    @radius.setter
    def radius(self, value):
        if value < 0:
            raise ValueError("Radius cannot be negative")
        self._radius = value
```

---

**Question**: What is pickling and how does `__reduce__` customize it?

**Answer**: Pickling serializes Python objects to a byte stream for storage or transmission. `__reduce__` returns a tuple describing how to reconstruct the object, allowing custom pickling behavior for objects with non-standard state (e.g., file handles, network connections).

```python
import pickle

class SafePickle:
    def __init__(self, data):
        self.data = data
    def __reduce__(self):
        return (self.__class__, (self.data,))
```

---

**Question**: What is the difference between a list comprehension and a generator expression?

**Answer**: A list comprehension (`[x for x in items]`) eagerly builds the entire list in memory. A generator expression (`(x for x in items)`) returns a lazy iterator that yields items one at a time, consuming far less memory for large sequences.

```python
squares_list = [x**2 for x in range(1000)]       # list in memory
squares_gen  = (x**2 for x in range(1000))       # lazy iterator
```

---

**Question**: What are `zip()`, `enumerate()`, `map()`, `filter()`, and `reduce()` used for?

**Answer**: `zip()` aggregates elements from multiple iterables into tuples. `enumerate()` returns index-value pairs. `map()` applies a function to every item. `filter()` selects items for which a function returns `True`. `reduce()` (from `functools`) cumulatively applies a function to items, reducing to a single value.

```python
names = ["Alice", "Bob", "Charlie"]
scores = [85, 92, 78]
pairs = list(zip(names, scores))         # [("Alice", 85), ...]
for i, name in enumerate(names):         # (0, "Alice"), (1, "Bob"), ...
    pass
squared = list(map(lambda x: x**2, [1, 2, 3]))
evens = list(filter(lambda x: x % 2 == 0, [1, 2, 3, 4]))
from functools import reduce
total = reduce(lambda a, b: a + b, [1, 2, 3, 4])  # 10
```

---

**Question**: What are the most useful functions in the `itertools` module?

**Answer**: `itertools` provides iterator-building tools: `chain` (flatten iterables), `cycle` (infinite repeat), `count` (infinite arithmetic progression), `islice` (slice an iterator), `product` (Cartesian product), `permutations`, `combinations`, `groupby` (consecutive key groups), and `zip_longest` (zip with fill value).

```python
from itertools import chain, count, groupby, islice
chained = list(chain([1, 2], [3, 4]))       # [1, 2, 3, 4]
first_5 = list(islice(count(10, 2), 5))     # [10, 12, 14, 16, 18]
```

---

**Question**: What key utilities does the `functools` module provide?

**Answer**: `lru_cache` memoizes function results with a least-recently-used eviction policy. `partial` freezes some arguments of a function, creating a new callable with fewer parameters. `wraps` copies metadata (name, docstring) from the original function to a decorator's wrapper.

```python
from functools import lru_cache, partial, wraps

@lru_cache(maxsize=128)
def fib(n):
    return n if n < 2 else fib(n-1) + fib(n-2)

def power(base, exp): return base ** exp
square = partial(power, exp=2)
```

---

**Question**: What are the `namedtuple`, `defaultdict`, `Counter`, and `deque` classes from `collections`?

**Answer**: `namedtuple` creates lightweight, immutable data classes. `defaultdict` supplies a default value for missing keys. `Counter` counts hashable items. `deque` is a double-ended queue with O(1) appends/pops from either end, ideal for queues and sliding windows.

```python
from collections import namedtuple, defaultdict, Counter, deque

Point = namedtuple("Point", ["x", "y"])
p = Point(1, 2)

dd = defaultdict(int)
dd["key"] += 1

cnt = Counter("abracadabra")
d = deque([1, 2, 3])
d.appendleft(0)
```

---

**Question**: How do threading, multiprocessing, and asyncio compare in Python?

**Answer**: Threading provides concurrent I/O (GIL-bound for CPU work). Multiprocessing spawns separate processes for true CPU parallelism but has higher overhead and IPC complexity. Asyncio offers cooperative single-threaded concurrency ideal for high-I/O, low-latency workloads with minimal overhead.

---

**Question**: How do you ensure thread safety with `threading.Lock` and `queue.Queue`?

**Answer**: `threading.Lock` provides exclusive access to a shared resource via `acquire()` / `release()` (or the `with` statement for automatic release). `queue.Queue` is a thread-safe FIFO queue that handles locking internally, making it the preferred way to exchange data between threads.

```python
import threading
from queue import Queue

lock = threading.Lock()
shared_data = []

def safe_append(item):
    with lock:
        shared_data.append(item)

q = Queue()
q.put("item")
item = q.get()
```

---

**Question**: What are `ThreadPoolExecutor` and `ProcessPoolExecutor` in `concurrent.futures`?

**Answer**: They provide a high-level interface for asynchronously executing callables using a pool of threads or processes. `ThreadPoolExecutor` is for I/O-bound tasks, `ProcessPoolExecutor` for CPU-bound tasks. Both use `submit()` to schedule work and return `Future` objects.

```python
from concurrent.futures import ThreadPoolExecutor

def fetch(url):
    return f"data from {url}"

with ThreadPoolExecutor(max_workers=4) as ex:
    futures = [ex.submit(fetch, f"url{i}") for i in range(10)]
    results = [f.result() for f in futures]
```

---

**Question**: What are Python logging best practices?

**Answer**: Use module-level loggers with `logging.getLogger(__name__)` for proper namespace tracking. Configure handlers (file, console) and formatters at the application entry point, not in libraries. Avoid `print()`, use structured logging with appropriate levels (DEBUG, INFO, WARNING, ERROR, CRITICAL).

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s"
)
logger = logging.getLogger(__name__)
logger.info("Application started")
```

---

**Question**: What does the `if __name__ == '__main__'` idiom do?

**Answer**: It ensures that code inside the block runs only when the script is executed directly, not when imported as a module. This allows the same file to serve both as a reusable module and as a standalone script, preventing side effects on import.

```python
def main():
    print("Running as script")

if __name__ == '__main__':
    main()
```

---

**Question**: How do type hints and the `typing` module improve Python code?

**Answer**: Type hints document expected argument and return types, enabling static analysis with tools like `mypy`. The `typing` module provides `List`, `Dict`, `Optional`, `Union`, `Callable`, `TypeVar`, and `Generic` for expressing complex types without sacrificing readability.

```python
from typing import List, Optional

def find_user(user_id: int) -> Optional[str]:
    users: List[str] = ["Alice", "Bob"]
    return users[user_id] if user_id < len(users) else None
```

---

**Question**: What is `@dataclass` and how does it simplify class definitions?

**Answer**: `@dataclass` auto-generates `__init__`, `__repr__`, and `__eq__` by default. It does not auto-generate `__hash__` - in fact it explicitly makes instances unhashable when `eq=True` (default) and `frozen=False` (default). `__hash__` is only generated when `frozen=True` or `eq=False`. Use `frozen=True` for immutability (all fields become read-only), and `order=True` for comparison dunder methods (`__lt__`, `__le__`, etc.).

```python
from dataclasses import dataclass

@dataclass
class Point:
    x: float
    y: float
    label: str = ""

p = Point(1.0, 2.0, "origin")
```

---

**Question**: What is the `Enum` class in Python?

**Answer**: `Enum` creates a set of symbolic names bound to unique, constant values. Enums are iterable, comparable by identity, and support `auto()` for automatic value assignment. Use them instead of magic strings or integers for fixed sets of options.

```python
from enum import Enum, auto

class Color(Enum):
    RED = auto()
    GREEN = auto()
    BLUE = auto()

print(Color.RED.name)    # "RED"
print(Color.RED.value)   # 1
```

---

**Question**: How does `pathlib` compare to `os.path` for filesystem operations?

**Answer**: `pathlib` provides an object-oriented interface (`Path` objects) with methods like `.read_text()`, `.write_bytes()`, `.glob()`, and `/` for path joining. It's more readable and composable than `os.path` string manipulation, which requires multiple imports and nested calls.

```python
from pathlib import Path

path = Path("data") / "logs" / "app.log"
content = path.read_text()
for py_file in Path("src").glob("**/*.py"):
    print(py_file.stat().st_size)
```

---

**Question**: What is the difference between `@dataclass` and `NamedTuple`?

**Answer**: Both create lightweight data containers with auto-generated methods. `NamedTuple` is immutable and tuple-like (indexable, unpackable, hashable by default). `@dataclass` is more flexible: it supports mutable fields, default factories, `__slots__`, and inheritance. Use `NamedTuple` for simple immutable records and `@dataclass` for more complex data objects.

---

**Question**: What is the walrus operator (`:=`) and when would you use it?

**Answer**: The walrus operator assigns a value to a variable as part of an expression, allowing assignment within conditions, comprehensions, or `while` loops. It reduces duplication where a value needs to be both tested and used.

```python
if (n := len(items)) > 10:
    print(f"Too many items: {n}")

results = [y for x in data if (y := process(x)) is not None]
```

---

**Question**: How does Python's `match` statement (structural pattern matching) work?

**Answer**: Introduced in Python 3.10, `match` compares a subject against patterns (literals, sequences, mappings, classes, guards) with destructuring. It's more expressive than chains of `if`/`elif`/`else`, especially for complex data structures.

```python
def handle(value):
    match value:
        case 0:
            print("Zero")
        case [x, y] if x == y:
            print("Equal pair")
        case {"name": name, "age": age}:
            print(f"{name} is {age}")
        case _:
            print("Unknown")
```

---

**Question**: What are the dictionary merge operators `|` and `|=`?

**Answer**: `|` merges two dictionaries into a new one (left-right conflict resolved by the right dict). `|=` updates a dictionary in-place with keys from another dict or an iterable of key-value pairs. They provide a cleaner alternative to `{**d1, **d2}` and `d1.update(d2)`.

```python
d1 = {"a": 1, "b": 2}
d2 = {"b": 3, "c": 4}
merged = d1 | d2       # {"a": 1, "b": 3, "c": 4}
d1 |= d2               # in-place: d1 now {"a": 1, "b": 3, "c": 4}
```

---

**Question**: How do you handle timezones in Python using `zoneinfo`?

**Answer**: `zoneinfo` (Python 3.9+) provides IANA timezone database support via `ZoneInfo` objects. Use `datetime.now(ZoneInfo(...))` for the current time in a timezone, and `.astimezone(ZoneInfo(...))` for converting between timezones. Avoid `datetime.replace(tzinfo=...)` as it merely tags a timezone without conversion, which can produce incorrect results during DST transitions.

```python
from datetime import datetime
from zoneinfo import ZoneInfo

utc = datetime.now(ZoneInfo("UTC"))
ny = utc.astimezone(ZoneInfo("America/New_York"))
```

---

**Question**: How does Python's `try`/`except`/`else`/`finally` work?

**Answer**: `try` contains code that may raise an exception. `except` catches and handles specific exceptions. `else` runs if no exception occurred (useful for code that should not trigger the except handler). `finally` always executes, regardless of exceptions, typically for cleanup.

```python
try:
    result = risky_operation()
except ValueError as e:
    print(f"ValueError: {e}")
except (TypeError, RuntimeError):
    print("Type or Runtime error")
else:
    print(f"Success: {result}")
finally:
    cleanup()
```

---

**Question**: How do you profile Python code with `cProfile` and `timeit`?

**Answer**: `cProfile` records function call statistics (call count, cumulative time) to identify bottlenecks. `timeit` measures execution time of small code snippets with high precision, disabling garbage collection for consistent results.

```python
# cProfile usage
import cProfile
cProfile.run("my_function()", sort="cumtime")

# timeit usage
import timeit
time = timeit.timeit("sum(range(100))", number=10000)
print(f"Total: {time:.4f}s")
```
