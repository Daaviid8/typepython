PEP: XXXX
Title: A Metadata Method for Dictionaries
Author: Daaviid8 <davidcortes1999@gmail.com>
Status: Draft
Type: Standards Track
Created: 04-Jul-2025
Python-Version: 3.14

Abstract
========
This PEP proposes the addition of a new method, `metadata()`, to the built-in dict class.
This method would return a special-purpose object containing a summary of the dictionary's
contents, including a count of keys, a detailed breakdown of value types (including nested types),
and the shallow memory usage for each value. The primary goal is to provide a standardized,
efficient way for developers to inspect the structure and memory footprint of dictionaries.

Motivation
==========
In modern Python applications, especially in fields like data science and web development,
dictionaries are used to hold heterogeneous and complex data. Developers frequently need
to understand the structure of these dictionaries: What kind of data do they hold?
How much memory are the values consuming?

Currently, answering these questions requires writing custom, boilerplate code that iterates
through the dictionary's items. This is repetitive and lacks a standard output format.
A built-in method would offer:
- **Standardization**: A common, reliable way to get structural information.
- **Efficiency**: A C-level implementation will be significantly faster than an equivalent loop in Python.
- **Improved Developer Experience**: Simplifies common debugging and analysis tasks.
- **Educational Value**: Helps visualize the composition of dictionaries.

Rationale
=========
The design is guided by simplicity and utility. A method (`my_dict.metadata()`) is more idiomatic
than a standalone function. It will return a dedicated, immutable summary object rather than
printing to stdout, allowing the developer to use the data programmatically. To ensure the method
is always fast, it will only report the shallow memory size of values, a limitation that will be clearly documented.

Specification
=============
A new method will be added to the `dict` type: `dict.metadata()`.

The method will return an instance of a new type, `dict_metadata`. This object will be immutable
and have the following attributes:
- `key_count`: An integer representing the total number of keys.
- `value_type_hints`: A mapping where keys are the dictionary's keys and values are a string
  representation of their type hint (e.g., `List[Union[int, str]]`).
- `memory_usage`: A mapping where keys are the dictionary's keys and values are the shallow size
  in bytes of the corresponding value.

The `dict_metadata` object will have a custom `__repr__` that generates a human-readable table.

Reference Implementation
========================
```python
import sys
from typing import Any, Dict, List, Set, Tuple, Union

def _get_type_hint_str(value: Any) -> str:
    """Recursively generates a type hint string for a given value."""
    # Your existing implementation here...
    
class dict_metadata:
    def __init__(self, d: Dict[str, Any]):
        self.key_count = len(d)
        self.value_type_hints = {k: _get_type_hint_str(v) for k, v in d.items()}
        self.memory_usage = {k: sys.getsizeof(v) for k, v in d.items()}
        self._d = d

    def __repr__(self) -> str:
        # Simplified __repr__
        header = f"<dict_metadata: {self.key_count} keys>\n"
        lines = [f"'{k}': type={self.value_type_hints[k]}, size={self.memory_usage[k]} bytes" for k in self._d.keys()]
        return header + "\n".join(lines)

# Example Usage:
# my_dict = {"a": [1, "b", {2}], "b": (1, 2)}
# print(my_dict.metadata())
