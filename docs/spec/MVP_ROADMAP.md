# Silicon Realm MVP 实施路线图

**Version:** 2.5  
**Date:** 2026-02-02  
**Status:** Planning  
**Based on:** Architecture v3.0

---

## MVP 目标

**简化验证**：直接利用 OpenClaw 的网关和会话管理能力，聚焦核心业务流程：
- 系统创世：King 根据用户材料划分领域，生成常驻 Lord
- Queen 编排：创世时创建 Lord 配置并启动容器（健康检查交给 Docker Compose）
- King ↔ Lord 业务流程：任务转接、用户交接
- 利用 OpenClaw 原生消息通道（无需自建 inbox/outbox）

---

## 技术栈

### 后端
| 组件 | 技术选型 |
|------|----------|
| Web Server | FastAPI |
| 数据库 | PostgreSQL |
| Agent 引擎 | OpenClaw |
| 消息队列 | Redis |
| 容器编排 | Docker Compose |

### 前端
| 组件 | 技术选型 |
|------|----------|
| 框架 | React |
| 路由 | TanStack Router |
| 状态管理 | TanStack Query |
| UI 组件库 | shadcn/ui |
| 样式 | Tailwind CSS (shadcn 预设) |

### 项目初始化 CLI
```bash
# 后端 (Python)
uv init backend
cd backend && uv add fastapi uvicorn sqlalchemy asyncpg

# 前端 (React + shadcn)
pnpm create vite frontend --template react-ts
cd frontend
pnpm add -D tailwindcss postcss autoprefixer
pnpm dlx shadcn@latest init
pnpm dlx shadcn@latest add button card table dialog form input
pnpm add @tanstack/react-router @tanstack/react-query
```

### MVP 简化
- 无用户管理（单用户：Admin）
- 无认证授权
- 专注跑通主业务流程

---

## 开发规范

### 架构原则
| 原则 | 说明 |
|------|------|
| **DDD** | 领域驱动设计，按业务领域组织代码 |
| **FDD** | 功能驱动开发，按功能模块迭代 |
| **TDD** | 测试驱动开发，先写测试再实现 |
| **KISS** | 保持简单，避免过度设计 |
| **DRY** | 不重复自己，抽取公共逻辑 |

### 版本控制
- **原子提交**：每个 commit 只做一件事
- **Conventional Commits** 规范：
  ```
  feat: 新功能
  fix: 修复 bug
  docs: 文档更新
  style: 代码格式
  refactor: 重构
  test: 测试相关
  chore: 构建/工具
  ```
- 示例：
  ```
  feat(queen): add genesis flow for creating lords
  fix(king): correct domain routing logic
  docs(readme): update setup instructions
  ```

---

## 核心架构

```
┌─────────────────────────────────────────────────────────────┐
│                        用户                                  │
│              (Telegram / Discord / Slack)                   │
└───────────────┬─────────────────────────┬───────────────────┘
                │                         │
                ▼                         ▼
┌───────────────────────┐     ┌───────────────────────┐
│  👑 King (常驻)        │     │  🏰 Lord A (常驻)     │
│  • 用户入口            │     │  • 技术领域           │
│  • 意图分析            │ ◄─► │  • 直接服务用户       │
│  • 任务转接            │     │                       │
└───────────────────────┘     └───────────────────────┘
                │                         │
                │     ┌───────────────────┤
                │     │                   │
                ▼     ▼                   ▼
┌─────────────────────────────────────────────────────────────┐
│                    👸 Queen (Python 服务)                    │
│  • 创世时创建 Lord 配置并启动容器                           │
│  • 领域重整时的容器编排                                     │
│  • 健康检查交给 Docker Compose (restart: unless-stopped)    │
└─────────────────────────────────────────────────────────────┘
```

**关键变化**：Lord 是常驻的，不是按需启动

---

## 系统生命周期

### 1. 系统创世 (Genesis)

**遵循 DDD 原则，通过多轮对话确认领域边界**

```
1. Admin 上传《公司业务白皮书》或其他材料
2. King 分析材料，生成初步领域划分方案
3. King 与 Admin 多轮对话，确认：
   - 领域边界是否清晰？
   - 是否存在职责重叠？
   - 是否需要拆分或合并？
4. Admin 确认最终方案
5. Queen 根据 Domain_Map 创建各 Lord 的配置目录
6. Queen 启动所有 Lord 容器（常驻运行）
7. 系统就绪，开始服务用户
```

**DDD 领域划分原则**：
- 基于业务能力而非技术栈划分
- 每个领域有明确的职责边界
- 领域之间低耦合、高内聚
- 避免职责重叠和模糊地带

### 2. 日常运行

```
用户 → King/Lord (通过各自的消息通道)
       │
       ├─ 简单问题 → King 直接回答
       │
       └─ 专业任务 → King 转接给对应 Lord
                     Lord 主动联系用户
                     Lord 执行任务
```

### 3. 领域重整 (Domain Restructure)

```
1. Lord/King 发现领域不适配，提出重整请求
2. King 制定重整方案（拆分/合并/调整边界）
3. 相关 Lord Review 方案
4. 圆桌会议投票
5. 人类最终确认
6. Queen 执行重整（创建/销毁/重配置 Lord 容器）
```

---

## Phase 0: 基础设施 (3 天)

### 目录结构

```
project/
├── realm/                           # Agent 配置（王国）
│   ├── crown/
│   │   ├── king/
│   │   │   ├── SOUL.md
│   │   │   ├── AGENTS.md
│   │   │   └── skills/
│   │   └── queen/
│   │       └── config.yaml
│   ├── fiefdoms/                    # Lord 配置（创世后生成）
│   ├── genesis/                     # 创世材料
│   ├── domain_map.json
│   └── address_book.json
├── backend/                         # FastAPI 后端
│   ├── src/
│   │   ├── domain/                  # DDD 领域层
│   │   │   ├── genesis/             # 创世领域
│   │   │   │   ├── entities.py
│   │   │   │   ├── services.py
│   │   │   │   └── repository.py
│   │   │   └── orchestration/       # 编排领域
│   │   │       ├── entities.py
│   │   │       ├── services.py
│   │   │       └── repository.py
│   │   ├── infrastructure/          # 基础设施层
│   │   │   ├── database.py
│   │   │   ├── docker_client.py
│   │   │   └── redis_client.py
│   │   ├── api/                     # API 层
│   │   │   ├── routes/
│   │   │   │   ├── genesis.py
│   │   │   │   └── lords.py
│   │   │   └── main.py
│   │   └── tests/                   # TDD 测试
│   │       ├── domain/
│   │       └── api/
│   ├── pyproject.toml
│   └── Dockerfile
├── frontend/                        # React 前端
│   ├── src/
│   │   ├── routes/                  # TanStack Router
│   │   │   ├── __root.tsx
│   │   │   ├── index.tsx            # 首页/仪表盘
│   │   │   ├── genesis.tsx          # 创世页面
│   │   │   └── lords.tsx            # Lord 管理
│   │   ├── components/
│   │   ├── api/                     # API 调用
│   │   └── main.tsx
│   ├── package.json
│   └── Dockerfile
├── docker-compose.yml
└── .env
```

### Docker Compose

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: silicon_realm
      POSTGRES_USER: realm
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks: [realm-net]

  redis:
    image: redis:7-alpine
    ports: ["6379:6379"]
    networks: [realm-net]
  
  backend:
    build: ./backend
    ports: ["8000:8000"]
    volumes:
      - ./realm:/realm
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - DATABASE_URL=postgresql://realm:${POSTGRES_PASSWORD}@postgres:5432/silicon_realm
      - REDIS_URL=redis://redis:6379
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
    depends_on: [postgres, redis]
    networks: [realm-net]

  frontend:
    build: ./frontend
    ports: ["3000:3000"]
    environment:
      - VITE_API_URL=http://localhost:8000
    depends_on: [backend]
    networks: [realm-net]

  king:
    image: openclaw/openclaw:latest
    volumes:
      - ./realm/crown/king:/app/workspace
      - ./realm:/realm
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - TELEGRAM_BOT_TOKEN=${KING_TELEGRAM_BOT_TOKEN}
    depends_on: [redis, backend]
    networks: [realm-net]
    restart: unless-stopped

  # Lord 容器由 backend 在创世后动态启动

networks:
  realm-net:
    driver: bridge

volumes:
  postgres_data:
```

### 任务清单

| 任务 | 完成标准 |
|------|----------|
| 创建目录结构 | `realm/`, `backend/`, `frontend/` 就位 |
| docker-compose.yml | 可启动所有服务 |
| 配置 King Telegram Bot | King 可接收用户消息 |
| 初始化 PostgreSQL | 数据库 schema 就绪 |
| 初始化前端项目 | React + TanStack Router 骨架 |

---

## Phase 1: 系统创世流程 (1 周)

### King 领域分析技能

```markdown
<!-- realm/crown/king/skills/analyze_domain/SKILL.md -->
---
emoji: 🗺️
---
# 领域分析

当 Admin 上传业务材料时，分析并划分业务领域。

## DDD 原则
1. **基于业务能力划分**：按业务领域而非技术栈
2. **边界清晰**：每个领域有明确的职责范围
3. **低耦合高内聚**：领域之间依赖最小化
4. **避免重叠**：不允许职责模糊地带

## 工作流程
1. 分析业务材料，识别核心业务能力
2. 生成初步领域划分方案
3. **与 Admin 多轮对话确认**：
   - 展示划分方案，解释每个领域的职责
   - 询问是否有职责重叠或边界模糊
   - 根据反馈调整方案
   - 重复直到 Admin 确认满意
4. 生成最终 domain_map.json

## 输入
- /realm/genesis/ 目录下的业务材料

## 输出
- /realm/domain_map.json - 领域划分方案

## 领域划分原则
1. 基于业务领域而非技术栈
2. 每个领域有明确的职责边界
3. 领域之间低耦合

## domain_map.json 格式
```json
{
  "version": "1.0",
  "created_at": "2026-02-02",
  "domains": [
    {
      "id": "tech",
      "name": "技术领主",
      "description": "负责技术开发、代码编写、系统维护",
      "keywords": ["代码", "开发", "编程", "API", "部署"],
      "telegram_bot_token_env": "TECH_LORD_TELEGRAM_BOT_TOKEN"
    },
    {
      "id": "finance",
      "name": "财务领主",
      "description": "负责财务分析、预算管理、成本核算",
      "keywords": ["财务", "预算", "成本", "报表", "审计"],
      "telegram_bot_token_env": "FINANCE_LORD_TELEGRAM_BOT_TOKEN"
    }
  ]
}
```
```

### Queen 创世流程（后端实现）

```python
# backend/src/domain/genesis/services.py

class GenesisService:
    """创世服务 - DDD 领域服务"""
    
    def __init__(self, docker_client: DockerClient, db: Database):
        self.docker = docker_client
        self.db = db
    
    async def create_lord(self, domain: DomainEntity) -> Lord:
        """创建 Lord 配置目录和容器"""
        lord_path = f'realm/fiefdoms/{domain.id}'
        os.makedirs(lord_path, exist_ok=True)
        
        # 生成 SOUL.md
        await self._generate_soul(lord_path, domain)
        
        # 生成 AGENTS.md
        await self._generate_agents(lord_path, domain)
        
        # 持久化到数据库
        lord = Lord(
            id=domain.id,
            name=domain.name,
            status='created',
        )
        await self.db.lords.create(lord)
        
        return lord
    
    async def start_all_lords(self) -> list[Lord]:
        """启动所有 Lord 容器"""
        lords = await self.db.lords.get_all()
        
        for lord in lords:
            await self._start_lord_container(lord)
            lord.status = 'running'
            await self.db.lords.update(lord)
        
        return lords
    
    async def _start_lord_container(self, lord: Lord):
        """启动单个 Lord 容器"""
        self.docker.containers.run(
            image='openclaw/openclaw:latest',
            name=f'lord-{lord.id}',
            detach=True,
            volumes={
                f'realm/fiefdoms/{lord.id}': {'bind': '/app/workspace', 'mode': 'rw'},
                'realm': {'bind': '/realm', 'mode': 'ro'},
            },
            environment={
                'ANTHROPIC_API_KEY': os.environ['ANTHROPIC_API_KEY'],
                'TELEGRAM_BOT_TOKEN': os.environ[f'{lord.id.upper()}_LORD_TELEGRAM_BOT_TOKEN'],
                'AGENT_ID': f'lord-{lord.id}',
            },
            network='realm-net',
            restart_policy={'Name': 'unless-stopped'},
        )
```

```python
# backend/src/api/routes/genesis.py

from fastapi import APIRouter, Depends
from src.domain.genesis.services import GenesisService

router = APIRouter(prefix="/api/genesis", tags=["genesis"])

@router.post("/domains")
async def create_domain(
    domain: DomainCreate,
    service: GenesisService = Depends()
):
    """创建领域（Admin 确认后调用）"""
    lord = await service.create_lord(domain)
    return {"lord_id": lord.id, "status": lord.status}

@router.post("/start")
async def start_genesis(service: GenesisService = Depends()):
    """启动所有 Lord 容器"""
    lords = await service.start_all_lords()
    return {"started": len(lords), "lords": [l.id for l in lords]}
```

### 创世流程（多轮对话）

```
1. Admin 将业务材料放入 realm/genesis/
2. Admin 通知 King: "请分析业务材料，划分领域"
3. King 分析材料，生成初步方案
4. King 向 Admin 展示方案，询问确认：
   "我建议划分为以下领域：
    - 技术领域：负责代码开发、系统维护
    - 财务领域：负责财务分析、预算管理
    
    请问：
    1. 这些领域边界是否清晰？
    2. 是否有职责重叠？
    3. 是否需要调整？"
5. Admin 反馈，King 调整方案
6. 重复 4-5 直到 Admin 确认满意
7. King 生成最终 domain_map.json
8. Admin 确认后通知 Queen: "执行创世"
9. Queen 创建 Lord 配置并启动容器
10. 系统就绪
```

### 任务清单

| 任务 | 完成标准 |
|------|----------|
| analyze_domain 技能 | King 能分析材料生成 domain_map |
| GenesisService | 后端能创建 Lord 配置并启动容器 |
| Genesis API | `/api/genesis/*` 接口可用 |
| 前端创世页面 | 能上传材料、查看方案、确认创世 |
| TDD 测试 | GenesisService 单元测试通过 |

---

## Phase 2: King ↔ Lord 业务流程 (1 周)

### King 配置

```markdown
<!-- realm/crown/king/AGENTS.md -->
# King Agent

## 职责
1. 用户入口，接收所有用户请求
2. 简单问题直接回答
3. 专业任务转接给对应 Lord

## 工作流
1. 收到用户消息
2. 分析意图，判断所属领域
3. 简单问题 → 直接回答
4. 专业任务 → 查询通讯录 → 转接给 Lord

## 转接方式
告诉用户对应 Lord 的联系方式，让用户直接联系 Lord。
或者通知 Lord 主动联系用户。
```

### King 转接技能

```markdown
<!-- realm/crown/king/skills/request_handover/SKILL.md -->
---
emoji: 🤝
---
# 请求转接

当需要将用户转接给 Lord 时，使用此技能。

## 方式一：告知用户 Lord 联系方式
"这个任务需要技术领主来处理，你可以直接联系他：@TechLordBot"

## 方式二：通知 Lord 主动联系用户
向 Redis 发送通知：
```bash
redis-cli -h redis PUBLISH lord:tech:notify \
  '{"user": "@user123", "context": "用户需要..."}'
```

Lord 收到通知后会主动联系用户。
```

### Lord 配置（通用模板）

```markdown
<!-- Lord AGENTS.md 通用模板 -->
# {Domain} Lord Agent

## 职责
{description}

## 工作流
1. 用户直接联系，或收到 King 转接通知
2. 确认需求细节
3. 执行任务
4. 交付成果

## 监听转接通知
订阅 Redis 频道 lord:{id}:notify，收到通知后主动联系用户。
```

### 通信流程

```
场景 1: 用户直接联系 Lord
用户 → Lord (通过 Lord 的 Telegram Bot)
Lord 直接处理任务

场景 2: 用户通过 King 转接
用户 → King: "帮我写个爬虫"
King: "这是技术任务，技术领主 @TechLordBot 可以帮你"
用户 → Lord: "King 让我找你写爬虫"
Lord 处理任务

场景 3: King 通知 Lord 主动联系
用户 → King: "帮我写个爬虫"
King → Redis: { notify Lord }
Lord → 用户: "你好，King 告诉我你需要爬虫..."
Lord 处理任务
```

### 任务清单

| 任务 | 完成标准 |
|------|----------|
| King AGENTS.md | 转接逻辑清晰 |
| query_address_book 技能 | 能查询 Lord 信息 |
| request_handover 技能 | 能通知 Lord |
| Lord 监听通知 | 能收到转接通知 |
| 前端 Lord 管理页面 | 能查看 Lord 状态 |
| 端到端测试 | 转接流程跑通 |

---

## Phase 3: 端到端验证 (3 天)

### 验证场景

| 场景 | 预期结果 |
|------|----------|
| 系统创世 | King 分析材料，Queen 创建 Lord |
| 用户直接联系 Lord | Lord 正常响应 |
| 用户通过 King 转接 | 转接流程正常 |

### 测试用例

```
测试 1: 系统创世
Admin 上传业务材料
King 生成 domain_map.json
Admin 确认
Queen 创建并启动 Lord
验证：所有 Lord 容器运行中

测试 2: 直接联系 Lord
用户 → Tech Lord: "帮我写个 Hello World"
Tech Lord 执行并返回结果

测试 3: 通过 King 转接
用户 → King: "帮我写个爬虫"
King 识别为技术任务
King 告知用户 Tech Lord 联系方式
用户联系 Tech Lord
Tech Lord 执行任务
```

### 任务清单

| 任务 | 完成标准 |
|------|----------|
| 创世流程测试 | 完整流程跑通 |
| 转接流程测试 | 用户能正确转接 |
| 容器恢复测试 | 异常能自动恢复 |
| 文档更新 | 使用说明完整 |

---

## MVP 里程碑

| 阶段 | 时长 | 产出 |
|------|------|------|
| Phase 0 | 3 天 | 基础设施就绪 |
| Phase 1 | 1 周 | 系统创世流程 |
| Phase 2 | 1 周 | King ↔ Lord 业务流程 |
| Phase 3 | 3 天 | 端到端验证 |
| **总计** | **~3 周** | **MVP 完成** |

---

## MVP 验证点

| 验证点 | 成功标准 |
|--------|----------|
| 系统创世 | King 能分析材料，Queen 能创建 Lord |
| Lord 常驻 | 所有 Lord 容器稳定运行（Docker Compose 管理） |
| 消息通道 | King/Lord 都能通过 Telegram 与用户交流 |
| 任务转接 | King 能正确转接任务给 Lord |

---

## MVP 不包含

- 领域重整流程（圆桌会议、投票）
- 双层会话（守护 + Topic）
- 记忆压缩
- Knight 沙箱
- Academy 知识库

---

## 附录

### A. API 接口清单

| 接口 | 方法 | 描述 |
|------|------|------|
| `/api/genesis/materials` | POST | 上传创世材料 |
| `/api/genesis/domains` | GET | 获取领域划分方案 |
| `/api/genesis/domains` | POST | 创建领域 |
| `/api/genesis/start` | POST | 启动所有 Lord |
| `/api/lords` | GET | 获取所有 Lord 列表 |
| `/api/lords/{id}` | GET | 获取单个 Lord 详情 |
| `/api/lords/{id}/status` | GET | 获取 Lord 容器状态 |
| `/api/address-book` | GET | 获取通讯录 |

### B. 数据库 Schema

```sql
-- Lords 表
CREATE TABLE lords (
    id VARCHAR(50) PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    keywords TEXT[],
    status VARCHAR(20) DEFAULT 'created',  -- created, running, stopped
    container_id VARCHAR(100),
    telegram_bot_username VARCHAR(100),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Domain Map 历史（用于审计）
CREATE TABLE domain_map_history (
    id SERIAL PRIMARY KEY,
    version VARCHAR(20) NOT NULL,
    content JSONB NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- 创世材料
CREATE TABLE genesis_materials (
    id SERIAL PRIMARY KEY,
    filename VARCHAR(255) NOT NULL,
    content TEXT NOT NULL,
    uploaded_at TIMESTAMP DEFAULT NOW()
);
```

### C. 前端页面清单

| 页面 | 路由 | 功能 | shadcn 组件 |
|------|------|------|-------------|
| 仪表盘 | `/` | 系统状态概览、Lord 运行状态 | Card, Badge |
| 创世 | `/genesis` | 上传材料、查看方案、确认创世 | Dialog, Form, Button |
| Lord 列表 | `/lords` | 查看所有 Lord、状态监控 | Table, Badge |
| Lord 详情 | `/lords/:id` | 单个 Lord 详情、日志查看 | Card, Tabs |

### D. 环境变量

```bash
# .env 示例
POSTGRES_PASSWORD=your_password
ANTHROPIC_API_KEY=your_api_key

# King Telegram Bot
KING_TELEGRAM_BOT_TOKEN=your_king_bot_token

# Lord Telegram Bots（创世后配置）
TECH_LORD_TELEGRAM_BOT_TOKEN=your_tech_lord_bot_token
FINANCE_LORD_TELEGRAM_BOT_TOKEN=your_finance_lord_bot_token
```

---

## 后续迭代路径

```
MVP (3 周)
  └─ 创世 + King/Lord 常驻 + 基础转接
      ↓
v0.2 (2 周)
  └─ 领域重整流程（圆桌会议简化版）
      ↓
v0.3 (2 周)
  └─ Knight 沙箱执行
      ↓
v0.4 (2 周)
  └─ 双层会话 + 记忆压缩
      ↓
v0.5 (2 周)
  └─ Academy 知识库
      ↓
v1.0
  └─ 完整圆桌会议 + 多 Lord 协作
```

---

*Version History*
- v2.0: Based on Architecture v3.0
- v2.1: Simplified MVP - focus on Queen orchestration
- v2.2: Use OpenClaw Gateway, Lord proactively contacts user
- v2.3: Lord is persistent, added Genesis flow
- v2.4: Added tech stack (FastAPI+PostgreSQL, React+TanStack Router), dev standards (DDD/FDD/TDD/KISS/DRY), Conventional Commits
- v2.5: Frontend uses shadcn/ui, added CLI init commands
