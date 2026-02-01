# Architecture Design Document: Silicon Realm
## Enterprise-Grade Agentic Organization System

**Version:** 2.6 (Final, Revised)  
**Date:** 2026-02-01  
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
| **无状态运行时 (Stateless Runtime)** | 领主按需唤醒 (Hydration)，用完即走，极低资源占用 |
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

### 3.2 三级角色的本质差异：权限与视角

既然"本体"相同，那么 King、Lord、Knight 的区别在于 **文件系统的访问视野**：

| 角色 | 挂载的根目录 (Scope) | 核心能力 (Capabilities) |
|------|---------------------|------------------------|
| **King** | `.realm/` (全局视野) | 创建新目录（新领域）、修改 Address Book、调用圆桌议长 |
| **Lord** | `.realm/fiefdoms/{domain_id}/` | 读取 Academy 知识、派发任务给 Knight、给 King 发送异步消息 |
| **Knight** | `.realm/fiefdoms/{domain_id}/workshops/{task_id}/` | 极窄视野，只能操作当前任务文件夹，调用受限的本地 CLI 工具 |

### 3.3 唤醒与休眠的具体实现

有了文件系统的视角，原本抽象的运行时机制变得极其具体：

#### 唤醒 (Hydration) — 渐进式加载

采用 **渐进式加载 + 动态上下文** 策略，保证上下文的针对性和纯洁：

```
┌─────────────────────────────────────────────────────────────┐
│                    Phase 1: 基础身份加载                     │
│  • soul.md — 人格与沟通风格                                 │
│  • agent.md — 职责范围与 SOP                                │
│  • rules.md — 合规性约束                                    │
│  • tools/ — 工具配置（由 Agent 框架自动加载）               │
│  ⚠️ 此时 Agent 只知道"我是谁"，不知道"要做什么"            │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    Phase 2: 任务上下文注入                   │
│  • 接收具体任务/消息                                        │
│  • 根据任务内容，Agent 主动检索相关文件                     │
│  • 按需加载：历史记录、相关文档、依赖信息                   │
│  ⚠️ 只加载与当前任务相关的上下文，避免信息污染              │
└─────────────────────────────────────────────────────────────┘
```

**核心原则**：
- **最小化初始上下文**：启动时只加载身份定义，不预加载业务数据
- **按需检索**：Agent 根据任务自主决定需要哪些额外上下文
- **上下文纯洁性**：避免无关信息干扰决策质量

```bash
# 1. Orchestrator（Queen）收到 Redis Streams 消息
# 2. 切换到对应 Agent 目录
cd .realm/fiefdoms/finance_lord/

# 3. 启动 Agent 引擎，仅注入基础身份
CLI_ENGINE --context=./soul.md,./agent.md,./rules.md --input="新消息内容"

# 4. Agent 启动后，根据任务内容自主检索
#    - 查询 Academy 获取相关 SOP
#    - 读取 memory/ 中的历史上下文
#    - 加载 workspace/ 中的相关文件
```

#### 思考与执行

```bash
# Agent 在该目录下工作
# - 读取 task.md
# - 编写代码或调用 MCP
# - 所有中间产物、临时思考写在 memory.log 或 workspace/ 中
```

#### 休眠 (Dehydration) — 记忆压缩

休眠不仅仅是保存状态，还需要对短期记忆进行 **压缩整理 (Memory Consolidation)**：

```
┌─────────────────────────────────────────────────────────────┐
│                    Phase 1: 短期记忆摘要                     │
│  • 分析本次会话的 memory.log                                │
│  • 提取关键决策、重要发现、待办事项                         │
│  • 生成结构化摘要 summary.json                              │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    Phase 2: 长期记忆归档                     │
│  • 将摘要向量化，存入 VectorDB (Qdrant)                     │
│  • 关联元数据：时间戳、任务ID、相关文件                     │
│  • 更新 memory/index.json 索引                              │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    Phase 3: 状态持久化                       │
│  • git add . && git commit                                  │
│  • 清理临时文件（保留摘要）                                 │
│  • 销毁进程/容器                                            │
└─────────────────────────────────────────────────────────────┘
```

**记忆分层**：

| 层级 | 存储位置 | 内容 | 生命周期 |
|------|----------|------|----------|
| **工作记忆** | `memory.log` | 当前会话的完整对话和思考 | 会话结束后清理 |
| **短期记忆** | `memory/sessions/{date}/` | 每次会话的摘要 | 保留 7 天 |
| **长期记忆** | VectorDB + `memory/archive/` | 重要决策、经验教训 | 永久保留 |

**摘要结构**：

```json
{
  "session_id": "2026-02-01-001",
  "timestamp": "2026-02-01T10:30:00Z",
  "task_summary": "完成用户认证模块的设计",
  "key_decisions": [
    "采用 JWT 方案",
    "Token 有效期设为 24 小时"
  ],
  "discoveries": [
    "现有代码库已有 bcrypt 依赖"
  ],
  "pending_items": [
    "等待用户确认刷新 Token 策略"
  ],
  "related_files": ["auth/jwt.py", "config/security.yaml"],
  "tags": ["authentication", "security", "jwt"]
}
```

```bash
# 休眠流程
# 1. 生成会话摘要
agent summarize --output=memory/sessions/2026-02-01/session_001.json

# 2. 向量化存入 Qdrant
queen index-memory --session=session_001.json --collection=lord_finance

# 3. Git 提交
git add . && git commit -m "Session 001: 完成用户认证模块设计"

# 4. 清理工作记忆
rm memory.log

# 5. 销毁进程
```

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

#### King Agent (国王)：意志与战略

| 属性 | 描述 |
|------|------|
| **本体** | 高级 LLM (如 Claude 3.5 Sonnet / O1) |
| **形态** | 常驻的高层思维进程 |
| **运行模式** | **常驻进程**，作为用户的第一接口人 |
| **职责** | 消息路由、领域管理、资源管理、系统初始化 (Genesis) 和演进 (Evolution) |
| **能力** | 基于 RAG 阅读企业文档，进行 **领域驱动设计 (DDD)**，划分领域边界 |
| **提案机制** | 生成的领域变更（拆分/合并）仅作为 **提案 (Proposal)**，必须经由 **人类内阁 (The Cabinet)** 在 Dashboard 上批准后生效 |
| **资源管理** | 统一分配和管理 Knight 配额，为每个领域分配固定配额 |
| **路由能力** | 用户不清楚找哪个 Lord 时，可通过 King 进行智能路由 |

**King 的核心职能**：
- **解读愿景**：将人类的模糊意图转化为王国的领域布局
- **外交与决策**：主持圆桌会议，平息领主间的争议
- **规则制定**：编写和修订各领域的 `rules.md`
- **审批建议**：审批 Queen 提交的系统级资源预警

#### Queen Agent (皇后)：编排与律法

| 属性 | 描述 |
|------|------|
| **本体** | 高性能系统服务（Go / Rust / Python 编写的逻辑代码 + 轻量级监督 AI） |
| **形态** | 王国的"心脏"，极低延迟的常驻守护进程 |
| **核心职能** | 编排调度、资源管控、环境构建、消息投递、安全沙盒 |

**Queen 的核心职能**：

| 职能 | 描述 |
|------|------|
| **复水与脱水 (Hydration/Dehydration)** | 唯一拥有"唤醒咒语"的角色，监听 Redis Streams 或文件系统，根据信号拉起对应的 CLI 进程 |
| **禁卫军管理 (Resource Guard)** | 严格执行 King 设定的配额，超支的 Knight 立即熔断销毁 |
| **环境构建** | 根据 `agent.md` 和 `soul.md` 动态拼接 Prompt，为领主们准备好"意识环境" |
| **信息投递** | 负责消息在 Inbox/Outbox 间的物理搬运，确保异步通信的可靠性 |
| **安全沙盒** | 管理 Kata Containers，确保每一个 Knight 都在隔离的"车间"里干活 |

```
┌─────────────────────────────────────────────────────────────┐
│                    The Royal Couple                          │
│                                                             │
│   ┌─────────────────────┐    ┌─────────────────────┐       │
│   │    👑 King          │    │    👸 Queen         │       │
│   │    (The Soul)       │    │    (The Engine)     │       │
│   │                     │    │                     │       │
│   │  • 战略规划         │    │  • 编排调度         │       │
│   │  • 领域设计         │    │  • 资源管控         │       │
│   │  • 规则制定         │    │  • 环境构建         │       │
│   │  • 争议仲裁         │    │  • 消息投递         │       │
│   │                     │    │  • 安全沙盒         │       │
│   └─────────────────────┘    └─────────────────────┘       │
│              ↓                         ↓                    │
│         意志与决策                执行与保障                │
└─────────────────────────────────────────────────────────────┘
```

#### Unified Address Book (统一通讯录)

| 属性 | 描述 |
|------|------|
| **技术** | 图数据库 (Graph DB) |
| **内容** | 存储所有 Agent (Lord/King) 和 Human (Staff) 的节点 |
| **关系** | 定义了 REPORTS_TO (汇报), DEPENDS_ON (依赖), CAN_CONTACT (可联系) 等边，用于权限控制和路由 |

---

### 4.2 领域层：Stateless Lords (无状态领主)

系统的"中层管理者"。

#### Lord 定义

Lord 不是常驻进程，而是存储在 DB 中的 **配置包 (Profile)**，包含：
- System Prompt
- Skill Set
- Memory ID

#### 运行时机制

| 阶段 | 描述 |
|------|------|
| **唤醒 (Hydration)** | 当有任务或会议时，Runtime 加载 Profile 启动一个通用 Agent 容器 |
| **思考 (Reasoning)** | 利用长期记忆 (Vector DB) 拆解任务 |
| **休眠 (Dehydration)** | 任务分发给 Knight 或会议结束后，保存上下文，销毁容器 |

#### 唤醒触发源

| 触发方式 | 描述 |
|----------|------|
| **用户直连** | 用户通过统一通讯录 (Address Book) 查找并直接联系对应 Lord |
| **King 路由** | 用户不确定找谁时，由 King 分析请求并路由到合适的 Lord |

#### 价值

- 极大降低资源成本
- 支持无限扩展领域数量
- 领域边界清晰，职责单一

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

### 4.4 执行层：Knights & Storage

系统的"手脚"与"仓库"。

#### Knight Pod

| 属性 | 描述 |
|------|------|
| **技术** | K8s Pod + Kata Containers (安全沙盒) + Stdio Tunnel |
| **特性** | 用完即毁，无记忆，纯执行 |
| **能力** | 运行代码、CLI 工具、爬虫 |

#### Spec 驱动执行模式

Knight 的无状态设计通过 **Spec 驱动** 模式保证执行质量：

```
┌─────────────────────────────────────────────────────────────┐
│                    Lord 规划阶段                             │
│  1. 接收用户话题/任务                                        │
│  2. 制定计划 (Plan)                                          │
│  3. 设计方案 (Design)                                        │
│  4. 拆解为任务清单 (Task List)                               │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    Knight 执行阶段                           │
│  1. 载入：规范文档 + 技能包 + 工具                           │
│  2. 执行：任务清单中的单个原子任务                           │
│  3. 交付：向 Lord 提交执行产物                               │
│  ⚠️ 无需知道整体上下文，只关注当前原子任务                   │
└─────────────────────────────────────────────────────────────┘
```

**核心要点**：
- Lord 负责"想清楚"，Knight 负责"做出来"
- 每个 Knight 任务是自包含的原子操作
- 通过规范文档传递必要信息，而非依赖记忆

#### Knight 任务生命周期

| 阶段 | 描述 |
|------|------|
| **派发** | Lord 通过 Sandbox 实时交互派发原子任务 |
| **执行** | Knight 在沙盒中执行任务（代码运行、文件操作等） |
| **成功** | 交付产物给 Lord（修订文件、新建文档等，视任务而定） |
| **失败处理** | 上报 Lord，由 Lord 决定：继续 / 重试 / 跳过 |
| **升级** | Lord 无法抉择时，询问用户 |

**通信方式**：Lord → Knight 采用 **Sandbox 实时交互**（Stdio Tunnel），无需消息队列

#### Knight 配额管理

| 场景 | 处理方式 |
|------|----------|
| **初始分配** | 领域创建时，King 为 Lord 分配固定 Knight 配额 |
| **配额不足** | Lord 向 King 提交配额申请 |
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
| 王室双星 (Royal Couple) | King 负责战略，Queen 负责执行，职责分明 |
| 圆桌会议 | 解决了 AI 协作无序问题 |
| 无状态领主 | 解决了大规模部署的成本问题 |
| 人类内阁 + 策略引擎 | 解决了 AI 的安全与合规问题 |
| 沙盒底座 | 保证了执行的绝对安全与环境一致性 |
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
