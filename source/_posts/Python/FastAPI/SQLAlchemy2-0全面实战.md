---
title: SQLAlchemy2.0全面实战
categories:
  - Python
  - FastAPI
date: 2026-05-28 21:54:21
tags:
---

# FastAPI学习笔记（三）：SQLAlchemy 2\.0 全面实战

## 本篇适用场景与核心定位

前面两篇我们搞定了 FastAPI 基础接口开发、异步底层原理、高并发与阻塞问题。但任何后端项目，最终核心都是**数据库读写、数据建模、事务一致性、查询性能**。

本篇以**企业通用后台：用户\-角色\-权限\-部门**真实业务场景切入，完整落地 SQLAlchemy 2\.0 全套知识点，覆盖**同步/异步双引擎、模型设计、会话管理、全套CRUD、高级查询、联表关系、懒加载优化、事务、分页、原生SQL、Alembic迁移、企业级封装**。

本篇为 FastAPI 项目数据层终极指南，学完可直接搭建可上线、可迭代、高性能、规范标准的生产级数据库架构，彻底告别手写SQL、模型混乱、事务失控、N\+1性能坑等常见问题。

**技术栈版本：SQLAlchemy 2\.0 \+ Python3\.10\+ \+ aiomysql/asyncmy \+ Alembic**

# 一、SQLAlchemy 2\.0 核心五大组件（必懂基石）

SQLAlchemy 2\.0 相比1\.x版本做了大量语法重构、异步标准化、类型注解增强，是目前Python生态最标准、最稳定、企业首选的ORM框架。其核心由五大核心组件构成，所有数据库操作都基于这五个组件实现。

### 1\.1 Engine（引擎）

Engine 是数据库连接的核心管理器，是整个ORM的入口，负责维护**连接池、连接创建、销毁、超时、重试**等底层逻辑。无论是同步还是异步操作，都必须先初始化Engine。Engine不直接执行SQL，只负责管理连接资源。

### 1\.2 Session（会话，重中之重）

Session 是ORM操作的**工作单元**，所有增删改查、事务、缓存、数据状态追踪都由Session完成。可以理解为：**一次数据库会话 = 一次事务生命周期**。Session具备对象状态管理能力，会自动追踪模型对象的新增、修改、删除状态，无需手动拼接SQL。

### 1\.3 DeclarativeBase（声明式基类）

2\.0版本全新标准基类，所有自定义ORM模型必须继承自 `DeclarativeBase`，替代1\.x的 `declarative\_base\(\)`。基类统一管理所有模型元数据、表结构映射，支持统一封装通用字段、通用方法，是企业级模型统一规范的基础。

### 1\.4 Mapped（类型注解）

SQLAlchemy2\.0 强类型核心，用于给模型字段添加静态类型注解，适配Python类型检查、IDE智能提示，彻底解决旧版本无类型、代码晦涩、报错难排查的问题。所有模型字段必须用 Mapped 包裹类型。

### 1\.5 mapped\_column（字段定义）

替代旧版本的 Column，是2\.0标准字段定义API，支持类型绑定、约束、默认值、索引、注释、nullable配置，语法更简洁、语义更清晰，完美适配 Mapped 类型注解。

# 二、数据库连接：同步/异步双引擎实战

FastAPI 异步项目**禁止使用同步引擎**，否则会阻塞事件循环，导致接口并发暴跌。本节完整讲解同步引擎、异步引擎配置，以及主流异步驱动 aiomysql、asyncmy 的使用区别。

依赖安装：`pip install sqlalchemy\[asyncio\] aiomysql asyncmy pymysql`

### 2\.1 同步 Engine（适用于脚本、定时任务、同步项目）

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import DeclarativeBase, sessionmaker

# 同步数据库连接URL
SYNC_DATABASE_URL = "mysql+pymysql://root:123456@localhost:3306/fastapi_db?charset=utf8mb4"

# 创建同步引擎
sync_engine = create_engine(
    SYNC_DATABASE_URL,
    pool_pre_ping=True,  # 每次使用前检测连接有效性，防止断连报错
    pool_recycle=3600,   # 1小时回收空闲连接
    pool_size=10,        # 连接池大小
    max_overflow=20      # 最大溢出连接数
)

# 同步会话工厂
SyncSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=sync_engine)

# 模型基类
class Base(DeclarativeBase):
    pass
```

### 2\.2 异步 Engine（FastAPI 生产首选）

异步引擎依托异步驱动，不阻塞事件循环，完美适配FastAPI异步接口，是项目标准配置。主流异步驱动有两种：

- **aiomysql**：老牌稳定、生态最全、兼容性最好，企业主流选择

- **asyncmy**：性能更高、轻量化，新项目优选

#### 2\.2\.1 aiomysql 异步引擎配置

```python
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker

ASYNC_DATABASE_URL = "mysql+aiomysql://root:123456@localhost:3306/fastapi_db?charset=utf8mb4"

# 创建异步引擎
async_engine = create_async_engine(
    ASYNC_DATABASE_URL,
    pool_pre_ping=True,
    pool_recycle=3600,
    pool_size=10,
    max_overflow=20,
    echo=False  # 生产关闭SQL日志，开发可开启True
)

# 异步会话工厂
AsyncSessionLocal = async_sessionmaker(autocommit=False, autoflush=False, bind=async_engine)
```

#### 2\.2\.2 asyncmy 高性能异步引擎

```python
# 连接URL仅驱动名不同，代码完全通用
ASYNC_DATABASE_URL = "mysql+asyncmy://root:123456@localhost:3306/fastapi_db?charset=utf8mb4"
```

# 三、ORM模型全套设计（企业级规范）

以用户表为核心，完整实现**主键、索引、唯一约束、默认值、时间字段、JSON字段、枚举字段**所有企业常用模型配置，所有字段遵循生产规范。

```python
from datetime import datetime
from enum import Enum as PyEnum
from sqlalchemy import Integer, String, DateTime, Boolean, JSON, Enum, UniqueConstraint, Index
from sqlalchemy.orm import Mapped, mapped_column

# 自定义枚举类
class UserStatus(PyEnum):
    NORMAL = 1
    DISABLE = 2
    LOCKED = 3

# 用户模型
class User(Base):
    __tablename__ = "sys_user"

    # 自增主键
    id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True, comment="主键ID")
    # 用户名：唯一索引、非空
    username: Mapped[str] = mapped_column(String(32), nullable=False, comment="用户名")
    # 密码
    password: Mapped[str] = mapped_column(String(128), nullable=False, comment="密码")
    # 昵称
    nickname: Mapped[str | None] = mapped_column(String(64), nullable=True, default="", comment="用户昵称")
    # 状态枚举
    status: Mapped[UserStatus] = mapped_column(Enum(UserStatus), default=UserStatus.NORMAL, comment="用户状态")
    # 手机号唯一约束
    phone: Mapped[str | None] = mapped_column(String(11), nullable=True, comment="手机号")
    # JSON字段：存储用户扩展信息
    extra_info: Mapped[dict | None] = mapped_column(JSON, nullable=True, default=dict, comment="扩展信息")
    # 布尔字段
    is_admin: Mapped[bool] = mapped_column(Boolean, default=False, comment="是否超级管理员")
    # 创建时间、更新时间
    create_time: Mapped[datetime] = mapped_column(DateTime, default=datetime.now, comment="创建时间")
    update_time: Mapped[datetime] = mapped_column(DateTime, default=datetime.now, onupdate=datetime.now, comment="更新时间")

    # 联合唯一约束：用户名唯一
    __table_args__ = (
        UniqueConstraint("username", name="uk_username"),
        UniqueConstraint("phone", name="uk_phone"),
        Index("idx_status_create", "status", "create_time"),  # 联合索引
    )
```

**核心知识点总结**：主键自增、唯一约束防重复、联合索引优化查询、枚举规范状态值、JSON存储动态扩展数据、时间字段自动赋值更新，完全贴合企业数据表设计标准。

# 四、Session会话全生命周期（重点难点）

90%的数据库报错、事务异常、数据脏读、连接泄露，都是因为不了解Session生命周期和核心方法。Session是单次请求的数据库操作容器，请求开始创建，请求结束销毁。

### 4\.1 Session完整生命周期

创建会话 → 执行CRUD → flush预提交 → commit事务提交 / rollback回滚 → refresh刷新数据 → close关闭释放连接。

### 4\.2 核心方法详解

- **add\(\)**：将模型对象加入Session缓存，未真正写入数据库

- **add\_all\(\)**：批量添加多个模型对象

- **flush\(\)**：预提交，将SQL发送到数据库，生成主键、触发约束，**未真正提交事务**，其他会话不可见

- **commit\(\)**：正式提交事务，数据持久化，其他会话可查询到数据

- **rollback\(\)**：事务回滚，撤销本次会话所有未提交操作

- **refresh\(\)**：重新从数据库查询最新数据，覆盖本地缓存对象

- **close\(\)**：关闭会话，归还连接至连接池，防止连接泄露

### 4\.3 标准异步Session使用模板

```python
async def demo_session():
    async with AsyncSessionLocal() as session:
        try:
            # 1. 新增对象到缓存
            user = User(username="test01", password="123456")
            session.add(user)
            # 2. 预提交，获取自增ID
            await session.flush()
            print("新增用户ID：", user.id)
            # 3. 正式提交事务
            await session.commit()
        except Exception as e:
            # 异常回滚
            await session.rollback()
            raise e
        finally:
            # 关闭会话，释放连接
            await session.close()
```

# 五、CRUD 全套完整实战

基于异步Session，完整实现 add、add\_all、select、update、delete 标准CRUD操作，全部为生产可直接复用代码。

### 5\.1 新增操作（单条/批量）

```python
# 单条新增
async def create_user():
    async with AsyncSessionLocal() as session:
        user = User(username="user01", password="123456", nickname="测试用户")
        session.add(user)
        await session.commit()
        await session.refresh(user)
        return user

# 批量新增
async def batch_create_user():
    async with AsyncSessionLocal() as session:
        user_list = [
            User(username="user02", password="123456"),
            User(username="user03", password="123456")
        ]
        session.add_all(user_list)
        await session.commit()
        return True
```

### 5\.2 查询操作

```python
from sqlalchemy import select

async def get_user_by_id(user_id: int):
    async with AsyncSessionLocal() as session:
        stmt = select(User).where(User.id == user_id)
        result = await session.execute(stmt)
        return result.scalar_one_or_none()
```

### 5\.3 更新操作

```python
from sqlalchemy import update

async def update_user(user_id: int, nickname: str):
    async with AsyncSessionLocal() as session:
        stmt = update(User).where(User.id == user_id).values(nickname=nickname)
        await session.execute(stmt)
        await session.commit()
        return True
```

### 5\.4 删除操作

```python
from sqlalchemy import delete

async def delete_user(user_id: int):
    async with AsyncSessionLocal() as session:
        stmt = delete(User).where(User.id == user_id)
        await session.execute(stmt)
        await session.commit()
        return True
```

# 六、核心查询语法大全（高频重点）

汇总企业开发100%用到的所有查询条件，包含精准匹配、模糊查询、多条件组合、范围、排序、分页基础，全覆盖无遗漏。

```python
from sqlalchemy import select, and_, or_

async def query_demo():
    async with AsyncSessionLocal() as session:
        # 1. where 精准查询
        stmt1 = select(User).where(User.username == "user01")

        # 2. filter 灵活查询
        stmt2 = select(User).filter(User.status == UserStatus.NORMAL)

        # 3. and_ 多条件且
        stmt3 = select(User).where(and_(User.is_admin == False, User.status == UserStatus.NORMAL))

        # 4. or_ 多条件或
        stmt4 = select(User).where(or_(User.id == 1, User.username == "user02"))

        # 5. like 模糊查询
        stmt5 = select(User).where(User.nickname.like("%测试%"))

        # 6. in_ 包含查询
        stmt6 = select(User).where(User.id.in_([1,2,3]))

        # 7. exists 存在查询
        from sqlalchemy import exists
        stmt7 = select(exists().where(User.username == "user01"))

        # 8. between 范围查询
        stmt8 = select(User).where(User.id.between(1, 100))

        # 9. order_by 排序
        stmt9 = select(User).order_by(User.create_time.desc())

        # 10. limit / offset 限制条数、偏移分页
        stmt10 = select(User).limit(10).offset(0)

        res = await session.execute(stmt10)
        return res.scalars().all()
```

# 七、高级查询（联表、子查询、聚合、分组）

### 7\.1 join / outerjoin 联表查询

inner join 内连接、outerjoin 左外连接，是多表关联查询核心。

### 7\.2 子查询

通过 subquery\(\) 实现嵌套查询，适配复杂业务筛选场景。

### 7\.3 聚合与分组

```python
from sqlalchemy import func, count

async def group_agg_query():
    async with AsyncSessionLocal() as session:
        # 按状态分组统计用户数量
        stmt = select(User.status, count(User.id)).group_by(User.status).having(count(User.id) > 0)
        res = await session.execute(stmt)
        return res.all()
```

### 7\.4 union 联合查询

合并多个查询结果，自动去重，union\_all 保留重复数据。

# 八、Relationship 关联关系（核心重难点）

ORM最强特性，彻底告别手写联表SQL，自动关联查询一对一、一对多、多对多数据，配合 back\_populates 实现双向关联。

### 8\.1 一对多关系（用户\-角色）

```python
from sqlalchemy.orm import relationship

# 角色模型
class Role(Base):
    __tablename__ = "sys_role"
    id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
    name: Mapped[str] = mapped_column(String(32), nullable=False)
    # 一对多反向关联：一个角色对应多个用户
    users: Mapped[list["User"]] = relationship(back_populates="role")

# 用户模型补充关联字段
class User(Base):
    # ... 省略原有字段
    role_id: Mapped[int | None] = mapped_column(Integer, nullable=True)
    # 多对一关联
    role: Mapped["Role"] = relationship(back_populates="users")
```

### 8\.2 一对一、多对多实现规范

一对一通过 `uselist=False` 实现，多对多需要中间关联表，搭配 secondary 参数实现自动关联，back\_populates 严格双向绑定，避免数据查询异常、重复数据。

# 九、懒加载、预加载与性能优化（解决N\+1致命问题）

### 9\.1 N\+1问题根源

默认懒加载机制：查询N条主数据后，遍历每条数据都会单独查询一次关联数据，产生1次主查询\+N次关联查询，数据库压力巨大。

### 9\.2 解决方案：预加载

- **joinedload**：联表左连接查询，适合一对一、多对一

- **selectinload**：批量IN查询，适合一对多、多对多，性能最优

```python
from sqlalchemy.orm import selectinload, joinedload

async def query_with_relation():
    async with AsyncSessionLocal() as session:
        # 一次性查询用户+关联角色，解决N+1
        stmt = select(User).options(selectinload(User.role))
        res = await session.execute(stmt)
        return res.scalars().all()
```

# 十、事务深度处理

### 10\.1 事务原理

SQLAlchemy 事务默认手动提交，单Session单次事务，要么全部成功commit，要么全部失败rollback，保证数据一致性。

### 10\.2 自动回滚与嵌套事务

```python
async def transaction_demo():
    async with AsyncSessionLocal() as session:
        async with session.begin():
            # 事务块内所有操作，异常自动回滚
            user1 = User(username="tx01", password="123456")
            user2 = User(username="tx02", password="123456")
            session.add_all([user1, user2])
            # 触发异常自动回滚
            1 / 0
```

# 十一、通用分页封装（企业级）

封装通用分页工具类，支持任意模型、任意条件分页查询，返回总条数、总页数、当前页数据，项目全局复用。

# 十二、原生SQL执行

复杂统计、特殊SQL场景可使用原生SQL，通过 text 封装SQL语句，execute 执行，安全防注入。

```python
from sqlalchemy import text

async def raw_sql_demo():
    async with AsyncSessionLocal() as session:
        sql = text("select * from sys_user where id = :user_id")
        res = await session.execute(sql, {"user_id": 1})
        return res.mappings().all()
```

# 十三、Alembic 数据迁移（生产必备）

Alembic 是SQLAlchemy官方迁移工具，实现**版本化数据库管理**，支持模型变更自动生成迁移脚本、升级、回滚，彻底解决手动改表结构导致的环境不一致问题。

### 13\.1 初始化迁移环境

`alembic init alembic`

### 13\.2 生成迁移脚本

`alembic revision \-\-autogenerate \-m \&\#34;init table\&\#34;`

### 13\.3 执行升级

`alembic upgrade head`

### 13\.4 版本回滚

`alembic downgrade \-1`

# 十四、SQLAlchemy 企业最佳实践封装

### 14\.1 通用基类封装

封装全局通用字段（创建时间、更新时间、创建人、更新人、删除标记），所有模型继承复用。

### 14\.2 通用CRUD父类封装

封装通用增删改查、分页、批量操作方法，所有业务CRUD直接继承，无需重复编写基础代码。

### 14\.3 FastAPI Session 依赖注入

通过Depends全局注入数据库会话，自动管理会话生命周期，请求结束自动关闭连接，杜绝连接泄露。

```python
from fastapi import Depends

async def get_db():
    async with AsyncSessionLocal() as session:
        try:
            yield session
        finally:
            await session.close()
```

## 本篇总结

本篇完整覆盖 SQLAlchemy2\.0 企业级全栈能力，从底层引擎、模型设计、会话原理、全套CRUD、高级查询、关联关系、性能优化、事务、分页、原生SQL、版本迁移到最终工程封装，形成一套可直接落地的生产级数据层架构，彻底解决FastAPI项目数据库开发不规范、性能差、事务乱、迭代难的问题。
