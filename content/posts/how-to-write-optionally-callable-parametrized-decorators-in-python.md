---
title: "如何在 Python 中编写可选参数的装饰器"
date: 2021-12-05T16:54:03+08:00
draft: flase
---

### 举个栗子


```python

import pytest

@pytest.fixture  # No need to write '@pytest.fixture()'
def app():
    ...

@pytest.fixture(scope="session")
def server():
    ...
```

### 具体实现

让我们用 `@logged` 为函数编写一个装饰器

它接受一个可选 `decimals` 参数来将计算结果四舍五入。如果 `decimals` 没有给出，我们根本不应该管它。

因此，可能的调用方式应该是：

```python

@logged()
def test():
    ...

@logged(decimals=2)
def test():
    ...

@logged # 相当于@logged() —— 实现这一点是这篇博文的目标
def test():
    ...
```

切入正题，这是带注释的解决方案：

```python

import functools
import typing

def logged(func: typing.Callable = None, decimals: int = None) -> typing.Callable:
    if func is None:
        print(f"Called with decimals={decimals}")
        return functools.partial(logged, decimals=decimals)

    print(f"Called without parameters, func={func}.")

    @functools.wraps(func)
    def decorated(*args: typing.Any, **kwargs: typing.Any) -> typing.Any:
        print(f"{func.__name__} called with args={args}, kwargs={kwargs}")
        result = func(*args, **kwargs)
        logged_result = result if decimals is None else round(result, decimals)
        print(f"Result: {logged_result}")
        return result

    return decorated
```

如果我们运行以下脚本：

```python

@logged
def add(x: float, y: float) -> float:
    return x + y

@logged(decimals=2)
def mult(x: float, y: float) -> float:
    return x * y

```

运行如下

```python

add(2, 2)
mult(3, 4)
```

我们得到以下输出：

```python

Called without parameters, func=<function add at 0x10d8d8b70>.
Called with decimals=2
Called without parameters, func=<function mult at 0x10db18a60>.
add called with args=(2, 2), kwargs={}
Result: 4
mult called with args=(3, 4), kwargs={}
Result: 12
```

### 通用实现

这个 100% 的通用实现没有任何注释和调试输出。只需将其复制粘贴到某处并根据您的需要进行调整即可。

```python

import functools
import typing


def decorate(func: typing.Callable = None, **options: typing.Any) -> typing.Callable:
    if func is None:
        return functools.partial(decorate, **options)

    @functools.wraps(func)
    def decorated(*args: typing.Any, **kwargs: typing.Any) -> typing.Any:
        return func(*args, **kwargs)

    return decorated

```
就是这样了。


