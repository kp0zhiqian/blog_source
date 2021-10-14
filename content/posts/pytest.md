---
title: "Pytest Notes"
date: 2020-10-20T00:11:23+08:00
draft: false
tags:
    - python
keywords:
    - pytest
---

## Test files and functions

`pytest` will run all files of format `test_*.py` or `*_test.py` in the current path. And run the functions name start with `test`.

## Run tests

`pytest` in the current directory will run the test
`pytest <filename> -v` will run a specific file.
`pytest -k <string> -v` will run the functions that containes a string `<string>`

## Test results

- PASSED(.): test pass
- FAILED(F): test fail
- SKIPPED(s): test skipped, you can use `@pytest.mark.skip()` or `@pytest.mark.skipif()` to skip the cases
- xfail(x): the test was meant to be failed, we can use `@pytest.mark.xfail()` to mark
- XPASS(X): test test should not pass
- ERROR(E): error

## Grouping tests
we can use markers to group tests by using `@pytest.mark.<markername>` to decorate the test functions.

`pytest` has its own build-in markers we can use:
```bash
~/python_learn/pytest » pytest --markers                                                                                                                                                               1 ↵ zhiqianguan@zhguan-mac
@pytest.mark.filterwarnings(warning): add a warning filter to the given test. see https://docs.pytest.org/en/stable/warnings.html#pytest-mark-filterwarnings 

@pytest.mark.skip(reason=None): skip the given test function with an optional reason. Example: skip(reason="no way of currently testing this") skips the test.

@pytest.mark.skipif(condition, ..., *, reason=...): skip the given test function if any of the conditions evaluate to True. Example: skipif(sys.platform == 'win32') skips the test if we are on the win32 platform. See https://docs.pytest.org/en/stable/reference.html#pytest-mark-skipif

@pytest.mark.xfail(condition, ..., *, reason=..., run=True, raises=None, strict=xfail_strict): mark the test function as an expected failure if any of the conditions evaluate to True. Optionally specify a reason for better reporting and run=False if you don't even want to execute the test function. If only specific exception(s) are expected, you can list them in raises, and if the test fails in other ways, it will be reported as a true failure. See https://docs.pytest.org/en/stable/reference.html#pytest-mark-xfail

@pytest.mark.parametrize(argnames, argvalues): call a test function multiple times passing in different arguments in turn. argvalues generally needs to be a list of values if argnames specifies only one name or a list of tuples of values if argnames specifies multiple names. Example: @parametrize('arg1', [1,2]) would lead to two calls of the decorated test function, one with arg1=1 and another with arg1=2.see https://docs.pytest.org/en/stable/parametrize.html for more info and examples.

@pytest.mark.usefixtures(fixturename1, fixturename2, ...): mark tests as needing all of the specified fixtures. see https://docs.pytest.org/en/stable/fixture.html#usefixtures 

@pytest.mark.tryfirst: mark a hook implementation function such that the plugin machinery will try to call it first/as early as possible.

@pytest.mark.trylast: mark a hook implementation function such that the plugin machinery will try to call it last/as late as possible.
```

Also we can use customized markers after we add them into the pytest configuration file, e.g. `pytest.ini`
```ini
[pytest]
markers = 
    great: for the great function
    others
```
use them in our test file
```python
import pytest

@pytest.mark.great
def test_greater():
    num = 100
    assert num > 100

@pytest.mark.great
def test_greater_equal():
    num = 100
    assert num == 100

@pytest.mark.others
def test_less():
    num = 100
    assert num < 200
```

run the selected test
```bash
pytest -m others -v
pytest -m great -v
pytest -m "others and not great"
pytest -m "not great"
```

## Parametrize testing
We could use `@pytest.mark.parametrize('<param1>, <param2>', [(<param_set>), (param_set)])` to run different parameters for a test function, each set of parameters will run one time of testing.

```python
# test_parametrize.py

@pytest.mark.parametrize('passwd',
                      ['123456',
                       'abcdefdfs',
                       'as52345fasdf4'])
def test_passwd_length(passwd):
    assert len(passwd) >= 8
```

Multiple parameters
```python
# test_parametrize.py

@pytest.mark.parametrize('user, passwd',
                         [('jack', 'abcdefgh'),
                          ('tom', 'a123456a')])
def test_passwd_md5(user, passwd):
    db = {
        'jack': 'e8dc4081b13434b45189a720b77b6818',
        'tom': '1702a132e769a623c1adb78353fc9503'
    }

    import hashlib

    assert hashlib.md5(passwd.encode()).hexdigest() == db[user]
```


## Error catch

We can use `pytest.raise()` to catch Exceptions and its information. When the exception happends, pytest will catch it and continue the assertion.

```python
import pytest
def test_raises():
    with pytest.raises(ZeroDivisionError):
        2 / 0
    assert 3 == 3
```

also, the exceiption we catched will registered in the context manager which we could pull information from.

```python
import pytest

def exc(x):
    if x == 0:
        raise ValueError("value not 0")
    return 2 / x

def test_raises():
    with pytest.raises(ValueError) as exec_info:
        exec(0)
    print(exec_info.type)
    print(exec_info.value.args)
```

Please note that the line that raise exceptions should be the last line of `with`, otherwise the rest of lines within `with` will not be excuted.

## Fixture

Fixture is some funtions we run before/after the actual test function, in other words, provide context for the test cases.
We use `@pytest.fixture` to define a fixture.

```python
import pytest
class Fruit:
    def __init__(self, name):
        self.name = name

    def __eq__(self, other):
        return self.name == other.name


@pytest.fixture
def my_fruit():
    return Fruit("apple")


@pytest.fixture
def fruit_basket(my_fruit):
    return [Fruit("banana"), my_fruit]


def test_my_fruit_in_basket(my_fruit, fruit_basket):
    assert my_fruit in fruit_basket
```

### Scope of fixture

There's a `scope` parameter that can control the fixture's scope.

`@pytest.fixture(scope="class")`

`scope` can be `session` > `module` > `class` > `function`
- function(default): run for every test function, each function will run it one time
- class: run fixture for a whole class, each class will run it one time
- module: run fixture for the whole module(i.e. python file), each module will run it one time
- session: run fixture for the whole session(be careful to use), each session will run it one time.

#### `autouse=True`
if we want to run a fixture for every test function(or class, or module, or session), we can use `@pytest.fixture(autouse=True)` to "auto use" it.

### Ways to use fixture

#### pass the fixture's name into test function

just like the above code example, we could pass the fixture name into a test functions and use the fixture's return value.

```python
import pytest

@pytest.fixture
def do_something_before():
    return "running-fixture"

def test_run(do_something_before):
    print(do_something_before)
    assert 1 == 1
```

```bash
~/python_learn/pytest » pytest -v -s                                                                                                                                                                       zhiqianguan@zhguan-mac
====================================================================================================== test session starts =======================================================================================================
platform darwin -- Python 3.8.3, pytest-6.2.5, py-1.10.0, pluggy-1.0.0 -- /Library/Frameworks/Python.framework/Versions/3.8/bin/python3.8
cachedir: .pytest_cache
rootdir: /Users/zhiqianguan/python_learn/pytest, configfile: pytest.ini
collected 1 item                                                                                                                                                                                                                 

test_basic.py::test_run running-fixture
PASSED

======================================================================================================= 1 passed in 0.01s ========================================================================================================
```
#### use `@pytest.mark.usefixtture()`

If a fixture has return values, `usefixtures` cannot get the return values. So if we need the return values of a fixgure, we can only pass the fixture name to the test function.

So we normally put some test setup in `usefixture` as we normally won't need any return values from it.

#### use many `@pytest.mark.usefixture()`

we could use many fixture for a single test function or class like:

```python
@pytest.mark.usefixture('test-2') # test-2 will run secondly
@pytest.mark.usefixture('test-1') # test-1 will run firstly
def test_func():
    print("test")
    assert 1 == 1
```

### pre-run and post-run

`yield` will seperate a fixture into two parts, pre-run and post-run.

```python
@pytest.fixture()
def db_conn():
    print("connected DB.")
    yield
    print("disconnected.")

def test_db(db_conn):
    db = {"test":1}
    assert db['test'] == 1
```

### rename a fixture
The name of a fixture is the name of the decorated function by default, but we can also rename a fixture as we want.
```python
@pytest.fixture(name="timer")
def time_cost():
    return 1
```