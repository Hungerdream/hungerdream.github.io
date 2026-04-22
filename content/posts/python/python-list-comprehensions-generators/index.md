---
title: "Python 列表推导式与生成器：高效编写代码的秘密"
date: 2026-04-22
draft: false
featuredImage: "img/wallhaven-l8671l.png"
tags: ["Python", "列表推导式", "生成器", "Pythonic"]
categories: ["Python"]
---


Python 以其简洁优雅的语法著称，而列表推导式（List Comprehension）和生成器（Generator）是 Pythonic 编程风格的典型代表。掌握这些技巧，不仅能让你的代码更简洁，还能显著提升性能。

本文将深入讲解列表推导式、集合推导式、字典推导式以及生成器的用法和最佳实践。

## 什么是列表推导式？

列表推导式是一种简洁创建列表的方式。它的基本语法是：

```python
[表达式 for 变量 in 可迭代对象]
```

### 传统方式 vs 列表推导式

```python
# 传统方式 - 创建 0-9 的平方列表
squares = []
for i in range(10):
    squares.append(i ** 2)
print(squares)  # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# 列表推导式 - 一行搞定
squares = [i ** 2 for i in range(10)]
print(squares)  # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```

### 性能对比

```python
import timeit

# 传统循环
def traditional():
    result = []
    for i in range(1000):
        result.append(i ** 2)
    return result

# 列表推导式
def comprehension():
    return [i ** 2 for i in range(1000)]

# 测试性能
t1 = timeit.timeit(traditional, number=1000)
t2 = timeit.timeit(comprehension, number=1000)

print(f"传统循环: {t1:.4f}秒")
print(f"列表推导式: {t2:.4f}秒")
print(f"列表推导式快 {(t1-t2)/t1*100:.1f}%")
```

## 基础列表推导式

### 简单示例

```python
# 基本语法
numbers = [1, 2, 3, 4, 5]

# 每个元素乘以 2
doubled = [x * 2 for x in numbers]
print(doubled)  # [2, 4, 6, 8, 10]

# 字符串处理
words = ['hello', 'world', 'python']
upper_words = [word.upper() for word in words]
print(upper_words)  # ['HELLO', 'WORLD', 'PYTHON']

# 数学运算
numbers = [1, 4, 9, 16, 25]
roots = [n ** 0.5 for n in numbers]
print(roots)  # [1.0, 2.0, 3.0, 4.0, 5.0]
```

### 处理各种数据结构

```python
# 处理元组
coordinates = [(1, 2), (3, 4), (5, 6)]
x_coords = [x for x, y in coordinates]
y_coords = [y for x, y in coordinates]
print(x_coords)  # [1, 3, 5]
print(y_coords)  # [2, 4, 6]

# 处理字典
students = {'Alice': 90, 'Bob': 85, 'Charlie': 92}
names = [name for name in students.keys()]
scores = [score for score in students.values()]
print(names)   # ['Alice', 'Bob', 'Charlie']
print(scores)  # [90, 85, 92]

# 处理字符串
sentence = "Python is awesome"
letters = [char for char in sentence]
print(letters)  # ['P', 'y', 't', 'h', 'o', 'n', ' ', 'i', 's', ' ', 'a', 'w', 'e', 's', 'o', 'm', 'e']
```

## 带条件的列表推导式

### 只包含满足条件的元素

```python
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# 只保留偶数
even_numbers = [x for x in numbers if x % 2 == 0]
print(even_numbers)  # [2, 4, 6, 8, 10]

# 只保留大于 5 的数
large_numbers = [x for x in numbers if x > 5]
print(large_numbers)  # [6, 7, 8, 9, 10]

# 只保留平方是偶数的数
numbers = [1, 2, 3, 4, 5]
result = [x**2 for x in numbers if x**2 % 2 == 0]
print(result)  # [4, 16]
```

### 多重条件

```python
# and 条件
numbers = range(1, 21)
result = [x for x in numbers if x > 5 if x < 15 if x % 2 == 0]
print(result)  # [6, 8, 10, 12, 14]

# 等同于
result = [x for x in numbers if x > 5 and x < 15 and x % 2 == 0]
print(result)  # [6, 8, 10, 12, 14]
```

### if-else 表达式

```python
# 在表达式中使用条件（三元表达式）
numbers = [1, 2, 3, 4, 5]

# 如果是偶数则乘以 2，否则保持原值
result = [x * 2 if x % 2 == 0 else x for x in numbers]
print(result)  # [1, 4, 3, 8, 5]

# 将数字分类
result = ['偶数' if x % 2 == 0 else '奇数' for x in numbers]
print(result)  # ['奇数', '偶数', '奇数', '偶数', '奇数']
```

## 嵌套循环

### 扁平化嵌套列表

```python
# 二维列表扁平化
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat = [num for row in matrix for num in row]
print(flat)  # [1, 2, 3, 4, 5, 6, 7, 8, 9]

# 传统方式
flat = []
for row in matrix:
    for num in row:
        flat.append(num)
```

### 生成所有组合

```python
# 扑克牌示例
ranks = ['A', '2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K']
suits = ['♠', '♥', '♦', '♣']

cards = [f"{rank}{suit}" for rank in ranks for suit in suits]
print(cards[:5])  # ['A♠', 'A♥', 'A♦', 'A♣', '2♠']

# 坐标组合
x = [1, 2, 3]
y = [4, 5]
points = [(xi, yi) for xi in x for yi in y]
print(points)  # [(1, 4), (1, 5), (2, 4), (2, 5), (3, 4), (3, 5)]
```

## 集合推导式

集合推导式语法与列表推导式类似，但使用花括号 `{}`，且结果自动去重。

```python
# 基础集合推导式
numbers = [1, 2, 2, 3, 3, 3, 4, 4, 4, 4]

# 平方后放入集合（自动去重）
squares = {x ** 2 for x in numbers}
print(squares)  # {1, 4, 9, 16}

# 从字符串提取唯一字符
text = "hello world"
unique_chars = {char for char in text if char != ' '}
print(unique_chars)  # {'h', 'e', 'l', 'o', 'w', 'r', 'd'}

# 带条件的集合推导式
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# 偶数的平方
even_squares = {x**2 for x in numbers if x % 2 == 0}
print(even_squares)  # {4, 16, 36, 64, 100}
```

### 集合运算

```python
# 使用集合推导式进行集合运算
a = [1, 2, 3, 4, 5]
b = [4, 5, 6, 7, 8]

# 交集
intersection = {x for x in a if x in b}
print(intersection)  # {4, 5}

# 并集
union = {x for x in a}.union({x for x in b})
print(union)  # {1, 2, 3, 4, 5, 6, 7, 8}

# 差集
diff = {x for x in a if x not in b}
print(diff)  # {1, 2, 3}
```

## 字典推导式

字典推导式使用 `{}` 和冒号 `:`，可以快速创建字典。

### 基础字典推导式

```python
# 基础语法：{key: value for ...}

# 数字到平方的映射
numbers = [1, 2, 3, 4, 5]
square_dict = {x: x ** 2 for x in numbers}
print(square_dict)  # {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}

# 字符串到长度的映射
words = ['apple', 'banana', 'cherry']
word_lengths = {word: len(word) for word in words}
print(word_lengths)  # {'apple': 5, 'banana': 6, 'cherry': 6}

# 反转字典（值作为键，键作为值）
original = {'a': 1, 'b': 2, 'c': 3}
reversed_dict = {v: k for k, v in original.items()}
print(reversed_dict)  # {1: 'a', 2: 'b', 3: 'c'}
```

### 带条件的字典推导式

```python
# 只包含满足条件的键值对
students = {'Alice': 90, 'Bob': 75, 'Charlie': 85, 'David': 95}

# 只保留分数大于等于 80 的学生
passed = {name: score for name, score in students.items() if score >= 80}
print(passed)  # {'Alice': 90, 'Charlie': 85, 'David': 95}

# 基于条件转换值
products = {'apple': 5, 'banana': 3, 'cherry': 10, 'date': 7}

# 价格翻倍
doubled_prices = {name: price * 2 for name, price in products.items()}
print(doubled_prices)  # {'apple': 10, 'banana': 6, 'cherry': 20, 'date': 14}

# 价格大于 5 的商品打 8 折
discounted = {name: price * 0.8 for name, price in products.items() if price > 5}
print(discounted)  # {'apple': 4.0, 'cherry': 8.0, 'date': 5.6}
```

### 复杂字典推导式

```python
# 两个列表组合成字典
keys = ['name', 'age', 'city']
values = ['Alice', 25, 'Beijing']

# 方法一：zip
info = {k: v for k, v in zip(keys, values)}
print(info)  # {'name': 'Alice', 'age': 25, 'city': 'Beijing'}

# 方法二：索引
info = {keys[i]: values[i] for i in range(len(keys))}
print(info)  # {'name': 'Alice', 'age': 25, 'city': 'Beijing'}

# 创建嵌套字典
students = ['Alice', 'Bob', 'Charlie']
grades = [90, 85, 92]

student_grades = {student: {'grade': grade, 'status': 'passed' if grade >= 60 else 'failed'}
                  for student, grade in zip(students, grades)}
print(student_grades)
# {'Alice': {'grade': 90, 'status': 'passed'},
#  'Bob': {'grade': 85, 'status': 'passed'},
#  'Charlie': {'grade': 92, 'status': 'passed'}}
```

## 生成器表达式

生成器表达式与列表推导式语法类似，但使用圆括号 `()` 而不是方括号 `[]`。

### 为什么要用生成器？

```python
import sys

# 列表：一次性生成所有元素，占用内存
list_comp = [x ** 2 for x in range(1000000)]
print(f"列表大小: {sys.getsizeof(list_comp)} bytes")  # 约 8MB

# 生成器：按需生成，不占内存
gen_exp = (x ** 2 for x in range(1000000))
print(f"生成器大小: {sys.getsizeof(gen_exp)} bytes")  # 约 200 bytes
```

### 生成器表达式

```python
# 语法： (表达式 for 变量 in 可迭代对象)

# 创建生成器
gen = (x ** 2 for x in range(5))
print(gen)  # <generator object <genexpr> at 0x...>

# 使用 next() 获取元素
print(next(gen))  # 0
print(next(gen))  # 1
print(next(gen))  # 4

# 遍历生成器
gen = (x ** 2 for x in range(5))
for value in gen:
    print(value)
# 0
# 1
# 4
# 9
# 16

# 转换为列表（会占用内存）
gen = (x ** 2 for x in range(5))
result = list(gen)
print(result)  # [0, 1, 4, 9, 16]
```

### 生成器 vs 列表推导式

```python
import timeit

# 列表推导式：返回完整列表
def using_list():
    return [x ** 2 for x in range(1000000)]

# 生成器表达式：返回生成器对象
def using_generator():
    return (x ** 2 for x in range(1000000))

t1 = timeit.timeit(using_list, number=1)
t2 = timeit.timeit(using_generator, number=1)

print(f"列表: {t1:.4f}秒")
print(f"生成器: {t2:.6f}秒")  # 几乎瞬间完成
```

## 生成器函数

生成器函数使用 `yield` 关键字，可以产生多个值。

### 基础生成器函数

```python
def count_up_to(n):
    """计数生成器"""
    count = 1
    while count <= n:
        yield count
        count += 1

# 使用
counter = count_up_to(5)
print(next(counter))  # 1
print(next(counter))  # 2
print(next(counter))  # 3

# 或者遍历
for num in count_up_to(5):
    print(num)
# 1
# 2
# 3
# 4
# 5
```

### yield 与 return

```python
def with_return(n):
    """使用 return - 函数结束"""
    for i in range(n):
        return i  # 只返回第一个值

def with_yield(n):
    """使用 yield - 函数变成生成器"""
    for i in range(n):
        yield i  # 每次返回一个值，函数暂停

# return 版本
result = with_return(5)
print(result)  # 0

# yield 版本
result = with_yield(5)
print(result)  # <generator object>
print(list(result))  # [0, 1, 2, 3, 4]
```

### 无限生成器

```python
# 无限序列生成器
def fibonacci():
    """斐波那契数列生成器"""
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

# 只获取前 10 个
fib = fibonacci()
for _ in range(10):
    print(next(fib))
# 0
# 1
# 1
# 2
# 3
# 5
# 8
# 13
# 21
# 34

def infinite_range(start=0, step=1):
    """无限整数序列"""
    while True:
        yield start
        start += step

# 偶数序列
evens = infinite_range(step=2)
for _ in range(10):
    print(next(evens))
# 0
# 2
# 4
# 6
# 8
# 10
# 12
# 14
# 16
# 18
```

### 生成器的高级用法

```python
def find_lines(filename):
    """逐行读取大文件"""
    with open(filename, 'r', encoding='utf-8') as f:
        for line in f:
            yield line.strip()

# 使用
for line in find_lines('huge_file.txt'):
    process(line)  # 每次只处理一行，不会把整个文件加载到内存

def batch_process(items, batch_size=100):
    """批量处理数据"""
    batch = []
    for item in items:
        batch.append(item)
        if len(batch) >= batch_size:
            yield batch
            batch = []
    if batch:
        yield batch

# 使用
for batch in batch_process(range(250), batch_size=100):
    print(f"处理批次: {batch}")
# 处理批次: [0, 1, 2, ..., 99]
# 处理批次: [100, 101, 102, ..., 199]
# 处理批次: [200, 201, 202, ..., 249]
```

## 生成器的状态

生成器有三种状态方法：

```python
def simple_gen():
    yield 1
    yield 2
    yield 3

gen = simple_gen()

print(next(gen))  # 1
print(next(gen))  # 2
print(next(gen))  # 3

try:
    print(next(gen))  # 抛出 StopIteration
except StopIteration:
    print("生成器耗尽")

# 重新使用生成器
gen = simple_gen()
result = list(gen)  # 消耗所有元素
print(result)  # [1, 2, 3]

# 再次使用
gen = simple_gen()
print(next(gen, '默认值'))  # 1 - 有默认值，不会抛出异常
print(next(gen, '默认值'))  # 2
print(next(gen, '默认值'))  # 3
print(next(gen, '默认值'))  # 默认值 - 提供默认值
```

## yield from

`yield from` 用于委托给子生成器：

```python
def gen1():
    yield 1
    yield 2

def gen2():
    yield 3
    yield 4

# 传统方式
def combined_traditional():
    for item in gen1():
        yield item
    for item in gen2():
        yield item

# 使用 yield from
def combined_yield_from():
    yield from gen1()
    yield from gen2()

# 测试
print(list(combined_yield_from()))  # [1, 2, 3, 4]

# 扁平化嵌套列表
def flatten(nested):
    for item in nested:
        if isinstance(item, (list, tuple)):
            yield from flatten(item)  # 递归展平
        else:
            yield item

nested = [1, [2, 3], [4, [5, 6]], 7]
print(list(flatten(nested)))  # [1, 2, 3, 4, 5, 6, 7]
```

## 实用案例

### 1. 读取大文件

```python
def read_large_file(filepath, chunk_size=1024):
    """分块读取大文件"""
    with open(filepath, 'rb') as f:
        while True:
            chunk = f.read(chunk_size)
            if not chunk:
                break
            yield chunk

# 使用示例
for chunk in read_large_file('huge_file.bin', chunk_size=8192):
    process(chunk)
```

### 2. 实时数据处理

```python
def stock_price_stream():
    """模拟股票价格流"""
    import random
    price = 100
    while True:
        price += random.uniform(-2, 2)
        yield price

# 获取价格突破阈值的事件
def price_alerts(threshold):
    price_gen = stock_price_stream()
    for price in price_gen:
        if abs(price - 100) > threshold:
            yield price, abs(price - 100)

for price, deviation in price_alerts(5):
    print(f"价格异常: {price:.2f}, 偏离: {deviation:.2f}")
```

### 3. 组合多个迭代器

```python
def interleave(*iterables):
    """交替合并多个迭代器"""
    iterators = [iter(it) for it in iterables]
    while iterators:
        for i, it in enumerate(iterators):
            try:
                yield next(it)
            except StopIteration:
                iterators.pop(i)
                break

# 使用
list1 = [1, 2, 3]
list2 = ['a', 'b', 'c']
list3 = [True, False]

result = list(interleave(list1, list2, list3))
print(result)  # [1, 'a', True, 2, 'b', False, 3, 'c']
```

## 性能优化技巧

### 何时使用列表推导式

```python
# ✅ 列表推导式：需要多次访问列表
numbers = [1, 2, 3, 4, 5]

# 好：一次性创建列表
squares = [x ** 2 for x in numbers]
for square in squares:  # 多次遍历
    print(square)
```

### 何时使用生成器

```python
# ✅ 生成器：数据量大，只遍历一次
numbers = range(1, 1000000)

# 好：使用生成器节省内存
squares = (x ** 2 for x in numbers)
for square in squares:  # 只需遍历一次
    print(square)
```

### 避免常见陷阱

```python
# 陷阱：循环变量泄漏
x = 'original'
result = [x for x in range(5)]  # x 变成了 4！
print(x)  # 4

# 正确：使用不同的变量名
y = 'original'
result = [x for x in range(5)]  # y 保持不变
print(y)  # 'original'

# 陷阱：重复使用生成器
gen = (x for x in range(5))
print(list(gen))  # [0, 1, 2, 3, 4]
print(list(gen))  # [] - 生成器已耗尽

# 正确：需要时重新创建
def gen_func():
    return (x for x in range(5))
print(list(gen_func()))  # [0, 1, 2, 3, 4]
print(list(gen_func()))  # [0, 1, 2, 3, 4]
```

## Pythonic 代码风格

### 推荐的写法

```python
# ✅ 简洁的过滤和转换
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# 获取偶数的平方
even_squares = [x**2 for x in numbers if x % 2 == 0]
print(even_squares)  # [4, 16, 36, 64, 100]

# 字典推导式
students = {'Alice': 90, 'Bob': 85, 'Charlie': 92}
passed = {name: score for name, score in students.items() if score >= 90}
print(passed)  # {'Alice': 90, 'Charlie': 92}

# 生成器用于大数据
data = range(1000000)
result = sum(x ** 2 for x in data)  # 不创建中间列表

# 使用 enumerate 获取索引
words = ['hello', 'world', 'python']
indexed = [(i, word) for i, word in enumerate(words)]

# 使用 zip 组合多个列表
names = ['Alice', 'Bob', 'Charlie']
ages = [25, 30, 35]
combined = {name: age for name, age in zip(names, ages)}
```

### 不推荐的写法

```python
# ❌ 嵌套过深的推导式
matrix = [[1, 2], [3, 4], [5, 6]]
result = [[cell for cell in row if cell % 2 == 0] for row in matrix if sum(row) > 4]
# 太复杂，难以理解

# 好：拆分成多步
filtered_rows = [row for row in matrix if sum(row) > 4]
result = [[cell for cell in row if cell % 2 == 0] for row in filtered_rows]
```

## 总结

| 类型 | 语法 | 特点 |
|------|------|------|
| 列表推导式 | `[x for x in items]` | 返回列表，占用内存 |
| 集合推导式 | `{x for x in items}` | 自动去重 |
| 字典推导式 | `{k: v for k, v in items}` | 创建字典 |
| 生成器表达式 | `(x for x in items)` | 惰性求值，节省内存 |
| 生成器函数 | `def gen(): yield x` | 可包含复杂逻辑 |

### 选择指南

- **列表推导式**：需要多次遍历，数据量不大
- **生成器表达式**：只遍历一次，数据量大
- **生成器函数**：需要复杂逻辑或状态
- **字典/集合推导式**：快速创建字典/集合

掌握这些技巧让你的 Python 代码更加简洁、高效和 Pythonic！

---

> 📚 推荐阅读
>
> - [PEP 202 - List Comprehensions](https://peps.python.org/pep-0202/)
> - [PEP 289 - Generator Expressions](https://peps.python.org/pep-0289/)
> - [Python Wiki: List Comprehensions](https://wiki.python.org/moin/ListComprehensions)
