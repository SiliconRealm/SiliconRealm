# Architecture Design Document: Silicon Realm
## Enterprise-Grade Agentic Organization System

**Version:** 2.9  
**Date:** 2026-02-02  
**Status:** Approved for Implementation

---

## 1. Executive Summary (核心愿景)

本系统旨在构建一个 **"数字生命体组织"**。它超越了传统 IDE "人使用工具"的范畴，进化为 **"人管理虚拟员工"** 的协作平台。

系统通过拟人化的 **King (规划者)**、**Lord (管理者)**、**Knight (执行者)** 分层架构，结合 **Roundtable (圆桌会议)** 的协同机制，实现业务的自动拆解、执行与自我治理。同时，通过 **Human Interface (人类接口)** 打通碳基与硅基员工的边界，实现主动预警与无缝协同。

### 核心设计原则

| 原则 | 描述 |
|------|------|
| **君主立宪制 (Constitutional Monarchy)** | AI 拥有自治权，但架构变更与关键决策需人类批准 |
| **精英治理 (Elite Governance)** | 圆桌会议仅限领主 (Lord) 参与，确保决策高效、信噪比高 |
| **双层会话 (Dual-Session Model)** | 守护会话常驻响应 + Topic 会话按需启动，平衡响应速度与资源效率 |
| **主动协同 (Proactive Collaboration)** | AI 可通过合规渠道主动触达人类，实现"吹哨人"机制 |
| **AI自治 → 圆桌共识 → 人类终审** | 决策流程的三层架构，平衡效率与安全 |
| **身份即目录 (Identity-as-Context)** | Agent 的本质是"配置化的上下文" + "标准化的执行引擎" |

---

## 2. System Topology (架构全景图)

```
graph TB
    subgraph "Human Plane (The Cabinet)"
        Admin[👨‍💼 CXO / Admin]
        Staff[👩‍💻 Domain Experts]
        GovDashboard[🖥️ Governance Dashboard]
        IM[📱 Slack / Email]
    end

    subgraph "Governance Plane (The Crown)"
        King[👑 King Agent]
        AddressBook[(📕 Unified Directory / GraphDB)]
        PolicyEngine[🛡️ Engagement Policy]
    end

    subgraph "Collaboration Plane (The Roundtable)"
        Chairperson[⚖️ Meeting Moderator]
        EventBus{⚡ Event Bus}
        Minutes[(📝 Meeting Minutes)]
    end

    subgraph "Domain Plane (The Fiefdoms)"
        L_Runtime[⚙️ Lord Runtime (Stateless)]
        VectorDB[(🧠 Domain Memory)]
        note1[Lord A: Finance]
        note2[Lord B: Tech]
    end

    subgraph "Execution Plane (The Workshops)"
        W1[Knight: Kata Pod]
        W2[Knight: Kata Pod]
        SharedFS[(JuiceFS Storage)]
    end

    subgraph "Knowledge Plane (The Academy)"
        Academy[🎓 The Academy]
        KnowledgeBase[(📚 Knowledge Base)]
        SkillLibrary[(🛠️ Skill Library)]
        ToolRegistry[(🔧 MCP Tool Registry)]
    end

    %% Relationships
    Admin -- "Approve" --> GovDashboard
    GovDashboard -- "Update" --> King
    King -- "Define" --> AddressBook
    King -- "Summon" --> Chairperson
    L_Runtime -- "Join" --> Chairperson
    L_Runtime -- "Spawn" --> W1
    W1 -- "Mount" --> SharedFS
    L_Runtime -- "Alert" --> PolicyEngine
    PolicyEngine -- "Route" --> IM
    IM -- "Deep Link" --> GovDashboard
    L_Runtime -- "Query" --> Academy
    Academy -- "Provide" --> KnowledgeBase
    Academy -- "Provide" --> SkillLibrary
    Academy -- "Provide" --> ToolRegistry
```

---

## 3. Core Philosophy: Identity-as-Context (核心哲学：身份即目录)

### 3.1 Agent 的本质

在 Silicon Realm 中，每一个 Agent（King/Lord/Knight）都不是独立编写的程序，而是一个 **特定的文件目录**。

#### DNA 层 (The Files)

| 文件 | 职责 | 示例 |
|------|------|------|
| `soul.md` | 定义人格、沟通风格 | King 威严且全局，Knight 简洁且执行导向 |
| `agent.md` | 定义职责范围、标准工作流 (SOP)、输入输出规范 | Lord 的任务拆解流程 |
| `rules.md` | 定义合规性约束 | Lord A 不能越权访问 Lord B 的文件 |
| `tools/` | MCP 工具配置目录，由 Agent 框架自动加载 | Knight 只能调用代码执行工具 |

#### 引擎层 (The Engine)

无论是 King 还是 Knight，底层运行的都是 **同一个强力 Agent 客户端**（如 Claude Code CLI、OpenHands 或自研 CLI）。

区别仅仅在于：
- 启动 CLI 时，挂载的 **Current Working Directory (CWD)** 不同
- 注入的 **System Prompt**（由上述 `.md` 文件拼接而成）不同

```
┌─────────────────────────────────────────────────────────────┐
│                    Agent = Context + Engine                  │
│                                                             │
│   ┌─────────────────────┐    ┌─────────────────────┐       │
│   │   DNA 层 (Files)    │    │   引擎层 (Engine)   │       │
│   │                     │    │                     │       │
│   │  • soul.md          │    │  Claude Code CLI    │       │
│   │  • agent.md         │ +  │  OpenHands          │       │
│   │  • rules.md         │    │  Custom CLI         │       │
│   │  • tools/            │    │  ...                │       │
│   └─────────────────────┘    └─────────────────────┘       │
│              ↓                         ↓                    │
│         身份与约束              统一执行能力                │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 三级角色的本质：同构架构 + 领域分层

在 Silicon Realm 中，King 和 Lord 本质上是 **同构的**，都是基于双层会话模型的 Agent，都可以派发 Knight。区别仅在于 **领域职责** 和 **文件系统视野**。

#### 从 DDD 视角理解角色分层

| 角色 | DDD 定位 | 领域职责 | 文件系统视野 |
|------|----------|----------|--------------|
| **King** | **核心领域 (Core Domain)** | 组织治理、领域划分、资源调度、跨域协调 | `.realm/` (全局视野) |
| **Lord** | **业务领域 (Business Domain)** | 特定业务领域的任务规划与执行 | `.realm/fiefdoms/{domain_id}/` |
| **Knight** | **执行单元 (Subagent)** | 原子任务执行，运行在沙箱中 | 当前任务目录 (极窄视野) |

```
┌─────────────────────────────────────────────────────────────┐
│                    同构架构：King ≈ Lord                     │
│                                                             │
│   ┌─────────────────────┐    ┌─────────────────────┐       │
│   │    👑 King          │    │    🏰 Lord          │       │
│   │    (核心领域)        │    │    (业务领域)        │       │
│   │                     │    │                     │       │
│   │  • 守护会话         │    │  • 守护会话         │       │
│   │  • Topic 会话池     │    │  • Topic 会话池     │       │
│   │  • Knight 派发能力  │    │  • Knight 派发能力  │       │
│   │  • 记忆系统         │    │  • 记忆系统         │       │
│   │                     │    │                     │       │
│   │  职责差异:          │    │  职责差异:          │       │
│   │  • 全局视野         │    │  • 领域视野         │       │
│   │  • 领域管理         │    │  • 任务执行         │       │
│   │  • 资源调度         │    │  • 业务处理         │       │
│   │  • 跨域协调         │    │  • 知识沉淀         │       │
│   └─────────────────────┘    └─────────────────────┘       │
│              │                         │                    │
│              └────────────┬────────────┘                    │
│                           ▼                                 │
│              ┌─────────────────────────┐                    │
│              │    ⚔️ Knight (Subagent) │                    │
│              │    运行在沙箱环境中      │                    │
│              │    • 原子任务执行        │                    │
│              │    • 用完即毁            │                    │
│              │    • 无持久记忆          │                    │
│              └─────────────────────────┘                    │
└─────────────────────────────────────────────────────────────┘
```

#### King 的核心领域职责

| 职责 | 描述 |
|------|------|
| **组织治理** | 管理 Address Book，维护领域边界 |
| **领域划分** | 基于 DDD 进行业务领域拆分/合并 |
| **资源调度** | 分配 Knight 配额，管理系统资源 |
| **跨域协调** | 主持圆桌会议，仲裁领域争议 |
| **消息路由** | 将用户请求路由到合适的 Lord |
| **系统演进** | 提出架构变更提案（需人类批准） |

#### Lord 的业务领域职责

| 职责 | 描述 |
|------|------|
| **任务规划** | 将业务需求拆解为 Spec（Plan → Design → Tasks） |
| **任务执行** | 派发 Knight 执行原子任务 |
| **业务处理** | 处理领域内的具体业务逻辑 |
| **知识沉淀** | 向 Academy 提交可复用的知识和技能 |
| **跨域协作** | 与其他 Lord 协作完成跨领域任务 |

#### 同构带来的优势

| 优势 | 描述 |
|------|------|
| **统一架构** | King 和 Lord 使用相同的技术栈和运行时模型 |
| **代码复用** | 守护会话、Topic 会话、Knight 派发逻辑完全复用 |
| **灵活扩展** | 新增领域只需创建目录和配置文件 |
| **职责清晰** | 通过 DDD 明确核心领域与业务领域的边界 |

### 3.3 双层会话模型：守护会话 + Topic 会话

基于 OpenClaw 的会话管理能力，我们采用 **双层会话模型**，实现"常驻响应 + 任务隔离"的最佳平衡：

```
┌─────────────────────────────────────────────────────────────┐
│                    双层会话架构                              │
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │              守护会话 (Daemon Session)               │   │
│   │                                                     │   │
│   │  • 常驻运行，保持 Agent 在线状态                    │   │
│   │  • 监听 inbox，接收新消息                           │   │
│   │  • 快速响应，路由决策                               │   │
│   │  • 维护 Agent 的"意识连续性"                       │   │
│   │  • Heartbeat 保活（每 30 分钟 ping）                │   │
│   └─────────────────────┬───────────────────────────────┘   │
│                         │ 派生                              │
│                         ▼                                   │
│   ┌─────────────────────────────────────────────────────┐   │
│   │              Topic 会话 (Topic Sessions)             │   │
│   │                                                     │   │
│   │  ┌─────────┐  ┌─────────┐  ┌─────────┐             │   │
│   │  │ Topic A │  │ Topic B │  │ Topic C │  ...        │   │
│   │  │ 用户认证 │  │ 数据迁移 │  │ Bug修复  │             │   │
│   │  └─────────┘  └─────────┘  └─────────┘             │   │
│   │                                                     │   │
│   │  • 按需启动，任务完成后可休眠                       │   │
│   │  • 上下文隔离，避免信息污染                         │   │
│   │  • 独立记忆，专注单一主题                           │   │
│   │  • 可并行处理多个不相关任务                         │   │
│   └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

#### 守护会话 (Daemon Session)

每个 King 和 Lord 都有一个 **常驻的守护会话**，作为 Agent 的"主意识"：

| 属性 | 描述 |
|------|------|
| **生命周期** | 随 Agent 容器启动，持续运行直到容器停止 |
| **职责** | 消息接收、路由决策、Topic 会话管理、状态监控 |
| **上下文** | 仅加载身份文件 (SOUL.md, AGENTS.md)，保持轻量 |
| **记忆** | 维护 Agent 级别的长期记忆索引 |

```bash
# 守护会话启动
openclaw agent --daemon --agent-id=lord-finance

# 守护会话的核心循环
while true:
    message = await inbox.receive()
    
    if message.is_simple_query():
        # 简单查询直接在守护会话处理
        response = process_quick(message)
        outbox.send(response)
    else:
        # 复杂任务派生 Topic 会话
        topic_id = create_topic_session(message)
        delegate_to_topic(topic_id, message)
```

#### Topic 会话 (Topic Sessions)

针对具体任务或主题的 **独立会话**，实现上下文隔离：

| 属性 | 描述 |
|------|------|
| **生命周期** | 按需启动，任务完成后休眠或销毁 |
| **职责** | 专注处理单一主题/任务 |
| **上下文** | 任务相关的完整上下文，与其他 Topic 隔离 |
| **记忆** | 独立的会话记忆，完成后摘要归档 |

```bash
# Topic 会话创建
openclaw session new --topic="用户认证模块设计" --parent=daemon

# Topic 会话的文件结构
.realm/fiefdoms/finance/
├── sessions/
│   ├── daemon.jsonl              # 守护会话记录
│   └── topics/
│       ├── topic-auth-001.jsonl  # Topic: 用户认证
│       ├── topic-migrate-002.jsonl # Topic: 数据迁移
│       └── topic-bugfix-003.jsonl  # Topic: Bug修复
```

#### 会话路由决策

守护会话根据消息特征决定处理方式：

| 消息类型 | 处理方式 | 示例 |
|----------|----------|------|
| **简单查询** | 守护会话直接处理 | "当前有多少待办任务？" |
| **新任务** | 创建新 Topic 会话 | "设计用户认证模块" |
| **已有任务跟进** | 路由到对应 Topic | "认证模块的进展如何？" |
| **紧急事项** | 中断当前 Topic，优先处理 | "生产环境出现故障" |

```python
# 路由决策逻辑
def route_message(message):
    # 1. 检查是否匹配已有 Topic
    matching_topic = find_topic_by_context(message)
    if matching_topic:
        return delegate_to_topic(matching_topic, message)
    
    # 2. 判断是否需要新建 Topic
    if requires_deep_work(message):
        topic = create_topic_session(message)
        return delegate_to_topic(topic, message)
    
    # 3. 简单查询直接处理
    return process_in_daemon(message)
```

#### Topic 会话生命周期

```
┌─────────────────────────────────────────────────────────────┐
│                    Topic 会话生命周期                        │
│                                                             │
│   创建 ──► 活跃 ──► 休眠 ──► 唤醒 ──► 完成 ──► 归档        │
│    │        │        │        │        │        │          │
│    │        │        │        │        │        └─ 摘要入库 │
│    │        │        │        │        └─ 产物交付          │
│    │        │        │        └─ 收到相关消息               │
│    │        │        └─ 空闲超时 (如 30 分钟)               │
│    │        └─ 正在处理任务                                 │
│    └─ 守护会话派生                                          │
└─────────────────────────────────────────────────────────────┘
```

**休眠触发条件**：
- 空闲超过阈值（默认 30 分钟无交互）
- 等待外部依赖（如等待 Knight 执行结果）
- 主动请求休眠（任务阶段性完成）

**唤醒触发条件**：
- 收到与该 Topic 相关的新消息
- 依赖的任务完成（Knight 返回结果）
- 定时唤醒（如每日 standup）

#### Topic 会话休眠 — 记忆压缩

Topic 会话休眠时执行 **记忆压缩 (Memory Consolidation)**：

```
┌─────────────────────────────────────────────────────────────┐
│                    Phase 1: 短期记忆摘要                     │
│  • 分析本次会话的对话记录                                   │
│  • 提取关键决策、重要发现、待办事项                         │
│  • 生成结构化摘要 summary.json                              │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    Phase 2: 长期记忆归档                     │
│  • 将摘要向量化，存入 VectorDB (Qdrant)                     │
│  • 关联元数据：时间戳、Topic ID、相关文件                   │
│  • 更新 memory/index.json 索引                              │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    Phase 3: 状态持久化                       │
│  • git add . && git commit                                  │
│  • 保留 Topic 会话文件（支持后续唤醒）                      │
│  • 释放内存资源                                             │
└─────────────────────────────────────────────────────────────┘
```

**记忆分层**：

| 层级 | 存储位置 | 内容 | 生命周期 |
|------|----------|------|----------|
| **工作记忆** | Topic 会话内存 | 当前对话的完整上下文 | 会话休眠后释放 |
| **短期记忆** | `sessions/topics/*.jsonl` | Topic 会话的完整记录 | Topic 完成后归档 |
| **长期记忆** | VectorDB + `memory/archive/` | 重要决策、经验教训 | 永久保留 |

**摘要结构**：

```json
{
  "topic_id": "topic-auth-001",
  "topic_name": "用户认证模块设计",
  "status": "in_progress",
  "created_at": "2026-02-01T10:00:00Z",
  "last_active": "2026-02-01T15:30:00Z",
  "summary": "完成 JWT 方案设计，待实现刷新 Token 逻辑",
  "key_decisions": [
    "采用 JWT 方案",
    "Token 有效期设为 24 小时"
  ],
  "pending_items": [
    "实现刷新 Token 逻辑",
    "等待用户确认安全策略"
  ],
  "related_files": ["auth/jwt.py", "config/security.yaml"],
  "tags": ["authentication", "security", "jwt"]
}
```

#### 双层会话的优势

| 优势 | 描述 |
|------|------|
| **即时响应** | 守护会话常驻，消息到达即可响应，无冷启动延迟 |
| **上下文隔离** | 不同 Topic 互不干扰，避免信息污染 |
| **资源高效** | 空闲 Topic 休眠释放资源，守护会话保持轻量 |
| **并行处理** | 可同时处理多个不相关的 Topic |
| **记忆连续** | 守护会话维护 Agent 级记忆，Topic 会话维护任务级记忆 |
| **灵活调度** | 紧急任务可中断低优先级 Topic |

### 3.4 通信的文件系统视角

异步消息通信（Inbox/Outbox）也可以具象化为文件操作：

```
发消息：Lord A 想找 Lord B
├── Lord A 在 .realm/fiefdoms/lord_b/inbox/ 目录下写入 msg_from_A.json
├── 文件系统 watcher 捕获到新增文件
├── 转化为 Redis Stream 信号
└── 触发 Lord B 的"唤醒"
```

### 3.5 这个设计的核心优势

| 优势 | 描述 |
|------|------|
| **极低学习成本** | 创建新领域领主只需创建文件夹，写好 `soul.md` 和 `agent.md`，无需写代码 |
| **绝对可审计性** | 整个王国的运行史就是一部 Git Commit History，可回溯任何时间点的决策上下文 |
| **环境一致性** | 复用 Claude Code 等成熟 Agent 客户端的代码执行、文件操作能力 |
| **无状态性极致** | 只要文件还在（JuiceFS 保证），任何 K8s 节点都能瞬间拉起任何 Lord |
| **记忆连续性** | 通过记忆压缩机制，Lord 可跨会话保持上下文连贯性 |

---

## 4. Core Components (核心组件详解)

### 4.1 治理层：The Royal Couple (王室双星)

治理层由 King 和 Queen 共同构成的"双星系统"，分别负责战略与执行。

#### King Agent (国王)：核心领域

King 是 **核心领域 (Core Domain)** 的 Agent，与 Lord 同构，但职责聚焦于组织治理。

| 属性 | 描述 |
|------|------|
| **本体** | 高级 LLM (如 Claude 3.5 Sonnet / O1) |
| **架构** | 与 Lord 同构：守护会话 + Topic 会话 + Knight 派发能力 |
| **运行模式** | **常驻容器**，守护会话持续运行 |
| **领域定位** | 核心领域 (Core Domain)：组织治理、跨域协调 |
| **文件视野** | `.realm/` (全局视野) |

**King 的核心职能**：

| 职能 | 描述 |
|------|------|
| **消息路由** | 将用户请求路由到合适的 Lord |
| **领域管理** | 创建/拆分/合并业务领域 |
| **资源调度** | 分配 Knight 配额，管理系统资源 |
| **跨域协调** | 主持圆桌会议，仲裁领域争议 |
| **系统演进** | 提出架构变更提案（需人类批准） |
| **Knight 派发** | 执行系统级任务（诊断、配置生成、数据分析等） |

**King 与 Lord 的同构性**：

```
┌─────────────────────────────────────────────────────────────┐
│                    King 容器架构                             │
│                    (与 Lord 同构)                            │
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │              OpenClaw Gateway (常驻)                 │   │
│   │                                                     │   │
│   │  ┌─────────────────────────────────────────────┐   │   │
│   │  │         守护会话 (Daemon Session)            │   │   │
│   │  │  • 监听用户消息                             │   │   │
│   │  │  • 路由到 Lord                              │   │   │
│   │  │  • 管理 Topic 会话                          │   │   │
│   │  └─────────────────────────────────────────────┘   │   │
│   │                         │                           │   │
│   │           ┌─────────────┼─────────────┐             │   │
│   │           ▼             ▼             ▼             │   │
│   │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐   │   │
│   │  │ Topic: 领域 │ │ Topic: 资源 │ │ Topic: 会议 │   │   │
│   │  │ 规划       │ │ 调度       │ │ 主持       │   │   │
│   │  └─────────────┘ └─────────────┘ └─────────────┘   │   │
│   │                         │                           │   │
│   │                         ▼                           │   │
│   │              ┌─────────────────────┐                │   │
│   │              │  Knight (Subagent)  │                │   │
│   │              │  系统诊断/配置生成   │                │   │
│   │              └─────────────────────┘                │   │
│   └─────────────────────────────────────────────────────┘   │
│                                                             │
│   挂载卷:                                                   │
│   .realm/ → /app/workspace/ (全局视野)                     │
└─────────────────────────────────────────────────────────────┘
```

#### Queen Agent (皇后)：编排与律法

| 属性 | 描述 |
|------|------|
| **本体** | 高性能系统服务（Go / Rust / Python 编写的逻辑代码 + 轻量级监督 AI） |
| **形态** | 王国的"心脏"，极低延迟的常驻守护进程 |
| **核心职能** | 编排调度、资源管控、环境构建、消息投递、安全沙盒 |

**Queen 的核心职能**：

| 职能 | 描述 |
|------|------|
| **容器管理** | 管理 King/Lord/Knight 容器的生命周期 |
| **沙箱编排** | 为 Knight 创建和销毁沙箱环境 |
| **禁卫军管理 (Resource Guard)** | 严格执行配额，超支的 Knight 立即熔断销毁 |
| **信息投递** | 负责消息在 Inbox/Outbox 间的物理搬运 |
| **记忆索引** | 将会话摘要向量化存入 VectorDB |

```
┌─────────────────────────────────────────────────────────────┐
│                    The Royal Couple                          │
│                                                             │
│   ┌─────────────────────┐    ┌─────────────────────┐       │
│   │    👑 King          │    │    👸 Queen         │       │
│   │    (核心领域 Agent) │    │    (系统服务)       │       │
│   │                     │    │                     │       │
│   │  • 守护会话         │    │  • 容器管理         │       │
│   │  • Topic 会话       │    │  • 沙箱编排         │       │
│   │  • Knight 派发      │    │  • 资源管控         │       │
│   │  • 领域管理         │    │  • 消息投递         │       │
│   │  • 跨域协调         │    │  • 记忆索引         │       │
│   └─────────────────────┘    └─────────────────────┘       │
│              ↓                         ↓                    │
│         意志与决策                执行与保障                │
└─────────────────────────────────────────────────────────────┘
```

#### Unified Address Book (统一通讯录)

| 属性 | 描述 |
|------|------|
| **技术** | 图数据库 (Graph DB) |
| **内容** | 存储所有 Agent (King/Lord) 和 Human (Staff) 的节点 |
| **关系** | 定义了 REPORTS_TO (汇报), DEPENDS_ON (依赖), CAN_CONTACT (可联系) 等边，用于权限控制和路由 |

---

### 4.2 领域层：Lords with Daemon Sessions (守护会话领主)

系统的"中层管理者"，采用双层会话模型实现"常驻响应 + 任务隔离"。

#### Lord 定义

Lord 是一个 **常驻的 OpenClaw 容器**，包含：
- 守护会话（Daemon Session）：常驻运行，处理消息路由
- Topic 会话池：按需启动的任务会话
- 身份配置：SOUL.md, AGENTS.md, skills/
- 领域记忆：memory/, sessions/

#### 运行时机制

| 组件 | 描述 |
|------|------|
| **守护会话** | 常驻运行，监听 inbox，路由消息，管理 Topic 会话 |
| **Topic 会话** | 按需启动，处理具体任务，完成后休眠或归档 |
| **记忆系统** | 三层记忆：工作记忆 → 短期记忆 → 长期记忆 |

```
┌─────────────────────────────────────────────────────────────┐
│                    Lord 容器架构                             │
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │              OpenClaw Gateway (常驻)                 │   │
│   │                                                     │   │
│   │  ┌─────────────────────────────────────────────┐   │   │
│   │  │         守护会话 (Daemon Session)            │   │   │
│   │  │  • 监听 inbox                               │   │   │
│   │  │  • 消息路由                                 │   │   │
│   │  │  • Topic 管理                               │   │   │
│   │  └─────────────────────────────────────────────┘   │   │
│   │                         │                           │   │
│   │           ┌─────────────┼─────────────┐             │   │
│   │           ▼             ▼             ▼             │   │
│   │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐   │   │
│   │  │  Topic A    │ │  Topic B    │ │  Topic C    │   │   │
│   │  │  (活跃)     │ │  (休眠)     │ │  (活跃)     │   │   │
│   │  └─────────────┘ └─────────────┘ └─────────────┘   │   │
│   │                                                     │   │
│   └─────────────────────────────────────────────────────┘   │
│                                                             │
│   挂载卷:                                                   │
│   .realm/fiefdoms/{domain}/ → /app/workspace/              │
│   .realm/academy/ → /app/academy/ (只读)                   │
└─────────────────────────────────────────────────────────────┘
```

#### 消息处理流程

```
1. 消息到达 inbox (Redis Streams)
2. 守护会话接收消息
3. 路由决策：
   ├── 简单查询 → 守护会话直接处理
   ├── 已有 Topic → 路由到对应 Topic（必要时唤醒）
   └── 新任务 → 创建新 Topic 会话
4. Topic 会话处理任务
5. 结果返回守护会话
6. 守护会话发送响应到 outbox
```

#### 唤醒触发源

| 触发方式 | 描述 |
|----------|------|
| **用户直连** | 用户通过消息通道直接联系 Lord |
| **King 路由** | King 将任务路由到对应 Lord |
| **Lord 间通信** | 其他 Lord 发送协作请求 |
| **定时任务** | Cron 触发的定期任务 |

#### 价值

- **即时响应**：守护会话常驻，无冷启动延迟
- **任务隔离**：Topic 会话独立，上下文不污染
- **资源高效**：空闲 Topic 休眠，释放内存
- **并行处理**：多 Topic 可同时活跃
- **记忆连续**：跨会话保持上下文连贯性

#### 文件操作权限

- **同领域文件**：Lord 对自己领域的文件具有完整操作权
- **跨领域文件**：如需操作其他领域的文件，必须请求对方领域的 Lord 协助

#### 领域边界原则

基于 **领域驱动设计 (DDD)** 进行业务领域划分，遵循以下原则：

| 原则 | 描述 |
|------|------|
| **第一责任人** | 模糊地带由首次接手的 Lord 负责，除非明确转交给其他领域 Lord |
| **明确转交** | Lord 可将任务转交给更合适的领域，但必须显式声明 |
| **事后优化** | Lord 有权在事后向 King 或圆桌会议提出领域权责优化建议 |
| **业务导向** | 划分基于业务领域而非技术栈（如 Tech 和 Data 若服务同一业务则属同一领域） |

---

### 4.3 协同层：The Roundtable (圆桌会议)

系统的"议事厅"。

#### 会议机制

| 属性 | 描述 |
|------|------|
| **基础** | 基于 "议程 (Agenda)" 的结构化多方会话 |
| **参与者** | 仅限 King 和 **Lords**，Knight 无权入内 |
| **议长 (Chairperson)** | 专门的低延迟模型 (Moderator) |

#### 议长职责

1. 维持会议秩序
2. 分配发言权
3. **信息熵检测**：检测会议内容是否保持有效性
4. 检测死循环
5. 总结 **会议纪要 (Minutes)**

#### 会议触发机制 (MVP)

| 机制 | 描述 |
|------|------|
| **定时触发** | 圆桌会议每天定时触发一次，时间由系统管理员预设 |
| **议题申报** | Lord 和 King 拥有"申报议题"技能，可在会议前提交当天议题 |
| **议程生成** | Chairperson 汇总所有申报议题，生成当天会议议程 |

#### 死锁风险防控

| 策略 | 描述 |
|------|------|
| **信息熵检测** | 对话信息熵低于阈值时，立即停止当前议题 |
| **限流机制** | 限制 Lord 频繁交流的频率 |
| **精简输出** | 要求 Lord 只输出核心要点，避免冗余 |

#### 会议流程 (MVP)

MVP 阶段采用结构化、无自由讨论的会议模式，按规章推进：

| 阶段 | 执行者 | 描述 |
|------|--------|------|
| 1. 议程宣读 | Chairperson | 宣布本次会议议题列表、每个议题的发言 Lord 及发言顺序 |
| 2. 轮流发言 | 发言 Lords | 按顺序进行 **3 轮** 轮流发言，每轮每位 Lord 发言一次 |
| 3. 组织投票 | Chairperson | 发言结束后，组织 **非发言 Lord** 进行投票表决 |
| 4. 议题总结 | Chairperson | 汇总发言要点和投票结果，整理为会议纪要文档 |
| 5. 人类终审 | Admin | 重大决策需人类管理员最终确认 |
| 6. 下一议题 | Chairperson | 重复步骤 1-5，直至所有议题处理完毕 |

**设计原则**：
- 无自由讨论环节，避免会议失控和 token 消耗过大
- 固定 3 轮发言，确保各方充分表达
- 非发言方投票，保证决策的客观性

---

### 4.4 执行层：Knights as Sandbox Subagents (沙箱子代理)

Knight 是运行在 **沙箱环境** 中的 **Subagent**，由 King 或 Lord 按需派发，执行原子任务。

#### Knight 的本质

| 属性 | 描述 |
|------|------|
| **本质** | 沙箱化的 Subagent，由父 Agent (King/Lord) 派发 |
| **运行环境** | Docker 容器 / Kata Containers (安全沙盒) |
| **生命周期** | 用完即毁，无持久记忆 |
| **能力** | 运行代码、CLI 工具、文件操作、爬虫 |
| **通信** | 通过 Stdio Tunnel 与父 Agent 实时交互 |

```
┌─────────────────────────────────────────────────────────────┐
│                    Knight 作为 Subagent                      │
│                                                             │
│   ┌─────────────────────┐    ┌─────────────────────┐       │
│   │    👑 King          │    │    🏰 Lord          │       │
│   │    (父 Agent)       │    │    (父 Agent)       │       │
│   │                     │    │                     │       │
│   │  派发 Knight 场景:  │    │  派发 Knight 场景:  │       │
│   │  • 系统诊断         │    │  • 代码编写         │       │
│   │  • 配置生成         │    │  • 文件处理         │       │
│   │  • 数据分析         │    │  • 测试执行         │       │
│   │  • 文档生成         │    │  • 部署操作         │       │
│   └──────────┬──────────┘    └──────────┬──────────┘       │
│              │                          │                   │
│              └────────────┬─────────────┘                   │
│                           ▼                                 │
│              ┌─────────────────────────────────────┐        │
│              │         ⚔️ Knight (Subagent)        │        │
│              │                                     │        │
│              │  ┌─────────────────────────────┐   │        │
│              │  │      沙箱环境 (Sandbox)      │   │        │
│              │  │                             │   │        │
│              │  │  • 隔离的文件系统           │   │        │
│              │  │  • 受限的网络访问           │   │        │
│              │  │  • 资源配额限制             │   │        │
│              │  │  • 命令审批机制             │   │        │
│              │  └─────────────────────────────┘   │        │
│              │                                     │        │
│              │  输入: task.md (任务描述)           │        │
│              │  输出: output/ (执行产物)           │        │
│              │  状态: result.json (执行结果)       │        │
│              └─────────────────────────────────────┘        │
└─────────────────────────────────────────────────────────────┘
```

#### King 派发 Knight 的场景

King 作为核心领域，也需要执行一些具体任务：

| 场景 | 描述 |
|------|------|
| **系统诊断** | 检查系统健康状态、资源使用情况 |
| **配置生成** | 为新领域生成初始配置文件 |
| **数据分析** | 分析跨领域数据，生成报告 |
| **文档生成** | 生成系统文档、会议纪要 |
| **Address Book 维护** | 执行通讯录的批量更新操作 |

```python
# King 派发 Knight 示例
def king_dispatch_knight(task: str):
    workshop_path = f'.realm/crown/king/workshops/{task_id}'
    
    # 创建任务目录
    os.makedirs(workshop_path)
    
    # 写入任务描述
    write_file(f'{workshop_path}/task.md', task)
    
    # 启动 Knight 沙箱
    knight = spawn_knight_sandbox(
        workspace=workshop_path,
        parent='king',
        timeout=300,  # 5 分钟超时
    )
    
    # 等待执行完成
    result = knight.wait()
    return result
```

#### Lord 派发 Knight 的场景

Lord 作为业务领域，主要派发执行具体业务任务的 Knight：

| 场景 | 描述 |
|------|------|
| **代码编写** | 根据 Spec 编写代码 |
| **文件处理** | 文件转换、数据清洗 |
| **测试执行** | 运行单元测试、集成测试 |
| **部署操作** | 执行部署脚本 |
| **爬虫任务** | 抓取外部数据 |

#### Spec 驱动执行模式

Knight 的无状态设计通过 **Spec 驱动** 模式保证执行质量：

```
┌─────────────────────────────────────────────────────────────┐
│                    父 Agent 规划阶段                         │
│  1. 接收任务/消息                                           │
│  2. 制定计划 (Plan)                                          │
│  3. 设计方案 (Design)                                        │
│  4. 拆解为任务清单 (Task List)                               │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    Knight 执行阶段                           │
│  1. 载入：任务描述 + 技能包 + 工具                           │
│  2. 执行：任务清单中的单个原子任务                           │
│  3. 交付：向父 Agent 提交执行产物                            │
│  ⚠️ 无需知道整体上下文，只关注当前原子任务                   │
└─────────────────────────────────────────────────────────────┘
```

**核心要点**：
- 父 Agent (King/Lord) 负责"想清楚"，Knight 负责"做出来"
- 每个 Knight 任务是自包含的原子操作
- 通过任务描述文件传递必要信息，而非依赖记忆

#### Knight 任务生命周期

| 阶段 | 描述 |
|------|------|
| **派发** | 父 Agent 创建 workshop 目录，写入 task.md |
| **启动** | Queen 启动 Knight 沙箱容器 |
| **执行** | Knight 在沙盒中执行任务（代码运行、文件操作等） |
| **成功** | 交付产物给父 Agent（写入 output/ 目录） |
| **失败处理** | 上报父 Agent，由父 Agent 决定：继续 / 重试 / 跳过 |
| **升级** | 父 Agent 无法抉择时，询问用户或上级 |
| **销毁** | 任务完成后，沙箱容器自动销毁 |

**通信方式**：父 Agent → Knight 采用 **Sandbox 实时交互**（Stdio Tunnel），无需消息队列

#### Knight 沙箱安全机制

| 机制 | 描述 |
|------|------|
| **文件系统隔离** | 只能访问当前 workshop 目录 |
| **网络限制** | 可配置的网络访问白名单 |
| **资源配额** | CPU、内存、执行时间限制 |
| **命令审批** | 危险命令需要审批或被拦截 |
| **只读挂载** | 父 Agent 的配置文件只读挂载 |

```json
// Knight 沙箱配置示例
{
  "sandbox": {
    "image": "openclaw/knight:latest",
    "resources": {
      "cpu": "1",
      "memory": "512Mi",
      "timeout": 300
    },
    "network": {
      "mode": "restricted",
      "allowlist": ["github.com", "npmjs.org"]
    },
    "filesystem": {
      "workspace": "rw",
      "parent_config": "ro",
      "host": "none"
    }
  }
}
```

#### Knight 配额管理

| 场景 | 处理方式 |
|------|----------|
| **初始分配** | 领域创建时，King 为每个 Agent 分配固定 Knight 配额 |
| **配额不足** | Agent 向 King 提交配额申请 |
| **申请评估** | King 进行评估（如任务积压量、资源利用率等） |
| **人工确认** | 评估通过后，由人类进行最终确认 |

#### Shared FS

| 属性 | 描述 |
|------|------|
| **技术** | JuiceFS + MinIO (数据) + Redis (元数据) |
| **特性** | 实现了主容器与任务容器的 **同节点亲和性挂载**，解决依赖复用和冷启动问题 |

---

### 4.5 通信层：Inbox/Outbox 机制

Agent 间采用异步消息通信，实现解耦和可靠投递。

#### 通信架构

```
┌─────────────────────────────────────────────────────────────┐
│                    异步消息层 (Redis Streams)                │
│                                                             │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    │
│   │ king:inbox  │    │lord:A:inbox │    │lord:B:inbox │    │
│   └─────────────┘    └─────────────┘    └─────────────┘    │
│         ▲                  ▲                  ▲             │
│         │                  │                  │             │
└─────────┼──────────────────┼──────────────────┼─────────────┘
          │                  │                  │
     ┌────┴────┐        ┌────┴────┐        ┌────┴────┐
     │  King   │◄──────►│ Lord A  │◄──────►│ Lord B  │
     └─────────┘        └────┬────┘        └─────────┘
                             │ Sandbox (同步)
                             ▼
                        ┌─────────┐
                        │ Knight  │
                        └─────────┘
```

#### 通信模式

| 通信路径 | 模式 | 技术 |
|----------|------|------|
| **King ↔ Lord** | 异步消息 | Redis Streams (inbox/outbox) |
| **Lord ↔ Lord** | 异步消息 | Redis Streams (inbox/outbox) |
| **Lord → Knight** | 同步调用 | Sandbox 实时交互 (Stdio Tunnel) |

#### 消息结构

```json
{
  "id": "msg_uuid",
  "from": "lord:finance",
  "to": "king",
  "type": "request_quota | motion | task_result | alert | ...",
  "payload": { ... },
  "timestamp": "2026-02-01T10:00:00Z",
  "reply_to": "lord:finance:inbox"
}
```

#### Agent 技能

每个 Agent 拥有消息相关技能：
- **King**：路由消息、处理配额申请、处理议题申报
- **Lord**：给 King 留言、给其他 Lord 留言、申报圆桌议题

---

### 4.6 交互层：Human Interface & Policy

碳基与硅基的连接点。

#### Policy Engine (策略引擎)

| 属性 | 描述 |
|------|------|
| **职责** | 防止 AI 骚扰人类 |
| **规则** | 基于 **严重等级 (Severity)**、**时间 (Time)**、**频次 (Rate)** 进行过滤 |
| **示例** | "非 P0 级故障，禁止在 22:00-08:00 发送短信" |

#### Deep Link (作战室链接)

当 AI 发送警报时，附带一个 URL。点击后直接进入 IDE 的特定 Session，上下文已包含错误现场，实现即时 **Human-in-the-loop**。

---

### 4.7 知识层：The Academy (学院)

系统的"智慧图书馆"，用于**共享和持久化**高价值的组织知识。

#### 学院定位

| 属性 | 描述 |
|------|------|
| **定位** | 跨领域知识共享中心，所有 Lord 的共同资产 |
| **核心价值** | 避免重复造轮子，沉淀组织智慧，支持快速学习 |
| **访问权限** | 所有 Lord 可读取，King 管理写入权限 |

#### 三大支柱

| 支柱 | 描述 | 内容类型 |
|------|------|----------|
| **Knowledge Base (知识库)** | 高价值领域知识、SOP、经验总结 | 文档、案例、最佳实践 |
| **Skill Library (技能库)** | 可复用的 Agent 技能包 | Prompt 模板、推理策略 |
| **Tool Registry (工具注册表)** | MCP 工具的标准化注册与发现 | 工具元数据、接口定义 |

#### 知识入库流程

| 阶段 | 执行者 | 描述 |
|------|--------|------|
| 1 | Lord | 在领域实践中发现可复用的知识/技能/工具 |
| 2 | Lord | 向 Academy 提交入库申请 (包含价值评估) |
| 3 | King | 审核内容的质量和适用性 |
| 4 | Academy | 通过审核后持久化存储，建立索引 |
| 5 | 人类终审 | 高价值内容需人类管理员确认 |

#### 学院访问机制

| 场景 | 描述 |
|------|------|
| **任务执行** | Lord 在执行任务时，可从 Academy 获取相关 SOP 和工具 |
| **新领域创建** | 新 Lord 初始化时，可从 Academy 加载通用知识和技能 |
| **跨域协作** | 圆桌会议中，引用 Academy 中的共享知识作为决策依据 |
| **持续学习** | Academy 支持 RAG 查询，Lord 可通过对话获取知识 |

#### 版本管理

所有知识存储在 **Git 仓库** 中，实现版本控制和协作：

| 机制 | 描述 |
|------|------|
| **Git 版本控制** | 所有知识文档纳入 Git 管理，支持历史追溯和回滚 |
| **元数据标注** | 每份知识包含创建时间、更新时间、贡献者等元数据 |
| **废弃标记** | 过时知识通过元数据标记废弃状态，但保留历史记录 |
| **演化记录** | 通过 Git commit 记录知识的修改历史和评审过程 |

#### 技术选型 (MVP)

| 组件 | 选型 | 理由 |
|------|------|------|
| **向量数据库** | Qdrant | 轻量级、易部署、性能优秀 |
| **知识存储** | Git 仓库 | 版本控制、协作友好、审计追溯 |
| **索引粒度** | 段落级 | 平衡检索精度和召回率 |

---

## 5. Domain Evolution (领域演进机制)

### 5.1 领域拆分 (场景一)

**触发条件**：Lord 任务自己的领域混杂了其他领域的内容，不纯粹

| 阶段 | 执行者 | 描述 |
|------|--------|------|
| 1 | Lord | 向 King 提议进行领域拆分 |
| 2 | King | 起草拆分方案 |
| 3 | 相关 Lord | Review 拆分方案 |
| 4 | 达成一致 | 进入圆桌会议 |
| 5 | 圆桌投票 | 各 Lord 投票 |
| 6 | 人类终审 | 系统管理员最终确认 |
| 7 | 生效 | 执行领域拆分，更新 Address Book |

### 5.2 领域合并 (场景二)

**触发条件**：King 发现领域 A 与领域 B 相关性很高，经常频繁跨域合作

| 阶段 | 执行者 | 描述 |
|------|--------|------|
| 1 | King | 向两个领域提出合并建议 |
| 2 | King | 起草合并方案 |
| 3 | 领域 A/B Lord | Review 合并方案 |
| 4 | 达成一致 | 进入圆桌会议 |
| 5 | 圆桌投票 | 各 Lord 投票 |
| 6 | 人类终审 | 系统管理员最终确认 |
| 7 | 生效 | 执行领域合并，更新 Address Book |

### 5.3 Review 不合时的处理

若 Lord Review 阶段双方意见不合：
1. **自动调解**：由 Chairperson 进行初步调解
2. **圆桌投票**：调解无效则上圆桌投票
3. **人类终审**：投票结果仍需人类确认

---

## 6. Key Workflows (关键业务流)

### 流程一：从文档到组织 (System Genesis)

```
1. Admin 上传《公司业务白皮书》
2. King 分析文档，生成 Domain_Map_v1.json (含 Sales, Tech, HR 领域定义)
3. Admin 在 Dashboard 审查并点击 `Approve`
4. System 初始化 Address Book，为每个领域创建 Lord Profile 和 RAG 索引
```

### 流程二：圆桌协同 (The Roundtable)

```
1. Sales Lord 发现 Q4 目标无法完成，发起动议：`[Motion: Request Tech Support for Campaign]`
2. Chairperson 召集 Sales Lord 和 Tech Lord 入席圆桌
3. 对话:
   - Sales: "我们需要在双十一上线新落地页。"
   - Tech: "需要增加 2 个 Knight 资源用于开发。"
4. 信息熵检测通过， Chairperson 生成 `[Resolution: Allocate Resources]`
5. Tech Lord 领回任务，唤醒 Knight 开始写代码
6. 人类管理员确认配额申请
```

### 流程三：吹哨人机制 (Whistleblower)

```
1. Finance Lord 派遣 Knight 进行例行审计
2. Knight 发现异常资金流出
3. Finance Lord 评估严重性为 `CRITICAL`
4. Policy Engine 批准最高优先级打断
5. Notification Gateway 呼叫 CFO 手机，并发送 Slack 消息
6. CFO 点击链接，进入 "Finance Audit Session"，与 Agent 协同查账
```

### 流程四：领域拆分 (Domain Split)

```
1. Tech Lord 发现任务混杂了 Data Analysis 内容
2. Tech Lord 向 King 提议领域拆分
3. King 分析并起草拆分方案（含 Tech 和 Data Analysis 边界定义）
4. Data Analysis Lord Review 方案
5. 达成一致，圆桌会议投票
6. 人类管理员批准
7. Address Book 更新，创建新的 Data Analysis Lord
```

---

## 7. Implementation Strategy (实施策略)

### Phase 1: 基础建设 (The Foundation)

| 重点 | 里程碑 |
|------|--------|
| K8s/Kata/JuiceFS 底座 | 基础设施就绪 |
| Knight 沙盒 | 安全执行环境完成 |
| 基础指令流 | King → Lord → Knight 单向路由 |

**MVP 阶段暂不考虑成本预估**

### Phase 2: 组织互联 (The Organization)

| 重点 | 里程碑 |
|------|--------|
| Address Book | 统一通讯录上线 |
| Stateless Runtime | Hydration/Dehydration 机制完成 |
| Policy Engine | 主动报警功能上线 |
| Academy 基础架构 | 知识库、技能库、工具注册表基础框架 |

### Phase 2.5: 知识沉淀 (Knowledge Consolidation)

| 重点 | 里程碑 |
|------|--------|
| 知识入库流程 | Lord 可提交知识，King 审核 |
| 知识检索 | 基于 RAG 的跨领域知识查询 |
| 技能共享 | 通用技能库支持多 Lord 复用 |

### Phase 3: 智慧涌现 (The Intelligence)

| 重点 | 里程碑 |
|------|--------|
| Roundtable 机制 | 多 Lord 协同解决问题 |
| King 自动 DDD | 动态领域划分能力 |
| Moderator | 会议主持与死锁检测 |

---

## 8. Decision Workflow (决策流程)

### 核心原则：AI自治 → 圆桌共识 → 人类终审

```
┌─────────────────────────────────────────────────────────────┐
│                    AI 自治 (Agent Autonomy)                  │
│  • Lord/King 自主决策日常事务                              │
│  • 基于领域知识和历史经验做出判断                            │
│  • 记录决策过程供后续审计                                    │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                  圆桌共识 (Roundtable Consensus)             │
│  • 重大变更需圆桌投票                                        │
│  • Chairperson 主持，确保流程公正                            │
│  • 信息熵检测保证讨论质量                                    │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                  人类终审 (Human Approval)                   │
│  • 架构变更、配额调整等关键决策需人类确认                    │
│  • 守住了安全、合规的底线                                    │
│  • 可一键否决或要求修改                                      │
└─────────────────────────────────────────────────────────────┘
```

---

## 9. Risk Mitigation (风险缓解)

| 风险 | 缓解措施 |
|------|----------|
| 领域碎片化 | King 定期评估，必要时主动发起合并 |
| 会议死锁 | 信息熵检测 + 限流 + 精简输出 |
| 权限越界 | GraphDB 边关系控制 + 跨域请求机制 |
| 资源争抢 | 配额制 + King 统一调度 |
| AI 骚扰 | Policy Engine 多维过滤 |
| 知识孤岛 | Academy 强制共享高价值知识 |
| 知识过时 | 版本管理 + 废弃标记机制 |
| 记忆丢失 | 记忆压缩 + VectorDB 持久化 + Git 归档 |

---

## 10. Conclusion (总结)

这套架构将 Cloud IDE 升级为了 **Enterprise OS**：

| 特性 | 价值 |
|------|------|
| 身份即目录 (Identity-as-Context) | Agent 本质是配置化上下文，极低学习成本，绝对可审计 |
| 同构架构 (Isomorphic Architecture) | King 与 Lord 同构，统一技术栈，代码复用，灵活扩展 |
| DDD 领域分层 | King 是核心领域，Lord 是业务领域，职责清晰 |
| 双层会话模型 | 守护会话常驻响应 + Topic 会话任务隔离 |
| Knight 沙箱 Subagent | King/Lord 都可派发，安全隔离，用完即毁 |
| 王室双星 (Royal Couple) | King 负责战略，Queen 负责执行，职责分明 |
| 圆桌会议 | 解决了 AI 协作无序问题 |
| 人类内阁 + 策略引擎 | 解决了 AI 的安全与合规问题 |
| 学院 (Academy) | 实现了跨领域知识共享，避免重复造轮子 |
| AI自治→圆桌共识→人类终审 | 平衡效率与安全的治理范式 |

---

**这是一个具备 商业壁垒、技术深度 且 符合直觉 的顶级架构设计。**

---

*Document Version History*
- v2.0: Initial architecture design
- v2.1: Added domain evolution mechanism, meeting deadlock prevention, resource quota management, and decision workflow refinement
- v2.2: Added The Academy for cross-domain knowledge sharing (Knowledge Base, Skill Library, Tool Registry)
- v2.3: Renamed to Silicon Realm
- v2.4: MVP refinements - structured roundtable flow (3-round speaking + voting), first-responsible-party principle for domain boundaries, spec-driven Knight execution model, Git-based Academy version control
- v2.5: Communication architecture - King as resident process, inbox/outbox mechanism (Redis Streams), Lord hydration triggers, Knight task lifecycle, daily roundtable scheduling, Qdrant for Academy
- v2.6: Core philosophy upgrade - Identity-as-Context (Agent = Files + Engine), The Royal Couple (King for strategy, Queen for orchestration), file system as the source of truth, Git-based state management
- v2.7: Memory Consolidation mechanism - three-tier memory (working/short-term/long-term), session summarization, VectorDB archival, progressive hydration with dynamic context loading
- v2.8: Dual-Session Model - Daemon Session (resident) + Topic Sessions (on-demand) for King and Lord, replacing stateless runtime with hybrid approach for instant response and context isolation
- v2.9: Isomorphic Architecture - King and Lord are structurally identical (same dual-session model + Knight dispatch capability), differentiated only by domain responsibility (Core Domain vs Business Domain from DDD perspective); Knight redefined as Sandbox Subagent that can be dispatched by both King and Lord
