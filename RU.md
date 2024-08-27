# Evrone Python Guidelines (RU)


## Содержание
- [Про код](#про-код)
  - [Основные принципы](#основные-принципы)
  - [Атомарность операций](#атомарность-операций)
  - [Логические блоки](#логические-блоки)
  - [Размеры методов, функций и модулей](#размеры-методов-функций-и-модулей)
  - [Импорты](#импорты)
  - [Файлы `__init__.py`](#файлы-__init__py)
  - [Докстринги](#докстринги)
- [Про Pull Request](#про-pull-request)
  - [Создание Pull Request](#создание-pull-request)
  - [Рефакторинг и Pull Request](#рефакторинг-и-pull-request)
  - [Размер Pull Request](#размер-pull-request)
- [Про тулинг](#про-тулинг)
- [Прочее](#прочее)
  - [Документация к REST API](#документация-к-rest-api)


## Про код

### Основные принципы
- **Поддерживаемость** (представьте, сможете ли вы понять свой код через год или через два)
- **Простота** (между сложным и простым решением следует выбрать простое)
- **Очевидность** (представьте, когда подключится новый программист, насколько ему будет понятно, почему именно **так** написан этот код)


### Атомарность операций
**1 действие ~ 1 строка**

Постарайтесь делать атомарные операции в коде - на каждой строке ровно **одно** действие. 

Плохо ❌:
```python
# 1. 3 действия на одной строке - 3 вызова функции
foo_result = foo(bar(spam(x)))

# 2. 3 действия на одной строке - вызов функции foo, get_c, from_b
foo_result = foo(a=a, b=b, c=get_c(from_b())

# 3. 3 действия на одной строке - фильтрация по аргументам, условное получение элементов (через or), вызов метода .value
result = [(a.value() or A, b or B) for a, b in iterator if a < b]

# 4. 4 действия на одной строке - из библиотеки / переменной foo идет получение атрибута bar, получение атрибута spam, получение атрибута hello и вызов calculate_weather
result = calculate_weather(foo.bar.spam.hello)
```

Хорошо ✅:
```python
# 1. делаем поочередный вызов каждой функции
spam_result = spam(x)
bar_result = bar(spam_result)
foo_result = foo(bar_result)

# 2. поочередно вызываем функции, результат пишем в переменную и используем ее при вызове foo
from_b_result = from_b()
c = get_c(from_b_result)
foo_result = foo(a=a, b=b, c=c)

# 3. последовательно проводим действия над списком - вначале фильтруем, вызываем метод .value у a, выбираем между элементами (or)
filtered_result = ((a, b) for a, b in iterator if a < b)
intermediate_result = ((a.value(), b) for a, b in filtered_result)
result = [(a or A, b or B) for a, b in intermediate_result]

# 4 . последовательно читаем атрибуты bar, spam, hello и вызываем функцию calculate_weather
bar = foo.bar
spam = bar.spam
hello = spam.hello
result = calculate_weather(hello)
```


**Почему?** Потому что код становится более читабельным, не нужно исполнять несколько выражений в голове во время чтения кода. Разбитый до простых атомных операций код воспринимается гораздо лучше, чем сложный уан-лайнер. Постарайтесь упростить свой код настолько, насколько это возможно - код чаще читается, чем пишется.


### Логические блоки

Постарайтесь делить код на логические блоки - так глазу программиста будет в разы проще прочитать и уловить суть.

Плохо ❌:
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

Хорошо ✅:
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

**Почему?** Кроме того, что это повышает читабельность, [Zen of Python](https://www.python.org/dev/peps/pep-0020/) рассказывает нам о том, как надо писать идиоматический код на Python.
Одно из высказываний звучит как "Sparse is better than dense." - "Разреженное лучше чем сжатое". Сжатый код сложнее прочитать чем разреженный.


### Размеры методов, функций и модулей

Предельный размер метода или функции - **50** строк.
Достижение предельного размера говорит о том, что функция (метод) делает слишком много - декомпозируйте действия внутри функции (метода). 


Предельный размер модуля - **300** строк.
Достижение предельного размера говорит о том, что модуль получил слишком много логики - декомпозируйте модуль на несколько.

Длина строки - 100 символов.


### Импорты

Рекомендуемый способ импортирования - абсолютный. 

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

**Почему?** Потому что абсолютный импорт явно определяет локацию (путь) модуля, который импортируется. 
При релативном импорте всегда нужно помнить путь и вычислять в уме локацию модулей `foo.py`, `bar.py` относительно `spam.py`


### Файлы `__init__.py`

В `__init__.py` файлах пишем только импорты.

**Почему?** Потому что `__init__.py` - последнее место, в которое посмотрит программист, который будет читать код в будущем.


### Докстринги
Рекомендуем добавлять докстринги в функции, методы и классы. 

**Почему?** Потому что программист, который впервые увидит ваш код, сможет быстрее понять, что в нем происходит. 
Код читается намного больше, чем пишется. 


## Про Pull Request

### Создание Pull Request 
**1 Pull Request = 1 issue**

Один Pull Request должен решать ровно одно issue.

**Почему?** Потому что ревьюверу сложнее держать контекст нескольких задач в голове и переключаться между ними. Когда PR содержит несколько issue - это часто приводит к тому, что PR увеличивается и требует больше времени и сил на ревью от ревьювера.


### Рефакторинг и Pull Request
Рефакторинг лучше всего выносить в отдельный Pull Request.

**Почему?** Когда рефакторинг идет вместе с решением определенного issue, то рефакторинг размывает контекст issue и вводит правки, которые не имеют отношения к данному PR. 


### Размер Pull Request
Итоговый дифф PR не должен превышать +/- 600 измененных строк.

Плохо ❌:

![bad](https://user-images.githubusercontent.com/8825727/113953748-6fc7ba80-9853-11eb-9673-827995e54f73.png)
```
Дифф 444 + 333 = 777
```

Хорошо ✅:

![good](https://user-images.githubusercontent.com/8825727/113953831-a30a4980-9853-11eb-854b-d4c4f6559f2c.png)
```
Дифф 222 + 111 = 333
```


**Почему?** Потому что чем больше PR - тем более он становится неконтролируемым и мерж производится "закрыв глаза и заткнув уши". 
Также, большинству ревьюверов будет сложно воспринять такой объем изменений за один раз.

## Про тулинг

Актуальные версии конфигов хранятся в [evrone-django-template](https://github.com/evrone/evrone-django-template).

- [pytest](https://pytest.org) - фреймворк для тестирования кода, [пример конфига](https://github.com/evrone/evrone-django-template/blob/master/%7B%7Bcookiecutter.project_name%7D%7D/pytest.ini)
- [poetry](https://python-poetry.org) - менеджер зависимостей и сборщик пакетов, [пример pyproject.toml](https://github.com/evrone/evrone-django-template/blob/master/%7B%7Bcookiecutter.project_name%7D%7D/pyproject.toml)
- [black](https://black.readthedocs.io/en/stable/) - PEP8 автоформаттер кода, [пример конфига](https://github.com/evrone/evrone-django-template/blob/master/%7B%7Bcookiecutter.project_name%7D%7D/pyproject.toml#L42-L59)
 - [isort](https://pycqa.github.io/isort/) - автоформаттер импортов, [пример конфига](https://github.com/evrone/evrone-django-template/blob/master/%7B%7Bcookiecutter.project_name%7D%7D/pyproject.toml#L61-L68)
- [flake8](https://flake8.pycqa.org/en/latest/) - валидатор соответствия PEP8, [пример конфига](https://github.com/evrone/evrone-django-template/blob/master/%7B%7Bcookiecutter.project_name%7D%7D/pyproject.toml#L95-L186)
- [pylint](https://pylint.readthedocs.io/en/latest/) - статический анализатор кода. Проверяет наличие ошибок, обеспечивает соблюдение стандартов, ищет подозрительный код и дает рекомендации по рефакторингу, [пример конфига](https://github.com/evrone/evrone-django-template/blob/master/%7B%7Bcookiecutter.project_name%7D%7D/pyproject.toml#L189-L269)
- [mypy](http://mypy.readthedocs.io) - валидатор аннотации типов, [пример конфига](https://github.com/evrone/evrone-django-template/blob/master/%7B%7Bcookiecutter.project_name%7D%7D/pyproject.toml#L89-L90), [пример команды для запуска с дополнительными аргументами](https://github.com/evrone/evrone-django-template/blob/master/%7B%7Bcookiecutter.project_name%7D%7D/Makefile#L75)
- [pre-commit](https://pre-commit.com) - фреймворк для управления `pre-commit` хуками, [пример конфига](https://github.com/evrone/evrone-django-template/blob/master/%7B%7Bcookiecutter.project_name%7D%7D/.pre-commit-config.yaml)

## Прочее

### Документация к REST API
Рекомендуемый формат документации - [OpenAPI](https://www.openapis.org).
Схема для OpenAPI должна генерироваться "на лету", чтобы обеспечивать клиентов API свежими изменениями.

**Почему?** Потому что это один из распространенных форматов для документирования REST API, который вышел из Swagger. Данный формат документации поддерживается большим количеством клиентов (Swagger, Postman, Insomnia Designer и многие другие). Также, рукописная документация имеет свойство быстро устаревать, а документация, которая генерируется напрямую из кода позволяет не думать о постоянном обновлении документации.


## Спонсор
[<img src="https://evrone.com/logo/evrone-sponsored-logo.png" width=300>](https://evrone.com/?utm_source=github.com&utm_campaign=evrone-python-codestyle)
