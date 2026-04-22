---
title: "Python 装饰器基础：理解与使用"
date: 2026-04-22
draft: false
featuredImage: "img/wallhaven-l8671l.png"
tags: ["Python", "装饰器", "函数式编程", "高级特性"]
categories: ["Python"]
---


装饰器是 Python 中最强大也最容易被误解的特性之一。它允许你在不修改原函数代码的情况下，给函数添加新功能。装饰器在框架、库和大型项目中无处不在：日志记录、性能测试、权限验证、缓存等都可以用它来实现。

本文将带你从零开始，彻底理解装饰器的原理和使用方法。

## 什么是装饰器？

### 装饰器的概念

装饰器本质上是一个函数，它接受一个函数作为参数，并返回一个新的函数。想象一下给函数"穿上一件外衣"——这就是装饰器的作用。

### 简单类比

```python
# 装饰器就像给函数添加包装
def my_function():
    return "Hello"

# 装饰前
result = my_function()  # "Hello"

# 装饰后（添加了日志）
result = logged(my_function)()  # "Hello" + 日志
```

### 基本语法

```python
# 使用 @decorator_name 语法
@my_decorator
def my_function():
    return "Hello"

# 等价于
def my_function():
    return "Hello"
my_function = my_decorator(my_function)
```

## 第一个装饰器

### 最简单的装饰器

```python
# 定义装饰器
def my_decorator(func):
    """一个简单的装饰器"""
    def wrapper():
        print("函数执行前")
        result = func()
        print("函数执行后")
        return result
    return wrapper

# 使用装饰器
@my_decorator
def say_hello():
    print("Hello!")

# 调用
say_hello()

# 输出:
# 函数执行前
# Hello!
# 函数执行后
```

### 装饰器的工作原理

```python
# 逐步分解装饰器的执行过程

# 1. 定义原始函数
def say_hello():
    print("Hello!")

# 2. 将函数传给装饰器
decorated = my_decorator(say_hello)

# 3. decorated 是 wrapper 函数
print(type(decorated))  # <class 'function'>

# 4. 调用 decorated
decorated()

# 5. @my_decorator 语法就是上面步骤的简写
@my_decorator
def say_hello():
    print("Hello!")
# 等价于
def say_hello():
    print("Hello!")
say_hello = my_decorator(say_hello)
```

## 带参数的装饰器

### 处理函数参数

```python
def trace_calls(func):
    """追踪函数调用的装饰器"""
    def wrapper(*args, **kwargs):
        print(f"调用 {func.__name__}，参数: args={args}, kwargs={kwargs}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} 返回: {result}")
        return result
    return wrapper

@trace_calls
def add(a, b):
    return a + b

@trace_calls
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

# 测试
add(3, 5)
# 输出:
# 调用 add，参数: args=(3, 5), kwargs={}
# add 返回: 8

greet("Alice")
# 输出:
# 调用 greet，参数: args=('Alice',), kwargs={}
# greet 返回: Hello, Alice!

greet("Bob", greeting="Hi")
# 输出:
# 调用 greet，参数: args=('Bob',), kwargs={'greeting': 'Hi'}
# greet 返回: Hi, Bob!
```

### `*args` 和 `**kwargs`

```python
def debug(func):
    """打印函数所有参数的装饰器"""
    def wrapper(*args, **kwargs):
        # 位置参数
        print(f"位置参数: {args}")
        # 关键字参数
        print(f"关键字参数: {kwargs}")
        # 调用原函数
        result = func(*args, **kwargs)
        return result
    return wrapper

@debug
def process(*args, **kwargs):
    print(f"处理数据: {args}, {kwargs}")

process(1, 2, 3, name="test", value=42)
# 输出:
# 位置参数: (1, 2, 3)
# 关键字参数: {'name': 'test', 'value': 42}
# 处理数据: (1, 2, 3), {'name': 'test', 'value': 42}
```

## 保留原函数信息

### 问题：函数元数据丢失

```python
def simple_decorator(func):
    def wrapper(*args, **kwargs):
        print("调用前")
        return func(*args, **kwargs)
    return wrapper

@simple_decorator
def hello():
    """这是 hello 函数"""
    return "Hello"

print(hello.__name__)  # wrapper - 函数名变成了 wrapper！
print(hello.__doc__)   # None - 文档字符串也丢失了！
```

### 解决方案：使用 functools.wraps

```python
import functools

def simple_decorator(func):
    @functools.wraps(func)  # 保留原函数元数据
    def wrapper(*args, **kwargs):
        print("调用前")
        return func(*args, **kwargs)
    return wrapper

@simple_decorator
def hello():
    """这是 hello 函数"""
    return "Hello"

print(hello.__name__)  # hello - 保留了原函数名！
print(hello.__doc__)   # 这是 hello 函数 - 保留了文档字符串！
```

### functools.wraps 的作用

```python
import functools

def my_decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

# functools.wraps 会复制以下属性：
# - __name__      函数名
# - __doc__       文档字符串
# - __qualname__  限定名称
# - __module__    模块名
# - __annotations__ 参数注解
# - __dict__      其他属性

# 手动复制（等价于 @functools.wraps）
def manual_wraps(wrapper, func):
    wrapper.__name__ = func.__name__
    wrapper.__doc__ = func.__doc__
    # ... 其他属性
    return wrapper
```

## 带参数的装饰器

### 装饰器工厂函数

```python
# 装饰器工厂：返回装饰器的函数
def repeat(times):
    """重复执行函数的装饰器工厂"""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            results = []
            for _ in range(times):
                result = func(*args, **kwargs)
                results.append(result)
            return results
        return wrapper
    return decorator

# 使用
@repeat(times=3)
def say_hello():
    return "Hello!"

print(say_hello())  # ['Hello!', 'Hello!', 'Hello!']

@repeat(times=2)
def add(a, b):
    return a + b

print(add(3, 5))  # [8, 8]
```

### 多层参数的装饰器

```python
def log(level):
    """带日志级别的装饰器工厂"""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            print(f"[{level}] 调用 {func.__name__}")
            return func(*args, **kwargs)
        return wrapper
    return decorator

# 使用
@log("DEBUG")
def debug_function():
    pass

@log("INFO")
def info_function():
    pass

@log("WARNING")
def warning_function():
    pass

debug_function()    # [DEBUG] 调用 debug_function
info_function()     # [INFO] 调用 info_function
warning_function()  # [WARNING] 调用 warning_function
```

## 类装饰器

### 函数装饰器 vs 类装饰器

```python
# 函数装饰器
def func_decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

# 类装饰器
class ClassDecorator:
    def __init__(self, func):
        self.func = func
    
    def __call__(self, *args, **kwargs):
        return self.func(*args, **kwargs)

@ClassDecorator
def decorated_function():
    return "Hello"
```

### 类作为装饰器

```python
import functools
import time

class Timer:
    """计时器装饰器类"""
    def __init__(self, func):
        functools.update_wrapper(self, func)
        self.func = func
    
    def __call__(self, *args, **kwargs):
        start = time.time()
        result = self.func(*args, **kwargs)
        end = time.time()
        print(f"{self.func.__name__} 执行耗时: {end - start:.4f}秒")
        return result

@Timer
def slow_function():
    time.sleep(1)
    return "完成"

@Timer
def fast_function():
    return "完成"

slow_function()   # slow_function 执行耗时: 1.0001秒
fast_function()   # fast_function 执行耗时: 0.0000秒
```

### 类装饰器带参数

```python
class Repeat:
    """带参数的类装饰器"""
    def __init__(self, times):
        self.times = times
    
    def __call__(self, func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            results = []
            for _ in range(self.times):
                results.append(func(*args, **kwargs))
            return results
        return wrapper

@Repeat(times=3)
def get_value():
    return 42

print(get_value())  # [42, 42, 42]
```

## 装饰器叠加

### 多个装饰器

```python
def decorator1(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print("装饰器 1 - 开始")
        result = func(*args, **kwargs)
        print("装饰器 1 - 结束")
        return result
    return wrapper

def decorator2(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print("装饰器 2 - 开始")
        result = func(*args, **kwargs)
        print("装饰器 2 - 结束")
        return result
    return wrapper

# 装饰器从上到下应用
@decorator1
@decorator2
def my_function():
    print("执行 my_function")

my_function()

# 执行顺序：
# 装饰器 1 - 开始
# 装饰器 2 - 开始
# 执行 my_function
# 装饰器 2 - 结束
# 装饰器 1 - 结束
```

### 装饰器顺序的影响

```python
# 顺序不同，结果不同

@decorator_a
@decorator_b
def process():
    pass
# 执行顺序：decorator_a → decorator_b → process → decorator_b → decorator_a

# 相当于
process = decorator_a(decorator_b(process))

# 示例
def uppercase(func):
    @functools.wraps(func)
    def wrapper():
        return func().upper()
    return wrapper

def add_exclamation(func):
    @functools.wraps(func)
    def wrapper():
        return func() + "!"
    return wrapper

@add_exclamation
@uppercase
def greet():
    return "hello"

print(greet())  # HELLO!

@uppercase
@add_exclamation
def greet2():
    return "hello"

print(greet2())  # hello!
```

## 实战装饰器

### 1. 日志装饰器

```python
import functools
import logging

def log(level="INFO"):
    """日志装饰器"""
    def decorator(func):
        logger = logging.getLogger(func.__module__)
        
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            logger.log(
                getattr(logging, level),
                f"调用 {func.__name__}({args}, {kwargs})"
            )
            try:
                result = func(*args, **kwargs)
                logger.log(
                    getattr(logging, level),
                    f"{func.__name__} 返回 {result}"
                )
                return result
            except Exception as e:
                logger.error(f"{func.__name__} 抛出异常: {e}")
                raise
        return wrapper
    return decorator

# 使用
@log("DEBUG")
def complex_function(x, y):
    return x / y

@log("ERROR")
def another_function():
    pass
```

### 2. 计时装饰器

```python
import functools
import time

def timer(func):
    """函数执行时间计时器"""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        end = time.perf_counter()
        print(f"{func.__name__} 执行耗时: {end - start:.4f}秒")
        return result
    return wrapper

def async_timer(func):
    """异步函数计时器"""
    @functools.wraps(func)
    async def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = await func(*args, **kwargs)
        end = time.perf_counter()
        print(f"{func.__name__} 执行耗时: {end - start:.4f}秒")
        return result
    return wrapper

@timer
def sort_list(n):
    return sorted([i for i in range(n, 0, -1)])

sort_list(10000)  # sort_list 执行耗时: 0.0023秒
```

### 3. 缓存装饰器（记忆化）

```python
import functools

def memoize(func):
    """缓存函数结果的装饰器"""
    cache = {}
    
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        # 创建可哈希的键
        key = str(args) + str(kwargs)
        
        if key not in cache:
            print(f"计算 {func.__name__}({args}, {kwargs})")
            cache[key] = func(*args, **kwargs)
        else:
            print(f"从缓存读取 {func.__name__}({args}, {kwargs})")
        
        return cache[key]
    return wrapper

@memoize
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

print(fibonacci(10))  # 使用缓存后，计算快很多
# 第一次计算并缓存每个值
# 后续从缓存读取
```

### 4. 参数验证装饰器

```python
import functools

def validate_args(**validators):
    """验证函数参数的装饰器"""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            # 获取函数参数名
            import inspect
            sig = inspect.signature(func)
            bound = sig.bind(*args, **kwargs)
            bound.apply_defaults()
            
            # 验证每个参数
            for param_name, validator in validators.items():
                value = bound.arguments.get(param_name)
                if not validator(value):
                    raise ValueError(f"参数 {param_name} 验证失败: {value}")
            
            return func(*args, **kwargs)
        return wrapper
    return decorator

@validate_args(age=lambda x: x >= 0, name=lambda x: len(x) > 0)
def create_user(name, age):
    return {"name": name, "age": age}

create_user("Alice", 25)  # 正常
# create_user("", -5)     # ValueError: 参数 age 验证失败
```

### 5. 重试装饰器

```python
import functools
import time

def retry(max_attempts=3, delay=1, exceptions=(Exception,)):
    """重试装饰器"""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            attempts = 0
            while attempts < max_attempts:
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    attempts += 1
                    if attempts >= max_attempts:
                        raise
                    print(f"尝试 {attempts} 失败，重试中... ({e})")
                    time.sleep(delay)
        return wrapper
    return decorator

@retry(max_attempts=3, delay=1, exceptions=(ConnectionError,))
def fetch_data():
    # 模拟不稳定操作
    import random
    if random.random() < 0.7:
        raise ConnectionError("网络不稳定")
    return "数据获取成功"

fetch_data()  # 可能会重试几次后成功
```

### 6. 权限验证装饰器

```python
import functools

class User:
    def __init__(self, name, role):
        self.name = name
        self.role = role

def require_permission(permission):
    """权限验证装饰器"""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            # 从参数或全局获取当前用户
            user = kwargs.get('user') or (args[0] if args else None)
            
            if not user or not hasattr(user, 'role'):
                raise PermissionError("需要登录")
            
            if user.role != permission and user.role != 'admin':
                raise PermissionError(f"需要 {permission} 权限")
            
            return func(*args, **kwargs)
        return wrapper
    return decorator

class Document:
    def __init__(self, title):
        self.title = title
    
    @require_permission('editor')
    def edit(self, user, content):
        print(f"{user.name} 编辑了文档: {content}")
    
    @require_permission('viewer')
    def view(self, user):
        print(f"{user.name}查看了文档: {self.title}")

admin = User("Admin", "admin")
editor = User("Alice", "editor")
viewer = User("Bob", "viewer")

doc = Document("报告")

doc.edit(user=editor, content="新内容")  # 可以
doc.edit(user=viewer, content="新内容")  # PermissionError

doc.view(user=viewer)  # 可以
doc.view(user=editor)  # 可以（admin 权限）
```

## functools 模块详解

### wraps 函数

```python
import functools

# functools.wraps(wrapped, assigned=WRAPPER_ASSIGNMENTS, updated=WRAPPER_UPDATES)

# 默认复制的属性
WRAPPER_ASSIGNMENTS = (
    '__module__', '__name__', '__qualname__', 
    '__annotations__', '__doc__'
)

# 默认更新的属性
WRAPPER_UPDATES = ('__dict__',)

# 示例：自定义复制的属性
def my_decorator(func):
    @functools.wraps(func, assigned=('__name__', '__doc__'))
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

### update_wrapper

```python
# wraps 实际上是 update_wrapper 的装饰器版本
def my_decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return functools.update_wrapper(wrapper, func)
```

### lru_cache

```python
import functools

# LRU (Least Recently Used) 缓存
@functools.lru_cache(maxsize=128)
def expensive_computation(n):
    print(f"计算 {n}...")
    return n * n

expensive_computation(10)  # 计算 10...
expensive_computation(10)  # 使用缓存，不再打印

# 使用参数
@functools.lru_cache(maxsize=None)  # 无限制缓存
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

# 查看缓存状态
print(fibonacci.cache_info())
# CacheInfo(hits=5, misses=11, maxsize=128, currsize=11)
```

## 装饰器在框架中的应用

### Flask 风格

```python
# Flask 中的装饰器用法示例
from functools import wraps

app = {}

def route(path):
    """Flask 风格路由装饰器"""
    def decorator(func):
        app[path] = func
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            return func(*args, **kwargs)
        return wrapper
    return decorator

@route('/')
def index():
    return '首页'

@route('/about')
def about():
    return '关于'

print(app)  # {'/': index, '/about': about}
```

### Django 风格

```python
# Django 的 method_decorator 示例
import functools

def login_required(func):
    """登录验证装饰器"""
    @functools.wraps(func)
    def wrapper(self, request, *args, **kwargs):
        if not request.user.is_authenticated:
            return redirect('/login')
        return func(self, request, *args, **kwargs)
    return wrapper

# 使用在类方法上
from django.utils.decorators import method_decorator

class ArticleView:
    @method_decorator(login_required)
    def edit(self, request):
        pass
```

## 装饰器的常见问题

### 1. 装饰器改变函数签名

```python
# 问题
@decorator
def func(x, y):
    return x + y

import inspect
print(inspect.signature(func))  # 显示 wrapper 的签名

# 解决
import functools

def decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    # 或手动更新签名
    wrapper.__signature__ = inspect.signature(func)
    return wrapper
```

### 2. 装饰器带可选参数

```python
# 问题：如何让装饰器参数可选？
def decorator(func=None, *, option=True):
    def actual_decorator(f):
        @functools.wraps(f)
        def wrapper(*args, **kwargs):
            if option:
                print("选项启用")
            return f(*args, **kwargs)
        return wrapper
    
    if func is None:
        return actual_decorator
    else:
        return actual_decorator(func)

# 使用方式一：带参数
@decorator(option=False)
def func1():
    pass

# 使用方式二：不带参数
@decorator
def func2():
    pass

# 使用方式三：括号调用
@decorator()
def func3():
    pass
```

### 3. 类方法装饰器

```python
class MyClass:
    def __init__(self):
        self._value = 0
    
    @property
    @decorator
    def value(self):
        """正确的装饰顺序"""
        return self._value
    
    # @decorator  # 错误顺序
    # @property
    # def value(self):
    #     return self._value
```

## 总结

### 装饰器语法总结

```python
# 基本装饰器
@decorator
def func():
    pass

# 等价于
def func():
    pass
func = decorator(func)

# 带参数的装饰器
@decorator(arg)
def func():
    pass

# 等价于
def func():
    pass
func = decorator(arg)(func)

# 多层装饰器
@decorator1
@decorator2
def func():
    pass

# 等价于
func = decorator1(decorator2(func))
```

### 装饰器执行顺序

1. 先执行最靠近函数的装饰器
2. 由内到外执行
3. 调用函数时反向执行

### 最佳实践

1. **始终使用 `@functools.wraps`** - 保留原函数元数据
2. **使用 `*args, **kwargs`** - 传递任意参数
3. **装饰器工厂函数** - 实现带参数的装饰器
4. **类装饰器** - 适合需要保存状态的装饰器
5. **文档说明** - 为装饰器添加清晰的文档

### 常用场景

| 场景 | 装饰器类型 |
|------|-----------|
| 日志记录 | 函数装饰器 |
| 性能计时 | 函数装饰器 |
| 缓存 | 函数装饰器 |
| 权限验证 | 函数/类装饰器 |
| 参数验证 | 函数装饰器 |
| 重试机制 | 函数装饰器 |
| 路由注册 | 函数装饰器 |

装饰器是 Python 中非常强大的特性，掌握它可以让你的代码更加简洁、可复用和优雅！

---

> 📚 推荐阅读
>
> - [PEP 318 - Decorators](https://peps.python.org/pep-0318/)
> - [Python decorators documentation](https://docs.python.org/3/library/functools.html)
> - [Real Python: Decorators](https://realpython.com/decorators/)
