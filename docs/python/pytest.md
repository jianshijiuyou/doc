> [官方文档](https://docs.pytest.org/en/latest/contents.html)

# 查看自带 fixture

``` bash
pytest --fixtures
```

# 在第一次（或N次）失败后停止

``` bash
pytest -x            # stop after first failure
pytest --maxfail=2    # stop after two failures
```

# 指定测试/选择测试

测试特定文件

``` bash
pytest test_mod.py
```

测试特定目录

``` bash
pytest testing/
```

测试特定函数

``` bash
pytest test_mod.py::test_func
```

测试特定类中的特定方法

``` bash
pytest test_mod.py::TestClass::test_method
```

# fixture 的使用

``` python
@pytest.fixture
def smtp_connection():
    import smtplib
    return smtplib.SMTP("smtp.gmail.com", 587, timeout=5)

def test_ehlo(smtp_connection):
    response, msg = smtp_connection.ehlo()
    assert response == 250
    assert 0 # for demo purposes
```

## conftest.py

可以将共用的 fixture 放到 conftest.py 文件中，pytest 会自动识别

## 共享范围

可以给 fixture 加上共享范围，共享范围内 fixture 只会创建一次，以后直接重用，重用范围有 `function` (defalut), `class`, `module`（同一个文件）, `package`, `session` .


``` python
# content of conftest.py
import pytest
import smtplib

@pytest.fixture(scope="module")
def smtp_connection():
    return smtplib.SMTP("smtp.gmail.com", 587, timeout=5)
```

## 模拟 teardown 效果

通过 `yield` 模拟 `setup(), teardown()` 效果

``` python
# content of conftest.py

import pytest

@pytest.fixture
def abc():
    print('==========start===========')
    yield "return abc"
    print('==========end===========')

# test_fixture.py

def test_one(abc):
    print(abc)
    assert 0

def test_two(abc):
    print(abc)
    assert 0

# print result

# ---------------- Captured stdout setup ----------
# ==========start===========
# ---------------- Captured stdout call -----------
# return abc
# ---------------- Captured stdout teardown --------
# ==========end===========

# ---------------- Captured stdout setup ----------
# ==========start===========
# ---------------- Captured stdout call -----------
# return abc
# ---------------- Captured stdout teardown --------
# ==========end===========
```

如果范围换成 `module`

结果是

``` python
# ---------------- Captured stdout setup ----------
# ==========start===========
# ---------------- Captured stdout call -----------
# return abc

# ---------------- Captured stdout call -----------
# return abc
# ---------------- Captured stdout teardown --------
# ==========end===========
```

### 第二种方式


``` python
import pytest

@pytest.fixture(scope="package")
def abc(request):

    print('==========start===========')

    def fin():
        print("====end====")

    request.addfinalizer(fin)
    # 可以添加多个函数
    # request.addfinalizer(fin)
    return "return abc"
```

## Fixtures 可以内省请求的测试上下文

``` python

# conftest.py

import pytest

@pytest.fixture(scope="module")
def abc(request):
    test_url = request.module.test_url
    print(test_url)
    return "return abc"

# test_fixture.py

test_url = "https://www.google.com"

def test_one(abc):
    print(abc)
    assert 0
```

## 参数化 fixtures

根据参数列表重复调用测试函数

``` python
import pytest

@pytest.fixture(scope="module", params=['arg1', 'arg2', 'arg3'])
def abc(request):   # request 自带的
    arg = request.param
    print(arg)
    return "return abc"

# ================================================================= FAILURES =================================================================
# ______________________________________________________________ test_one[arg1] ______________________________________________________________

# ....
# ---------------------------------------------------------- Captured stdout setup -----------------------------------------------------------
# arg1
# ----------------------------------------------------------- Captured stdout call -----------------------------------------------------------
# return abc
# ______________________________________________________________ test_two[arg1] ______________________________________________________________

# ....
# ----------------------------------------------------------- Captured stdout call -----------------------------------------------------------
# return abc
# ______________________________________________________________ test_one[arg2] ______________________________________________________________

# ....
# ---------------------------------------------------------- Captured stdout setup -----------------------------------------------------------
# arg2
# ----------------------------------------------------------- Captured stdout call -----------------------------------------------------------
# return abc
# ______________________________________________________________ test_two[arg2] ______________________________________________________________

# ....
# ----------------------------------------------------------- Captured stdout call -----------------------------------------------------------
# return abc
# ______________________________________________________________ test_one[arg3] ______________________________________________________________

# ....
# ---------------------------------------------------------- Captured stdout setup -----------------------------------------------------------
# arg3
# ----------------------------------------------------------- Captured stdout call -----------------------------------------------------------
# return abc
# ______________________________________________________________ test_two[arg3] ______________________________________________________________

# ....
# ----------------------------------------------------------- Captured stdout call -----------------------------------------------------------
# return abc
# ========================================================= 6 failed in 0.05 seconds =========================================================
```

查看具体函数使用参数的情况

``` bash
pytest --collect-only

<Module 'test_fixture.py'>
  <Function 'test_one[arg1]'>
  <Function 'test_two[arg1]'>
  <Function 'test_one[arg2]'>
  <Function 'test_two[arg2]'>
  <Function 'test_one[arg3]'>
  <Function 'test_two[arg3]'>
```

# monkeypatch

动态替换

``` python
def get_db():
    return "db"

def test_one(monkeypatch):
    def mockreturn():
        return 'mockreturn'
    monkeypatch.setattr('test_fixture.get_db', mockreturn)
    print(get_db())
    assert 0

# ----------- Captured stdout call -------
# mockreturn
```

# 跳过测试

``` python
import pytest

@pytest.mark.skip(reason="skip")
def test_one(monkeypatch):
    assert 0
```

# 参数化测试函数

根据参数重复执行测试函数，避免编写重复的参数不同，逻辑相同的测试函数

``` python
import pytest

@pytest.mark.parametrize(('test_input', 'expected'), [
    ("3+5", 8),
    ("2+4", 6),
    ("6*9", 42),
])
def test_one(test_input, expected):
    assert eval(test_input) == expected

# ================================================================= FAILURES =================================================================
# _____________________________________________________________ test_one[6*9-42] _____________________________________________________________

# test_input = '6*9', expected = 42

#     @pytest.mark.parametrize(('test_input', 'expected'), [
#         ("3+5", 8),
#         ("2+4", 6),
#         ("6*9", 42),
#     ])
#     def test_one(test_input, expected):
# >       assert eval(test_input) == expected
# E       AssertionError: assert 54 == 42
# E        +  where 54 = eval('6*9')

# test_fixture.py:9: AssertionError
# ==================================================== 1 failed, 2 passed in 0.05 seconds ====================================================
```

# 其他参数选项

## 输出日志

`pytest -s`

通过 `-s` 选项可以在测试的时候同时看到输出到终端的日志