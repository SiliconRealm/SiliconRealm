# Silicon Realm MVP 实施路线图

**Version:** 6.0  
**Date:** 2026-02-02  
**Status:** Planning  
**Based on:** Architecture v3.5 + OpenClaw Multi-Agent

---

## MVP 目标

**极简验证**：利用 OpenClaw 原生多Agent能力，单实例运行所有Agent，验证核心机制：

| 机制 | 验证目标 |
|:-----|:---------|
| 系统创世 | King 分析材料、划分领域、配置新 Agent |
| 任务转接 | King ↔ Lord 消息路由 |
| Agent 间通信 | 通过 OpenClaw sessions 或 Telegram |
| 圆桌会议 | 多 Agent 协作决策 |
| 领域重整 | 领域拆分/合并 |

---

## 技术架构

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenClaw 单实例                           │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                 openclaw.json                        │   │
│  │  agents.list: [king, lord-tech, lord-finance, ...]  │   │
│  │  bindings: [telegram → agent 映射]                   │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                 │
│  │  👑 King  │  │🏰 Lord A │  │🏰 Lord B │  ...           │
│  │ workspace │  │ workspace │  │ workspace │                │
│  └──────────┘  └──────────┘  └──────────┘                 │
│                                                             │
│  共享: /agents/address_book.json, /agents/shared/skills/   │
└─────────────────────────────────────────────────────────────┘
                          │
                          ▼
              Telegram / Discord / TUI
```

**核心简化**：
- ❌ 不需要 Docker Compose
- ❌ 不需要多个容器
- ❌ 不需要 Docker SDK Tool
- ✅ 单个 OpenClaw 实例
- ✅ 配置文件定义 Agent
- ✅ 消息自动路由

---

## 目录结构

```
silicon-realm/
├── agents/
│   ├── king/                    # King workspace
│   │   ├── SOUL.md
│   │   ├── AGENTS.md
│   │   └── skills/
│   │
│   ├── tech/                    # Tech Lord workspace (创世后)
│   │   ├── SOUL.md
│   │   ├── AGENTS.md
│   │   └── skills/
│   │
│   ├── shared/                  # 共享资源
│   │   └── skills/
│   │       ├── telegram_notify/
│   │       ├── roundtable_speak/
│   │       └── roundtable_vote/
│   │
│   ├── templates/               # Lord 模板
│   ├── genesis/                 # 创世材料
│   ├── roundtable/              # 会议记录
│   ├── address_book.json
│   └── domain_map.json
│
├── openclaw.json                # OpenClaw 多Agent配置
├── .env
└── README.md
```

---

## 核心配置

### openclaw.json

```json
{
  "agents": {
    "defaults": {
      "sandbox": { "mode": "all", "scope": "agent" }
    },
    "list": [
      {
        "id": "king",
        "name": "King",
        "workspace": "./agents/king",
        "sandbox": { "mode": "off" }
      }
    ]
  },
  "bindings": [
    {
      "agentId": "king",
      "match": { "provider": "telegram", "accountId": "*" }
    }
  ]
}
```

### 创世后动态添加 Lord

```json
{
  "agents": {
    "list": [
      { "id": "king", "workspace": "./agents/king", "sandbox": { "mode": "off" } },
      { "id": "lord-tech", "workspace": "./agents/tech", "sandbox": { "mode": "all" } },
      { "id": "lord-finance", "workspace": "./agents/finance", "sandbox": { "mode": "all" } }
    ]
  },
  "bindings": [
    { "agentId": "king", "match": { "provider": "telegram", "peer": { "id": "king_chat" } } },
    { "agentId": "lord-tech", "match": { "provider": "telegram", "peer": { "id": "tech_chat" } } },
    { "agentId": "lord-finance", "match": { "provider": "telegram", "peer": { "id": "finance_chat" } } }
  ]
}
```

---

## 交付物清单

### 设定文档
| 文件 | 描述 |
|:-----|:-----|
| agents/king/SOUL.md | King 人格定义 |
| agents/king/AGENTS.md | King 职责与工作流 |
| agents/templates/lord/SOUL.md.template | Lord 人格模板 |
| agents/templates/lord/AGENTS.md.template | Lord 职责模板 |

### Skills
| 技能 | 所属 | 描述 |
|:-----|:-----|:-----|
| analyze_domain | King | 分析材料，划分领域 |
| genesis | King | 创建 Lord 配置，更新 openclaw.json |
| convene_roundtable | King | 召集圆桌会议 |
| execute_restructure | King | 执行领域重整 |
| telegram_notify | Shared | Agent 间通信 |
| roundtable_speak | Shared | 圆桌发言 |
| roundtable_vote | Shared | 圆桌投票 |
| propose_restructure | Shared | 提议重整 |

### 配置文件
| 文件 | 描述 |
|:-----|:-----|
| openclaw.json | OpenClaw 多Agent配置 |
| .env | 环境变量 |
| agents/address_book.json | 通讯录 |

---

## 实施阶段

### Phase 1: King 基础 (2天)

**任务**：
- [ ] 编写 agents/king/SOUL.md
- [ ] 编写 agents/king/AGENTS.md
- [ ] 实现 analyze_domain 技能
- [ ] 实现 genesis 技能（更新 openclaw.json）
- [ ] 配置 openclaw.json 初始版本
- [ ] 测试 King 通过 Telegram 响应

**genesis 技能核心逻辑**：
```python
# 创世时更新 openclaw.json
def add_lord_to_config(domain_id, workspace_path):
    config = load_openclaw_json()
    config["agents"]["list"].append({
        "id": f"lord-{domain_id}",
        "workspace": workspace_path,
        "sandbox": {"mode": "all", "scope": "agent"}
    })
    save_openclaw_json(config)
    # OpenClaw 会自动热加载新配置
```

### Phase 2: Agent 通信 (1天)

**任务**：
- [ ] 实现 telegram_notify 技能
- [ ] 测试 King → Lord 消息传递
- [ ] 测试 Lord → King 消息传递

### Phase 3: Lord 模板与创世 (2天)

**任务**：
- [ ] 编写 Lord SOUL.md 模板
- [ ] 编写 Lord AGENTS.md 模板
- [ ] 完善 genesis 技能（使用模板生成配置）
- [ ] 端到端测试创世流程

### Phase 4: 圆桌会议 (2天)

**任务**：
- [ ] 实现 convene_roundtable 技能
- [ ] 实现 roundtable_speak 技能
- [ ] 实现 roundtable_vote 技能
- [ ] 定义会议记录格式
- [ ] 端到端测试圆桌流程

### Phase 5: 领域重整 (1天)

**任务**：
- [ ] 实现 propose_restructure 技能
- [ ] 实现 execute_restructure 技能
- [ ] 测试领域拆分流程

### Phase 6: 验证 (1天)

**任务**：
- [ ] 完整创世流程测试
- [ ] 任务转接流程测试
- [ ] 圆桌会议流程测试
- [ ] 领域重整流程测试
- [ ] 文档完善

---

## MVP 里程碑

| 阶段 | 时长 | 产出 |
|:-----|:-----|:-----|
| Phase 1 | 2天 | King 基础能力 |
| Phase 2 | 1天 | Agent 间通信 |
| Phase 3 | 2天 | Lord 模板与创世 |
| Phase 4 | 2天 | 圆桌会议机制 |
| Phase 5 | 1天 | 领域重整机制 |
| Phase 6 | 1天 | 端到端验证 |
| **总计** | **9天** | **MVP 完成** |

---

## 快速启动

```bash
# 1. 克隆项目
git clone <repo> silicon-realm && cd silicon-realm

# 2. 配置环境变量
cp .env.example .env
# 编辑 .env，填入 ANTHROPIC_API_KEY 和 Telegram Token

# 3. 启动 OpenClaw
openclaw

# 4. 通过 Telegram 与 King 对话
# 或使用 TUI: openclaw chat
```

---

*Version History*
- v5.0: Complete Mechanism Verification with Docker
- v6.0: **OpenClaw Native Multi-Agent** - 单实例运行，无需Docker Compose，9天完成MVP
