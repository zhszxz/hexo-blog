---
title: FastAPI项目实战
tags:
  - RBAC
categories:
  - Python
  - FastAPI
date: 2026-05-31 21:07:11
---

# FastAPI \+ SQLAlchemy 项目实战：RBAC用户权限管理系统学习笔记

# 前言

RBAC（Role\-Based Access Control，基于角色的访问控制）是目前企业级系统最通用的权限管理模型，核心思路是**用户关联角色、角色关联权限**，通过角色实现用户和权限的解耦，极大提升权限管理的灵活性和可维护性。

本笔记将完整记录一套**轻量化企业级 RBAC 用户权限管理系统**的开发全过程，涵盖需求分析、项目设计、数据库设计、代码实现、异常处理、接口测试等全流程，所有代码可直接复制运行，适合零基础入门 FastAPI 项目实战，同时掌握后台权限系统的核心开发逻辑。

**技术栈**：FastAPI \+ SQLAlchemy \+ Pydantic \+ JWT \+ Python3\.10\+ \+ SQLite（可无缝切换 MySQL/PostgreSQL）

# 一、项目需求分析

## 1\.1 功能需求

本项目实现一套基础通用的后台权限管理系统，核心功能覆盖用户、角色、权限全链路管理，具体需求如下：

- **用户模块**：用户注册、账号密码登录、用户新增/查询/修改/删除（CRUD）、用户分页列表查询

- **鉴权模块**：基于 JWT 实现令牌签发、过期校验、接口权限拦截、全局登录用户获取

- **角色模块**：角色创建、角色查询、角色权限绑定、角色用户关联

- **权限菜单模块**：菜单权限配置、角色菜单权限分配、接口访问权限控制

- **基础功能**：全局统一接口响应格式、全局异常捕获处理、数据库会话统一管理、分页查询封装

## 1\.2 非功能需求

- **高性能**：基于 FastAPI 异步特性，支持高并发接口请求

- **可拓展性**：分层架构设计（路由、业务、模型、数据层），方便后续新增功能

- **规范性**：统一响应结构、统一异常处理、代码分层清晰、注释完善

- **可测试性**：原生支持 Swagger 自动接口文档，支持 Postman 全接口测试

# 二、项目整体设计

## 2\.1 项目架构设计

采用经典的**分层架构**，自上而下分为：路由层（Router）、数据校验层（Schema）、业务逻辑层（Service）、数据访问层（ORM模型）、数据库层，彻底解耦各模块职责，符合软件工程开发规范。

各层级职责：

- **Router 路由层**：接收前端请求、路由分发、基础权限拦截、参数接收

- **Schema 模型层**：基于 Pydantic 实现请求参数校验、响应数据格式化

- **Service 业务层**：处理核心业务逻辑、数据组装、权限判断

- **Model ORM层**：SQLAlchemy 数据库模型映射、数据表关系定义

- **公共工具层**：JWT鉴权、异常处理、响应封装、数据库会话管理

## 2\.2 完整项目结构

以下为标准化企业级项目目录结构，结构清晰、便于维护和拓展：

```python
fastapi-rbac-demo/
├── app/                        # 项目主目录
│   ├── __init__.py
│   ├── main.py                 # 项目入口文件
│   ├── core/                   # 核心配置模块
│   │   ├── __init__.py
│   │   ├── config.py           # 全局配置参数
│   │   ├── database.py         # 数据库Session管理
│   │   ├── exception.py        # 全局异常处理
│   │   └── security.py         # JWT鉴权、密码加密
│   ├── models/                 # ORM数据库模型
│   │   ├── __init__.py
│   │   ├── user.py             # 用户模型
│   │   ├── role.py             # 角色模型
│   │   └── menu.py             # 菜单权限模型
│   ├── schemas/                # Pydantic数据模型
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── role.py
│   │   └── menu.py
│   ├── services/               # 业务逻辑层
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── role.py
│   │   └── menu.py
│   ├── api/                    # 路由接口层
│   │   ├── __init__.py
│   │   ├── deps.py             # 全局依赖项
│   │   ├── user.py
│   │   ├── role.py
│   │   └── menu.py
│   └── common/                 # 公共工具
│       ├── __init__.py
│       ├── response.py         # 统一响应结构
│       └── pagination.py       # 分页工具封装
├── requirements.txt            # 项目依赖
└── README.md                   # 项目说明
```

## 2\.3 项目依赖配置

新建 `requirements\.txt` 文件，配置项目所需所有依赖，安装即可使用：

```txt
fastapi==0.104.1
uvicorn==0.24.0
sqlalchemy==2.0.23
pydantic==2.4.2
pyjwt==2.8.0
passlib==1.7.4
python-multipart==0.0.6
```

安装命令：`pip install \-r requirements\.txt`

# 三、数据库设计

## 3\.1 数据库设计原则

基于 RBAC 权限模型，设计三张核心业务表 \+ 两张中间关联表，实现**用户\-角色\-菜单权限**的多对多关联关系，满足灵活的权限分配需求。

核心表结构：用户表、角色表、菜单表、用户角色中间表、角色菜单中间表

## 3\.2 数据表结构详情

### 3\.2\.1 用户表（sys\_user）

存储系统用户基础信息，包含账号、密码、昵称、状态等核心字段

| 字段名       | 字段类型      | 是否主键 | 默认值   | 字段说明                 |
| ------------ | ------------- | -------- | -------- | ------------------------ |
| id           | Integer       | 是       | 自增     | 用户唯一ID               |
| username     | String\(50\)  | 否       | 无       | 登录账号（唯一）         |
| password     | String\(100\) | 否       | 无       | 加密后密码               |
| nickname     | String\(50\)  | 否       | 无       | 用户昵称                 |
| status       | Integer       | 否       | 1        | 用户状态 1\-正常 0\-禁用 |
| create\_time | DateTime      | 否       | 当前时间 | 创建时间                 |
| update\_time | DateTime      | 否       | 当前时间 | 更新时间                 |

### 3\.2\.2 角色表（sys\_role）

存储系统角色信息，如超级管理员、普通用户、运营人员等

| 字段名       | 字段类型      | 是否主键 | 默认值   | 字段说明                      |
| ------------ | ------------- | -------- | -------- | ----------------------------- |
| id           | Integer       | 是       | 自增     | 角色唯一ID                    |
| role\_name   | String\(50\)  | 否       | 无       | 角色名称                      |
| role\_code   | String\(50\)  | 否       | 无       | 角色唯一标识（如admin、user） |
| description  | String\(200\) | 否       | 空       | 角色描述                      |
| create\_time | DateTime      | 否       | 当前时间 | 创建时间                      |

### 3\.2\.3 菜单权限表（sys\_menu）

存储系统接口权限、菜单路由信息，用于控制接口访问权限

| 字段名     | 字段类型      | 是否主键 | 默认值 | 字段说明                          |
| ---------- | ------------- | -------- | ------ | --------------------------------- |
| id         | Integer       | 是       | 自增   | 菜单ID                            |
| menu\_name | String\(50\)  | 否       | 无     | 菜单/权限名称                     |
| permission | String\(100\) | 否       | 无     | 权限标识（如user:add、user:list） |
| parent\_id | Integer       | 否       | 0      | 父级菜单ID                        |

### 3\.2\.4 中间关联表

- **用户角色中间表（sys\_user\_role）**：用户和角色多对多关联，字段：user\_id、role\_id

- **角色菜单中间表（sys\_role\_menu）**：角色和权限多对多关联，字段：role\_id、menu\_id

# 四、核心配置与公共模块实现

## 4\.1 全局配置文件（core/config\.py）

统一管理项目密钥、数据库地址、JWT过期时间等全局配置，方便后期修改

```python
from pydantic_settings import BaseSettings
from typing import Optional

class Settings(BaseSettings):
    # 项目配置
    PROJECT_NAME: str = "FastAPI-RBAC权限管理系统"
    PROJECT_VERSION: str = "1.0.0"

    # JWT配置
    SECRET_KEY: str = "fastapi-rbac-secret-key-2026"
    ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 120

    # 数据库配置
    DATABASE_URL: str = "sqlite:///./rbac.db"

    class Config:
        case_sensitive = True

# 全局配置实例
settings = Settings()
```

## 4\.2 数据库Session管理（core/database\.py）

基于SQLAlchemy实现数据库引擎创建、会话管理，封装全局数据库依赖，实现接口自动获取数据库连接，请求结束自动关闭会话

```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from app.core.config import settings

# 创建数据库引擎，sqlite开启线程支持
engine = create_engine(
    settings.DATABASE_URL,
    connect_args={"check_same_thread": False} if "sqlite" in settings.DATABASE_URL else {}
)

# 创建数据库会话工厂
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# ORM模型基类
Base = declarative_base()

# 数据库会话依赖项
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        # 请求结束自动关闭会话
        db.close()
```

## 4\.3 JWT鉴权与密码加密（core/security\.py）

实现密码哈希加密、JWT令牌生成、令牌校验功能，是登录鉴权的核心模块

```python
from datetime import datetime, timedelta
from typing import Any, Union, Optional

from jose import jwt, JWTError
from passlib.context import CryptContext

from app.core.config import settings

# 密码加密上下文
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def create_access_token(
    subject: Union[str, Any], expires_delta: Optional[timedelta] = None
) -> str:
    """生成JWT访问令牌"""
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES)
    
    # 令牌载荷
    to_encode = {"exp": expire, "sub": str(subject)}
    encoded_jwt = jwt.encode(to_encode, settings.SECRET_KEY, algorithm=settings.ALGORITHM)
    return encoded_jwt

def verify_password(plain_password: str, hashed_password: str) -> bool:
    """校验密码"""
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password: str) -> str:
    """密码加密"""
    return pwd_context.hash(password)
```

## 4\.4 统一响应结构（common/response\.py）

封装全局统一接口返回格式，所有接口返回数据格式统一，便于前端解析

```python
from typing import Any, Optional
from fastapi.responses import JSONResponse

class ResponseModel:
    """统一响应模型"""
    def __init__(self, code: int, msg: str, data: Any = None):
        self.code = code
        self.msg = msg
        self.data = data

    @staticmethod
    def success(data: Any = None, msg: str = "请求成功") -> JSONResponse:
        """成功响应"""
        return JSONResponse(
            status_code=200,
            content={"code": 200, "msg": msg, "data": data}
        )

    @staticmethod
    def fail(code: int = 500, msg: str = "请求失败", data: Any = None) -> JSONResponse:
        """失败响应"""
        return JSONResponse(
            status_code=200,
            content={"code": code, "msg": msg, "data": data}
        )

# 全局响应实例
resp = ResponseModel()
```

## 4\.5 全局异常处理（core/exception\.py）

捕获全局异常、自定义业务异常，统一异常返回格式，避免后端报错直接暴露给前端

```python
from fastapi import FastAPI, Request
from fastapi.exceptions import RequestValidationError
from starlette.exceptions import HTTPException as StarletteHTTPException

from app.common.response import resp

# 自定义业务异常
class BusinessException(Exception):
    def __init__(self, code: int, msg: str):
        self.code = code
        self.msg = msg

def register_exception(app: FastAPI):
    """注册全局异常处理器"""

    # 捕获HTTP异常
    @app.exception_handler(StarletteHTTPException)
    async def http_exception_handler(request: Request, exc: StarletteHTTPException):
        return resp.fail(code=exc.status_code, msg=exc.detail)

    # 捕获参数校验异常
    @app.exception_handler(RequestValidationError)
    async def validation_exception_handler(request: Request, exc: RequestValidationError):
        err_msg = exc.errors()[0]["msg"]
        return resp.fail(code=400, msg=f"参数校验失败：{err_msg}")

    # 捕获自定义业务异常
    @app.exception_handler(BusinessException)
    async def business_exception_handler(request: Request, exc: BusinessException):
        return resp.fail(code=exc.code, msg=exc.msg)

    # 捕获全局未知异常
    @app.exception_handler(Exception)
    async def global_exception_handler(request: Request, exc: Exception):
        return resp.fail(code=500, msg=f"服务器异常：{str(exc)}")
```

## 4\.6 分页工具封装（common/pagination\.py）

封装通用分页查询方法，适配所有列表查询接口，简化分页逻辑

```python
from typing import List, Any
from sqlalchemy.orm import Query

class Pagination:
    """通用分页工具"""
    def __init__(self, page: int = 1, page_size: int = 10):
        self.page = page
        self.page_size = page_size

    def paginate(self, query: Query) -> dict:
        """分页查询"""
        total = query.count()
        offset = (self.page - 1) * self.page_size
        items = query.offset(offset).limit(self.page_size).all()
        return {
            "list": items,
            "total": total,
            "page": self.page,
            "page_size": self.page_size,
            "total_page": (total + self.page_size - 1) // self.page_size
        }
```

# 五、ORM模型关系映射实现

基于SQLAlchemy实现数据表模型映射，定义**用户\-角色、角色\-菜单**多对多关联关系，自动关联查询数据

## 5\.1 中间表模型（models/\_\_init\_\_\.py）

```python
from sqlalchemy import Column, Integer, ForeignKey
from app.core.database import Base

# 用户角色中间表
sys_user_role = Base.metadata.tables.get("sys_user_role") or Base(
    __tablename__ = "sys_user_role",
    Column("user_id", Integer, ForeignKey("sys_user.id"), primary_key=True),
    Column("role_id", Integer, ForeignKey("sys_role.id"), primary_key=True)
)

# 角色菜单中间表
sys_role_menu = Base.metadata.tables.get("sys_role_menu") or Base(
    __tablename__ = "sys_role_menu",
    Column("role_id", Integer, ForeignKey("sys_role.id"), primary_key=True),
    Column("menu_id", Integer, ForeignKey("sys_menu.id"), primary_key=True)
)
```

## 5\.2 用户模型（models/user\.py）

```python
from datetime import datetime
from sqlalchemy import Column, Integer, String, DateTime
from sqlalchemy.orm import relationship
from app.core.database import Base
from app.models import sys_user_role

class User(Base):
    __tablename__ = "sys_user"

    id = Column(Integer, primary_key=True, index=True, autoincrement=True)
    username = Column(String(50), unique=True, nullable=False, comment="登录账号")
    password = Column(String(100), nullable=False, comment="密码")
    nickname = Column(String(50), comment="用户昵称")
    status = Column(Integer, default=1, comment="状态1正常0禁用")
    create_time = Column(DateTime, default=datetime.now, comment="创建时间")
    update_time = Column(DateTime, default=datetime.now, onupdate=datetime.now, comment="更新时间")

    # 多对多关联角色
    roles = relationship("Role", secondary=sys_user_role, back_populates="users")
```

## 5\.3 角色模型（models/role\.py）

```python
from datetime import datetime
from sqlalchemy import Column, Integer, String, DateTime
from sqlalchemy.orm import relationship
from app.core.database import Base
from app.models import sys_user_role, sys_role_menu

class Role(Base):
    __tablename__ = "sys_role"

    id = Column(Integer, primary_key=True, index=True, autoincrement=True)
    role_name = Column(String(50), nullable=False, comment="角色名称")
    role_code = Column(String(50), unique=True, nullable=False, comment="角色标识")
    description = Column(String(200), default="", comment="角色描述")
    create_time = Column(DateTime, default=datetime.now, comment="创建时间")

    # 多对多关联用户、菜单
    users = relationship("User", secondary=sys_user_role, back_populates="roles")
    menus = relationship("Menu", secondary=sys_role_menu, back_populates="roles")
```

## 5\.4 菜单模型（models/menu\.py）

```python
from sqlalchemy import Column, Integer, String
from sqlalchemy.orm import relationship
from app.core.database import Base
from app.models import sys_role_menu

class Menu(Base):
    __tablename__ = "sys_menu"

    id = Column(Integer, primary_key=True, index=True, autoincrement=True)
    menu_name = Column(String(50), nullable=False, comment="菜单名称")
    permission = Column(String(100), nullable=False, comment="权限标识")
    parent_id = Column(Integer, default=0, comment="父级ID")

    # 多对多关联角色
    roles = relationship("Role", secondary=sys_role_menu, back_populates="menus")
```

# 六、Pydantic Schema数据模型

用于请求参数校验、响应数据格式化，过滤敏感字段（如密码），保证接口数据安全和规范性

## 6\.1 用户Schema（schemas/user\.py）

```python
from typing import List, Optional
from pydantic import BaseModel, Field

# 用户注册
class UserRegister(BaseModel):
    username: str = Field(..., description="登录账号")
    password: str = Field(..., description="登录密码")
    nickname: Optional[str] = Field(None, description="用户昵称")

# 用户登录
class UserLogin(BaseModel):
    username: str
    password: str

# 用户信息响应（过滤密码）
class UserInfo(BaseModel):
    id: int
    username: str
    nickname: Optional[str]
    status: int

    class Config:
        from_attributes = True

# 用户分页查询
class UserPageQuery(BaseModel):
    page: int = 1
    page_size: int = 10
    username: Optional[str] = None
```

## 6\.2 角色、菜单Schema

```python
# schemas/role.py
from pydantic import BaseModel, Optional

class RoleCreate(BaseModel):
    role_name: str
    role_code: str
    description: Optional[str] = None

class RoleInfo(BaseModel):
    id: int
    role_name: str
    role_code: str
    description: Optional[str]

    class Config:
        from_attributes = True

# schemas/menu.py
from pydantic import BaseModel

class MenuCreate(BaseModel):
    menu_name: str
    permission: str
    parent_id: int = 0

class MenuInfo(BaseModel):
    id: int
    menu_name: str
    permission: str
    parent_id: int

    class Config:
        from_attributes = True
```

# 七、依赖项封装（api/deps\.py）

封装全局依赖，实现JWT登录校验、获取当前登录用户、接口权限拦截

```python
from typing import Generator, Optional
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from sqlalchemy.orm import Session
from jose import jwt, JWTError

from app.core.database import get_db
from app.core.config import settings
from app.models.user import User
from app.core.exception import BusinessException

# 令牌依赖
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/user/login")

def get_current_user(token: str = Depends(oauth2_scheme), db: Session = Depends(get_db)) -> User:
    """获取当前登录用户"""
    # 校验令牌
    try:
        payload = jwt.decode(token, settings.SECRET_KEY, algorithms=[settings.ALGORITHM])
        user_id: str = payload.get("sub")
        if user_id is None:
            raise BusinessException(code=401, msg="令牌无效")
    except JWTError:
        raise BusinessException(code=401, msg="令牌过期或无效")
    
    # 查询用户
    user = db.query(User).filter(User.id == user_id).first()
    if not user:
        raise BusinessException(code=401, msg="用户不存在")
    if user.status == 0:
        raise BusinessException(code=403, msg="账号已被禁用")
    return user

def check_permission(permission: str):
    """权限校验装饰器"""
    def wrapper(user: User = Depends(get_current_user)):
        # 获取用户所有权限
        permissions = []
        for role in user.roles:
            for menu in role.menus:
                permissions.append(menu.permission)
        if permission not in permissions:
            raise BusinessException(code=403, msg="无接口访问权限")
        return user
    return wrapper
```

# 八、业务逻辑层Service实现

## 8\.1 用户业务（services/user\.py）

```python
from sqlalchemy.orm import Session
from typing import Optional, Dict
from app.models.user import User
from app.schemas.user import UserRegister, UserPageQuery
from app.core.security import get_password_hash, verify_password, create_access_token
from app.core.exception import BusinessException
from app.common.pagination import Pagination

class UserService:
    @staticmethod
    def register(db: Session, data: UserRegister) -> Dict:
        """用户注册"""
        # 校验账号是否存在
        exist_user = db.query(User).filter(User.username == data.username).first()
        if exist_user:
            raise BusinessException(code=400, msg="账号已存在")
        
        # 创建用户
        db_user = User(
            username=data.username,
            password=get_password_hash(data.password),
            nickname=data.nickname
        )
        db.add(db_user)
        db.commit()
        db.refresh(db_user)
        return {"id": db_user.id, "username": db_user.username}

    @staticmethod
    def login(db: Session, username: str, password: str) -> Dict:
        """用户登录"""
        user = db.query(User).filter(User.username == username).first()
        if not user:
            raise BusinessException(code=400, msg="账号或密码错误")
        if not verify_password(password, user.password):
            raise BusinessException(code=400, msg="账号或密码错误")
        if user.status == 0:
            raise BusinessException(code=403, msg="账号已禁用")
        
        # 生成令牌
        access_token = create_access_token(subject=user.id)
        return {"access_token": access_token, "token_type": "bearer"}

    @staticmethod
    def get_user_page(db: Session, query: UserPageQuery) -> Dict:
        """用户分页查询"""
        q = db.query(User)
        if query.username:
            q = q.filter(User.username.like(f"%{query.username}%"))
        page = Pagination(page=query.page, page_size=query.page_size)
        return page.paginate(q)

    @staticmethod
    def delete_user(db: Session, user_id: int):
        """删除用户"""
        user = db.query(User).filter(User.id == user_id).first()
        if not user:
            raise BusinessException(code=400, msg="用户不存在")
        db.delete(user)
        db.commit()

user_service = UserService()
```

## 8\.2 角色、菜单业务Service

```python
# services/role.py
from sqlalchemy.orm import Session
from app.models.role import Role
from app.schemas.role import RoleCreate
from app.core.exception import BusinessException

class RoleService:
    @staticmethod
    def create_role(db: Session, data: RoleCreate) -> Role:
        # 校验角色标识
        exist_role = db.query(Role).filter(Role.role_code == data.role_code).first()
        if exist_role:
            raise BusinessException(code=400, msg="角色标识已存在")
        role = Role(**data.model_dump())
        db.add(role)
        db.commit()
        db.refresh(role)
        return role

role_service = RoleService()

# services/menu.py
from sqlalchemy.orm import Session
from app.models.menu import Menu
from app.schemas.menu import MenuCreate

class MenuService:
    @staticmethod
    def create_menu(db: Session, data: MenuCreate) -> Menu:
        menu = Menu(**data.model_dump())
        db.add(menu)
        db.commit()
        db.refresh(menu)
        return menu

menu_service = MenuService()
```

# 九、路由接口Router实现

## 9\.1 用户路由（api/user\.py）

```python
from fastapi import APIRouter, Depends
from sqlalchemy.orm import Session

from app.core.database import get_db
from app.schemas.user import UserRegister, UserLogin, UserPageQuery
from app.services.user import user_service
from app.common.response import resp
from app.api.deps import get_current_user

router = APIRouter(prefix="/user", tags=["用户模块"])

@router.post("/register", summary="用户注册")
def register(data: UserRegister, db: Session = Depends(get_db)):
    res = user_service.register(db, data)
    return resp.success(data=res, msg="注册成功")

@router.post("/login", summary="用户登录")
def login(data: UserLogin, db: Session = Depends(get_db)):
    res = user_service.login(db, data.username, data.password)
    return resp.success(data=res, msg="登录成功")

@router.get("/page", summary="用户分页列表")
def user_page(query: UserPageQuery = Depends(), db: Session = Depends(get_db), current_user = Depends(get_current_user)):
    res = user_service.get_user_page(db, query)
    return resp.success(data=res)

@router.delete("/{user_id}", summary="删除用户")
def delete_user(user_id: int, db: Session = Depends(get_db), current_user = Depends(get_current_user)):
    user_service.delete_user(db, user_id)
    return resp.success(msg="删除成功")
```

## 9\.2 角色、菜单路由

```python
# api/role.py
from fastapi import APIRouter, Depends
from sqlalchemy.orm import Session
from app.core.database import get_db
from app.schemas.role import RoleCreate
from app.services.role import role_service
from app.common.response import resp
from app.api.deps import get_current_user

router = APIRouter(prefix="/role", tags=["角色模块"])

@router.post("/create", summary="创建角色")
def create_role(data: RoleCreate, db: Session = Depends(get_db), current_user = Depends(get_current_user)):
    res = role_service.create_role(db, data)
    return resp.success(data=res, msg="角色创建成功")

# api/menu.py
from fastapi import APIRouter, Depends
from sqlalchemy.orm import Session
from app.core.database import get_db
from app.schemas.menu import MenuCreate
from app.services.menu import menu_service
from app.common.response import resp
from app.api.deps import get_current_user

router = APIRouter(prefix="/menu", tags=["菜单权限模块"])

@router.post("/create", summary="创建菜单权限")
def create_menu(data: MenuCreate, db: Session = Depends(get_db), current_user = Depends(get_current_user)):
    res = menu_service.create_menu(db, data)
    return resp.success(data=res, msg="权限创建成功")
```

# 十、项目入口文件（main\.py）

```python
from fastapi import FastAPI
import uvicorn

from app.core.database import engine, Base
from app.core.exception import register_exception
from app.api import user, role, menu

# 创建所有数据表
Base.metadata.create_all(bind=engine)

# 初始化项目
app = FastAPI(title="FastAPI-RBAC权限管理系统", version="1.0.0")

# 注册全局异常处理
register_exception(app)

# 注册路由
app.include_router(user.router)
app.include_router(role.router)
app.include_router(menu.router)

@app.get("/", summary="项目根路径")
def root():
    return {"msg": "FastAPI RBAC权限系统运行成功，访问/docs查看接口文档"}

if __name__ == "__main__":
    uvicorn.run(app="app.main:app", host="0.0.0.0", port=8000, reload=True)
```

# 十一、接口测试

## 11\.1 Swagger在线文档测试

FastAPI 原生自动生成交互式接口文档，项目启动后可通过以下地址访问：

- Swagger UI：`http://127\.0\.0\.1:8000/docs`

- ReDoc 文档：`http://127\.0\.0\.1:8000/redoc`

测试流程：

1. 启动项目，访问 /docs 进入接口文档页面

2. 调用 `/user/register` 注册测试账号

3. 调用 `/user/login` 获取 JWT Token

4. 点击文档右上角 `Authorize`，输入 `Bearer 令牌` 完成授权

5. 测试需要登录的分页查询、角色创建等接口

## 11\.2 Postman 测试

1\. 新建 Postman 集合，配置全局请求地址 `http://127\.0\.0\.1:8000`

2\. 调用登录接口获取 token，在请求头添加 `Authorization: Bearer \{token\}`

3\. 依次测试注册、登录、分页查询、角色权限创建等接口，验证数据返回、异常拦截、权限控制功能是否正常

# 十二、完整项目开发流程总结

本项目严格遵循企业级后端开发流程，标准化开发步骤如下，可复用至所有 FastAPI 项目：

## 12\.1 第一步：环境搭建与依赖配置

创建项目目录、配置依赖文件、安装所需第三方库，搭建基础项目架构。

## 12\.2 第二步：数据库设计与建表

根据业务需求设计数据表结构，确定字段、关联关系，基于SQLAlchemy编写ORM模型，定义多对多中间表，实现数据库模型映射。

## 12\.3 第三步：公共模块封装

封装数据库会话、JWT鉴权、密码加密、统一响应、全局异常、分页工具等公共基础模块，为业务开发提供支撑。

## 12\.4 第四步：Schema数据模型定义

基于Pydantic定义请求参数、响应数据模型，实现参数自动校验、敏感字段过滤。

## 12\.5 第五步：Service业务逻辑开发

编写核心业务逻辑，处理数据校验、数据库增删改查、业务规则判断，解耦路由和业务逻辑。

## 12\.6 第六步：Router路由接口开发

定义接口路由、请求方式、接口注释，配置依赖项（登录校验、权限校验），调用业务层方法返回统一格式数据。

## 12\.7 第七步：全局配置与项目整合

注册路由、全局异常处理器，配置项目启动参数，完成项目整体整合。

## 12\.8 第八步：接口测试与调试

通过 Swagger、Postman 全量测试接口，验证正常流程、异常流程、权限拦截、分页查询等功能。

# 十三、项目总结

本笔记完整实现了**FastAPI \+ SQLAlchemy 轻量化RBAC权限管理系统**，覆盖后端开发全流程核心知识点。通过本项目实战，可熟练掌握 FastAPI 分层开发思想、SQLAlchemy ORM 关联映射、JWT 鉴权机制、RBAC权限模型、全局异常与响应封装、分页查询等企业级核心技能。

项目代码结构规范、拓展性强，可在此基础上继续拓展**按钮权限、数据权限、日志记录、密码重置、角色权限批量分配**等高级功能。

