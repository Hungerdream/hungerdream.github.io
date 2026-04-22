---
title: "Python 日志处理完全指南：logging 模块从入门到精通"
date: 2026-04-22
draft: false
featuredImage: "img/wallhaven-l8671l.png"
tags: ["Python", "logging", "日志", "调试"]
categories: ["Python"]
---


任何程序都需要日志系统来记录运行状态、调试问题和追踪错误。Python 内置的 `logging` 模块提供了完整的日志解决方案，远比 `print()` 语句更专业、更灵活。

本文将带你全面掌握 Python logging 模块，从基础用法到高级配置。

## 为什么不用 print()？

很多初学者习惯用 `print()` 来调试程序，这在小型脚本中是可以的。但对于正式项目，日志系统有不可替代的优势：

| 特性 | print() | logging |
|------|---------|---------|
| 日志级别 | 无 | DEBUG/INFO/WARNING/ERROR/CRITICAL |
| 输出位置 | 仅控制台 | 文件、控制台、网络等 |
| 格式控制 | 简单 | 完全可定制 |
| 性能影响 | 较大 | 可控制 |
| 多线程安全 | 一般 | 安全 |
| 可关闭调试信息 | 困难 | 容易 |

## 日志级别

logging 模块定义了 5 个日志级别，从低到高：

```python
import logging

# 级别由低到高
logging.debug("调试信息 - 开发时使用")
logging.info("一般信息 - 程序运行状态")
logging.warning("警告信息 - 潜在问题")
logging.error("错误信息 - 程序出错")
logging.critical("严重错误 - 严重问题")
```

### 级别对应关系

```python
# 数值越小，级别越低
logging.DEBUG    = 10
logging.INFO     = 20
logging.WARNING  = 30
logging.ERROR    = 40
logging.CRITICAL = 50
```

### 设置日志级别

默认只显示 WARNING 及以上级别的日志：

```python
import logging

# 创建一个 logger
logger = logging.getLogger(__name__)

# 设置级别为 DEBUG（显示所有日志）
logger.setLevel(logging.DEBUG)

# 默认只会显示 WARNING 及以上的日志
# DEBUG 和 INFO 不会显示
```

## 基础配置

### 快速配置

```python
import logging

# 最简单的方式 - 一行配置
logging.basicConfig(level=logging.DEBUG)

# 现在所有级别都会显示
logging.debug("这条会显示")
logging.info("这条也会显示")
```

### basicConfig 常用参数

```python
import logging

logging.basicConfig(
    level=logging.DEBUG,           # 设置级别
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',  # 格式
    datefmt='%Y-%m-%d %H:%M:%S',  # 日期格式
    filename='app.log',            # 输出到文件
    filemode='a',                  # 文件模式：a=追加，w=覆盖
    encoding='utf-8'               # 文件编码
)

logging.debug("这条会写入文件")
```

## Logger 对象

### 创建 Logger

```python
import logging

# 创建命名 logger
logger = logging.getLogger('my_app')
logger.setLevel(logging.DEBUG)

# 也可以用 __name__ 自动获取模块名
logger = logging.getLogger(__name__)
```

### Handler（处理器）

Handler 决定日志输出到哪里：

```python
import logging

logger = logging.getLogger('my_app')
logger.setLevel(logging.DEBUG)

# 控制台处理器
console_handler = logging.StreamHandler()
console_handler.setLevel(logging.INFO)

# 文件处理器
file_handler = logging.FileHandler('app.log', encoding='utf-8')
file_handler.setLevel(logging.DEBUG)

# 添加处理器到 logger
logger.addHandler(console_handler)
logger.addHandler(file_handler)

# 同时输出到控制台和文件
logger.debug("调试信息")  # 只在文件中
logger.info("一般信息")   # 控制台和文件都有
```

### Formatter（格式化器）

```python
import logging

# 创建 formatter
formatter = logging.Formatter(
    '%(asctime)s - %(name)s - %(levelname)s - %(filename)s:%(lineno)d - %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S'
)

# 创建 handler 并设置 formatter
console_handler = logging.StreamHandler()
console_handler.setFormatter(formatter)

# 添加到 logger
logger = logging.getLogger('my_app')
logger.addHandler(console_handler)

logger.info("格式化的日志信息")
```

### Formatter 格式说明

| 格式 | 说明 | 示例 |
|------|------|------|
| %(name)s | Logger 名称 | my_app |
| %(levelname)s | 日志级别 | DEBUG/INFO |
| %(message)s | 日志消息 | 消息内容 |
| %(asctime)s | 时间 | 2024-01-01 10:30:00 |
| %(filename)s | 文件名 | app.py |
| %(funcName)s | 函数名 | main |
| %(lineno)d | 行号 | 42 |
| %(process)d | 进程 ID | 1234 |
| %(thread)d | 线程 ID | 5678 |

### 组合示例

```python
import logging

def setup_logger():
    """配置日志系统"""
    # 创建 logger
    logger = logging.getLogger('my_app')
    logger.setLevel(logging.DEBUG)
    
    # 防止重复添加 handler
    if logger.handlers:
        return logger
    
    # 创建 formatter
    detailed_formatter = logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - [%(filename)s:%(lineno)d] - %(message)s',
        datefmt='%Y-%m-%d %H:%M:%S'
    )
    
    simple_formatter = logging.Formatter(
        '%(levelname)s - %(message)s'
    )
    
    # 控制台处理器 (只显示 INFO 及以上)
    console_handler = logging.StreamHandler()
    console_handler.setLevel(logging.INFO)
    console_handler.setFormatter(simple_formatter)
    
    # 文件处理器 (记录所有级别)
    file_handler = logging.FileHandler('app.log', encoding='utf-8')
    file_handler.setLevel(logging.DEBUG)
    file_handler.setFormatter(detailed_formatter)
    
    # 添加处理器
    logger.addHandler(console_handler)
    logger.addHandler(file_handler)
    
    return logger

# 使用
logger = setup_logger()
logger.debug("调试信息 - 只在文件")
logger.info("一般信息 - 控制台和文件")
logger.warning("警告信息")
logger.error("错误信息")
```

## 日志轮转

日志文件会不断增长，使用 RotatingFileHandler 或 TimedRotatingFileHandler 可以自动管理日志文件。

### 按大小轮转

```python
import logging
from logging.handlers import RotatingFileHandler

logger = logging.getLogger('my_app')
logger.setLevel(logging.DEBUG)

# 10MB 分割，保留 5 个备份
handler = RotatingFileHandler(
    'app.log',
    maxBytes=10 * 1024 * 1024,  # 10MB
    backupCount=5,
    encoding='utf-8'
)
handler.setLevel(logging.DEBUG)

formatter = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)

logger.addHandler(handler)

# 当文件达到 10MB 时，会自动轮转
# 保留 app.log, app.log.1, app.log.2 ... app.log.5
```

### 按时间轮转

```python
import logging
from logging.handlers import TimedRotatingFileHandler

logger = logging.getLogger('my_app')
logger.setLevel(logging.DEBUG)

# 每天午夜轮转，保留 7 天
handler = TimedRotatingFileHandler(
    'app.log',
    when='midnight',      # 何时轮转：midnight, H, D, W0-W6
    interval=1,           # 间隔
    backupCount=7,         # 保留 7 个备份
    encoding='utf-8'
)

# 文件名后缀格式
handler.suffix = '%Y-%m-%d.log'

formatter = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)

logger.addHandler(handler)

# 生成的日志文件：
# app.log.2024-01-01.log
# app.log.2024-01-02.log
# ...
```

## 实战：配置文件方式

### 使用字典配置

```python
import logging
import logging.config

# 日志配置字典
LOGGING_CONFIG = {
    'version': 1,
    'disable_existing_loggers': False,
    
    'formatters': {
        'detailed': {
            'format': '%(asctime)s - %(name)s - %(levelname)s - [%(filename)s:%(lineno)d] - %(message)s',
            'datefmt': '%Y-%m-%d %H:%M:%S'
        },
        'simple': {
            'format': '%(levelname)s - %(message)s'
        }
    },
    
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
            'level': 'INFO',
            'formatter': 'simple',
            'stream': 'ext://sys.stdout'
        },
        'file': {
            'class': 'logging.handlers.RotatingFileHandler',
            'level': 'DEBUG',
            'formatter': 'detailed',
            'filename': 'app.log',
            'maxBytes': 10485760,  # 10MB
            'backupCount': 5,
            'encoding': 'utf-8'
        }
    },
    
    'loggers': {
        'my_app': {
            'level': 'DEBUG',
            'handlers': ['console', 'file'],
            'propagate': False
        }
    },
    
    'root': {
        'level': 'WARNING',
        'handlers': ['console']
    }
}

# 应用配置
logging.config.dictConfig(LOGGING_CONFIG)

# 使用
logger = logging.getLogger('my_app')
logger.debug("调试信息")
logger.info("一般信息")
```

### 使用 YAML 配置文件

```python
# logging.yaml
version: 1
disable_existing_loggers: False

formatters:
  default:
    format: '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    datefmt: '%Y-%m-%d %H:%M:%S'

handlers:
  console:
    class: logging.StreamHandler
    level: INFO
    formatter: default
    stream: ext://sys.stdout

  file:
    class: logging.handlers.RotatingFileHandler
    level: DEBUG
    formatter: default
    filename: app.log
    maxBytes: 10485760
    backupCount: 5
    encoding: utf-8

root:
  level: DEBUG
  handlers: [console, file]
```

```python
import logging
import logging.config
import yaml

# 加载 YAML 配置
with open('logging.yaml', 'r', encoding='utf-8') as f:
    config = yaml.safe_load(f)

logging.config.dictConfig(config)

logger = logging.getLogger(__name__)
logger.info("配置完成")
```

## 在类中使用日志

### 类的实例日志器

```python
import logging

class MyClass:
    def __init__(self, value):
        self.logger = logging.getLogger(f'{__name__}.MyClass')
        self.value = value
        self.logger.info(f'MyClass 实例化，value={value}')
    
    def do_something(self):
        self.logger.debug("执行 do_something")
        try:
            result = self.value * 2
            self.logger.info(f"计算结果: {result}")
            return result
        except Exception as e:
            self.logger.error(f"发生错误: {e}")
            raise

# 使用
obj = MyClass(10)
obj.do_something()
```

### 模块级日志器

```python
# mymodule.py
import logging

logger = logging.getLogger(__name__)

class MyClass:
    def __init__(self, value):
        self.value = value
        logger.info(f'MyClass 初始化，value={value}')
    
    def do_something(self):
        logger.debug("执行操作")
        return self.value * 2

# 外部使用时，可以统一配置
# import mymodule
# mymodule.logger.setLevel(logging.DEBUG)
```

## 日志与异常

### 记录异常信息

```python
import logging

logger = logging.getLogger(__name__)

try:
    result = 10 / 0
except Exception as e:
    # 方式一：只记录异常消息
    logger.error(f"发生错误: {e}")
    
    # 方式二：记录异常信息和堆栈（推荐）
    logger.exception("发生异常")
    
    # 方式三：显式传递 exc_info=True
    logger.error("发生错误", exc_info=True)
```

### traceback 模块辅助

```python
import logging
import traceback

logger = logging.getLogger(__name__)

try:
    # 可能出错的代码
    pass
except Exception:
    # 完整堆栈跟踪
    logger.error("异常详情:\n" + traceback.format_exc())
```

## 多模块日志

### 项目结构

```
myproject/
├── main.py
├── utils/
│   ├── __init__.py
│   └── helper.py
└── config.py
```

### main.py

```python
import logging
import utils.helper

def setup_logging():
    """统一配置所有模块的日志"""
    logging.basicConfig(
        level=logging.DEBUG,
        format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
        datefmt='%Y-%m-%d %H:%M:%S'
    )

def main():
    setup_logging()
    logger = logging.getLogger('main')
    logger.info("程序启动")
    
    utils.helper.process()
    
    logger.info("程序结束")

if __name__ == '__main__':
    main()
```

### utils/helper.py

```python
import logging

logger = logging.getLogger('utils.helper')

def process():
    logger.info("开始处理")
    # ... 处理逻辑
    logger.info("处理完成")
```

输出示例：

```
2024-01-01 10:00:00 - main - INFO - 程序启动
2024-01-01 10:00:00 - utils.helper - INFO - 开始处理
2024-01-01 10:00:00 - utils.helper - INFO - 处理完成
2024-01-01 10:00:00 - main - INFO - 程序结束
```

## 过滤日志

### 自定义过滤器

```python
import logging

class ContextFilter(logging.Filter):
    """添加上下文信息的过滤器"""
    
    def __init__(self):
        super().__init__()
        self.user_id = None
        self.request_id = None
    
    def filter(self, record):
        record.user_id = getattr(self, 'user_id', 'anonymous')
        record.request_id = getattr(self, 'request_id', 'no-request')
        return True

# 使用
logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

filter_obj = ContextFilter()
handler = logging.StreamHandler()
handler.addFilter(filter_obj)

logger.addHandler(handler)

# 设置上下文
filter_obj.user_id = 'user123'
filter_obj.request_id = 'req-456'

logger.info("带上下文的日志")
```

### 根据级别过滤

```python
import logging

class LevelFilter(logging.Filter):
    """只允许特定级别的日志通过"""
    
    def __init__(self, level):
        super().__init__()
        self.level = level
    
    def filter(self, record):
        return record.levelno == self.level

# 只记录 ERROR 级别
handler = logging.StreamHandler()
handler.addFilter(LevelFilter(logging.ERROR))

logger = logging.getLogger('my_app')
logger.addHandler(handler)
```

## 日志与 JSON

### 结构化日志输出

```python
import logging
import json
from datetime import datetime

class JSONFormatter(logging.Formatter):
    """JSON 格式的日志格式化器"""
    
    def format(self, record):
        log_data = {
            'timestamp': datetime.utcnow().isoformat(),
            'level': record.levelname,
            'logger': record.name,
            'message': record.getMessage(),
            'module': record.module,
            'function': record.funcName,
            'line': record.lineno
        }
        
        # 添加异常信息
        if record.exc_info:
            log_data['exception'] = self.formatException(record.exc_info)
        
        # 添加额外字段
        if hasattr(record, 'user_id'):
            log_data['user_id'] = record.user_id
        
        return json.dumps(log_data)

# 使用
handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())

logger = logging.getLogger('my_app')
logger.setLevel(logging.DEBUG)
logger.addHandler(handler)

# 添加额外字段
extra = {'user_id': 'user123'}
logger.info("用户登录", extra=extra)
```

## 性能优化

### 避免字符串拼接

```python
import logging

logger = logging.getLogger(__name__)

# 低效 - 总是拼接字符串
logger.debug("用户数据: " + str(user_data))

# 高效 - 只在需要时拼接
logger.debug("用户数据: %s", user_data)
```

### 使用 lazy formatting

```python
# 方式一：% 格式化
logger.debug("值: %s", expensive_calculation())

# 方式二：Lazy 包装
from logging import StringFormatterMessage as _

# 不要这样
if logger.isEnabledFor(logging.DEBUG):
    logger.debug(f"复杂消息: {some_expensive_operation()}")
```

## 常见问题

### 1. 日志重复输出

```python
# 问题：多次配置导致重复输出
logging.basicConfig(level=logging.DEBUG)  # 第一次
logging.basicConfig(level=logging.DEBUG)  # 第二次

# 解决：检查并移除现有 handler
logger = logging.getLogger(__name__)
if not logger.handlers:
    handler = logging.StreamHandler()
    logger.addHandler(handler)
```

### 2. 模块日志不生效

```python
# 问题：子模块日志没有正确配置
import mymodule
mymodule.logger.setLevel(logging.DEBUG)  # 需要手动设置

# 解决：使用父 logger 统一配置
logging.getLogger('mymodule').setLevel(logging.DEBUG)
logging.getLogger('mymodule.submodule').setLevel(logging.DEBUG)
```

### 3. 中文日志乱码

```python
# 确保文件编码正确
handler = logging.FileHandler('app.log', encoding='utf-8')

# 确保源文件保存为 UTF-8
# -*- coding: utf-8 -*-
```

## 最佳实践

1. **使用命名 logger**

   ```python
   logger = logging.getLogger(__name__)  # 推荐
   logger = logging.getLogger('myapp')  # 也可以
   ```

2. **不要使用 root logger**

   ```python
   # 不好
   logging.info("message")
   
   # 好
   logger = logging.getLogger(__name__)
   logger.info("message")
   ```

3. **配置只执行一次**

   ```python
   if not logger.handlers:
       setup_logger()
   ```

4. **合理的日志级别**
   - DEBUG: 详细调试信息
   - INFO: 程序运行状态
   - WARNING: 警告但不影响运行
   - ERROR: 错误但程序可继续
   - CRITICAL: 严重错误导致程序无法运行

5. **不要记录敏感信息**

   ```python
   # 不好
   logger.info(f"用户密码: {password}")
   
   # 好
   logger.info("用户登录成功")
   ```

## 总结

Python 的 logging 模块提供了完整的日志处理能力：

- 📊 **多级别**：DEBUG/INFO/WARNING/ERROR/CRITICAL
- 📍 **多输出**：控制台、文件、网络
- 🔄 **轮转**：按大小或时间自动切换文件
- 🎨 **格式化**：完全可定制的输出格式
- 🔧 **过滤**：按条件筛选日志
- ⚡ **性能**：支持延迟格式化和异步日志

掌握这些内容，你就能为 Python 项目建立完善的日志系统了！

---

> 📚 推荐阅读
>
> - [Python logging 官方文档](https://docs.python.org/3/library/logging.html)
> - [Logging Cookbook](https://docs.python.org/3/howto/logging-cookbook.html)
> - [十二要素应用之日志](https://12factor.net/zh_cn/logs)
