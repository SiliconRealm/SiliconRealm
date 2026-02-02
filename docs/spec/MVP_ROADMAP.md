# Silicon Realm MVP 实施路线图

**Version:** 4.0  
**Date:** 2026-02-02  
**Status:** Planning  
**Based on:** Architecture v3.5

---

## MVP 目标

**极简验证**：纯 Agent 架构，通过 Telegram/TUI 交互，聚焦核心业务流程：
- 系统创世：King 根据用户材料划分领域，通过 Docker SDK Tool 启动 Lord 容器
- King ↔ Lord 业务流程：任务转接、Agent 间通信
- Agent 自治：King/Lord 通过 Skills 和 MCP Tools 获得能力，自主管理资源
- 利用 OpenClaw 原生消息通道（Telegram/TUI）

---

## 技术栈

| 组件 | 技术选型 |
|------|----------|
| Agent 引擎 | OpenClaw |
| 人机交互 | Telegram Bot / OpenClaw TUI |
| Agent 间通信 | Telegram Bot API (telegram_notify Tool) |
| 容器编排 | Docker Compose |
| 数据存储 | JSON 文件 (address_book.json, domain_map.json) |

### 项目初始化 CLI
```bash
# 创建项目根目录
mkdir silicon-realm && cd silicon-realm

# 创建 agents 目录结构
mkdir -p agents/{king/skills,genesis}

# 创建 docs 目录
mkdir -p docs/spec

# 创建共享技能目录
mkdir -p agents/shared/skills/telegram_notify
```

### MVP 简化
- **无前端** - 通过 Telegram/TUI 交互
- **无后端** - Agent 自治，无需 API 服务
- **无数据库** - JSON 文件存储配置
- **Agent 自治** - King 通过 Skills/Tools 自主管理

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
  feat(king): add genesis skill for creating lords
  fix(king): correct domain routing logic
  docs(readme): update setup instructions
  ```

---

## 核心架构

```
┌─────────────────────────────────────────────────────────────┐
│                        用户                                  │
│              (Telegram / OpenClaw TUI)                      │
└───────────────┬─────────────────────────┬───────────────────┘
                │                         │
                ▼                         ▼
┌───────────────────────┐     ┌───────────────────────┐
│  👑 King (常驻)        │     │  🏰 Lord A (常驻)     │
│  • 用户入口            │     │  • 技术领域           │
│  • 意图分析            │ ◄─► │  • 直接服务用户       │
│  • 任务转接            │     │                       │
│  • 通过 Skills/Tools   │     │  • 通过 Skills/Tools  │
│    自主管理资源        │     │    获得领域能力       │
└───────────────────────┘     └───────────────────────┘
        │                               │
        └───────────┬───────────────────┘
                    │
                    ▼
        ┌───────────────────────┐
        │  📁 Shared Resources   │
        │  • address_book.json  │
        │  • domain_map.json    │
        │  • shared/skills/     │
        └───────────────────────┘
```

**核心理念**：Agent 自治
- King/Lord 通过 Skills 获得领域知识
- King/Lord 通过 MCP Tools (Docker SDK, Telegram API) 操作外部系统
- 通过共享的 JSON 文件实现配置共享
- Agent 间通过 Telegram 消息通信
- 不需要前后端，不需要数据库

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
5. King 通过 Docker SDK Tool 创建各 Lord 的配置目录
6. King 通过 Docker SDK Tool 启动所有 Lord 容器（常驻运行）
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
6. King 通过 Docker SDK Tool 执行重整（创建/销毁/重配置 Lord 容器）
```

---

## Phase 0: 基础设施 (1 天)

### 目录结构

```
silicon-realm/
├── agents/                          # Agent 配置
│   ├── king/                        # King
│   │   ├── SOUL.md
│   │   ├── AGENTS.md
│   │   └── skills/
│   │       ├── analyze_domain/      # 领域分析
│   │       ├── genesis/             # 创世执行
│   │       └── query_address_book/  # 查询通讯录
│   ├── shared/                      # 共享资源
│   │   └── skills/
│   │       └── telegram_notify/     # Agent 间通信工具
│   ├── {domain_id}/                 # Lord（创世后生成）
│   │   ├── SOUL.md
│   │   ├── AGENTS.md
│   │   └── skills/
│   ├── genesis/                     # 创世材料
│   ├── domain_map.json              # 领域划分方案
│   └── address_book.json            # 通讯录
│
├── docs/                            # 文档
│   └── spec/
│
├── docker-compose.yml
├── .env
└── README.md
```

**结构说明**：
- `agents/` - Agent 配置（King 和各 Lord 平级）
- `agents/shared/` - 共享技能（如 telegram_notify）
- `docs/` - 项目文档
- **无 apps/ 目录** - 纯 Agent 架构，无前后端

### Docker Compose

```yaml
version: '3.8'

services:
  # ============ Agent 层 ============
  king:
    image: openclaw/openclaw:latest
    volumes:
      - ./agents/king:/app/workspace
      - ./agents:/agents              # 可读写，用于创建 Lord 配置
      - ./agents/shared/skills:/app/skills:ro  # 共享技能
      - /var/run/docker.sock:/var/run/docker.sock  # Docker SDK Tool
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - TELEGRAM_BOT_TOKEN=${KING_TELEGRAM_BOT_TOKEN}
    networks: [realm-net]
    restart: unless-stopped

  # Lord 容器由 King 通过 Docker SDK Tool 动态创建
  # Lord 挂载:
  #   - ./agents/{domain_id}:/app/workspace
  #   - ./agents:/agents:ro (只读访问 address_book.json)
  #   - ./agents/shared/skills:/app/skills:ro (共享技能)

networks:
  realm-net:
    driver: bridge
```

**关键点**：
- 只有 King 一个服务，Lord 由 King 动态创建
- 共享技能目录挂载给所有 Agent
- King 对 agents/ 可读写，Lord 只读

### 任务清单

| 任务 | 完成标准 |
|------|----------|
| 创建项目结构 | `agents/`, `docs/` 就位 |
| docker-compose.yml | King 可启动 |
| 配置 King Telegram Bot | King 可接收用户消息 |
| telegram_notify 工具 | 共享技能可用 |

---

## Phase 1: 系统创世流程 (3 天)

### King 领域分析技能

```markdown
<!-- agents/king/skills/analyze_domain/SKILL.md -->
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
- /agents/genesis/ 目录下的业务材料

## 输出
- /agents/domain_map.json - 领域划分方案

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
    }
  ]
}
```
```

### King 创世执行技能

```markdown
<!-- agents/king/skills/genesis/SKILL.md -->
---
emoji: 🌅
tools:
  - docker_sdk
  - filesystem
---
# 创世执行

当 Admin 确认领域划分方案后，执行创世流程。

## 工作流程
1. 读取 /agents/domain_map.json
2. 为每个领域创建配置目录和文件
3. 通过 Docker SDK Tool 启动 Lord 容器
4. 更新 Address Book

## 创建 Lord 配置
对于每个领域，创建以下文件：
- /agents/{domain_id}/SOUL.md - Lord 的灵魂定义
- /agents/{domain_id}/AGENTS.md - Lord 的工作流程

## 启动 Lord 容器
使用 Docker SDK Tool 执行：
```python
docker.containers.run(
    image='openclaw/openclaw:latest',
    name=f'lord-{domain_id}',
    detach=True,
    volumes={
        f'agents/{domain_id}': {'bind': '/app/workspace'},
        'agents': {'bind': '/agents', 'mode': 'ro'},
        'agents/shared/skills': {'bind': '/app/skills', 'mode': 'ro'},
    },
    environment={
        'ANTHROPIC_API_KEY': os.environ['ANTHROPIC_API_KEY'],
        'TELEGRAM_BOT_TOKEN': os.environ[telegram_bot_token_env],
    },
    network='realm-net',
    restart_policy={'Name': 'unless-stopped'},
)
```

## 更新 Address Book
创建/更新 /agents/address_book.json
```

### 创世流程（通过 Telegram/TUI 对话）

```
1. Admin 将业务材料放入 agents/genesis/
2. Admin → King (Telegram): "请分析业务材料，划分领域"
3. King 分析材料，生成初步方案
4. King → Admin: "我建议划分为以下领域：
    - 技术领域：负责代码开发、系统维护
    - 财务领域：负责财务分析、预算管理
    
    请问：
    1. 这些领域边界是否清晰？
    2. 是否有职责重叠？"
5. Admin 反馈，King 调整方案
6. 重复 4-5 直到 Admin 确认满意
7. Admin → King: "确认，开始创世"
8. King 执行创世：
   - 创建各 Lord 配置目录
   - 通过 Docker SDK Tool 启动 Lord 容器
   - 更新 Address Book
   - 通过 telegram_notify 通知各 Lord 上线
9. King → Admin: "创世完成，已启动 N 个领主"
```

### 任务清单

| 任务 | 完成标准 |
|------|----------|
| analyze_domain 技能 | King 能分析材料生成 domain_map |
| genesis 技能 | King 能创建 Lord 配置并启动容器 |
| Docker SDK Tool | King 能通过 Tool 操作 Docker |
| 创世流程测试 | 通过 Telegram 完成创世 |

---

## Phase 2: King ↔ Lord 业务流程 (3 天)

### King 配置

```markdown
<!-- agents/king/AGENTS.md -->
# King Agent

## 职责
1. 用户入口，接收所有用户请求
2. 简单问题直接回答
3. 专业任务转接给对应 Lord

## 工作流
1. 收到用户消息
2. 分析意图，判断所属领域
3. 简单问题 → 直接回答
4. 专业任务 → 查询 Address Book → 转接给 Lord

## 转接方式
1. 告知用户对应 Lord 的联系方式
2. 或通过 telegram_notify 通知 Lord 主动联系用户
```

### King 转接技能

```markdown
<!-- agents/king/skills/request_handover/SKILL.md -->
---
emoji: 🤝
tools:
  - telegram_notify
---
# 请求转接

当需要将用户转接给 Lord 时，使用此技能。

## 工作流程
1. 读取 /agents/address_book.json 查询目标 Lord 信息
2. 方式一：告知用户对应 Lord 的联系方式
3. 方式二：通过 Telegram Notify Tool 通知 Lord 主动联系用户

## 方式一：告知用户 Lord 联系方式
"这个任务需要技术领主来处理，你可以直接联系他：@TechLordBot"

## 方式二：通知 Lord 主动联系用户
使用 telegram_notify tool 发送消息给 Lord：
```json
{
  "target_bot": "tech",
  "message": "用户 @user123 需要帮助写爬虫，请主动联系他。\n上下文：用户想抓取某网站的商品数据..."
}
```
Lord 收到消息后会主动联系用户。

## Address Book 格式
```json
{
  "version": "1.0",
  "updated_at": "2026-02-02",
  "king": {
    "telegram_bot": "@KingBot",
    "telegram_chat_id": "123456789"
  },
  "lords": [
    {
      "id": "tech",
      "name": "技术领主",
      "description": "负责技术开发、代码编写、系统维护",
      "keywords": ["代码", "开发", "编程"],
      "telegram_bot": "@TechLordBot",
      "telegram_chat_id": "987654321",
      "container_name": "lord-tech"
    }
  ]
}
```
```

### Telegram Notify Tool (MCP)

```python
# agents/king/skills/telegram_notify/tool.py
"""
Telegram Notify Tool - Agent 间通信工具

通过 Telegram Bot API 向其他 Agent 发送消息，实现 Agent 间通信。
"""
import os
import json
import httpx

def load_address_book():
    """加载通讯录"""
    with open('/agents/address_book.json', 'r') as f:
        return json.load(f)

async def notify_agent(target_id: str, message: str) -> dict:
    """
    向目标 Agent 发送通知消息
    
    Args:
        target_id: 目标 Agent ID (如 "tech", "finance", "king")
        message: 要发送的消息内容
    
    Returns:
        {"success": True/False, "message": "..."}
    """
    address_book = load_address_book()
    
    # 查找目标 Agent
    target = None
    if target_id == "king":
        target = address_book.get("king")
        bot_token = os.environ.get("KING_TELEGRAM_BOT_TOKEN")
    else:
        for lord in address_book.get("lords", []):
            if lord["id"] == target_id:
                target = lord
                bot_token = os.environ.get(f"{target_id.upper()}_LORD_TELEGRAM_BOT_TOKEN")
                break
    
    if not target:
        return {"success": False, "message": f"Agent '{target_id}' not found"}
    
    chat_id = target.get("telegram_chat_id")
    if not chat_id:
        return {"success": False, "message": f"No chat_id for agent '{target_id}'"}
    
    # 发送 Telegram 消息
    url = f"https://api.telegram.org/bot{bot_token}/sendMessage"
    async with httpx.AsyncClient() as client:
        response = await client.post(url, json={
            "chat_id": chat_id,
            "text": message,
            "parse_mode": "Markdown"
        })
    
    if response.status_code == 200:
        return {"success": True, "message": f"Notified {target_id}"}
    else:
        return {"success": False, "message": f"Telegram API error: {response.text}"}
```

```markdown
<!-- agents/king/skills/telegram_notify/SKILL.md -->
---
emoji: 📨
tools:
  - notify_agent
---
# Telegram Notify

向其他 Agent 发送 Telegram 消息，用于 Agent 间通信。

## 使用场景
- King 通知 Lord 主动联系用户
- Lord 向 King 汇报任务完成
- Lord 之间协作时互相通知

## 工具调用
```python
notify_agent(target_id="tech", message="用户需要帮助...")
```

## 参数
- target_id: 目标 Agent ID，可以是 "king" 或 Lord 的 id（如 "tech", "finance"）
- message: 消息内容，支持 Markdown 格式
```

### Lord 配置（通用模板）

```markdown
<!-- Lord AGENTS.md 通用模板 -->
# {Domain} Lord Agent

## 职责
{description}

## 工作流
1. 用户直接联系，或收到 King/其他 Lord 的 Telegram 通知
2. 确认需求细节
3. 执行任务
4. 交付成果
5. 如需协作，通过 telegram_notify 工具联系其他 Agent

## 查询其他 Agent
读取 /agents/address_book.json 获取其他 Agent 的信息。

## 通知其他 Agent
使用 telegram_notify 工具发送消息。
```

### 通信流程

```
场景 1: 用户直接联系 Lord
用户 → Lord (通过 Lord 的 Telegram Bot)
Lord 直接处理任务

场景 2: 用户通过 King 转接（告知联系方式）
用户 → King: "帮我写个爬虫"
King 查询 address_book.json
King: "这是技术任务，技术领主 @TechLordBot 可以帮你"
用户 → Lord: "King 让我找你写爬虫"
Lord 处理任务

场景 3: King 通知 Lord 主动联系用户
用户 → King: "帮我写个爬虫"
King 使用 telegram_notify 工具通知 Tech Lord
Tech Lord 收到 Telegram 消息
Tech Lord → 用户: "你好，King 告诉我你需要爬虫..."
Tech Lord 处理任务

场景 4: Lord 之间协作
Tech Lord 发现需要财务数据
Tech Lord 使用 telegram_notify 通知 Finance Lord
Finance Lord 提供数据
Tech Lord 完成任务
```

### 任务清单

| 任务 | 完成标准 |
|------|----------|
| King AGENTS.md | 转接逻辑清晰 |
| query_address_book 技能 | 能读取 address_book.json 查询 Agent 信息 |
| telegram_notify 工具 | 能通过 Telegram API 向其他 Agent 发消息 |
| 端到端测试 | 转接流程跑通（含 Agent 间通信） |

---

## Phase 3: 端到端验证 (1 天)

### 验证场景

| 场景 | 预期结果 |
|------|----------|
| 系统创世 | King 分析材料，自主创建并启动 Lord |
| 用户直接联系 Lord | Lord 正常响应 |
| 用户通过 King 转接 | 转接流程正常 |

### 测试用例

```
测试 1: 系统创世
Admin 上传业务材料
King 生成 domain_map.json
Admin 确认
King 通过 Docker SDK Tool 创建并启动 Lord
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
| Agent 间通信测试 | telegram_notify 正常工作 |
| 文档更新 | 使用说明完整 |

---

## MVP 里程碑

| 阶段 | 时长 | 产出 |
|------|------|------|
| Phase 0 | 1 天 | 基础设施就绪 |
| Phase 1 | 3 天 | 系统创世流程 |
| Phase 2 | 3 天 | King ↔ Lord 业务流程 |
| Phase 3 | 1 天 | 端到端验证 |
| **总计** | **~1 周** | **MVP 完成** |

---

## MVP 验证点

| 验证点 | 成功标准 |
|--------|----------|
| 系统创世 | King 能通过 Telegram 对话完成创世 |
| Lord 常驻 | 所有 Lord 容器稳定运行 |
| 消息通道 | King/Lord 都能通过 Telegram 与用户交流 |
| 任务转接 | King 能正确转接任务给 Lord |
| Agent 间通信 | Agent 能通过 telegram_notify 互相通信 |

---

## MVP 不包含

- 前端 Web UI
- 后端 API 服务
- 数据库
- 领域重整流程（圆桌会议、投票）
- 双层会话（守护 + Topic）
- 记忆压缩
- Knight 沙箱
- Academy Knowledge Base

---

## 附录

### A. 环境变量

```bash
# .env 示例
ANTHROPIC_API_KEY=your_api_key

# King Telegram Bot
KING_TELEGRAM_BOT_TOKEN=your_king_bot_token

# Lord Telegram Bots（创世后配置）
TECH_LORD_TELEGRAM_BOT_TOKEN=your_tech_lord_bot_token
FINANCE_LORD_TELEGRAM_BOT_TOKEN=your_finance_lord_bot_token
```

### B. Address Book 格式

```json
{
  "version": "1.0",
  "updated_at": "2026-02-02",
  "king": {
    "telegram_bot": "@KingBot",
    "telegram_chat_id": "123456789"
  },
  "lords": [
    {
      "id": "tech",
      "name": "技术领主",
      "description": "负责技术开发、代码编写、系统维护",
      "keywords": ["代码", "开发", "编程"],
      "telegram_bot": "@TechLordBot",
      "telegram_chat_id": "987654321",
      "container_name": "lord-tech"
    }
  ]
}
```

---

## 后续迭代路径

```
MVP (1 周)
  └─ 纯 Agent 架构 + Telegram 交互
      ↓
v0.2 (1 周)
  └─ Web UI 仪表盘（可选）
      ↓
v0.3 (2 周)
  └─ 领域重整流程（圆桌会议简化版）
      ↓
v0.4 (2 周)
  └─ Knight 沙箱执行
      ↓
v0.5 (2 周)
  └─ 双层会话 + 记忆压缩
      ↓
v1.0
  └─ Academy Knowledge Base + 完整圆桌会议
```

---

*Version History*
- v2.0: Based on Architecture v3.0
- v2.1: Simplified MVP - focus on Canal orchestration
- v2.2: Use OpenClaw Gateway, Lord proactively contacts user
- v2.3: Lord is persistent, added Genesis flow
- v2.4: Added tech stack (FastAPI+PostgreSQL, React+TanStack Router)
- v2.5: Frontend uses shadcn/ui, added CLI init commands
- v2.6: Separated Canal from Backend
- v2.7: Changed realm/ to agents/ with flat structure
- v2.8: Renamed Queen to Canal
- v2.9: Unified naming convention (English first)
- v3.0: **Agent Autonomy** - Removed Canal, King manages Lords via Docker SDK Tool
- v3.1: **Removed Redis** - Use shared address_book.json
- v3.2: **Agent-to-Agent Communication** - Added telegram_notify MCP Tool
- v4.0: **Pure Agent Architecture** - Removed frontend/backend/database, Telegram/TUI only, MVP reduced to 1 week
