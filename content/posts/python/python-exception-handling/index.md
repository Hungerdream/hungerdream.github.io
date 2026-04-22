---
title: "Python 异常处理完全指南：try/except/finally 实战"
date: 2026-04-22
draft: false
featuredImage: "img/wallhaven-l8671l.png"
tags: ["Python", "异常处理", "调试", "错误处理"]
categories: ["Python"]
---


程序运行中难免会遇到各种错误：文件不存在、网络连接失败、数据格式错误...如何优雅地处理这些错误，而不是让程序直接崩溃？Python 的异常处理机制就是答案。

本文将全面讲解 Python 的异常处理，从基础语法到高级技巧。

## 为什么需要异常处理？

### 没有异常处理的问题

```python
# 没有异常处理的代码
def read_file(filename):
    with open(filename, 'r') as f:  # 如果文件不存在，程序直接崩溃
        return f.read()

# 调用
content = read_file('not_exists.txt')  # FileNotFoundError!
print("这段代码不会执行")
```

### 有异常处理的好处

```python
# 有异常处理的代码
def read_file(filename):
    try:
        with open(filename, 'r') as f:
            return f.read()
    except FileNotFoundError:
        return "文件不存在"

# 调用
content = read_file('not_exists.txt')  # 优雅处理
print(f"结果: {content}")  # "文件不存在"
```

## 基础语法

### try-except 结构

```python
try:
    # 可能出错的代码
    result = 10 / 0
except ZeroDivisionError:
    # 处理特定异常
    print("不能除以零!")
```

### 基本结构

```python
try:
    # 可能抛出异常的代码
    risky_code()
except ExceptionType1:
    # 处理 ExceptionType1
    handler1()
except ExceptionType2:
    # 处理 ExceptionType2
    handler2()
else:
    # try 代码块执行成功时执行（可选）
    success_handler()
finally:
    # 无论是否异常都执行（可选）
    cleanup()
```

## 捕获异常

### 捕获特定异常

```python
# 捕获单个异常
try:
    num = int(input("请输入数字: "))
    result = 10 / num
except ValueError:
    print("输入的不是有效数字")
except ZeroDivisionError:
    print("不能除以零")

# 捕获多个异常（元组形式）
try:
    with open('file.txt', 'r') as f:
        data = f.read()
except (FileNotFoundError, PermissionError):
    print("文件无法访问")
```

### 捕获所有异常

```python
# 方式一：捕获 Exception
try:
    result = risky_operation()
except Exception as e:
    print(f"发生错误: {e}")

# 方式二：使用 except:（不推荐，不知道具体异常）
try:
    result = risky_operation()
except:
    print("发生错误")

# 方式三：捕获 BaseException（包含 SystemExit 等）
try:
    sys.exit()
except BaseException as e:
    print(f"捕获到: {type(e).__name__}")
```

### 异常对象

```python
try:
    x = 1 / 0
except ZeroDivisionError as e:
    # 异常对象属性
    print(f"异常类型: {type(e).__name__}")  # ZeroDivisionError
    print(f"异常消息: {e}")                  # division by zero
    print(f"异常描述: {str(e)}")             # division by zero
```

## 常见异常类型

### Python 内置异常

```python
# 常见异常类型及示例

# ValueError - 值错误
int("abc")           # ValueError: invalid literal for int()

# TypeError - 类型错误
"1" + 1             # TypeError: can only concatenate str

# IndexError - 索引错误
lst = [1, 2, 3]
lst[10]              # IndexError: list index out of range

# KeyError - 字典键错误
d = {'a': 1}
d['b']               # KeyError: 'b'

# FileNotFoundError - 文件不存在
open('not_exists.txt')

# AttributeError - 属性错误
[].append           # 正常
[].nonexistent      # AttributeError

# ImportError - 导入错误
import nonexistent_module

# StopIteration - 迭代器耗尽
it = iter([1, 2])
next(it); next(it); next(it)  # StopIteration

# AssertionError - 断言失败
assert 1 == 2

# KeyboardInterrupt - 用户中断
# Ctrl+C 触发

# SystemExit - 程序退出
import sys
sys.exit()
```

### 异常层次结构

```
BaseException
├── SystemExit
├── KeyboardInterrupt
├── GeneratorExit
└── Exception
    ├── StopIteration
    ├── ArithmeticError
    │   ├── FloatingPointError
    │   ├── OverflowError
    │   └── ZeroDivisionError
    ├── LookupError
    │   ├── IndexError
    │   └── KeyError
    ├── OSError (IOError)
    │   ├── FileNotFoundError
    │   └── PermissionError
    ├── TypeError
    ├── ValueError
    └── ...
```

## raise 语句

### 抛出异常

```python
# 基本语法
raise ExceptionType("错误消息")

# 示例
def validate_age(age):
    if age < 0:
        raise ValueError("年龄不能为负数")
    if age > 150:
        raise ValueError("年龄不合理")
    return age

try:
    validate_age(-5)
except ValueError as e:
    print(e)  # 年龄不能为负数
```

### 重新抛出异常

```python
def read_config():
    try:
        with open('config.json', 'r') as f:
            return json.load(f)
    except FileNotFoundError:
        # 重新包装异常
        raise RuntimeError("配置文件不存在，请检查路径") from None
    except json.JSONDecodeError as e:
        # 保留原始异常链
        raise RuntimeError("配置文件格式错误") from e

try:
    read_config()
except RuntimeError as e:
    print(f"错误: {e}")
    if e.__cause__:
        print(f"原始原因: {e.__cause__}")
```

### 异常链

```python
# 隐式异常链（异常在 except 块中发生时）
try:
    int("abc")
except ValueError as e:
    try:
        1 / 0
    except ZeroDivisionError:
        raise ValueError("包装错误")  # __cause__ = None, __context__ = ValueError

# 显式异常链
try:
    int("abc")
except ValueError as e:
    raise RuntimeError("新错误") from e  # __cause__ = ValueError
```

## else 子句

`else` 在 try 代码块成功执行后执行：

```python
try:
    with open('data.txt', 'r') as f:
        data = f.read()
except FileNotFoundError:
    data = None
    print("文件不存在，使用默认值")
else:
    print("文件读取成功!")  # 只在成功时执行
    print(f"文件长度: {len(data)}")

print("程序继续执行")
```

## finally 子句

`finally` 无论是否异常都执行，常用于清理资源：

### 基本用法

```python
try:
    file = open('data.txt', 'w')
    file.write("Hello")
except Exception as e:
    print(f"错误: {e}")
finally:
    # 无论如何都会执行
    print("清理资源")
    file.close()  # 确保文件关闭
```

### 与 context manager 对比

```python
# 方式一：手动关闭
file = None
try:
    file = open('data.txt', 'r')
    content = file.read()
finally:
    if file:
        file.close()

# 方式二：context manager（推荐）
with open('data.txt', 'r') as file:
    content = file.read()
# 自动关闭，无需 finally
```

## 资源清理模式

### 数据库连接

```python
import sqlite3

# 不推荐的写法
conn = sqlite3.connect('test.db')
try:
    cursor = conn.cursor()
    cursor.execute('SELECT * FROM users')
    result = cursor.fetchall()
finally:
    conn.close()  # 确保关闭

# 推荐的写法（Python 3.7+）
conn = sqlite3.connect('test.db')
try:
    cursor = conn.cursor()
    cursor.execute('SELECT * FROM users')
    result = cursor.fetchall()
except Exception as e:
    conn.rollback()
    raise
else:
    conn.commit()
finally:
    conn.close()
```

### 网络请求

```python
import requests

def fetch_data(url):
    session = requests.Session()
    try:
        response = session.get(url, timeout=10)
        response.raise_for_status()
        return response.json()
    except requests.RequestException as e:
        raise RuntimeError(f"请求失败: {e}") from e
    finally:
        session.close()
```

### 锁的释放

```python
import threading

lock = threading.Lock()

def thread_safe_operation():
    lock.acquire()
    try:
        # 临界区代码
        shared_resource += 1
    finally:
        lock.release()  # 确保锁被释放

# 或者使用 context manager
from contextlib import contextmanager

@contextmanager
def acquired(lock):
    lock.acquire()
    try:
        yield
    finally:
        lock.release()

def thread_safe_operation():
    with acquired(lock):
        shared_resource += 1
```

## 自定义异常

### 创建自定义异常

```python
# 基础自定义异常
class ValidationError(Exception):
    """验证错误异常"""
    pass

def validate_email(email):
    if '@' not in email:
        raise ValidationError(f"无效的邮箱地址: {email}")

try:
    validate_email("invalid_email")
except ValidationError as e:
    print(f"验证失败: {e}")

# 带详细信息的异常
class APIError(Exception):
    """API 错误"""
    def __init__(self, message, status_code=None, response=None):
        super().__init__(message)
        self.status_code = status_code
        self.response = response

# 使用自定义异常
import requests

def get_user(user_id):
    response = requests.get(f'https://api.example.com/users/{user_id}')
    if response.status_code == 404:
        raise APIError(
            f"用户 {user_id} 不存在",
            status_code=404,
            response=response.text
        )
    return response.json()
```

### 异常类最佳实践

```python
# 推荐：继承自具体异常而不是 Exception
class NetworkError(ConnectionError):
    """网络连接错误"""
    pass

class TimeoutError(ConnectionError):
    """请求超时错误"""
    pass

# 添加额外属性
class ConfigurationError(Exception):
    """配置错误"""
    def __init__(self, message, config_key=None, config_value=None):
        super().__init__(message)
        self.config_key = config_key
        self.config_value = config_value

    def __str__(self):
        msg = super().__str__()
        if self.config_key:
            msg += f" (key={self.config_key}, value={self.config_value})"
        return msg

try:
    raise ConfigurationError("配置值无效", "database_url", "invalid")
except ConfigurationError as e:
    print(f"{e} - 键: {e.config_key}, 值: {e.config_value}")
```

## 异常处理最佳实践

### 1. 精准捕获异常

```python
# ❌ 过于宽泛
try:
    do_something()
except Exception:
    pass

# ✅ 精准捕获
try:
    do_something()
except ValueError:
    handle_value_error()
except FileNotFoundError:
    handle_file_not_found()
except Exception:
    handle_unexpected_error()  # 最后捕获所有未处理的异常
```

### 2. 异常与业务逻辑分离

```python
# ❌ 异常处理与业务逻辑混杂
def process_order(order):
    try:
        validate_order(order)
        charge_customer(order)
        ship_order(order)
        send_confirmation(order)
    except ValidationError:
        print("订单验证失败")
    except PaymentError:
        print("支付失败")
    except ShippingError:
        print("发货失败")

# ✅ 分离异常处理
def process_order(order):
    # 业务逻辑 - 只处理正常流程
    validate_order(order)
    charge_customer(order)
    ship_order(order)
    send_confirmation(order)

# 异常处理在调用方
try:
    process_order(order)
except ValidationError:
    print("订单验证失败")
except PaymentError:
    print("支付失败")
except ShippingError:
    print("发货失败")
```

### 3. 记录异常而非静默处理

```python
import logging

logger = logging.getLogger(__name__)

try:
    risky_operation()
except Exception as e:
    # ❌ 静默处理
    pass

# ✅ 记录异常
    logger.exception("操作失败")  # 自动包含 traceback
    # 或
    logger.error(f"操作失败: {e}", exc_info=True)
```

### 4. 不要过度使用异常

```python
# ❌ 滥用异常处理控制流程
try:
    if items[10]:
        pass
except IndexError:
    print("索引超出范围")

# ✅ 使用条件判断
if len(items) > 10 and items[10]:
    pass

# ❌ 过度使用 try-except
try:
    value = int(user_input)
except ValueError:
    value = 0

# ✅ 使用默认值
value = int(user_input) if user_input.isdigit() else 0

# 或者
try:
    value = int(user_input)
except ValueError:
    value = int(user_input)  # 尝试默认值
```

## 异常处理与性能

```python
import timeit

# try-except 带来的开销
def with_exception():
    try:
        return 1
    except:
        return 0

def without_exception():
    return 1

# 测试
print(timeit.timeit(with_exception, number=1000000))
print(timeit.timeit(without_exception, number=1000000))

# 经验法则：
# - 异常控制流：开销明显
# - 异常捕获（不发生异常时）：开销很小
# - 只在异常情况使用，不要用于正常流程控制
```

## 常见场景处理

### 1. 文件操作

```python
# 读取文件的多种方式

# 方式一：try-except-finally
file = None
try:
    file = open('data.txt', 'r', encoding='utf-8')
    content = file.read()
except FileNotFoundError:
    print("文件不存在")
    content = None
except UnicodeDecodeError:
    print("编码错误")
    content = None
finally:
    if file:
        file.close()

# 方式二：context manager（推荐）
try:
    with open('data.txt', 'r', encoding='utf-8') as file:
        content = file.read()
except FileNotFoundError:
    content = None
# 文件自动关闭
```

### 2. JSON 解析

```python
import json

# 安全解析 JSON
def safe_json_loads(json_string, default=None):
    try:
        return json.loads(json_string)
    except json.JSONDecodeError as e:
        print(f"JSON 解析错误: {e}")
        return default

# 测试
print(safe_json_loads('{"valid": true}'))      # {'valid': True}
print(safe_json_loads('invalid json'))          # None
print(safe_json_loads('invalid', default={}))  # {}
```

### 3. 字典安全访问

```python
# 安全获取字典值
def safe_get(dictionary, *keys, default=None):
    """安全获取嵌套字典的值"""
    result = dictionary
    for key in keys:
        try:
            result = result[key]
        except (KeyError, TypeError, IndexError):
            return default
    return result

# 使用
data = {'user': {'profile': {'name': 'Alice'}}}
print(safe_get(data, 'user', 'profile', 'name'))       # Alice
print(safe_get(data, 'user', 'address', 'city'))        # None
print(safe_get(data, 'user', 'address', 'city', default='未知'))  # 未知
```

### 4. 类型转换安全

```python
def safe_int(value, default=0):
    """安全转换为整数"""
    try:
        return int(value)
    except (ValueError, TypeError):
        return default

def safe_float(value, default=0.0):
    """安全转换为浮点数"""
    try:
        return float(value)
    except (ValueError, TypeError):
        return default

def safe_bool(value):
    """安全转换为布尔值"""
    if isinstance(value, bool):
        return value
    try:
        return bool(value)
    except (ValueError, TypeError):
        return False

# 测试
print(safe_int("123"))      # 123
print(safe_int("abc"))       # 0
print(safe_int("42.5"))      # 42
print(safe_float("3.14"))    # 3.14
print(safe_float("abc"))     # 0.0
```

### 5. 网络请求

```python
import requests
from requests.exceptions import (
    ConnectionError,
    Timeout,
    HTTPError,
    RequestException
)

def robust_request(url, method='GET', max_retries=3):
    """带重试的健壮请求"""
    for attempt in range(max_retries):
        try:
            if method == 'GET':
                response = requests.get(url, timeout=10)
            elif method == 'POST':
                response = requests.post(url, timeout=10)
            else:
                raise ValueError(f"不支持的 HTTP 方法: {method}")
            
            response.raise_for_status()
            return response.json()
            
        except ConnectionError:
            if attempt < max_retries - 1:
                print(f"连接失败，重试 {attempt + 1}/{max_retries}")
                continue
            raise RuntimeError("无法连接到服务器")
            
        except Timeout:
            if attempt < max_retries - 1:
                print(f"请求超时，重试 {attempt + 1}/{max_retries}")
                continue
            raise RuntimeError("请求超时")
            
        except HTTPError as e:
            if response.status_code == 404:
                raise RuntimeError(f"资源不存在: {url}")
            elif response.status_code >= 500:
                if attempt < max_retries - 1:
                    print(f"服务器错误，重试")
                    continue
            raise RuntimeError(f"HTTP 错误: {e}")
            
        except RequestException as e:
            raise RuntimeError(f"请求失败: {e}")
```

## 异常处理的测试

```python
import pytest

def divide(a, b):
    if b == 0:
        raise ValueError("除数不能为零")
    return a / b

# 测试正常情况
def test_divide_success():
    assert divide(10, 2) == 5

# 测试异常情况
def test_divide_by_zero():
    with pytest.raises(ValueError) as exc_info:
        divide(10, 0)
    assert "除数不能为零" in str(exc_info.value)

# 测试异常属性
def test_custom_exception_attributes():
    class APIError(Exception):
        def __init__(self, message, code):
            super().__init__(message)
            self.code = code

    with pytest.raises(APIError) as exc_info:
        raise APIError("Not Found", 404)
    
    assert exc_info.value.code == 404
```

## 调试技巧

### 使用 traceback 模块

```python
import traceback

try:
    1 / 0
except Exception:
    # 打印完整堆栈
    traceback.print_exc()
    
    # 获取堆栈字符串
    stack_trace = traceback.format_exc()
    print(f"堆栈信息: {stack_trace}")
```

### 使用 logging 模块

```python
import logging

logger = logging.getLogger(__name__)

try:
    do_something()
except Exception:
    logger.exception("操作失败")  # 自动记录完整 traceback
```

### 使用 IPython 调试

```python
# 在异常发生后进入调试器
try:
    do_something()
except Exception:
    import pdb
    pdb.post_mortem()
    
# 或使用 ipdb
# pip install ipdb
try:
    do_something()
except Exception:
    import ipdb
    ipdb.post_mortem()
```

## 总结

### 异常处理关键字

| 关键字 | 作用 |
|--------|------|
| `try` | 包裹可能出错的代码 |
| `except` | 捕获并处理异常 |
| `else` | try 成功执行后运行 |
| `finally` | 无论是否异常都执行 |
| `raise` | 抛出异常 |

### 最佳实践

1. **精准捕获**：只捕获能处理的异常
2. **记录日志**：不要静默处理异常
3. **资源清理**：使用 finally 或 context manager
4. **异常链**：保留原始异常信息
5. **分层处理**：在合适的位置处理异常
6. **不要滥用**：异常用于异常情况，不是流程控制

### 异常处理流程

```
try:
    # 可能抛出异常的代码
    risky_code()
except SpecificError:
    # 处理特定错误
    handle_specific()
except AnotherError:
    # 处理另一种错误
    handle_another()
else:
    # try 成功完成时执行
    success_handler()
finally:
    # 清理资源
    cleanup()
```

掌握异常处理，让你的 Python 程序更加健壮！

---

> 📚 推荐阅读
>
> - [Python 官方文档：错误与异常](https://docs.python.org/zh-cn/3/tutorial/errors.html)
> - [异常处理最佳实践](https://docs.python.org/3/library/exceptions.html)
> - [Real Python: Exception Handling](https://realpython.com/python-exceptions/)
