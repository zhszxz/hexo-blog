---
title: FastAPI核心基础
tags:
  - Pydantic
  - Depends
  - Middleware
categories:
  - Python
  - FastAPI
date: 2026-05-28 21:29:21
---

# FastAPI学习笔记（一）：FastAPI 核心基础

# 一、FastAPI 核心原理

### 1\.1 FastAPI 是什么

FastAPI 是一款基于 Python 3\.8及以上版本、依托原生类型提示语法开发的**现代化、高性能、工程化** Web API 开发框架。相较于传统的 Flask、Django 框架，FastAPI 针对性解决了传统 Python 后端框架性能弱、无自动校验、无原生接口文档、工程化薄弱等痛点，是目前 Python 微服务、后台管理系统、移动端接口、第三方对接接口开发的主流首选框架。

FastAPI 的核心设计理念是**高效开发、类型安全、高性能、零冗余配置**。框架内置了完整的数据校验、接口文档、异常处理、依赖注入能力，无需引入大量第三方插件，开箱即用。同时完美兼容同步、异步两种开发模式，兼顾开发效率与服务并发性能，适配从小型接口项目到中大型分布式微服务项目的全场景开发。

核心特性详细解析：

- **全自动数据校验**：依托 Pydantic 实现运行时参数校验、类型转换、数据清洗，无需手写 if 判断校验参数，大幅减少冗余代码与BUG。

- **零成本自动接口文档**：根据代码路由、参数、模型自动生成标准化 OpenAPI 文档，支持在线调试、参数预览、接口测试，彻底告别手写接口文档。

- **同步异步双兼容**：同时支持普通同步函数与异步协程函数，开发者可根据业务场景自由选择，无需改造框架底层。

- **超高并发性能**：基于 ASGI 异步协议构建，性能对标 Go、Node\.js 框架，远超传统 WSGI 框架。

- **强类型安全**：依托 Python 类型提示，实现编码阶段语法校验、运行时数据校验，降低类型错误、参数错误等线上问题。

- **原生工程化支持**：内置依赖注入、中间件、生命周期事件，天然适配分层解耦的企业项目架构。

### 1\.2 ASGI 协议详解

想要理解 FastAPI 的高性能优势，必须先搞懂 ASGI 协议。ASGI 全称 Asynchronous Server Gateway Interface，即异步服务器网关接口，是 Python 官方推出的、用于替代传统 WSGI 的新一代 Web 服务协议标准。

在 Python Web 发展历程中，WSGI 协议长期占据主流，Flask、Django、Tornado 等老牌框架均基于 WSGI 开发。但 WSGI 存在致命短板：**仅支持同步阻塞模式**，单进程同一时间只能处理一个请求，遇到 IO 阻塞（数据库查询、网络请求、文件读写）时，线程会被挂起，无法处理新请求，并发能力极差，完全无法适配高并发业务场景。

而 ASGI 协议是 WSGI 的超集，完美兼容 WSGI 同步逻辑，同时新增了**异步非阻塞、长连接、多任务并发**能力，支持 HTTP、WebSocket、服务推送等多种协议场景。

两大协议核心对比：

- **WSGI**：纯同步阻塞模型，无异步支持，单线程串行处理请求，并发性能弱，仅适配简单Web业务，代表框架：Flask、Django。

- **ASGI**：同步\+异步双模式，事件循环驱动非阻塞处理请求，支持高并发、长连接、实时通信，适配绝大多数企业级高并发场景，代表框架：FastAPI、Starlette、Quart。

FastAPI 底层基于 Starlette（轻量化 ASGI 核心框架）封装，天生适配 ASGI 协议，这也是 FastAPI 性能远超传统 Python 框架的核心底层原因。

### 1\.3 FastAPI vs Flask 深度对比

Flask 是 Python 经典轻量 Web 框架，主打极简灵活，而 FastAPI 是现代化工程化框架，两者定位、能力、适用场景差异极大，也是面试高频考点。下面从企业开发核心维度做全方位对比：

| 对比维度     | Flask                                                        | FastAPI                                                      |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 底层协议     | WSGI 同步协议，阻塞式请求处理                                | ASGI 双协议，同步异步兼容，非阻塞并发                        |
| 数据参数校验 | 无内置校验能力，需手动编写大量if判断，或引入第三方库，极易出现参数漏洞 | 内置Pydantic全自动校验，支持类型、长度、范围、正则、自定义规则，运行时自动拦截脏数据 |
| 接口文档     | 无原生文档，需手动集成Flask\-RESTX、flask\-swagger等插件，配置繁琐、格式不统一 | 原生自动生成OpenAPI标准文档，无需任何配置，支持在线调试、参数预览、接口导出 |
| 类型提示支持 | 不强制、不校验，类型提示仅作注释，运行时无任何约束，极易出现类型报错 | 强类型约束，编码阶段IDE提示，运行时强制校验，从根源规避类型错误 |
| 并发性能     | 单线程阻塞，并发能力弱，高并发场景极易超时、报错             | 异步非阻塞，单机QPS是Flask的3\-10倍，适配高并发接口场景      |
| 工程化能力   | 极简轻量化，无内置分层、依赖、异常机制，大型项目需手动封装大量基础能力 | 原生支持分层架构、依赖注入、全局异常、中间件，开箱即用企业级架构 |

**场景选型建议**：小型静态网站、极简演示项目可选用Flask；所有**接口开发、后端服务、微服务、线上商用项目**，优先选择FastAPI。

### 1\.4 同步与异步开发模式详解

FastAPI 最大的优势之一就是**同步、异步完全兼容**，开发者无需强制使用异步，可根据业务场景灵活选型，这也是新手最容易困惑的知识点。

1、同步函数（def 普通函数）

同步模式是串行执行逻辑，代码从上到下依次运行，上一行代码执行完毕后，才会执行下一行。同步模式兼容性极强，支持所有第三方库、数据库驱动、工具类，不会出现兼容报错。

**适用场景**：常规数据库CRUD操作、简单业务计算、CPU密集型任务、老旧第三方库调用、团队快速开发。90%的普通后台管理系统、业务接口，使用同步模式完全足够。

2、异步函数（async def 协程函数）

异步模式基于 Python 事件循环机制，采用非阻塞执行逻辑。当代码遇到IO阻塞操作（数据库查询、HTTP请求、文件读写、Redis查询）时，会主动让出CPU，处理其他请求，等待IO操作完成后再回归执行剩余逻辑，极大提升并发处理能力。

**适用场景**：高并发接口、批量网络请求、异步数据库、消息队列、长连接服务、秒杀、限流等高吞吐业务场景。

FastAPI 智能适配两种模式：同步函数会自动放入线程池执行，异步函数直接通过事件循环调度，开发者无需手动处理事件循环，极大降低了异步开发门槛。

### 1\.5 FastAPI 为什么快

FastAPI 的高性能并非噱头，而是由底层架构、技术选型、执行机制共同决定的，核心原因分为四点：

- **双核心底层架构**：底层依托 Starlette（高性能异步Web内核）处理请求调度，依托 Pydantic（极速数据校验库）处理参数解析与校验，两个库均由C语言底层优化，执行效率极高。

- **ASGI非阻塞并发**：摒弃WSGI阻塞模型，基于ASGI事件循环机制，单进程可同时处理成千上万个请求，无线程阻塞开销。

- **预编译类型校验**：基于类型提示做静态预编译校验，运行时无需冗余解析，参数校验开销远低于手动校验和第三方校验库。

- **极简内核设计**：框架内核极简，无冗余中间层、无多余逻辑封装，请求链路极短，请求响应延迟极低。

在同等服务器配置下，FastAPI 单机吞吐量远超 Flask、Django，基本持平 Go、Java 轻量接口框架性能，完全满足线上生产环境高并发需求。

### 1\.6 OpenAPI 规范原理

OpenAPI 是目前全球通用的 RESTful 接口描述行业规范，定义了接口请求方式、路径、参数、数据类型、响应格式、错误码等统一标准。在传统开发中，开发者需要手动编写接口文档，耗时且容易和代码不一致、更新不及时。

FastAPI 深度适配 OpenAPI 3\.0 最新规范，具备**代码即文档**的能力。框架会在项目启动时，自动扫描所有路由、参数注解、Pydantic模型、接口描述，自动解析生成标准化的 OpenAPI 接口描述文件，全程无需开发者手动编写任何文档内容。

核心价值：统一前后端对接标准、支持前端自动生成请求代码、支持接口自动化测试、方便接口对接与迭代维护，彻底解决前后端文档不一致、对接沟通成本高的问题。

### 1\.7 Swagger 自动文档使用详解

FastAPI 基于 OpenAPI 规范，内置两套开箱即用的可视化接口文档，启动项目后可直接通过浏览器访问，是开发、联调、测试的核心工具。

- **/docs 交互式调试文档（Swagger UI）**：最常用的文档地址，界面友好，支持在线查看接口参数、参数类型、必填项、注释说明，支持**在线直接发送请求、调试接口、查看响应结果**，开发联调必备。

- **/redoc 静态规范文档**：简洁只读式文档，排版规整，适合对外展示、接口查阅、文档导出，不支持在线调试。

开发规范：所有接口必须添加注释、参数描述，自动同步到文档中，保证文档可读性与准确性。生产环境可根据需求关闭文档地址，避免接口信息泄露。

# 二、项目初始化与环境搭建

### 2\.1 虚拟环境 venv 原理与使用

Python 全局环境会存在包版本冲突、依赖混乱的问题，不同项目可能需要同一个库的不同版本，直接使用全局环境会导致项目运行报错、环境崩溃。因此，所有正规 Python 项目，都必须使用**虚拟环境**隔离项目依赖。

venv 是 Python3\.3\+ 官方内置的虚拟环境工具，无需额外安装，轻量化、无第三方依赖，足够满足绝大多数项目的环境隔离需求。虚拟环境会为当前项目创建独立的 Python 解释器、依赖包目录，仅对当前项目生效，不影响全局环境。

```bash
# 创建虚拟环境（项目根目录执行）
python -m venv venv

# 激活虚拟环境（Windows CMD/PowerShell）
venv\Scripts\activate

# 激活虚拟环境（Mac/Linux 终端）
source venv/bin/activate

# 退出虚拟环境
deactivate
```

激活环境后，终端会出现 venv 标识，此时安装的所有依赖包仅作用于当前项目，完美实现环境隔离。

### 2\.2 requirements\.txt 依赖管理规范

requirements\.txt 是 Python 项目通用的依赖版本管理文件，用于记录项目所有依赖包及对应版本，保证团队协作、服务器部署时，所有人、所有环境的依赖版本完全一致，避免因版本差异导致的运行报错。

企业开发规范：必须固定核心依赖版本，禁止使用默认最新版本，避免框架版本迭代带来的兼容性问题。

```bash
# 手动写入核心依赖（固定版本，稳定可靠）
echo "fastapi==0.104.1
uvicorn==0.24.0
pydantic-settings==2.1.0
python-multipart==0.0.6" > requirements.txt

# 批量安装所有依赖
pip install -r requirements.txt

# 导出当前环境所有依赖（更新依赖后使用）
pip freeze > requirements.txt
```

### 2\.3 uvicorn 服务启动详解

uvicorn 是 FastAPI 官方推荐的 ASGI 高性能服务器，专门用于运行 FastAPI、Starlette 异步项目，替代传统的 gunicorn\+gevent 组合，原生支持异步、热更新、多进程部署。

区分开发环境与生产环境启动命令，是项目部署的基础规范：

```bash
# 开发环境启动：热更新、单进程、本地访问
uvicorn main:app --reload

# 生产环境启动：多进程、全网访问、高性能
uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4
```

参数详细说明：

- `main`：对应项目根目录的 main\.py 入口文件名称

- `app`：对应 main\.py 中初始化的 FastAPI 实例变量名

- `\-\-reload`：开启热更新，代码修改后自动重启服务，仅开发环境使用，生产环境必须关闭（损耗性能）

- `\-\-host 0\.0\.0\.0`：允许所有IP访问服务，部署服务器必备

- `\-\-port 8000`：指定服务启动端口

- `\-\-workers 4`：开启4个进程，充分利用服务器多核CPU，提升并发能力

### 2\.4 最简项目结构与入口文件解析

入门阶段采用单文件最简结构，快速跑通项目、熟悉基础流程，适合新手练习、接口测试。main\.py 是整个项目的唯一入口文件，所有服务初始化、路由注册、中间件配置、全局配置均依托入口文件完成。

```python
# main.py
from fastapi import FastAPI

# 初始化FastAPI全局实例，配置项目基础信息
app = FastAPI(title="FastAPI入门项目", version="1.0", description="FastAPI基础学习演示项目")

# 根路由：项目测试入口
@app.get("/")
def index():
    return {"msg": "Hello FastAPI！项目启动成功"}
```

启动服务后，访问 `http://127.0.0.1:8000` 即可看到返回结果，访问 `/docs`可查看自动生成的接口文档。

# 三、RESTful 标准路由系统

路由是 Web 接口的核心，负责**接收前端请求、匹配接口路径、分发业务逻辑**。FastAPI 完全遵循 RESTful 接口设计规范，标准化五种请求方法，对应增删改查四大业务操作，是企业接口统一规范，所有业务接口必须遵循该设计模式，保证接口语义清晰、统一规范。

RESTful 规范核心语义：GET用于查询、POST用于新增、PUT用于全量更新、PATCH用于局部更新、DELETE用于删除，禁止乱用请求方法。

```python
# GET：查询数据（无副作用、幂等）
@app.get("/get")
def get_demo():
    return {"method": "GET", "desc": "查询数据，适用于列表、详情查询"}

# POST：新增数据（非幂等，用于创建资源）
@app.post("/post")
def post_demo():
    return {"method": "POST", "desc": "新增数据，适用于创建用户、订单等资源"}

# PUT：全量更新（幂等，覆盖全部字段）
@app.put("/put")
def put_demo():
    return {"method": "PUT", "desc": "全量更新数据，覆盖目标资源所有字段"}

# DELETE：删除数据（幂等，删除指定资源）
@app.delete("/delete")
def delete_demo():
    return {"method": "DELETE", "desc": "删除数据，适用于资源删除操作"}

# PATCH：局部更新（幂等，仅更新传入字段）
@app.patch("/patch")
def patch_demo():
    return {"method": "PATCH", "desc": "局部更新数据，仅修改传入的字段，不影响其他字段"}
```

补充知识点：幂等性指一次和多次请求，对服务器资源的影响一致，查询、更新、删除接口必须保证幂等，新增接口无需幂等。

# 四、请求参数处理

前端与后端的所有数据交互，本质都是**参数传递与解析**。FastAPI 原生支持7种主流传参方式，完全覆盖浏览器、小程序、APP、第三方对接的所有传参场景，且全部支持自动类型校验、自动解析、自动文档生成，无需手动处理。

### 4\.1 Path 路径参数

路径参数是拼接在URL路径中的动态参数，常用于传递唯一性资源标识（用户ID、订单ID、文章ID），属于 Restful 规范核心参数，适合查询单条详情、删除指定资源场景。使用 Path 可实现参数精细化校验、必填约束、注释说明。

```python
from fastapi import Path

@app.get("/user/{user_id}")
def get_user(user_id: int = Path(..., description="用户唯一ID，正整数", ge=1)):
    return {"user_id": user_id, "desc": "通过用户ID查询用户详情"}
```

### 4\.2 Query 查询参数

查询参数是URL问号后的参数，常用于列表查询、分页、筛选、搜索场景，参数灵活、可传可不传，支持默认值、范围校验、长度校验，是日常开发使用频率最高的参数类型。

```python
from fastapi import Query

@app.get("/query")
def query_demo(name: str, age: int = Query(None, ge=0, le=150, description="用户年龄，0-150之间")):
    return {"name": name, "age": age}
```

### 4\.3 Body 请求体参数

Body 请求体参数是 POST/PUT/PATCH 接口专用参数，以 JSON 格式传递，支持复杂对象、多层级数据、大批量数据，是新增、修改接口的核心传参方式，支持复杂结构数据传输。

```python
from fastapi import Body

@app.post("/body")
def body_demo(data: dict = Body(..., description="自定义JSON请求体数据")):
    return {"data": data, "msg": "JSON请求体解析成功"}
```

### 4\.4 Header 请求头参数

Header 参数存储在请求头中，常用于传递全局身份信息、设备信息、令牌信息，最典型场景就是传递 Token 登录凭证、设备型号、请求来源标识。

```python
from fastapi import Header

@app.get("/header")
def header_demo(token: str = Header(None, description="用户登录令牌")):
    return {"token": token, "msg": "请求头参数获取成功"}
```

### 4\.5 Cookie 参数

Cookie 参数由浏览器自动携带，用于存储客户端临时会话信息、登录状态，常用于网页端项目的会话保持、免登录校验场景。

```python
from fastapi import Cookie

@app.get("/cookie")
def cookie_demo(session: str = Cookie(None, description="客户端会话Cookie")):
    return {"session": session, "msg": "Cookie参数获取成功"}
```

### 4\.6 Form 表单参数

Form 表单参数是传统表单提交格式，适配前端 form 表单提交、登录页面提交场景，不支持复杂JSON结构，仅支持简单键值对参数，常用于账号密码登录、简单表单提交。

```python
from fastapi import Form

@app.post("/form")
def form_demo(username: str = Form(...), password: str = Form(...)):
    return {"username": username, "msg": "表单提交成功"}
```

### 4\.7 File 文件上传参数

FastAPI 原生支持文件上传，通过 UploadFile 接收文件流，支持单文件、多文件上传，可获取文件名、文件大小、文件类型，适配头像上传、图片素材、文档上传等所有文件业务场景。

```python
from fastapi import UploadFile, File

@app.post("/upload")
def upload_file(file: UploadFile = File(..., description="上传文件")):
    return {"filename": file.filename, "size": file.size, "msg": "文件上传成功"}
```

# 五、Pydantic 数据模型（核心重点）

Pydantic 是 FastAPI 的**灵魂核心**，也是区别于其他框架的最大优势。所有接口的参数校验、数据解析、格式转换、敏感字段过滤，全部依托 Pydantic 实现。Pydantic 基于类型提示，在运行时自动完成数据校验，非法参数直接拦截并返回标准错误，彻底解放手动参数校验代码，是企业开发必须精通的核心知识点。

### 5\.1 BaseModel、Optional、默认值

BaseModel 是 Pydantic 所有数据模型的基类，所有自定义请求、响应模型都需要继承此类。Optional 用于定义可选参数，搭配默认值可实现参数非必传、自动填充默认值的业务需求。

```python
from pydantic import BaseModel
from typing import Optional

# 基础用户模型：实现必填、可选、默认值配置
class UserBase(BaseModel):
    name: str               # 必填字符串参数
    age: Optional[int] = None  # 可选参数，不传默认None
    status: bool = True     # 非必传，默认启用状态
```

### 5\.2 Field 精细化校验与 alias 别名

Field 是 Pydantic 精细化校验核心工具，支持字段长度、数值范围、正则匹配、必填标识、字段描述等精细化约束。alias 别名可以实现**前端传参名与后端变量名不一致**的映射适配，解决前后端命名规范差异问题。

```python
from pydantic import Field

class UserField(BaseModel):
    # 用户名：必填、2-20位字符串、带描述
    username: str = Field(..., min_length=2, max_length=20, description="用户登录账号")
    # 别名映射：前端传age，后端自动赋值给user_age，默认值18，范围0-120
    user_age: int = Field(18, ge=0, le=120, alias="age")
```

### 5\.3 validator 与 model\_validator 自定义校验

基础字段校验无法满足复杂业务规则，Pydantic 提供两种自定义校验器：单字段校验器 validator、全局模型校验器 model\_validator，可自定义任意业务校验规则，适配密码强度、手机号格式、联动校验等复杂场景。

```python
from pydantic import validator, model_validator

class UserValidator(BaseModel):
    age: int
    password: str

    # 单字段独立校验：年龄合法性校验
    @validator("age")
    def check_age(cls, v):
        if v < 0:
            raise ValueError("年龄不能为负数")
        return v

    # 模型全局校验：多字段联动校验
    @model_validator(mode="after")
    def check_pwd(self):
        if len(self.password) < 6:
            raise ValueError("密码长度不能小于6位")
        return self
```

### 5\.4 Enum枚举、常用高阶数据类型

企业开发中常用枚举、时间、高精度小数、JSON扩展字段等特殊类型，Pydantic 原生支持解析校验，无需手动转换格式，适配状态枚举、时间筛选、金额计算、扩展字段存储场景。

```python
from enum import Enum
from datetime import datetime
from decimal import Decimal
from pydantic import Json

# 角色枚举：固定可选值，杜绝非法参数
class UserRole(str, Enum):
    ADMIN = "admin"
    USER = "user"

# 高阶类型模型
class UserType(BaseModel):
    role: UserRole        # 枚举角色
    create_time: datetime # 自动解析时间格式
    money: Decimal        # 高精度金额，避免浮点误差
    ext: Json             # 扩展JSON字段
```

### 5\.5 嵌套模型、泛型模型

嵌套模型用于处理多层级复杂JSON数据，适配地址、详情、配置等嵌套业务结构；泛型模型用于封装通用统一响应体，实现接口返回格式统一、代码复用，是中大型项目必备能力。

```python
from typing import Generic, TypeVar

# 嵌套子模型
class Address(BaseModel):
    province: str
    city: str

# 嵌套主模型
class UserNested(BaseModel):
    name: str
    address: Address

# 通用泛型响应模型
T = TypeVar("T")
class ResModel(Generic[T]):
    code: int
    data: T
```

# 六、标准化响应模型

接口响应是后端与前端数据交互的最终载体，企业项目必须统一响应格式、过滤敏感数据、标准化分页结构，保证前端对接统一、报错规范、数据安全。FastAPI 响应模型可自动过滤字段、统一返回结构，从框架层面规范输出。

### 6\.1 response\_model 响应过滤

核心作用：**自动过滤多余字段、隐藏敏感数据**，比如密码、密钥、隐私字段，无需手动删除，保证接口数据安全，统一返回字段规范。

```python
# 定义对外展示的响应模型，仅暴露name和age
class UserResp(BaseModel):
    name: str
    age: int

@app.get("/user/resp", response_model=UserResp)
def get_user_resp():
    # 后端原始数据包含密码，响应时自动过滤
    return {"name": "张三", "age": 20, "password": "123456"}
```

### 6\.2 统一返回格式 \+ 通用分页结构

所有业务接口必须采用统一响应格式，包含状态码、提示信息、业务数据；分页接口统一封装分页参数，避免每个接口自定义格式，降低前端对接成本。

```python
from typing import List

# 全局统一响应体
class Result(BaseModel):
    code: int = 200
    msg: str = "success"
    data: dict | list | None = None

# 通用分页响应体（继承统一响应，拓展分页字段）
class PageResult(Result):
    total: int = 0
    page: int = 1
    page_size: int = 10

@app.get("/page", response_model=PageResult)
def page_demo():
    return {
        "total": 100,
        "page": 1,
        "page_size": 10,
        "data": [{"name": "张三"}]
    }
```

# 七、全局异常处理体系

项目运行过程中必然会出现参数错误、权限不足、业务报错、系统异常等问题，原生异常返回格式混乱、前端难以处理。企业项目必须搭建**标准化异常处理体系**，统一所有错误返回格式，区分参数异常、业务异常、系统异常，方便前端捕获错误、提示用户、后端排查日志。

### 7\.1 基础 HTTPException 主动抛错

用于接口内主动抛出请求异常，适配参数错误、权限不足、资源不存在等常规请求错误。

```python
from fastapi import HTTPException

@app.get("/error")
def error_demo():
    # 主动抛出400参数错误
    raise HTTPException(status_code=400, detail="请求参数不合法，请检查参数")
```

### 7\.2 全局异常捕获与自定义业务异常

通过全局异常拦截器，统一捕获参数校验异常、自定义业务异常、系统未知异常，格式化返回统一JSON结构，彻底解决报错格式混乱问题。

```python
from fastapi import Request
from fastapi.responses import JSONResponse
from fastapi.exceptions import ValidationError

# 自定义业务异常类（统一业务报错）
class BizException(Exception):
    def __init__(self, code: int, msg: str):
        self.code = code
        self.msg = msg

# 全局参数校验异常拦截
@app.exception_handler(ValidationError)
async def validation_error_handler(request: Request, exc: ValidationError):
    return JSONResponse(status_code=200, content={"code": 400, "msg": "参数校验失败", "data": exc.errors()})

# 全局自定义业务异常拦截
@app.exception_handler(BizException)
async def biz_error_handler(request: Request, exc: BizException):
    return JSONResponse(status_code=200, content={"code": exc.code, "msg": exc.msg, "data": None})
```

# 八、依赖注入 Depends（核心重点）

依赖注入是 FastAPI 最核心、最优雅的架构设计，是实现**代码复用、解耦、权限控制、统一预处理**的核心机制。简单来说，就是把多个接口共用的逻辑（登录校验、权限校验、数据库连接、分页参数）抽离为公共依赖，按需注入到接口中，避免代码冗余，是企业项目分层架构的基石。

### 8\.1 函数依赖与核心原理

函数依赖是最基础的依赖用法，将公共逻辑封装为函数，通过 Depends 注入接口，自动执行预处理逻辑，适合登录校验、参数预处理等通用场景。

```python
from fastapi import Depends, Header, HTTPException

# 公共依赖：全局Token登录校验
def get_token(token: str = Header(None)):
    if not token:
        raise HTTPException(401, "用户未登录，请先登录")
    return token

# 接口注入依赖，自动执行登录校验
@app.get("/depends")
def depends_demo(token: str = Depends(get_token)):
    return {"token": token, "msg": "登录校验通过"}
```

### 8\.2 类依赖、多层依赖

类依赖适合封装多参数、可复用的参数实体（分页参数、查询参数）；多层依赖支持依赖嵌套，实现逻辑分层复用，适配复杂权限、多级校验场景。

```python
# 类依赖：通用分页参数封装
class PageParams:
    def __init__(self, page: int = 1, page_size: int = 10):
        self.page = page
        self.page_size = page_size

# 二级依赖：基于Token获取用户信息
def get_user_info(token: str = Depends(get_token)):
    return {"user_id": 1001, "username": "管理员"}

# 同时注入类依赖和函数依赖（多层依赖）
@app.get("/class_dep")
def class_dep(page: PageParams = Depends(), user: dict = Depends(get_user_info)):
    return {"page": page.page, "page_size": page.page_size, "user": user}
```

### 8\.3 权限依赖 \&amp; 数据库依赖

企业项目最常用的两大依赖：权限校验依赖实现接口权限管控，数据库会话依赖实现全局数据库连接复用，是所有后台系统的核心基础。

```python
# 权限依赖：管理员权限校验
def check_admin(user: dict = Depends(get_user_info)):
    if user.get("user_id") != 1001:
        raise HTTPException(403, "权限不足，仅管理员可访问")
    return user

# 数据库会话依赖：全局数据库连接
def get_db():
    db = "数据库会话对象"
    try:
        yield db
    finally:
        # 请求结束后关闭连接
        pass

# 同时注入数据库依赖和权限依赖
@app.get("/admin")
def admin_api(db = Depends(get_db), user = Depends(check_admin)):
    return {"db": db, "user": user, "msg": "管理员接口访问成功"}
```

# 九、中间件 Middleware

中间件是全局请求拦截器，会拦截项目所有的HTTP请求，在请求执行前、响应后执行统一逻辑，适合全局日志、请求耗时统计、跨域处理、全局拦截、权限预处理等全局通用逻辑。

### 9\.1 请求日志、请求耗时中间件

用于全局记录所有接口的请求路径、请求耗时，方便线上排查接口超时、请求异常问题，是项目运维必备中间件。

```python
import time
import logging
from fastapi import Request

# 初始化日志
logging.basicConfig(level=logging.INFO)

# 全局请求日志&耗时中间件
@app.middleware("http")
async def log_middleware(request: Request, call_next):
    start_time = time.time()
    # 执行接口逻辑
    response = await call_next(request)
    # 统计耗时
    cost = round((time.time() - start_time) * 1000, 2)
    logging.info(f"请求路径：{request.url}，请求耗时：{cost}ms")
    return response
```

### 9\.2 CORS 跨域全局配置

前后端分离项目必然存在跨域问题，FastAPI 可通过中间件全局配置跨域，允许前端域名跨域请求，彻底解决浏览器跨域拦截问题。

```python
from fastapi.middleware.cors import CORSMiddleware

# 全局跨域配置
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],      # 允许所有域名跨域（生产环境建议指定具体域名）
    allow_credentials=True,   # 允许携带Cookie
    allow_methods=["*"],      # 允许所有请求方法
    allow_headers=["*"],      # 允许所有请求头
)
```

# 十、项目生命周期事件

FastAPI 提供项目启动、关闭两大生命周期事件，用于处理项目全局初始化和资源释放逻辑。项目启动时执行初始化（连接数据库、加载配置、初始化缓存），项目关闭时执行资源释放（关闭连接、清空缓存）。

```python
# 项目启动生命周期：服务启动时执行一次
@app.on_event("startup")
async def startup_event():
    print("✅ 项目启动成功，初始化数据库、缓存、全局配置")

# 项目关闭生命周期：服务停止时执行一次
@app.on_event("shutdown")
async def shutdown_event():
    print("❌ 项目关闭，释放数据库连接、关闭资源")
```

# 十一、环境配置管理

正规项目必须区分**开发环境、测试环境、生产环境**，通过环境变量管理数据库地址、密钥、端口、环境标识等敏感配置，禁止硬编码配置，避免配置泄露、环境切换繁琐问题。通过 dotenv \+ pydantic\-settings 实现优雅配置管理。

### 11\.1 环境文件配置

项目根目录新建 \.env 文件，存储所有全局环境变量，不提交代码仓库，保证配置安全。

```env
# .env 环境配置文件
ENV=dev
DB_URL=mysql://root:123456@localhost:3306/test
SECRET_KEY=123456789abcdef
```

### 11\.2 配置读取封装

通过 pydantic\-settings 自动读取\.env文件配置，全局统一调用，支持类型校验、默认值、环境区分。

```python
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    # 加载环境文件
    model_config = SettingsConfigDict(env_file=".env", env_file_encoding="utf-8")
    ENV: str
    DB_URL: str
    SECRET_KEY: str

# 全局单例配置实例
settings = Settings()
# 全局调用
print("数据库地址：", settings.DB_URL)
```

# 十二、企业级分层项目结构

单文件项目仅适合测试学习，中大型商用项目必须采用**分层解耦架构**，按照职责拆分目录，代码各司其职、便于维护、迭代、多人协作，是90%企业通用的 FastAPI 项目架构。

```plain
app/
├── api/                # 路由接口层：仅接收请求、参数校验、调用业务
│   ├── __init__.py
│   ├── user.py
│   └── common.py
├── schemas/            # 数据模型层：Pydantic入参、出参、校验模型
│   ├── __init__.py
│   └── user_schema.py
├── models/            # ORM模型层：数据库表结构映射
│   ├── __init__.py
│   └── user_model.py
├── service/           # 业务逻辑层：核心业务、事务、流程处理
│   ├── __init__.py
│   └── user_service.py
├── dao/               # 数据操作层：纯数据库CRUD操作
│   ├── __init__.py
│   └── user_dao.py
├── core/              # 核心配置层：全局核心能力
│   ├── __init__.py
│   ├── config.py      # 全局配置
│   ├── exceptions.py  # 全局异常
│   └── dependencies.py # 全局依赖
├── utils/             # 工具层：通用工具函数
│   ├── __init__.py
│   ├── logger.py
│   └── common.py
├── main.py            # 项目唯一入口
└── .env               # 环境变量配置
```

### 分层职责详细说明

- **api路由层**：只负责接收前端请求、参数接收、依赖注入、调用service业务，禁止写任何业务逻辑和数据库操作。

- **schemas模型层**：统一管理所有请求入参、响应出参、数据校验规则，规范前后端数据交互格式。

- **models数据库层**：映射数据库表结构，定义字段类型、关联关系、约束规则。

- **service业务层**：处理核心业务逻辑、事务控制、业务校验、多DAO组合查询，是项目核心逻辑载体。

- **dao数据层**：仅负责纯数据库增删改查，无任何业务逻辑，实现数据操作复用。

- **core核心层**：存放全局配置、全局异常、全局依赖、中间件等基础核心能力。

- **utils工具层**：存放加密、日志、时间处理、正则、文件处理等通用工具方法。

该分层架构完全遵循**单一职责、高内聚低耦合**的设计思想，适配长期迭代、多人协作、线上商用项目。
