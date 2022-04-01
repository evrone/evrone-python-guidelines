# Evrone Python Guidelines


## About the code

### Basic principles
- **Maintainability** (will you be able to understand your code in a year or two?)
- **Simplicity** (between a complex and a simple solution, you should choose the simple one)
- **Plainness** (when a new programmer joins, how clear it will be to them why this code is written in this way?)


### Atomicity of operations
**1 action ~ 1 line**

Try to do atomic operations in your code — there should be exactly one operation on each line.

Bad ❌:
```python
# 1. 3 actions on one line - 3 function calls
foo_result = foo(bar(spam(x)))

# 2. 3 actions on one line - function call foo, get_c, from_b
foo_result = foo(a=a, b=b, c=get_c(from_b())

# 3. 3 actions on one line - filtering by arguments, conditionally getting elements (via or), calling a method .value
result = [(a.value() or A, b or B) for a, b in iterator if a < b]

# 4. 4 actions on one line - from library/variable foo comes bar attribute getting, spam attribute getting, hello attribute getting and calculate_weather call
result = calculate_weather(foo.bar.spam.hello)
```

Good ✅:
```python
# 1. make a call to each function in turn
spam_result = spam(x)
bar_result = bar(spam_result)
foo_result = foo(bar_result)

# 2. call the functions one by one, write the result to a variable and use it when calling foo
from_b_result = from_b()
c = get_c(from_b_result)
foo_result = foo(a=a, b=b, c=c)

# 3. sequentially perform actions on the list - first filter, then call the .value method of a, and choose between elements (or)
filtered_result = ((a, b) for a, b in iterator if a < b)
intermediate_result = ((a.value(), b) for a, b in filtered_result)
result = [(a or A, b or B) for a, b in intermediate_result]

# 4 . sequentially read the attributes bar, spam, hello and call the function calculate_weather
bar = foo.bar
spam = bar.spam
hello = spam.hello
result = calculate_weather(hello)
```


**Why?** Because the code becomes more readable, and there is no need to execute several statements in your head while reading the code. Code broken down into simple atomic operations is perceived much better than complex one-liners. Try to simplify your code as much as possible — code is more often read than written.


### Logical blocks

Try to divide the code into logical blocks — this way it will be much easier for the programmer to read and understand the essence.

Bad ❌:
```python
def register_model(self, app_label, model):
    model_name = model._meta.model_name
    app_models = self.all_models[app_label]
    if model_name in app_models:
        if (model.__name__ == app_models[model_name].__name__ and
                model.__module__ == app_models[model_name].__module__):
            warnings.warn(
                "Model '%s.%s' was already registered. "
                "Reloading models is not advised as it can lead to inconsistencies, "
                "most notably with related models." % (app_label, model_name),
                RuntimeWarning, stacklevel=2)
        else:
            raise RuntimeError(
                "Conflicting '%s' models in application '%s': %s and %s." %
                (model_name, app_label, app_models[model_name], model))
    app_models[model_name] = model
    self.do_pending_operations(model)
    self.clear_cache()
```

Good ✅:
```python
def register_model(self, app_label, model):
    model_name = model._meta.model_name
    app_models = self.all_models[app_label]
    
    if model_name in app_models:
        if (
            model.__name__ == app_models[model_name].__name__ and
            model.__module__ == app_models[model_name].__module__
        ):
            warnings.warn(
                "Model '%s.%s' was already registered. "
                "Reloading models is not advised as it can lead to inconsistencies, "
                "most notably with related models." % (app_label, model_name),
                RuntimeWarning, stacklevel=2)
            
        else:
            raise RuntimeError(
                "Conflicting '%s' models in application '%s': %s and %s." %
                (model_name, app_label, app_models[model_name], model))
        
    app_models[model_name] = model
    
    self.do_pending_operations(model)
    self.clear_cache()
```

**Why?** In addition to improving readability, The Zen of Python teaches us how to write idiomatic Python code. One of the statements claims that "sparse is better than dense." Compressed code is harder to read than sparse code.


### Sizes of methods, functions, and modules
The size limit for a method or function is 50 lines. Reaching the size limit indicates that the function (method) is doing too much — so decompose the actions inside the function (method).
The module size limit is 300 lines. Reaching the size limit indicates that the module has received too much logic — so decompose the module into several ones.
The line length is 100 characters.
Imports
The recommended import method is absolute.
Bad ❌:
# spam.py
from . import foo, bar
Good ✅:
# spam.py
from some.absolute.path import foo, bar
Why? Because absolute import explicitly defines the location (path) of the module that is being imported. With relative imports, you always need to remember the path and calculate in your mind the location of the modules foo.py, bar.py relative to spam.py
Files __init__.py
Only write imports in __init__.py files.
Why? Because __init__.py is the last place a programmer will look when they read the code in the future.
Docstrings
We recommend adding docstrings to functions, methods, and classes.
Why? Because the programmer who sees your code for the first time will be able to quickly understand what is happening in it. Code is read much more than it is written.
About Pull Requests
Creating Pull Requests
1 Pull Request = 1 issue
One Pull Request must solve exactly one issue.
Why? Because it is more difficult for a reviewer to keep the context of several tasks in their head and switch between them. When a PR contains several issues, then the PR often increases and requires more time and effort for the review from the reviewer.
Refactoring and Pull Requests
Refactoring is best done in a separate Pull Request.
Why? When refactoring goes along with resolving a specific issue, the refactoring blurs the context of the issue and introduces changes that are not related to that PR.
Pull Request Size
The resulting PR diff should not exceed +/- 600 changed lines.
Bad ❌:

Diff 444 + 333 = 777
 
Good ✅:
Diff 222 + 111 = 333
 
Why? Because the more PR involves, the more uncontrollable it becomes, and the merge is made "with eyes closed and ears shut." Also, most reviewers will find it difficult to accept a large volume of changes at once.
About tooling
Testing (pytest)
pytest - code testing framework
Recommended config in pytest.ini:
[pytest]
DJANGO_SETTINGS_MODULE = settings.local
python_files = tests.py test_*.py *_tests.py
 
Package manager (poetry)
poetry - dependency manager and package builder
Code formatting (Black)
Black - PEP8 code auto-formatter
Recommended config in pyproject.toml:
[tool.black]
line-length = 100
target-version = ['py38']
exclude = '''
(
  \.eggs
  |\.git
  |\.hg
  |\.mypy_cache
  |\.nox
  |\.tox
  |\.venv
  |_build
  |buck-out
  |build
  |dist
)
'''
 
 Imports formatting (isort)
isort - import block auto-formatter
Recommended config in pyproject.toml:
[tool.isort]
line_length = 100
sections = ["FUTURE", "STDLIB", "DJANGO", "THIRDPARTY", "FIRSTPARTY", "LOCALFOLDER"]
multi_line_output = 3
known_django = "django"
profile = "django"
src_paths = "app"
lines_after_imports = 2
 
Linter (flake8)
flake8 - PEP8 conformance validator
Recommended config in .flake8:
[flake8]
max-line-length = 100
max-complexity = 5
exclude = .venv,venv,**/migrations/*,snapshots
per-file-ignores =
    tests/**: S101
    **/tests/**: S101
Type checker (mypy)
mypy - checker for static typing
Recommended config mypy.ini:
[mypy]
ignore_missing_imports = True
allow_untyped_globals = True

[mypy-*.migrations.*]
ignore_errors = True
 
Pre-commit hooks (pre-commit)
pre-commit - framework for managing pre-commit hooks
Recommended config .pre-commit-config.yaml:
default_language_version:
    python: python3.8

repos:
  - repo: local
    hooks:
      - id: black
        name: black
        entry: black app
        language: python
        types: [python]

      - id: isort
        name: isort
        entry: isort app
        language: python
        types: [python]

      - id: flake8
        name: flake8
        entry: flake8 server
        language: python
        types: [python]
Other
REST API Documentation
The recommended documentation format is OpenAPI. The schema for OpenAPI should be generated “on the fly” to provide API clients with fresh changes.
Why? Because it's one of the common formats for documenting REST APIs that come out of Swagger. This documentation format is supported by a large number of clients (Swagger, Postman, Insomnia Designer, and many others). Also, handwritten documentation tends to quickly become outdated, and documentation that is generated directly from the code allows you to avoid constantly thinking about updating the documentation.


