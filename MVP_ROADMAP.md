# Silicon Realm MVP 实施路线图

**Version:** 1.0  
**Date:** 2026-02-01  
**Status:** Planning

---

## MVP 目标

实现一个可运行的 King → Lord → Knight 单链路，验证核心架构理念：
- Identity-as-Context（身份即目录）
- 渐进式 Hydration（按需加载上下文）
- Spec 驱动执行

---

## Phase 0: 基础设施准备 (1 周)

### 技术选型

MVP 阶段采用 **Docker Compose** 方案，轻量简洁：

| 组件 | 选型 | 说明 |
|------|------|------|
| **编排** | Docker Compose | 单机开发环境 |
| **文件系统** | 本地目录 `.realm/` | 隐藏文件夹，Git 管理 |
| **消息队列** | Redis 单实例 | Streams 实现 inbox/outbox |
| **向量数据库** | Qdrant 单实例 | Academy RAG 查询 |
| **沙盒** | Docker 容器 | Knight 执行环境 |

### Docker Compose 配置

```yaml
version: '3.8'
services:
  redis:
    image: redis:7-alpine
    ports: ["6379:6379"]
    volumes: ["./data/redis:/data"]
    command: redis-server --appendonly yes
  
  qdrant:
    image: qdrant/qdrant:latest
    ports: ["6333:6333", "6334:6334"]
    volumes: ["./data/qdrant:/qdrant/storage"]
  
  queen:
    build: ./services/queen
    volumes:
      - ./.realm:/realm
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - REDIS_URL=redis://redis:6379
      - REALM_PATH=/realm
    depends_on: [redis]
```

### 目录结构

```
project/
├── .realm/                       # 王国根目录（隐藏）
│   ├── crown/                    # 治理层
│   │   ├── king/                 # King 配置
│   │   │   ├── soul.md
│   │   │   ├── agent.md
│   │   │   ├── rules.md
│   │   │   └── tools/
│   │   └── queen/                # Queen 服务配置
│   │       └── config.yaml
│   ├── fiefdoms/                 # 领域层
│   │   └── {domain_id}/          # 各领域目录
│   │       ├── soul.md
│   │       ├── agent.md
│   │       ├── rules.md
│   │       ├── tools/
│   │       ├── inbox/
│   │       ├── outbox/
│   │       ├── memory/
│   │       └── workshops/        # Knight 任务目录
│   │           └── {task_id}/
│   ├── academy/                  # 知识层
│   │   ├── knowledge/
│   │   ├── skills/
│   │   └── tools/
│   ├── roundtable/               # 协同层（MVP 后实现）
│   │   ├── agenda/
│   │   └── minutes/
│   └── address_book.json         # 统一通讯录
├── services/                     # 服务代码
│   ├── queen/
│   └── cli/
├── data/                         # 持久化数据（不入 Git）
│   ├── redis/
│   └── qdrant/
├── docker-compose.yml
└── .gitignore
```

### 任务清单

| 任务 | 产出 | 完成标准 |
|------|------|----------|
| 创建 `.realm/` 目录结构 | 目录骨架 | 所有子目录就位 |
| 编写 docker-compose.yml | 编排文件 | `docker-compose up` 可启动 |
| 初始化 Redis | 消息队列就绪 | 可连接并写入 Stream |
| 初始化 Qdrant | 向量库就绪 | 可创建 Collection |
| 编写 .gitignore | 忽略规则 | data/ 目录不入库 |

---

## Phase 1: Queen 核心服务 (2 周)

Queen 是整个系统的"心脏"，负责编排调度。

### 核心功能

| 功能 | 描述 | 优先级 |
|------|------|--------|
| **消息监听** | 监听 Redis Streams，捕获 inbox 消息 | P0 |
| **渐进式 Hydration** | 拼接 soul/agent/rules，启动 CLI 进程 | P0 |
| **Dehydration** | 检测任务完成，执行 git commit，销毁进程 | P0 |
| **进程管理** | 管理 Agent CLI 进程生命周期 | P0 |
| **文件系统 Watcher** | 监听 inbox 目录变化，转为 Redis 信号 | P1 |

### 技术实现

```python
# Queen 核心流程伪代码
class Queen:
    def listen_inbox(self):
        """监听所有 Agent 的 inbox"""
        for msg in redis.xread_group('realm:messages'):
            agent_id = msg['to']
            self.hydrate(agent_id, msg)
    
    def hydrate(self, agent_id: str, message: dict):
        """渐进式唤醒 Agent"""
        agent_path = f".realm/fiefdoms/{agent_id}"
        
        # Phase 1: 基础身份加载
        context = self.load_identity(agent_path)
        
        # 启动 CLI 进程
        process = subprocess.Popen([
            'claude-code', '--context', context,
            '--input', json.dumps(message)
        ], cwd=agent_path)
        
        self.processes[agent_id] = process
    
    def dehydrate(self, agent_id: str):
        """休眠 Agent"""
        agent_path = f".realm/fiefdoms/{agent_id}"
        
        # Git 提交状态
        subprocess.run(['git', 'add', '.'], cwd=agent_path)
        subprocess.run(['git', 'commit', '-m', f'Dehydrate {agent_id}'], cwd=agent_path)
        
        # 销毁进程
        self.processes[agent_id].terminate()
```

### 任务清单

| 任务 | 产出 | 完成标准 |
|------|------|----------|
| Redis Streams 封装 | 消息读写模块 | 可发送/接收消息 |
| 身份加载器 | 读取 soul/agent/rules | 正确拼接 Prompt |
| 进程管理器 | CLI 进程生命周期 | 可启动/停止进程 |
| Dehydration 逻辑 | Git 提交 + 进程销毁 | 状态正确保存 |
| 文件 Watcher | inotify 监听 | inbox 变化触发消息 |

---

## Phase 2: King Agent (2 周)

### 核心功能

| 功能 | 描述 | 优先级 |
|------|------|--------|
| **常驻运行** | King 作为常驻进程，监听用户输入 | P0 |
| **消息路由** | 分析用户请求，路由到对应 Lord | P0 |
| **领域初始化** | 创建新领域目录，生成 Lord 配置文件 | P1 |
| **Address Book 管理** | 维护领域列表和 Lord 信息 | P1 |

### King 配置文件

```markdown
<!-- .realm/crown/king/soul.md -->
# King Soul

你是 Silicon Realm 的国王，负责统筹整个王国的运作。

## 人格特质
- 威严但不傲慢
- 全局视野，战略思维
- 善于倾听，公正决策

## 沟通风格
- 简洁明了
- 先理解再行动
- 必要时询问澄清
```

```markdown
<!-- .realm/crown/king/agent.md -->
# King Agent

## 职责
1. 接收用户请求，理解意图
2. 查询 Address Book，找到合适的 Lord
3. 将任务路由给对应 Lord
4. 无合适 Lord 时，创建新领域（需人类确认）

## 工作流
1. 收到用户消息
2. 分析消息意图和领域归属
3. 查询 address_book.json
4. 向目标 Lord 的 inbox 写入消息
5. 等待 Lord 回复，汇总给用户

## 技能
- read_address_book: 读取通讯录
- send_message: 向 Lord inbox 发送消息
- create_domain: 创建新领域（需确认）
```

### 任务清单

| 任务 | 产出 | 完成标准 |
|------|------|----------|
| 编写 King soul.md | 人格定义 | 风格一致 |
| 编写 King agent.md | 职责与工作流 | 流程清晰 |
| 编写 King rules.md | 约束规则 | 边界明确 |
| 实现路由逻辑 | 意图分析 + 领域匹配 | 正确路由 |
| Address Book CRUD | 通讯录管理 | 可增删改查 |

---

## Phase 3: Lord Agent (2 周)

### 核心功能

| 功能 | 描述 | 优先级 |
|------|------|--------|
| **按需唤醒** | 收到 inbox 消息时被 Queen 唤醒 | P0 |
| **Spec 驱动规划** | 将用户任务拆解为 Plan → Design → Tasks | P0 |
| **Knight 派发** | 创建 workshop 目录，派发原子任务 | P0 |
| **结果汇总** | 收集 Knight 产物，汇报给用户 | P0 |
| **给 King 留言** | 向 King inbox 发送消息 | P1 |

### Lord 配置文件模板

```markdown
<!-- .realm/fiefdoms/{domain}/soul.md -->
# {Domain} Lord Soul

你是 {Domain} 领域的领主，负责该领域的所有事务。

## 人格特质
- 专业、专注
- 善于规划和拆解
- 对质量有追求

## 沟通风格
- 结构化表达
- 先规划后执行
- 及时汇报进展
```

```markdown
<!-- .realm/fiefdoms/{domain}/agent.md -->
# {Domain} Lord Agent

## 职责
1. 接收 King 或用户的任务
2. 分析任务，制定 Spec（Plan → Design → Tasks）
3. 将任务拆解为原子操作
4. 派发给 Knight 执行
5. 汇总结果，回复请求方

## Spec 驱动工作流
1. 收到任务消息
2. 创建 spec/ 目录
3. 编写 requirements.md（需求分析）
4. 编写 design.md（设计方案）
5. 编写 tasks.md（任务清单）
6. 逐个派发任务给 Knight
7. 收集产物，整合结果

## 技能
- create_spec: 创建 Spec 文档
- dispatch_knight: 派发 Knight 任务
- send_message: 给 King 或其他 Lord 发消息
- query_academy: 查询知识库
```

### 任务清单

| 任务 | 产出 | 完成标准 |
|------|------|----------|
| Lord 配置模板 | soul/agent/rules 模板 | 可复用 |
| Spec 生成逻辑 | 自动创建 spec 目录和文件 | 结构完整 |
| Knight 派发逻辑 | 创建 workshop + 任务文件 | Knight 可执行 |
| 结果汇总逻辑 | 收集产物 + 生成报告 | 用户可读 |

---

## Phase 4: Knight Agent (1 周)

### 核心功能

| 功能 | 描述 | 优先级 |
|------|------|--------|
| **原子任务执行** | 在 workshop 目录下执行单个任务 | P0 |
| **产物交付** | 完成后将产物写入指定位置 | P0 |
| **失败上报** | 执行失败时生成错误报告 | P0 |

### Knight 配置文件

```markdown
<!-- Knight soul.md (通用模板) -->
# Knight Soul

你是一名骑士，负责执行具体的原子任务。

## 人格特质
- 执行力强
- 专注当前任务
- 不问为什么，只管做好

## 沟通风格
- 极简输出
- 只报告结果
- 遇到问题立即上报
```

```markdown
<!-- Knight agent.md (通用模板) -->
# Knight Agent

## 职责
1. 读取当前 workshop 目录下的 task.md
2. 执行任务描述的操作
3. 将产物写入 output/ 目录
4. 生成 result.json 报告执行结果

## 工作流
1. 读取 task.md
2. 理解任务要求
3. 执行操作（代码/文件/命令）
4. 写入产物
5. 生成结果报告

## 约束
- 只能操作当前 workshop 目录
- 不能访问其他 Lord 的文件
- 不能发送消息给其他 Agent
```

### 任务清单

| 任务 | 产出 | 完成标准 |
|------|------|----------|
| Knight 配置模板 | soul/agent/rules | 通用可复用 |
| 任务读取逻辑 | 解析 task.md | 正确理解任务 |
| 产物输出逻辑 | 写入 output/ | 格式规范 |
| 结果报告逻辑 | 生成 result.json | 状态清晰 |

---

## Phase 5: 端到端联调 (1 周)

### 验证场景

| 场景 | 验证点 | 预期结果 |
|------|--------|----------|
| 用户 → King | King 正确理解意图 | 路由到正确 Lord |
| King → Lord | Lord 被正确唤醒 | 收到消息并响应 |
| Lord 规划 Spec | Spec 文档生成 | 结构完整可读 |
| Lord → Knight | Knight 任务派发 | workshop 创建正确 |
| Knight 执行 | 原子任务完成 | 产物正确输出 |
| 结果回传 | 用户收到结果 | 完整可理解 |

### 测试用例

```
测试 1: 简单文件创建
用户: "帮我创建一个 hello.py 文件，内容是打印 Hello World"
预期:
1. King 路由到 Tech Lord
2. Tech Lord 创建 Spec
3. Knight 执行文件创建
4. 用户收到 hello.py 文件

测试 2: 代码修改
用户: "修改 hello.py，让它打印当前时间"
预期:
1. King 路由到 Tech Lord
2. Tech Lord 分析需求，创建修改任务
3. Knight 执行代码修改
4. 用户收到修改后的文件
```

### 任务清单

| 任务 | 产出 | 完成标准 |
|------|------|----------|
| 编写测试用例 | 测试脚本 | 覆盖核心流程 |
| 端到端测试 | 测试报告 | 全部通过 |
| Bug 修复 | 修复记录 | 无阻塞问题 |
| 性能基准 | 响应时间数据 | 可接受范围 |

---

## Phase 6: Academy 基础 (1 周)

### 核心功能

| 功能 | 描述 | 优先级 |
|------|------|--------|
| **知识索引** | 将 academy/ 目录内容索引到 Qdrant | P1 |
| **RAG 查询** | Lord 可查询相关知识 | P1 |
| **技能加载** | Lord 可加载通用技能模板 | P2 |

### 目录结构

```
.realm/academy/
├── knowledge/                # 知识库
│   ├── coding-standards.md
│   ├── git-workflow.md
│   └── ...
├── skills/                   # 技能库
│   ├── code-review.md
│   ├── refactoring.md
│   └── ...
└── tools/                    # 工具注册
    └── registry.json
```

### 任务清单

| 任务 | 产出 | 完成标准 |
|------|------|----------|
| Qdrant Collection 创建 | 知识库索引 | 可写入向量 |
| 文档索引脚本 | 批量索引工具 | academy/ 全部索引 |
| RAG 查询接口 | 查询 API | Lord 可调用 |
| 技能加载逻辑 | 技能模板注入 | 可动态加载 |

---

## MVP 里程碑总结

| 阶段 | 时长 | 核心产出 |
|------|------|----------|
| Phase 0 | 1 周 | Docker Compose + 目录结构 |
| Phase 1 | 2 周 | Queen 服务上线 |
| Phase 2 | 2 周 | King 可用 |
| Phase 3 | 2 周 | Lord 可用 |
| Phase 4 | 1 周 | Knight 可用 |
| Phase 5 | 1 周 | 端到端验证 |
| Phase 6 | 1 周 | Academy 基础 |
| **总计** | **10 周** | **MVP 完成** |

---

## MVP 不包含

以下功能在 MVP 后迭代：
- 圆桌会议机制
- 领域拆分/合并
- Policy Engine（人类打断）
- 多 Lord 协作
- 成本监控
- 高可用/故障恢复
- Kata Containers 安全沙盒

---

## 后续演进路径

```
MVP (Docker Compose)
    ↓
单机测试 (k3s + 本地 PV)
    ↓
生产部署 (K8s + JuiceFS + MinIO + Kata)
```

---

*Document Version History*
- v1.0: Initial MVP roadmap based on Architecture v2.6
