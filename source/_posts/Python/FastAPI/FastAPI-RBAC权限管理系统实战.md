---
title: FastAPI RBAC权限管理系统实战
tags:
  - FastAPI
  - RBAC
  - SQLAlchemy
  - Vue
categories:
  - Python
  - FastAPI
date: 2026-06-10 22:00:00
---

# FastAPI RBAC 权限管理系统实战

本文记录一个基于 FastAPI 的 RBAC 用户权限管理系统从需求分析、架构设计、数据库建模、接口实现到前后端联调的完整开发过程。项目定位不是生产级后台模板，而是一个具备完整业务闭环的学习型工程：通过尽量小的系统边界覆盖 Web 后端开发中常见的核心主题，包括分层架构、ORM 关系建模、JWT 登录认证、接口级权限控制、统一响应、前端登录态管理和权限驱动的页面控制。

项目路径如下：

```text
D:\pycharm_workspace\python_code\FastAPI\fastapi-rbac-admin
```

技术栈如下：

- 后端：FastAPI、SQLAlchemy 2.x、Pydantic、JWT、SQLite
- 前端：Vue3、Vite、Element Plus、Pinia、Axios
- 权限模型：RBAC，即 User、Role、Permission 三层模型
- 文档：README、设计文档、Swagger 自动接口文档
- 初始化数据：admin、editor、viewer 三类演示账号

选择 RBAC 作为实战主题的原因在于，它比普通 CRUD 更能体现后端系统设计的真实问题。一个权限系统必须处理用户身份、角色关系、权限粒度、接口拦截、前端菜单控制等多个环节，能够较好地串联 FastAPI 的依赖注入、SQLAlchemy 的多对多建模以及前后端协作方式。

## 一、需求分析

### 1.1 项目目标

本项目的目标是实现一个可本机运行、可通过浏览器操作、具备接口级权限校验能力的用户权限管理系统。系统面向学习和演示场景，因此优先考虑代码结构清晰、运行门槛低、核心流程完整，而不是引入过多生产级基础设施。

从使用者角度看，系统应支持以下能力：

1. 用户能够使用账号和密码登录系统。
2. 登录成功后，前端能够获取当前用户信息、角色和权限列表。
3. 管理员能够维护用户、角色和权限数据。
4. 用户可以绑定多个角色。
5. 角色可以绑定多个权限。
6. 不同角色登录后看到的菜单和操作按钮不同。
7. 后端接口必须执行真正的权限校验，不能只依赖前端隐藏按钮。

从学习角度看，系统应覆盖以下知识点：

1. FastAPI 路由组织和接口文档生成。
2. FastAPI 依赖注入在鉴权中的应用。
3. Pydantic 请求模型和响应模型设计。
4. SQLAlchemy 2.x 声明式模型和多对多关系。
5. JWT 的签发、解析和过期校验。
6. 密码哈希存储，而不是保存明文密码。
7. 前端 Axios 请求拦截器和登录态管理。
8. 前端基于权限标识控制菜单和按钮显示。

### 1.2 功能需求

系统按照后台管理场景拆分为四个功能模块。

第一是登录认证模块。该模块负责账号密码登录、JWT Token 签发、当前登录用户查询。登录接口不需要权限，但当前用户接口需要携带有效 Token。

第二是用户管理模块。该模块负责用户列表查询、用户创建、用户更新、用户删除、用户角色分配。用户数据包括用户名、昵称、密码哈希、启用状态、创建时间等信息。

第三是角色管理模块。该模块负责角色列表查询、角色创建、角色更新、角色删除、角色权限分配。角色是用户和权限之间的中间层，用于降低权限直接分配给用户带来的维护成本。

第四是权限管理模块。该模块负责权限列表查询、权限创建、权限更新、权限删除。权限以字符串标识的方式保存，例如 `user:read`、`role:update`、`permission:delete`，便于在接口层进行声明式拦截。

### 1.3 非功能需求

系统虽然是学习项目，但仍需要满足若干基本非功能要求。

可运行性方面，项目应能在本机快速启动。数据库选择 SQLite，避免学习阶段被 MySQL 或 PostgreSQL 的安装、账号、端口、驱动等问题分散注意力。

可读性方面，后端按 api、schemas、models、services、core、common 分层，前端按 api、router、stores、views 分层。每一层只负责自己的职责，避免把数据库查询、鉴权逻辑和 HTTP 响应全部写在路由函数中。

可扩展性方面，权限标识使用统一格式，接口通过 `require_permission("user:read")` 声明需要的权限。以后新增模块时，只需要增加权限数据并在接口上声明即可。

安全性方面，密码必须哈希存储；接口必须校验 JWT；敏感接口必须校验权限；前端按钮隐藏不作为安全边界。

## 二、总体设计

### 2.1 RBAC 模型设计

RBAC 是 Role-Based Access Control，即基于角色的访问控制。它的核心思想是：用户不直接拥有权限，而是通过角色间接获得权限。

```text
User  <->  Role  <->  Permission
```

在该模型中：

- 一个用户可以拥有多个角色。
- 一个角色可以分配给多个用户。
- 一个角色可以拥有多个权限。
- 一个权限可以分配给多个角色。

相比直接给用户分配权限，RBAC 的优势是维护成本更低。例如系统中有 20 个运营人员都需要相同权限，如果直接给每个用户分配权限，一旦权限变化就需要修改 20 次；如果通过“运营编辑”角色管理，只需要修改该角色一次。

本项目内置三个角色：

| 角色编码 | 角色名称 | 权限范围 |
| --- | --- | --- |
| `admin` | 系统管理员 | 拥有全部权限 |
| `editor` | 运营编辑 | 拥有查询权限和部分维护权限 |
| `viewer` | 只读访客 | 仅拥有查询权限 |

权限标识采用 `资源:动作` 的格式：

| 权限标识 | 含义 |
| --- | --- |
| `user:read` | 查看用户 |
| `user:create` | 创建用户 |
| `user:update` | 更新用户 |
| `user:delete` | 删除用户 |
| `role:read` | 查看角色 |
| `permission:read` | 查看权限 |

这种格式简单、直观，并且适合直接放在接口依赖中。

### 2.2 后端分层设计

后端目录结构如下：

```text
app/
├── api/          # 路由层：HTTP 接口、接口级权限依赖
├── common/       # 公共结构：统一响应
├── core/         # 核心能力：配置、数据库、JWT、密码哈希
├── models/       # ORM 模型：User、Role、Permission
├── schemas/      # Pydantic 模型：请求与响应结构
└── services/     # 业务层：数据库查询、对象转换、业务规则
```

路由层只处理 HTTP 语义，包括路径、方法、依赖和返回结果。业务层负责具体查询、创建、更新、删除和对象转换。模型层只描述数据库结构和表关系。这样的拆分能够避免路由函数膨胀，也便于后续补充测试。

### 2.3 前端分层设计

前端目录结构如下：

```text
frontend/src/
├── api/          # Axios 实例和接口封装
├── router/       # Vue Router 路由与登录守卫
├── stores/       # Pinia 保存 token、用户和权限
└── views/        # 页面：登录、控制台、用户、角色、权限
```

前端不保存复杂业务规则，只负责展示、交互和调用接口。登录成功后，Pinia 保存当前用户和权限列表；路由守卫负责检查是否登录；页面按钮通过 `auth.has("permission-code")` 判断是否展示。

需要特别说明的是，前端权限判断只用于改善体验，不用于保证安全。真正的安全边界在后端接口。

## 三、数据库设计

### 3.1 数据表说明

系统包含五张核心表：

| 表名 | 说明 |
| --- | --- |
| `users` | 用户表 |
| `roles` | 角色表 |
| `permissions` | 权限表 |
| `user_roles` | 用户角色关联表 |
| `role_permissions` | 角色权限关联表 |

`users` 表保存用户身份信息。密码字段保存哈希值，而不是明文密码。`is_active` 用于控制账号是否启用。

`roles` 表保存角色编码、角色名称和描述。角色编码用于程序识别，角色名称用于页面展示。

`permissions` 表保存权限标识、权限名称、分组和描述。权限标识是接口鉴权的核心字段。

`user_roles` 和 `role_permissions` 是典型多对多关联表。它们不需要额外业务字段，因此可以直接用 SQLAlchemy 的 `Table` 定义。

### 3.2 ORM 关键代码

用户和角色之间、角色和权限之间都是多对多关系，SQLAlchemy 建模如下：

```python
user_roles = Table(
    "user_roles",
    Base.metadata,
    Column("user_id", Integer, ForeignKey("users.id", ondelete="CASCADE"), primary_key=True),
    Column("role_id", Integer, ForeignKey("roles.id", ondelete="CASCADE"), primary_key=True),
)

role_permissions = Table(
    "role_permissions",
    Base.metadata,
    Column("role_id", Integer, ForeignKey("roles.id", ondelete="CASCADE"), primary_key=True),
    Column("permission_id", Integer, ForeignKey("permissions.id", ondelete="CASCADE"), primary_key=True),
)
```

用户模型定义了 `roles` 关系：

```python
class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True, index=True)
    username: Mapped[str] = mapped_column(String(50), unique=True, index=True)
    nickname: Mapped[str] = mapped_column(String(80))
    hashed_password: Mapped[str] = mapped_column(String(255))
    is_active: Mapped[bool] = mapped_column(Boolean, default=True)

    roles: Mapped[list["Role"]] = relationship(
        secondary=user_roles,
        back_populates="users",
    )
```

角色模型定义了 `users` 和 `permissions` 两个关系：

```python
class Role(Base):
    __tablename__ = "roles"

    id: Mapped[int] = mapped_column(primary_key=True, index=True)
    name: Mapped[str] = mapped_column(String(50), unique=True, index=True)
    label: Mapped[str] = mapped_column(String(80))
    description: Mapped[str] = mapped_column(String(255), default="")

    users: Mapped[list[User]] = relationship(
        secondary=user_roles,
        back_populates="roles",
    )
    permissions: Mapped[list["Permission"]] = relationship(
        secondary=role_permissions,
        back_populates="roles",
    )
```

权限模型再反向关联角色：

```python
class Permission(Base):
    __tablename__ = "permissions"

    id: Mapped[int] = mapped_column(primary_key=True, index=True)
    code: Mapped[str] = mapped_column(String(100), unique=True, index=True)
    name: Mapped[str] = mapped_column(String(80))
    group: Mapped[str] = mapped_column(String(50), default="default")
    description: Mapped[str] = mapped_column(String(255), default="")

    roles: Mapped[list[Role]] = relationship(
        secondary=role_permissions,
        back_populates="permissions",
    )
```

这里需要关注两个设计点。

第一，`username`、`role.name`、`permission.code` 都设置了唯一约束，避免出现重复账号、重复角色编码和重复权限标识。

第二，关联表外键设置 `ondelete="CASCADE"`。当用户、角色或权限被删除时，关联数据能够自动清理，避免产生无意义的孤立关系。

## 四、后端实现

### 4.1 配置与数据库会话

项目使用 `pydantic-settings` 管理配置。学习项目默认使用 SQLite，同时保留通过环境变量切换数据库的能力。

```python
class Settings(BaseSettings):
    app_name: str = "FastAPI RBAC Admin"
    api_prefix: str = "/api/v1"
    database_url: str = f"sqlite:///{BASE_DIR / 'rbac.db'}"
    secret_key: str = "change-this-secret-key-in-production"
    algorithm: str = "HS256"
    access_token_expire_minutes: int = 60 * 8
```

数据库会话通过 FastAPI 依赖提供：

```python
def get_db() -> Generator[Session, None, None]:
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

这种写法保证每个请求有独立的数据库 Session，并在请求结束后关闭连接。对于学习项目，这是理解 FastAPI 依赖生命周期的一个典型例子。

### 4.2 密码哈希与 JWT

密码不能明文保存。项目使用 `passlib` 对密码进行哈希：

```python
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password: str) -> str:
    return pwd_context.hash(password)
```

登录成功后，后端签发 JWT。Token 中的 `sub` 保存用户 ID，`exp` 保存过期时间。

```python
def create_access_token(subject: str) -> str:
    expire = datetime.now(timezone.utc) + timedelta(
        minutes=settings.access_token_expire_minutes
    )
    payload = {"sub": subject, "exp": expire}
    return jwt.encode(payload, settings.secret_key, algorithm=settings.algorithm)
```

解析 Token 时，如果签名错误或 Token 过期，则返回 `None`：

```python
def decode_access_token(token: str) -> str | None:
    try:
        payload = jwt.decode(
            token,
            settings.secret_key,
            algorithms=[settings.algorithm],
        )
        return payload.get("sub")
    except JWTError:
        return None
```

### 4.3 当前用户依赖

FastAPI 的依赖注入非常适合做认证和鉴权。项目使用 `HTTPBearer` 读取请求头中的 Bearer Token。

```python
bearer_scheme = HTTPBearer()

def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(bearer_scheme),
    db: Session = Depends(get_db),
) -> User:
    user_id = decode_access_token(credentials.credentials)
    if not user_id:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="登录已过期，请重新登录",
        )
    return get_user(db, int(user_id))
```

该依赖的价值在于，后续所有需要登录的接口不需要重复编写 Token 解析逻辑，只需要声明 `Depends(get_current_user)` 即可。

### 4.4 接口级权限依赖

接口级权限控制是本项目的核心。首先需要从用户的角色中汇总权限标识：

```python
def permission_codes(user: User) -> list[str]:
    codes = {
        permission.code
        for role in user.roles
        for permission in role.permissions
    }
    return sorted(codes)
```

然后定义一个权限检查依赖：

```python
def require_permission(code: str) -> Callable[[User], User]:
    def checker(current_user: User = Depends(get_current_user)) -> User:
        if code not in permission_codes(current_user):
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail=f"缺少权限：{code}",
            )
        return current_user

    return checker
```

这个函数返回另一个依赖函数，因此可以在接口上写出非常直观的声明：

```python
@router.get(
    "",
    response_model=ApiResponse,
    summary="查询用户列表",
    dependencies=[Depends(require_permission("user:read"))],
)
def list_users(db: Session = Depends(get_db)) -> ApiResponse:
    """用户列表接口：查询所有用户及其角色、权限，用于后台表格展示。"""
    return ok([rbac.user_to_read(user) for user in rbac.list_users(db)])
```

这种写法的优点是权限需求直接贴在接口定义上，阅读接口时就能知道访问该接口需要什么权限。

### 4.5 登录接口

登录接口负责校验用户名和密码，并返回访问令牌：

```python
@router.post("/login", response_model=ApiResponse, summary="账号密码登录")
def login(payload: LoginRequest, db: Session = Depends(get_db)) -> ApiResponse:
    """登录接口：校验用户名密码，成功后返回 JWT 令牌。"""
    user = authenticate_user(db, payload.username, payload.password)
    token = create_access_token(str(user.id))
    return ok(TokenResponse(access_token=token).model_dump(), "登录成功")
```

`authenticate_user` 的职责是查询用户、校验密码和判断账号是否启用：

```python
def authenticate_user(db: Session, username: str, password: str) -> User:
    user = get_user_by_username(db, username)
    if not user or not verify_password(password, user.hashed_password):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="用户名或密码错误",
        )
    if not user.is_active:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="用户已被禁用",
        )
    return user
```

这里把认证逻辑放在 service 层，而不是写在路由层，可以让路由函数保持简洁。

### 4.6 用户创建与角色分配

创建用户时，除了保存基础信息，还需要根据 `role_ids` 查询角色并建立关联：

```python
def create_user(db: Session, payload: UserCreate) -> User:
    if get_user_by_username(db, payload.username):
        raise HTTPException(
            status_code=status.HTTP_409_CONFLICT,
            detail="用户名已存在",
        )

    roles = (
        list(db.scalars(select(Role).where(Role.id.in_(payload.role_ids))).all())
        if payload.role_ids
        else []
    )
    user = User(
        username=payload.username,
        nickname=payload.nickname,
        hashed_password=get_password_hash(payload.password),
        is_active=payload.is_active,
        roles=roles,
    )
    db.add(user)
    db.commit()
    return get_user(db, user.id)
```

这里有三个关键点。

第一，创建用户前检查用户名是否重复。

第二，密码入库前必须哈希。

第三，角色关联不直接操作关联表，而是通过 SQLAlchemy relationship 赋值完成。这种方式更符合 ORM 的使用习惯。

### 4.7 对象转换与统一响应

项目定义了统一响应结构：

```python
class ApiResponse(BaseModel):
    code: int = 0
    message: str = "success"
    data: Any = None

def ok(data: Any = None, message: str = "success") -> ApiResponse:
    return ApiResponse(code=0, message=message, data=data)
```

实际开发中需要注意，SQLAlchemy 对象不能随意混入响应结构。为了保证响应稳定，项目在 service 层显式转换为普通字典：

```python
def role_to_read(role: Role) -> dict:
    return {
        "id": role.id,
        "name": role.name,
        "label": role.label,
        "description": role.description,
        "permissions": [
            permission_to_read(permission)
            for permission in role.permissions
        ],
        "created_at": role.created_at,
    }
```

这样做虽然比直接返回 ORM 对象多一些代码，但响应结构更可控，也更容易排查序列化问题。

## 五、前端实现

### 5.1 Axios 请求封装

前端通过 Axios 拦截器统一处理 Token：

```javascript
const http = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL || 'http://127.0.0.1:8000/api/v1',
  timeout: 10000
})

http.interceptors.request.use((config) => {
  const token = localStorage.getItem('rbac_token')
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})
```

响应拦截器处理 401 状态。如果 Token 失效，则清理本地 Token 并跳转登录页：

```javascript
http.interceptors.response.use(
  (response) => response.data,
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem('rbac_token')
      router.push('/login')
    }
    return Promise.reject(error)
  }
)
```

### 5.2 Pinia 登录态管理

Pinia store 保存 token、当前用户和权限列表：

```javascript
export const useAuthStore = defineStore('auth', {
  state: () => ({
    token: localStorage.getItem('rbac_token') || '',
    user: null,
    permissions: []
  }),
  actions: {
    async login(form) {
      const res = await loginApi(form)
      this.token = res.data.access_token
      localStorage.setItem('rbac_token', this.token)
      await this.loadProfile()
    },
    async loadProfile() {
      const res = await meApi()
      this.user = res.data
      this.permissions = res.data.permissions || []
    },
    has(code) {
      return this.permissions.includes(code)
    }
  }
})
```

`has` 方法是前端权限控制的统一入口。页面不需要关心权限列表如何存储，只需要调用 `auth.has("user:create")`。

### 5.3 路由守卫

路由守卫负责检查登录状态和页面访问权限：

```javascript
router.beforeEach(async (to) => {
  const auth = useAuthStore()
  if (to.path === '/login') return true
  if (!auth.token) return '/login'
  if (!auth.user) await auth.loadProfile()
  if (to.meta.permission && !auth.has(to.meta.permission)) return '/'
  return true
})
```

该逻辑保证未登录用户不能进入后台页面，同时也避免低权限用户直接输入 URL 访问无权限页面。需要再次强调，前端路由守卫不能替代后端接口鉴权，它只是用户体验层面的控制。

### 5.4 按钮级权限控制

用户管理页面中的新增按钮通过权限控制显示：

```vue
<el-button
  v-if="auth.has('user:create')"
  type="primary"
  @click="openCreate"
>
  新增用户
</el-button>
```

删除按钮同理：

```vue
<el-button
  v-if="auth.has('user:delete')"
  size="small"
  type="danger"
  @click="remove(row)"
>
  删除
</el-button>
```

这种写法让权限控制比较直观，但如果页面中大量按钮都需要权限判断，后续可以封装为自定义指令，例如 `v-permission="'user:create'"`。

## 六、初始化数据设计

学习项目必须降低首次运行成本，因此提供了初始化脚本 `scripts/init_db.py`。脚本做三件事：

1. 创建数据库表。
2. 初始化权限标识。
3. 初始化角色和演示用户。

权限种子数据示例：

```python
PERMISSIONS = [
    ("user:read", "查看用户", "用户管理"),
    ("user:create", "创建用户", "用户管理"),
    ("user:update", "更新用户", "用户管理"),
    ("user:delete", "删除用户", "用户管理"),
    ("role:read", "查看角色", "角色管理"),
    ("permission:read", "查看权限", "权限管理"),
]
```

角色授权逻辑如下：

```python
admin_role.permissions = list(permission_map.values())
editor_role.permissions = [
    p for code, p in permission_map.items()
    if code.endswith(":read") or code in {"user:update"}
]
viewer_role.permissions = [
    p for code, p in permission_map.items()
    if code.endswith(":read")
]
```

演示账号如下：

| 用户名 | 密码 | 角色 |
| --- | --- | --- |
| admin | admin123 | 系统管理员 |
| editor | editor123 | 运营编辑 |
| viewer | viewer123 | 只读访客 |

初始化脚本设计为可重复执行。重复执行时不会重复插入权限、角色和用户，而是更新角色权限关系。

## 七、运行与验证

### 7.1 后端运行

```powershell
cd D:\pycharm_workspace\python_code\FastAPI\fastapi-rbac-admin
python -m venv .venv
.\.venv\Scripts\python -m pip install -r requirements.txt
.\.venv\Scripts\python scripts\init_db.py
.\.venv\Scripts\python -m uvicorn app.main:app --reload
```

后端启动后访问：

```text
http://127.0.0.1:8000/docs
```

Swagger 页面可以看到中文接口摘要，例如“账号密码登录”“查询用户列表”“创建角色”等。

### 7.2 前端运行

```powershell
cd D:\pycharm_workspace\python_code\FastAPI\fastapi-rbac-admin\frontend
npm install
npm run dev
```

前端地址：

```text
http://127.0.0.1:5173
```

### 7.3 接口验证

本项目至少需要验证三类场景。

第一，管理员登录后可以访问用户、角色、权限接口。

第二，只读用户登录后可以访问查询接口。

第三，只读用户访问创建、更新、删除接口时应返回 403。

示例测试结果：

```text
admin login 200
admin users 200
admin roles 200
viewer create 403 缺少权限：user:create
```

这说明后端权限拦截已经生效。即使前端隐藏了按钮，也仍然需要后端测试验证，因为接口才是权限安全的边界。

## 八、开发过程中的问题与处理

### 8.1 ORM 对象序列化问题

开发过程中曾遇到 Pydantic 无法序列化 SQLAlchemy 对象的问题。原因是统一响应结构中的 `data` 字段混入了 ORM 对象。解决方案是将 ORM 对象显式转换为普通字典，再交给 FastAPI 返回。

这一问题说明，在真实项目中应当明确区分数据库对象、业务对象和响应对象。直接返回 ORM 对象虽然方便，但当对象包含 relationship、懒加载属性或复杂类型时，很容易出现序列化问题。

### 8.2 bcrypt 版本兼容问题

Python 3.13 环境下，`passlib==1.7.4` 与新版 `bcrypt` 存在兼容警告或异常。最终将 `bcrypt` 锁定为 `4.0.1`，保证初始化脚本和密码哈希逻辑稳定运行。

依赖版本锁定是学习项目也需要重视的事项。否则同一份代码在不同时间安装依赖，可能因为上游包更新而出现不同结果。

### 8.3 Hexo PowerShell 执行策略问题

本机 PowerShell 执行策略会拦截 `hexo.ps1`。部署笔记时使用以下方式绕过：

```powershell
npx.cmd hexo clean
npx.cmd hexo generate
npx.cmd hexo deploy
```

这不是 Hexo 本身的问题，而是 Windows PowerShell 对 `.ps1` 脚本执行的限制。

## 九、项目复盘

这个项目的价值不在于代码规模，而在于它把一个后台系统的主干流程串联起来。

从需求分析看，系统先确定了用户、角色、权限和接口鉴权的边界，避免一开始就陷入页面细节。

从设计看，RBAC 模型让权限维护从“给用户分配权限”转为“给角色分配权限”，符合后台系统常见实践。

从数据库看，多对多关系是权限系统的关键。通过 `user_roles` 和 `role_permissions` 两张关联表，可以清晰表达用户、角色、权限之间的关系。

从后端实现看，FastAPI 的依赖注入非常适合认证和鉴权。`get_current_user` 解决“用户是谁”，`require_permission` 解决“用户能不能访问该接口”。

从前端实现看，Pinia 保存用户权限，Axios 统一携带 Token，Router 控制页面访问，按钮根据权限显示。前端形成了较完整的管理后台体验。

从工程实践看，README、设计文档、初始化脚本、固定依赖版本、接口测试和部署笔记同样重要。它们决定了项目未来能否被自己或他人快速理解和复现。

## 十、后续扩展方向

如果要将该项目继续推进到更接近生产的程度，可以考虑以下方向：

1. 引入 Alembic 管理数据库迁移，替代 `Base.metadata.create_all`。
2. 增加刷新 Token，缩短 Access Token 有效期。
3. 增加操作日志，记录谁在什么时候修改了用户、角色和权限。
4. 增加菜单表，将“页面菜单权限”和“接口操作权限”拆开。
5. 增加部门或租户字段，支持更复杂的数据范围权限。
6. 增加 pytest 自动化测试，覆盖登录、鉴权和 CRUD。
7. 使用 MySQL 或 PostgreSQL 替代 SQLite。
8. 前端封装 `v-permission` 指令，减少页面中的重复权限判断。

通过这个项目可以得出一个结论：学习 FastAPI 不应只停留在单个接口示例上。只有把认证、数据库、业务分层、权限控制、前后端联调和部署文档放到同一个项目中，才能真正理解 FastAPI 在实际开发中的使用方式。
