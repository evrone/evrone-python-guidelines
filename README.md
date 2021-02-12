# Evrone Python Codestyle


## Пакетный менеджер

[poetry](https://python-poetry.org) - менеджер зависимостей и сборщик пакетов 


## Форматирование кода

[black](https://black.readthedocs.io/en/stable/) - автоформаттер кода по PEP8

Рекомендуемый конфиг в `pyproject.toml`:
```toml

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

```


[isort](https://pycqa.github.io/isort/) - автоформаттер блока импортов

Рекомендуемый конфиг в `pyproject.toml`:
```toml

[tool.isort]
line_length = 100
sections = ["FUTURE", "STDLIB", "DJANGO", "THIRDPARTY", "FIRSTPARTY", "LOCALFOLDER"]
multi_line_output = 3
known_django = "django"
profile = "django"
src_paths = "app"

```


## Типизация

[mypy](http://mypy.readthedocs.io) - чекер для статической типизации


Рекомендуемый конфиг `mypy.ini`:

```ini
[mypy]
ignore_missing_imports = True
allow_untyped_globals = True

[mypy-*.migrations.*]
ignore_errors = True

```


## Импорты

Плохо ❌:
```python
# spam.py
from . import foo, bar
```

Хорошо ✅:
```python

# spam.py
from some.absolute.path import foo, bar
```

**Почему?** Потому что абсолютный импорт явно определяет локацию (путь) модуля, который импортируется. При релативном
импорте всегда нужно помнить путь и вычислять в уме локацию модулей `foo.py`, `bar.py` относительно `spam.py`


## Размеры методов, функций и модулей

Предельный размер метода или функции - **50** строк.
Достижение предельного размера говорит о том, что функция (метод) делает слишком много - декомпозируйте действия внутри функции (метода). 


Предельный размер модуля - **300** строк.
Достижение предельного размера говорит о том, что модуль получил слишком много логики - декомпозируйте модуль на несколько.

Длина строки - 100 символов.


## Файлы `__init__.py`

В `__init__.py` файлах пишем только импорты.

**Почему?** Потому что `__init__.py` - последнее место, в которое посмотрит программист, который будет читать код в будущем.


## Докстринги
Рекомендуем добавлять докстринги в функции, методы и классы. 

**Почему?** Потому что программист, который впервые увидит ваш код, сможет быстрее понять, что в нем происходит. 
Код читается намного больше, чем пишется. 


## Pull Request
Дифф PR не должен превышать +/- 1000 строк. 

**Почему?** Потому что чем больше PR - тем более он становится неконтролируемым и мерж производится "закрыв глаза и заткнув уши". 
Также, большинству ревьюверов будет сложно воспринять такой объем изменений за один раз.