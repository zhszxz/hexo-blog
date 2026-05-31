---
title: FastAPI+SQLAlchemy项目实战
categories:
  - Python
  - FastAPI
date: 2026-05-28 22:14:53
tags:
---

# FastAPI + SQLAlchemy 项目实战（用户权限管理系统）

# 一、项目介绍

本篇笔记将使用：

- FastAPI
- SQLAlchemy 2.x
- MySQL
- JWT
- Pydantic
- Alembic

实现一个完整的：

# 用户权限管理系统（RBAC）

功能包含：

- 用户注册
- 用户登录
- JWT 鉴权
- 用户 CRUD
- 角色管理
- 菜单管理
- 用户角色绑定
- 分页查询
- 全局异常处理
- 统一响应结构
- Swagger 接口测试

项目采用：

- Router
- Service
- Schema
- ORM
- Model

分层架构。

------

# 二、项目结构设计

推荐采用企业级目录结构：

```bash
fastapi_rbac/
│
├── app/
│   ├── main.py
│   │
│   ├── core/
│   │   ├── config.py
│   │   ├── database.py
│   │   ├── security.py
│   │   ├── response.py
│   │   └── exception.py
│   │
│   ├── models/
│   │   ├── user.py
│   │   ├── role.py
│   │   ├── menu.py
│   │   └── user_role.py
│   │
│   ├── schemas/
│   │   ├── user.py
│   │   ├── role.py
│   │   └── menu.py
│   │
│   ├── service/
│   │   ├── user_service.py
│   │   └── auth_service.py
│   │
│   ├── api/
│   │   ├── user.py
│   │   ├── auth.py
│   │   └── role.py
│   │
│   └── utils/
│       └── password.py
│
├── requirements.txt
└── alembic.ini
```

------

# 三、安装依赖

```bash
pip install fastapi uvicorn sqlalchemy pymysql
pip install python-jose passlib[bcrypt]
pip install python-multipart
pip install alembic
```

requirements.txt：

```txt
fastapi
uvicorn
sqlalchemy
pymysql
python-jose
passlib[bcrypt]
python-multipart
alembic
```

------

# 四、数据库设计

# 1、用户表

```sql
CREATE TABLE sys_user (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    nickname VARCHAR(50),
    is_active TINYINT DEFAULT 1,
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

------

# 2、角色表

```sql
CREATE TABLE sys_role (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    role_name VARCHAR(50) NOT NULL,
    role_code VARCHAR(50) UNIQUE NOT NULL
);
```

------

# 3、菜单表

```sql
CREATE TABLE sys_menu (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    menu_name VARCHAR(50),
    path VARCHAR(100),
    component VARCHAR(100)
);
```

------

# 4、用户角色中间表

```sql
CREATE TABLE sys_user_role (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT,
    role_id BIGINT
);
```

------

# 5、角色菜单中间表

```sql
CREATE TABLE sys_role_menu (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    role_id BIGINT,
    menu_id BIGINT
);
```

------

# 五、数据库 Session 管理

# 1、创建数据库连接

app/core/database.py

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, DeclarativeBase

DATABASE_URL = "mysql+pymysql://root:123456@127.0.0.1:3306/fastapi_rbac"

engine = create_engine(
    DATABASE_URL,
    echo=True
)

SessionLocal = sessionmaker(
    autoflush=False,
    autocommit=False,
    bind=engine
)

class Base(DeclarativeBase):
    pass
```

------

# 2、创建 Session 依赖

```python
from app.core.database import SessionLocal

def get_db():
    db = SessionLocal()

    try:
        yield db
    finally:
        db.close()
```

FastAPI 推荐使用 Depends 自动管理数据库连接生命周期。

------

# 六、ORM 关系映射

# 1、用户模型

app/models/user.py

```python
from sqlalchemy import String
from sqlalchemy.orm import Mapped
from sqlalchemy.orm import mapped_column
from sqlalchemy.orm import relationship

from app.core.database import Base

class User(Base):
    __tablename__ = "sys_user"

    id: Mapped[int] = mapped_column(primary_key=True)

    username: Mapped[str] = mapped_column(
        String(50),
        unique=True
    )

    password: Mapped[str] = mapped_column(String(255))

    nickname: Mapped[str] = mapped_column(String(50))

    roles = relationship(
        "Role",
        secondary="sys_user_role",
        back_populates="users"
    )
```

------

# 2、角色模型

app/models/role.py

```python
from sqlalchemy import String
from sqlalchemy.orm import Mapped
from sqlalchemy.orm import mapped_column
from sqlalchemy.orm import relationship

from app.core.database import Base

class Role(Base):
    __tablename__ = "sys_role"

    id: Mapped[int] = mapped_column(primary_key=True)

    role_name: Mapped[str] = mapped_column(String(50))

    role_code: Mapped[str] = mapped_column(
        String(50),
        unique=True
    )

    users = relationship(
        "User",
        secondary="sys_user_role",
        back_populates="roles"
    )
```

------

# 3、中间表

app/models/user_role.py

```python
from sqlalchemy import ForeignKey
from sqlalchemy.orm import Mapped
from sqlalchemy.orm import mapped_column

from app.core.database import Base

class UserRole(Base):
    __tablename__ = "sys_user_role"

    id: Mapped[int] = mapped_column(primary_key=True)

    user_id: Mapped[int] = mapped_column(
        ForeignKey("sys_user.id")
    )

    role_id: Mapped[int] = mapped_column(
        ForeignKey("sys_role.id")
    )
```

------

# 七、Schema 数据校验

# 1、用户 Schema

app/schemas/user.py

```python
from pydantic import BaseModel

class UserCreate(BaseModel):
    username: str
    password: str
    nickname: str

class UserLogin(BaseModel):
    username: str
    password: str

class UserResponse(BaseModel):
    id: int
    username: str
    nickname: str

    class Config:
        from_attributes = True
```

------

# 八、密码加密

app/utils/password.py

```python
from passlib.context import CryptContext

pwd_context = CryptContext(
    schemes=["bcrypt"],
    deprecated="auto"
)

def hash_password(password: str):
    return pwd_context.hash(password)

def verify_password(
    plain_password: str,
    hashed_password: str
):
    return pwd_context.verify(
        plain_password,
        hashed_password
    )
```

------

# 九、用户注册功能

# 1、Service 层

app/service/user_service.py

```python
from sqlalchemy.orm import Session

from app.models.user import User
from app.schemas.user import UserCreate
from app.utils.password import hash_password

def create_user(
    db: Session,
    user: UserCreate
):
    db_user = User(
        username=user.username,
        password=hash_password(user.password),
        nickname=user.nickname
    )

    db.add(db_user)

    db.commit()

    db.refresh(db_user)

    return db_user
```

------

# 2、Router 层

app/api/user.py

```python
from fastapi import APIRouter
from fastapi import Depends
from sqlalchemy.orm import Session

from app.schemas.user import UserCreate
from app.service.user_service import create_user
from app.core.database import get_db

router = APIRouter(
    prefix="/user",
    tags=["用户管理"]
)

@router.post("/register")
def register(
    user: UserCreate,
    db: Session = Depends(get_db)
):
    return create_user(db, user)
```

------

# 十、JWT 鉴权

# 1、JWT 原理

JWT：

JSON Web Token

由三部分组成：

```txt
header.payload.signature
```

服务端登录成功后：

- 生成 token
- 返回给前端
- 前端后续请求携带 token
- 服务端解析 token 获取用户信息

JWT 是目前前后端分离项目最主流的认证方式。

------

# 2、创建 JWT 工具

app/core/security.py

```python
from datetime import datetime
from datetime import timedelta

from jose import jwt

SECRET_KEY = "fastapi"
ALGORITHM = "HS256"

def create_access_token(data: dict):

    to_encode = data.copy()

    expire = datetime.utcnow() + timedelta(hours=2)

    to_encode.update({
        "exp": expire
    })

    return jwt.encode(
        to_encode,
        SECRET_KEY,
        algorithm=ALGORITHM
    )
```

------

# 3、登录接口

app/service/auth_service.py

```python
from sqlalchemy.orm import Session

from app.models.user import User
from app.schemas.user import UserLogin

from app.utils.password import verify_password

from app.core.security import create_access_token

def login(
    db: Session,
    user: UserLogin
):

    db_user = db.query(User).filter(
        User.username == user.username
    ).first()

    if not db_user:
        return None

    if not verify_password(
        user.password,
        db_user.password
    ):
        return None

    token = create_access_token({
        "user_id": db_user.id,
        "username": db_user.username
    })

    return {
        "access_token": token,
        "token_type": "bearer"
    }
```

------

# 4、登录路由

app/api/auth.py

```python
from fastapi import APIRouter
from fastapi import Depends

from sqlalchemy.orm import Session

from app.schemas.user import UserLogin

from app.service.auth_service import login

from app.core.database import get_db

router = APIRouter(
    prefix="/auth",
    tags=["认证模块"]
)

@router.post("/login")
def user_login(
    user: UserLogin,
    db: Session = Depends(get_db)
):
    return login(db, user)
```

------

# 十一、Depends 获取当前用户

# 1、解析 JWT

```python
from jose import jwt
from jose import JWTError

from fastapi import Depends
from fastapi.security import OAuth2PasswordBearer

from sqlalchemy.orm import Session

from app.models.user import User
from app.core.database import get_db

oauth2_scheme = OAuth2PasswordBearer(
    tokenUrl="/auth/login"
)

def get_current_user(
    token: str = Depends(oauth2_scheme),
    db: Session = Depends(get_db)
):

    try:

        payload = jwt.decode(
            token,
            SECRET_KEY,
            algorithms=[ALGORITHM]
        )

        user_id = payload.get("user_id")

        user = db.query(User).filter(
            User.id == user_id
        ).first()

        return user

    except JWTError:
        return None
```

------

# 2、使用 Depends 获取用户

```python
@router.get("/info")
def user_info(
    current_user: User = Depends(get_current_user)
):
    return current_user
```

------

# 十二、用户 CRUD

# 1、查询用户

```python
@router.get("/{user_id}")
def get_user(
    user_id: int,
    db: Session = Depends(get_db)
):
    return db.query(User).filter(
        User.id == user_id
    ).first()
```

------

# 2、修改用户

```python
@router.put("/{user_id}")
def update_user(
    user_id: int,
    nickname: str,
    db: Session = Depends(get_db)
):

    user = db.query(User).filter(
        User.id == user_id
    ).first()

    user.nickname = nickname

    db.commit()

    return user
```

------

# 3、删除用户

```python
@router.delete("/{user_id}")
def delete_user(
    user_id: int,
    db: Session = Depends(get_db)
):

    user = db.query(User).filter(
        User.id == user_id
    ).first()

    db.delete(user)

    db.commit()

    return {
        "msg": "删除成功"
    }
```

------

# 十三、角色权限系统（RBAC）

RBAC：

Role Based Access Control

即：

基于角色的权限控制。

企业项目中通常不会直接给用户分配权限，而是：

```txt
用户 -> 角色 -> 权限
```

例如：

```txt
管理员
运营
财务
访客
```

不同角色拥有不同菜单权限。

------

# 1、用户与角色

```txt
用户
  ↓
角色
  ↓
菜单权限
```

------

# 2、角色接口

```python
@router.post("/assign-role")
def assign_role(
    user_id: int,
    role_id: int,
    db: Session = Depends(get_db)
):

    relation = UserRole(
        user_id=user_id,
        role_id=role_id
    )

    db.add(relation)

    db.commit()

    return {
        "msg": "角色分配成功"
    }
```

------

# 3、查询用户角色

```python
@router.get("/{user_id}/roles")
def get_user_roles(
    user_id: int,
    db: Session = Depends(get_db)
):

    user = db.query(User).filter(
        User.id == user_id
    ).first()

    return user.roles
```

------

# 十四、分页查询

企业项目中：

绝对不能一次查询全部数据。

必须分页。

------

# 1、分页参数

```python
page: int = 1
page_size: int = 10
```

------

# 2、分页实现

```python
@router.get("/list")
def user_list(
    page: int = 1,
    page_size: int = 10,
    db: Session = Depends(get_db)
):

    query = db.query(User)

    total = query.count()

    users = query.offset(
        (page - 1) * page_size
    ).limit(page_size).all()

    return {
        "total": total,
        "items": users
    }
```

------

# 十五、统一响应结构

企业项目必须统一响应格式。

否则前端很难统一处理。

------

# 1、统一响应结构

app/core/response.py

```python
def success(
    data=None,
    msg="success",
    code=200
):

    return {
        "code": code,
        "msg": msg,
        "data": data
    }

def fail(
    msg="fail",
    code=500
):

    return {
        "code": code,
        "msg": msg
    }
```

------

# 2、接口统一返回

```python
@router.get("/list")
def user_list():

    return success(
        data={
            "items": [],
            "total": 0
        }
    )
```

------

# 十六、全局异常处理

# 1、自定义异常

```python
class ApiException(Exception):

    def __init__(
        self,
        code: int,
        msg: str
    ):
        self.code = code
        self.msg = msg
```

------

# 2、注册异常处理器

app/core/exception.py

```python
from fastapi.responses import JSONResponse

def register_exception(app):

    @app.exception_handler(ApiException)
    async def api_exception_handler(
        request,
        exc: ApiException
    ):

        return JSONResponse({
            "code": exc.code,
            "msg": exc.msg
        })
```

------

# 3、main.py 注册

```python
from app.core.exception import register_exception

register_exception(app)
```

------

# 十七、Swagger 接口测试

FastAPI 最大优势之一：

自动生成 Swagger 文档。

启动项目：

```bash
uvicorn app.main:app --reload
```

访问：

```txt
http://127.0.0.1:8000/docs
```

即可看到：

- 注册接口
- 登录接口
- 用户 CRUD
- JWT 鉴权

等所有接口。

------

# 十八、Swagger JWT 认证

使用：

```python
OAuth2PasswordBearer
```

后：

Swagger 会自动出现：

```txt
Authorize
```

按钮。

输入：

```txt
Bearer token
```

即可完成认证测试。

------

# 十九、Postman 接口测试

# 1、注册

```http
POST /user/register
```

Body：

```json
{
  "username": "admin",
  "password": "123456",
  "nickname": "管理员"
}
```

------

# 2、登录

```http
POST /auth/login
```

返回：

```json
{
  "access_token": "xxx",
  "token_type": "bearer"
}
```

------

# 3、携带 Token

Headers：

```txt
Authorization: Bearer xxx
```

------

# 二十、项目完整开发流程

企业项目标准开发流程：

------

# 第一步：建表

先设计：

- 用户
- 角色
- 权限
- 中间表

数据库结构。

------

# 第二步：编写 ORM

使用 SQLAlchemy：

```python
class User(Base):
```

映射数据库表。

------

# 第三步：编写 Schema

使用 Pydantic：

```python
class UserCreate(BaseModel):
```

完成：

- 参数校验
- 数据转换
- 返回结构

------

# 第四步：编写 Service

Service 负责：

- 业务逻辑
- 数据库操作
- 权限判断

例如：

```python
create_user()
login()
```

------

# 第五步：编写 Router

Router 负责：

- 接收请求
- 参数解析
- 调用 Service

例如：

```python
@router.post("/login")
```

------

# 第六步：统一异常

统一：

- 参数错误
- 登录失败
- 权限不足
- 数据不存在

处理逻辑。

------

# 第七步：接口测试

使用：

- Swagger
- Postman

完成接口联调。

------

# 二十一、main.py 完整整合

```python
from fastapi import FastAPI

from app.api.user import router as user_router
from app.api.auth import router as auth_router

from app.core.exception import register_exception

app = FastAPI()

app.include_router(user_router)
app.include_router(auth_router)

register_exception(app)
```

------

# 二十二、Alembic 数据库迁移

初始化：

```bash
alembic init alembic
```

生成迁移：

```bash
alembic revision --autogenerate -m "init"
```

执行迁移：

```bash
alembic upgrade head
```

企业项目强烈推荐使用 Alembic 管理数据库版本。

------

# 二十三、项目优化方向

实际企业项目中还会继续扩展：

------

# 1、Redis 登录缓存

例如：

- Token 黑名单
- 登录状态
- 验证码

------

# 2、权限拦截器

例如：

```python
/admin/*
```

只有管理员可访问。

------

# 3、菜单树

前端动态菜单：

```txt
系统管理
  用户管理
  角色管理
```

------

# 4、日志系统

记录：

- 登录日志
- 操作日志
- 异常日志

------

# 5、异步 SQLAlchemy

FastAPI 最大优势之一：

支持 async。

企业项目推荐：

```python
create_async_engine
AsyncSession
```

提升并发性能。

------

# 6、配置管理

使用：

```python
pydantic-settings
```

管理：

- 数据库
- Redis
- JWT
- 邮件

等配置。

------

# 二十四、FastAPI 企业项目最佳实践

推荐：

------

# 1、严格分层

```txt
Router
↓
Service
↓
DAO
↓
Database
```

------

# 2、禁止 Router 写 SQL

Router 只负责：

- 接收参数
- 返回响应

业务逻辑必须放 Service。

------

# 3、统一异常

不要：

```python
return {
    "error": "xxx"
}
```

应该：

```python
raise ApiException()
```

------

# 4、统一响应

所有接口：

```json
{
  "code": 200,
  "msg": "success",
  "data": {}
}
```

------

# 5、JWT 不存数据库

JWT：

无状态认证。

不需要 Session。

适合：

- 前后端分离
- 微服务

------

# 二十五、FastAPI 与 Flask/Django 对比

| 框架    | 特点                         |
| ------- | ---------------------------- |
| Flask   | 灵活，但需要自己搭建         |
| Django  | 大而全                       |
| FastAPI | 高性能 + 自动文档 + 类型提示 |

FastAPI 优势：

- 开发效率高
- 自动 Swagger
- 异步支持
- 类型安全
- 性能接近 Go

------

# 二十六、完整项目接口列表

| 模块 | 接口         |
| ---- | ------------ |
| 用户 | 注册         |
| 用户 | 登录         |
| 用户 | 用户列表     |
| 用户 | 用户详情     |
| 用户 | 修改用户     |
| 用户 | 删除用户     |
| 角色 | 分配角色     |
| 权限 | 获取当前用户 |

------

# 二十七、项目运行流程

启动：

```bash
uvicorn app.main:app --reload
```

访问：

```txt
http://127.0.0.1:8000/docs
```

流程：

```txt
注册用户
↓
登录获取 token
↓
Swagger Authorize
↓
访问受保护接口
↓
角色权限控制
```

------

# 二十八、总结

本篇完整实现了：

# FastAPI + SQLAlchemy 用户权限管理系统

核心内容包括：

- 项目结构设计
- 数据库设计
- SQLAlchemy ORM
- 用户 CRUD
- JWT 鉴权
- RBAC 权限系统
- 分页查询
- 统一响应
- 全局异常处理
- Session 管理
- Swagger/Postman 测试
- 企业级开发流程

通过本项目，你已经掌握了：

# FastAPI 企业级后端开发核心能力

后续建议继续学习：

- Redis
- Celery
- Docker
- Nginx
- 微服务
- Kafka
- 异步任务
- OAuth2
- WebSocket

这些内容会进一步提升你的 FastAPI 实战能力。
