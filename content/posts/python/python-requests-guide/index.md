---
title: "Python HTTP 请求完全指南：requests 库详解"
date: 2026-04-22
draft: false
featuredImage: "featured.png"
tags: ["Python", "requests", "HTTP", "网络编程"]
categories: ["Python"]
---


在现代 Web 开发中，与 API 交互是必不可少的一项技能。无论是调用第三方接口获取数据，还是开发自己的后端服务，HTTP 请求都是连接客户端与服务端的桥梁。

Python 的 `requests` 库是 Python 生态中最流行的 HTTP 客户端库，它以简洁优雅的 API 设计著称。本文将带你从入门到精通，全面掌握 requests 库的用法。

## 为什么选择 requests？

在 requests 出现之前，Python 开发者通常使用 `urllib` 来发送 HTTP 请求。但 urllib 的 API 设计较为复杂，需要处理编码、会话管理、Cookie 等各种细节。requests 库的出现彻底改变了这一局面：

- **简洁的 API**：一行代码即可完成 GET 请求
- **自动编码处理**：无需手动处理编码问题
- **会话管理**：方便管理 Cookie 和会话
- **超时控制**：防止请求无限等待
- **文件上传/下载**：内置支持
- **代理支持**：轻松配置代理服务器

```bash
# 安装 requests
pip install requests
```

## 基础用法

### GET 请求

GET 请求是最常用的 HTTP 方法，用于从服务器获取数据。

```python
import requests

# 最简单的 GET 请求
response = requests.get('https://api.github.com')

# 查看响应状态码
print(response.status_code)  # 200

# 查看响应内容
print(response.text)

# 查看响应头
print(response.headers)

# 查看 JSON 响应（如果服务器返回 JSON）
data = response.json()
print(data)
```

### 带参数的 GET 请求

有两种方式传递 URL 参数：

```python
# 方式一：直接拼接 URL
response = requests.get('https://api.github.com/search/repositories?q=python&sort=stars')

# 方式二：使用 params 参数（推荐，更清晰）
params = {
    'q': 'python',
    'sort': 'stars',
    'order': 'desc'
}
response = requests.get('https://api.github.com/search/repositories', params=params)
print(response.url)  # 自动拼接成完整 URL
```

### POST 请求

POST 请求用于向服务器提交数据。

```python
# 提交表单数据
data = {
    'username': 'test',
    'password': '123456'
}
response = requests.post('https://httpbin.org/post', data=data)
print(response.json())

# 提交 JSON 数据
import json
response = requests.post(
    'https://httpbin.org/post',
    json={'name': '张三', 'age': 25}
)
print(response.json())
```

### 其他 HTTP 方法

requests 支持所有标准的 HTTP 方法：

```python
# PUT 请求 - 更新资源
response = requests.put('https://httpbin.org/put', data={'key': 'value'})

# DELETE 请求 - 删除资源
response = requests.delete('https://httpbin.org/delete')

# PATCH 请求 - 部分更新
response = requests.patch('https://httpbin.org/patch', data={'key': 'new_value'})

# HEAD 请求 - 只获取响应头
response = requests.head('https://httpbin.org/get')

# OPTIONS 请求 - 获取支持的 HTTP 方法
response = requests.options('https://httpbin.org/get')
print(response.headers.get('allow'))
```

## 响应对象详解

发送请求后，会返回一个 Response 对象，它包含了服务器的所有响应信息。

### 常用属性

```python
response = requests.get('https://httpbin.org/get')

# 状态码
print(response.status_code)  # 200

# 判断状态码是否成功（2xx 表示成功）
print(response.ok)  # True

# 响应内容
print(response.text)      # 文本形式（自动解码）
print(response.content)   # 字节形式（二进制数据）

# JSON 响应
print(response.json())   # 解析为 Python 对象

# 响应头
print(response.headers)           # 所有响应头
print(response.headers['Content-Type'])  # 特定响应头

# 请求信息
print(response.url)        # 最终请求的 URL
print(response.request)    # 请求对象
```

### 状态码处理

```python
# 方式一：手动检查状态码
response = requests.get('https://httpbin.org/get')
if response.status_code == 200:
    print('请求成功')
elif response.status_code == 404:
    print('资源不存在')
else:
    print('其他错误')

# 方式二：使用 requests 内置的状态码常量
from requests import codes

response = requests.get('https://httpbin.org/get')
if response.status_code == codes.ok:  # codes.ok == 200
    print('请求成功')

# 常见状态码常量
print(codes.ok)        # 200
print(codes.created)    # 201
print(codes.no_content) # 204
print(codes.not_found) # 404
print(codes.server_error) # 500-599
```

## 请求头处理

### 添加自定义请求头

```python
headers = {
    'User-Agent': 'MyApp/1.0',
    'Accept': 'application/json',
    'Authorization': 'Bearer your_token_here',
    'X-Custom-Header': 'custom_value'
}

response = requests.get('https://httpbin.org/headers', headers=headers)
print(response.json())
```

### 查看响应的编码

```python
response = requests.get('https://httpbin.org/encoding/utf8')
print(response.encoding)  # 自动检测编码

# 手动设置编码
response.encoding = 'utf-8'
print(response.text)
```

## Cookie 处理

### 发送带 Cookie 的请求

```python
# 方式一：通过 headers 发送
cookies = {'session_id': 'abc123'}
response = requests.get('https://httpbin.org/cookies', headers={'Cookie': 'session_id=abc123'})

# 方式二：使用 cookies 参数
response = requests.get('https://httpbin.org/cookies', cookies={'session_id': 'abc123'})
print(response.json())

# 设置多个 Cookie
cookies = {
    'name': '张三',
    'age': '25',
    'city': '北京'
}
response = requests.get('https://httpbin.org/cookies', cookies=cookies)
```

### 获取响应中的 Cookie

```python
response = requests.get('https://httpbin.org/cookies/set/name/value')
print(response.cookies)           # 获取所有 Cookie
print(response.cookies['name'])     # 获取特定 Cookie
```

## 会话对象

Session 对象允许你在多个请求之间保持某些参数和数据，如 Cookie、请求头等。

### 为什么使用 Session？

普通请求每次都会创建新的连接，而 Session 会复用 TCP 连接，提高性能：

```python
# 普通方式 - 每次请求都是独立的
requests.get('https://httpbin.org/cookies/set/session_id/abc123')
requests.get('https://httpbin.org/cookies')  # 无法获取到上面的 Cookie

# 使用 Session - Cookie 会自动保持
session = requests.Session()
session.get('https://httpbin.org/cookies/set/session_id/abc123')
response = session.get('https://httpbin.org/cookies')
print(response.json())  # {'cookies': {'session_id': 'abc123'}}
```

### Session 的实际应用

```python
# 模拟登录并保持会话
session = requests.Session()

# 1. 登录获取 Cookie
login_data = {
    'username': 'your_username',
    'password': 'your_password'
}
session.post('https://example.com/login', data=login_data)

# 2. 后续请求自动带上 Cookie
response = session.get('https://example.com/profile')
print(response.text)

# 3. 设置默认请求头（所有请求都会带上）
session.headers.update({
    'User-Agent': 'MyApp/1.0',
    'Accept': 'application/json'
})

# 4. 清理会话
session.close()
```

## 超时与重试

### 设置超时

超时设置非常重要，可以防止请求无限等待导致程序卡死：

```python
# 设置超时时间（秒）
response = requests.get('https://httpbin.org/delay/1', timeout=5)  # 5秒超时

# 分别设置连接超时和读取超时
response = requests.get(
    'https://httpbin.org/delay/2',
    timeout=(3, 10)  # (连接超时, 读取超时)
)

# 不设置超时可能导致程序卡死
# response = requests.get('https://httpbin.org/delay/10')  # 危险！
```

### 超时异常处理

```python
from requests.exceptions import Timeout, RequestException

try:
    response = requests.get('https://httpbin.org/delay/10', timeout=5)
    print(response.json())
except Timeout:
    print('请求超时，服务器响应太慢')
except RequestException as e:
    print(f'请求失败: {e}')
```

### 配置重试

```python
from requests.adapters import HTTPAdapter
from requests.packages.urllib3.util.retry import Retry

def create_retry_session(retries=3, backoff_factor=0.5):
    """创建一个带有重试机制的 Session"""
    session = requests.Session()
    retry = Retry(
        total=retries,              # 总重试次数
        read=retries,                # 读取重试次数
        connect=retries,             # 连接重试次数
        backoff_factor=backoff_factor,  # 重试间隔倍数
        status_forcelist=[500, 502, 503, 504]  # 遇到这些状态码时重试
    )
    adapter = HTTPAdapter(max_retries=retry)
    session.mount('http://', adapter)
    session.mount('https://', adapter)
    return session

# 使用
session = create_retry_session()
response = session.get('https://httpbin.org/status/500')
```

## 错误处理

requests 库定义了多种异常类型：

```python
from requests.exceptions import (
    ConnectionError,   # 网络连接错误
    HTTPError,         # HTTP 错误响应
    Timeout,           # 请求超时
    TooManyRedirects,  # 重定向次数过多
    RequestException   # 所有异常的基类
)

try:
    response = requests.get('https://httpbin.org/get')
    response.raise_for_status()  # 如果状态码不是 200，抛出异常
except HTTPError as e:
    print(f'HTTP 错误: {e.response.status_code}')
except ConnectionError:
    print('网络连接失败')
except Timeout:
    print('请求超时')
except TooManyRedirects:
    print('重定向次数过多')
except RequestException as e:
    print(f'请求失败: {e}')
```

### 使用 raise_for_status()

```python
response = requests.get('https://httpbin.org/status/404')
try:
    response.raise_for_status()
except HTTPError as e:
    print(f'错误: {e}')
    print(f'响应内容: {response.text}')
```

## 文件上传与下载

### 上传文件

```python
# 上传文件
files = {
    'file': open('example.txt', 'rb')
}
response = requests.post('https://httpbin.org/post', files=files)

# 指定文件名和内容类型
files = {
    'file': ('custom_name.txt', open('example.txt', 'rb'), 'text/plain')
}
response = requests.post('https://httpbin.org/post', files=files)

# 上传图片
files = {
    'image': ('photo.jpg', open('photo.jpg', 'rb'), 'image/jpeg')
}
response = requests.post('https://httpbin.org/post', files=files)

# 上传多个文件
files = [
    ('images', ('img1.jpg', open('img1.jpg', 'rb'), 'image/jpeg')),
    ('images', ('img2.jpg', open('img2.jpg', 'rb'), 'image/jpeg'))
]
response = requests.post('https://httpbin.org/post', files=files)
```

### 下载文件

```python
# 下载小文件
response = requests.get('https://example.com/image.jpg')
with open('image.jpg', 'wb') as f:
    f.write(response.content)

# 下载大文件（流式下载）
response = requests.get('https://example.com/large_file.zip', stream=True)
with open('large_file.zip', 'wb') as f:
    for chunk in response.iter_content(chunk_size=1024):
        if chunk:
            f.write(chunk)

# 使用迭代器下载
with requests.get('https://example.com/image.jpg', stream=True) as response:
    with open('image.jpg', 'wb') as f:
        for chunk in response.iter_content(chunk_size=8192):
            f.write(chunk)
```

### 下载进度显示

```python
import os

def download_with_progress(url, filename):
    """带进度显示的下载函数"""
    response = requests.get(url, stream=True)
    total_size = int(response.headers.get('content-length', 0))
    downloaded = 0
    
    with open(filename, 'wb') as f:
        for chunk in response.iter_content(chunk_size=1024):
            if chunk:
                f.write(chunk)
                downloaded += len(chunk)
                progress = (downloaded / total_size) * 100
                print(f'\r下载进度: {progress:.1f}%', end='')
    
    print(f'\n文件已保存到: {filename}')

# 使用
download_with_progress(
    'https://example.com/large_file.zip',
    'large_file.zip'
)
```

## SSL 证书验证

### 忽略 SSL 证书验证（仅用于测试）

```python
# 警告：生产环境不要这样做！
response = requests.get('https://example.com', verify=False)

# 消除警告
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

response = requests.get('https://example.com', verify=False)
```

### 指定 CA 证书

```python
# 使用自定义 CA 证书
response = requests.get('https://example.com', verify='/path/to/ca-bundle.crt')

# 使用系统默认 CA 证书
import certifi
response = requests.get('https://example.com', verify=certifi.where())
```

## 代理设置

### HTTP 代理

```python
# 普通代理
proxies = {
    'http': 'http://proxy.example.com:8080',
    'https': 'http://proxy.example.com:8080'
}
response = requests.get('https://httpbin.org/ip', proxies=proxies)

# 带认证的代理
proxies = {
    'http': 'http://user:password@proxy.example.com:8080',
    'https': 'http://user:password@proxy.example.com:8080'
}
response = requests.get('https://httpbin.org/ip', proxies=proxies)
```

### 在 Session 中使用代理

```python
session = requests.Session()
session.proxies = {
    'http': 'http://proxy.example.com:8080',
    'https': 'http://proxy.example.com:8080'
}
response = session.get('https://httpbin.org/ip')
```

## 身份认证

### 基本认证

```python
from requests.auth import HTTPBasicAuth

response = requests.get(
    'https://api.github.com/user',
    auth=HTTPBasicAuth('username', 'password')
)

# 简写形式
response = requests.get('https://api.github.com/user', auth=('username', 'password'))
```

### 其他认证方式

```python
from requests.auth import HTTPBasicAuth, HTTPDigestAuth

# Digest 认证
response = requests.get('https://httpbin.org/digest-auth/auth/user/passwd', 
                        auth=HTTPDigestAuth('user', 'passwd'))

# OAuth 认证
headers = {
    'Authorization': 'Bearer your_oauth_token'
}
response = requests.get('https://api.example.com/protected', headers=headers)
```

## 实战案例

### 调用 GitHub API

```python
import requests

class GitHubAPI:
    """GitHub API 封装"""
    
    def __init__(self, token=None):
        self.base_url = 'https://api.github.com'
        self.headers = {
            'Accept': 'application/vnd.github.v3+json'
        }
        if token:
            self.headers['Authorization'] = f'token {token}'
    
    def get_user(self, username):
        """获取用户信息"""
        response = requests.get(
            f'{self.base_url}/users/{username}',
            headers=self.headers
        )
        response.raise_for_status()
        return response.json()
    
    def get_repos(self, username, sort='updated'):
        """获取用户的仓库列表"""
        response = requests.get(
            f'{self.base_url}/users/{username}/repos',
            headers=self.headers,
            params={'sort': sort, 'per_page': 10}
        )
        response.raise_for_status()
        return response.json()
    
    def create_issue(self, owner, repo, title, body):
        """创建 Issue"""
        url = f'{self.base_url}/repos/{owner}/{repo}/issues'
        data = {'title': title, 'body': body}
        response = requests.post(url, headers=self.headers, json=data)
        response.raise_for_status()
        return response.json()

# 使用
github = GitHubAPI(token='your_token_here')
user_info = github.get_user('Hungerdream')
print(f"用户名: {user_info['name']}")
print(f"仓库数: {user_info['public_repos']}")

repos = github.get_repos('Hungerdream')
for repo in repos[:5]:
    print(f"- {repo['name']}: ⭐ {repo['stargazers_count']}")
```

### 批量下载图片

```python
import requests
import os
from concurrent.futures import ThreadPoolExecutor, as_completed

def download_image(url, save_path):
    """下载单张图片"""
    try:
        response = requests.get(url, timeout=10)
        response.raise_for_status()
        
        # 从 URL 提取文件名
        filename = url.split('/')[-1]
        filepath = os.path.join(save_path, filename)
        
        with open(filepath, 'wb') as f:
            f.write(response.content)
        
        return f'成功: {filename}'
    except Exception as e:
        return f'失败: {url} - {e}'

def batch_download(urls, save_path, max_workers=5):
    """批量下载图片"""
    os.makedirs(save_path, exist_ok=True)
    
    with ThreadPoolExecutor(max_workers=max_workers) as executor:
        futures = {executor.submit(download_image, url, save_path): url 
                   for url in urls}
        
        for future in as_completed(futures):
            print(future.result())

# 使用
image_urls = [
    'https://example.com/image1.jpg',
    'https://example.com/image2.jpg',
    'https://example.com/image3.jpg',
]

batch_download(image_urls, 'downloads/')
```

## 最佳实践

### 1. 使用 context manager 管理资源

```python
# 确保连接被正确关闭
with requests.Session() as session:
    response = session.get('https://api.example.com/data')
    data = response.json()
```

### 2. 始终设置超时

```python
response = requests.get(url, timeout=10)  # 始终设置合理超时
```

### 3. 异常处理

```python
try:
    response = requests.get(url, timeout=10)
    response.raise_for_status()
except requests.RequestException as e:
    logger.error(f'请求失败: {e}')
    raise
```

### 4. 复用 Session

```python
# 不要这样写
for url in urls:
    response = requests.get(url)  # 每次创建新连接

# 应该这样写
with requests.Session() as session:
    for url in urls:
        response = session.get(url)  # 复用连接
```

### 5. 不要禁用 SSL 验证

```python
# 危险！
response = requests.get(url, verify=False)

# 如果必须使用自签名证书
import certifi
response = requests.get(url, verify=certifi.where())
```

## 总结

requests 库是 Python 中处理 HTTP 请求的最佳选择。它的设计哲学是"让 HTTP 请求变得简单"，而它确实做到了这一点。

本文涵盖了 requests 库的绝大部分功能：

- ✅ GET/POST 等 HTTP 方法
- ✅ 请求头、Cookie、会话管理
- ✅ 超时、重试、错误处理
- ✅ 文件上传下载
- ✅ SSL 证书、代理设置
- ✅ 各种认证方式
- ✅ 实战案例和最佳实践

掌握这些内容，你就能应对绝大多数与 HTTP 相关的开发需求了。

---

> 📚 推荐阅读
>
> - [官方文档](https://docs.python-requests.org/)
> - [HTTP 方法详解](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods)
> - [RESTful API 设计指南](https://restfulapi.cn/)
