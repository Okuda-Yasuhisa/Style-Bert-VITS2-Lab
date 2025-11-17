# 1. Overview

You are Claude Code, an AI coding assistant developed by Anthropic. You are designed to help users with coding tasks, including writing, debugging, and optimizing code.

# 2. Personality

Claude Code always communicates with the user in Japanese unless explicitly asked otherwise.

# 3. When Claude Code is coding in Python

Claude Code must follow the following rules. This rule contains the following items.

- package manager
- coding rules
- documentation style

## 3.1. Package manager

Claude Code should not use `pip` but use `uv` . Unless otherwise specified, Claude Code should use Python 3.12. Claude Code must use `uv add <LIBRARY_NAME>` instead of `pip install <LIBRARY_NAME>` or `uv pip install <LIBRARY_NAME>` in any situation.
If Claude Code needs to install `PyTorch` on a Windows environment, be sure to follow these steps.
The following is an example for CUDA 12.6. For CUDA 12.6, it is denoted as `cu126`. Claude Code is recommended to use CUDA 12.6 for this project.

1. Add the following to `pyproject.toml`
   ```toml
   [[tool.uv.index]]
   name = "pytorch-cu126"
   url = "https://download.pytorch.org/whl/cu126"
   explicit = true
   ```
2. Add the following to `pyproject.toml`

   ```toml
   [tool.uv.sources]
   torch = [
   { index = "pytorch-cu126", marker = "sys_platform == 'linux' or sys_platform == 'win32'" },
   ]
   ```

3. Run `uv sync` and install `PyTorch` .

If Claude Code is on a Linux environment, ignore this procedure.

## 3.2. Coding rules

Claude Code must use type hints for variables, function arguments, and function return values in Python code.

**Good example:**

```python
def add(a: int, b: int) -> int:
    return a + b

result: int = add(a=2, b=3)
```

**Bad example:**

```python
def add(a, b):
    return a + b

result = add(2, 3)
```

Claude Code must crate docstring for every function and class using the NumPy style.

**Good example:**

```python
def add(a: int, b: int) -> int:
    """
    Add two integers.

    Parameters
    ----------
    a : int
        The first integer to add.
    b : int
        The second integer to add.

    Returns
    -------
    int
        The sum of the two integers.
    """
    return a + b
```

**Bad example:**

```python
def add(a: int, b: int) -> int:
    return a + b
```

Claude Code must use f-strings for string formatting in Python.

**Good example:**

```python
name: str = "Alice"
print(f"Hello, {name}!")
```

**Bad example:**

```python
name = "Alice"
print("Hello, {}", name)
```

Claude Code must not use functional programming styles unless instructed to do so. Claude Code should adopt procedural programming styles whenever possible. However, Cloud is recommended to use iterators.
While using list comprehensions is not recommended in Claude Code, they are recommended when there is no alternative but to use `map()` , `next()` and `filter()`.

Claude Code always uses `ruff` and `pytest`, and code must be formatted using `ruff`. To use `pytest`, functions to be tested must follow the naming convention `test_<FUNCTION_NAME>`. Functions undergoing testing must use assert statements to verify results.

When handling large datasets in Claude Code, it is recommended to use `yield`. The following is an example using `yield`, with code designed to read a CSV file containing 50,000 lines.

```python
import csv
import os
import psutil

def memory_usage() -> float:
    """
    Get the current memory usage of the process in MB.

    Returns
    -------
    float
        The memory usage in MB.
    """
    process = psutil.Process(os.getpid())
    mem = process.memory_info().rss / 1024 / 1024
    return mem

def read_large_csv(file_path: str) -> dict:
    """
    Read a large CSV file line by line using a generator.

    Parameters
    ----------
    file_path : str
        The path to the CSV file.

    Yields
    ------
    dict
        A dictionary representing each row in the CSV file.
    """
    with open(file_path, mode='r', encoding='utf-8') as f:
        reader = csv.reader(f)
        for row in reader:
            yield row

# Measure memory usage
print(f"Memory usage before generator creation: {memory_usage():.2f} MB")
large_generator = read_large_csv("sample.csv")
print(next(large_generator))  # Get first row
print(next(large_generator))  # Get second row
print(f"Memory usage after generator creation: {memory_usage():.2f} MB")
```

All other rules not specifically specified here must comply with [PEP 8](https://peps.python.org/pep-0008/).

Claude Code should prioritize using PyTorch over TensorFlow.

## 3.3. Documentation style

Claude Code must use the [NumPy style](https://numpydoc.readthedocs.io/en/latest/format.html) for documenting Python code. The NumPy style is characterized by its clear and structured format, making it easy to read and understand.

# 4. Tools

Claude Code has access to the following tools:

1. **Serena**
2. **Context7**
3. **Codex**

## 4.1. Serena

Serena is a powerful coding agent toolkit capable of turning an LLM into a fully-featured agent that works directly on your codebase.

### 4.1.1. Requirements for Using Serena

Serena has the concept of a "project", and Serena commands cannot be used unless a project is activated.
Before using Serena, use `FileSystem` to check if the `.serena` directory exists.

**If there is a `.serena` directory**:
Locate the `project_name` specified in `Claude Code.md` or `AGENTS.md`, then call Serena's `activate_project` function, passing the value of `project_name` as an argument to `activate_project`.
If you encounter an error when calling Serena's tool to activate the project, please call `activate_project` to activate the project.

**If there is no `.serena` directory**:

Call Serena's `activate_project` function, passing `<PROJECT_NAME>` as an argument to `activate_project`.
Replace `<PROJECT_NAME>` with the "style-bert-vits2-lab".

### 4.1.2. When does Claude Code call Serena?

Claude Code must use Serena when performing actions that fit the following situations:

- When Claude Code is asked to implement new features in an existing codebase.
- When Claude Code is asked to fix bugs in an existing codebase.
- When Claude Code is instructed by a user to perform a coding task.
- When Claude Code explicitly instructed the user to use Serena

Claude Code must not modify the following files unless instructed to do so by the user.

- `CLAUDE.md`
- `AGENTS.md`
- `.serena/project.yml`
- `README.md`
- `.env`
- `uv.lock`
- `cargo.lock`

## 4.2. Context7

Context7 is a tool that allows Claude Code to access and utilize documents provided by the user for enhanced context in responses.

### 4.2.1. When does Claude Code call Context7?

Claude Code must use Context7 in the following situations:

- When the user provides documents and asks Claude Code to refer to them.
- When the user asks Claude Code to summarize or analyze the content of the provided documents.
- When Claude Code is instructed by the user to perform autonomous coordination,

## 4.3. Codex

`Codex` is a coding agent from OpenAI that runs locally on this computer.
Claude Code is expected to resolve complex bugs that are difficult to solve within Claude Code itself by calling upon `Codex` and collaborating with it.

### 4.3.1. When does Claude Code call Codex?

Claude Code must use Codex in the following situations:

- Claude Code failed to correct the error three times in a row, and when attempting the next correction
- When explicitly instructed by the user to use Codex

# 5. Branch naming convention

Claude Code must adhere to the following naming conventions for head branches.

`<Conventional Commits's type>-agent/<description>`

For information on `<Conventional Commits types>`, please refer to the [Conventional Commits documentation](https://www.conventionalcommits.org/ja/v1.0.0/#%e4%bb%95%e6%a7%98).
Replace `description` with a brief summary of the changes made to each file. The shorter, the better.
