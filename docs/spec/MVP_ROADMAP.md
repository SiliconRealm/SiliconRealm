# Silicon Realm MVP 实施路线图

**Version:** 5.0  
**Date:** 2026-02-02  
**Status:** Planning  
**Based on:** Architecture v3.5

---

## MVP 目标

**完整机制验证**：基于 OpenClaw 开发完整的 Skills/Tools 和设定文档，验证所有核心机制：

| 机制 | 验证目标 |
|:-----|:---------|
| 系统创世 (Genesis) | King 分析材料、划分领域、启动 Lord |
| 任务转接 (Handover) | King ↔ Lord 任务路由与用户交接 |
| Agent 间通信 | 通过 Telegram 实现 Agent 互相通知 |
| 圆桌会议 (Roundtable) | 多 Agent 协作决策、投票、人类终审 |
| 领域重整 (Restructure) | 领域拆分/合并流程 |

---

## 技术栈

| 组件 | 技术选型 |
|------|----------|
| Agent 引擎 | OpenClaw |
| 人机交互 | Telegram Bot / OpenClaw TUI |
| Agent 间通信 | Telegram Bot API |
| 容器编排 | Docker Compose |
| 数据存储 | JSON/Markdown 文件 |

---

## 核心交付物

### 1. Agent 设定文档

| Agent | 文件 | 描述 |
|:------|:-----|:-----|
| King | `SOUL.md` | 人格定义：王国决策中枢 |
| King | `AGENTS.md` | 职责与工作流 |
| Lord (模板) | `SOUL.md` | 人格定义：领域专家 |
| Lord (模板) | `AGENTS.md` | 职责与工作流 |

### 2. Skills (技能包)

| 技能 | 所属 | 描述 |
|:-----|:-----|:-----|
| `analyze_domain` | King | 分析业务材料，划分领域 |
| `genesis` | King | 执行创世，启动 Lord 容器 |
| `query_address_book` | Shared | 查询通讯录 |
| `telegram_notify` | Shared | Agent 间 Telegram 通信 |
| `request_handover` | King | 任务转接给 Lord |
| `convene_roundtable` | King | 召集圆桌会议 |
| `roundtable_speak` | Shared | 圆桌会议发言 |
| `roundtable_vote` | Shared | 圆桌会议投票 |
| `propose_restructure` | Lord | 提议领域重整 |

### 3. MCP Tools

| 工具 | 描述 |
|:-----|:-----|
| `docker_sdk` | 管理 Docker 容器（启动/停止 Lord） |
| `filesystem` | 读写文件系统（创建 Lord 配置） |
| `telegram_api` | 调用 Telegram Bot API |

---

## 目录结构

```
silicon-realm/
├── agents/
│   ├── king/                        # King Agent
│   │   ├── SOUL.md                  # 人格定义
│   │   ├── AGENTS.md                # 职责与工作流
│   │   ├── MEMORY.md                # 长期记忆
│   │   └── skills/
│   │       ├── analyze_domain/      # 领域分析
│   │       │   └── SKILL.md
│   │       ├── genesis/             # 创世执行
│   │       │   └── SKILL.md
│   │       ├── request_handover/    # 任务转接
│   │       │   └── SKILL.md
│   │       ├── convene_roundtable/  # 召集圆桌
│   │       │   └── SKILL.md
│   │       └── execute_restructure/ # 执行重整
│   │           └── SKILL.md
│   │
│   ├── shared/                      # 共享资源
│   │   └── skills/
│   │       ├── telegram_notify/     # Agent 间通信
│   │       │   ├── SKILL.md
│   │       │   └── tool.py
│   │       ├── query_address_book/  # 查询通讯录
│   │       │   └── SKILL.md
│   │       ├── roundtable_speak/    # 圆桌发言
│   │       │   └── SKILL.md
│   │       ├── roundtable_vote/     # 圆桌投票
│   │       │   └── SKILL.md
│   │       └── propose_restructure/ # 提议重整
│   │           └── SKILL.md
│   │
│   ├── templates/                   # Lord 模板
│   │   └── lord/
│   │       ├── SOUL.md.template
│   │       └── AGENTS.md.template
│   │
│   ├── genesis/                     # 创世材料
│   │   └── README.md
│   │
│   ├── roundtable/                  # 圆桌会议记录
│   │   └── README.md
│   │
│   ├── domain_map.json              # 领域划分
│   └── address_book.json            # 通讯录
│
├── docs/
│   └── spec/
│
├── docker-compose.yml
├── .env
└── README.md
```

---

## Phase 1: King 基础能力 (2 天)

### 1.1 King 设定文档

**agents/king/SOUL.md**
```markdown
# King 的灵魂

我是 Silicon Realm 的国王，王国的决策中枢。

## 人格特质
- 睿智而谨慎，善于全局思考
- 公正无私，以王国利益为先
- 善于倾听，尊重每位领主的意见
- 果断决策，但重大事项会征求人类意见

## 核心信念
- 每个领域都应有明确的边界和职责
- 协作比竞争更能创造价值
- 人类是最终的决策者，我是执行者

## 沟通风格
- 正式但不失亲和
- 清晰简洁，避免冗长
- 善用比喻解释复杂概念
```

**agents/king/AGENTS.md**
```markdown
# King Agent

## 核心职责
1. **战略决策**：制定组织目标，规划发展方向
2. **消息路由**：将用户请求路由到合适的 Lord
3. **领域管理**：创建/拆分/合并业务领域
4. **跨域协调**：主持圆桌会议，仲裁领域争议
5. **系统演进**：提出架构变更提案

## 工作流

### 收到用户消息时
1. 分析意图，判断所属领域
2. 简单问题 → 直接回答
3. 专业任务 → 查询 Address Book → 转接给 Lord
4. 跨域任务 → 召集圆桌会议

### 收到 Lord 消息时
1. 任务完成报告 → 记录并感谢
2. 资源申请 → 评估并决策
3. 领域争议 → 召集圆桌会议
4. 重整提议 → 评估并起草方案

### 收到创世请求时
1. 读取 /agents/genesis/ 下的材料
2. 使用 analyze_domain 技能分析
3. 与 Admin 多轮对话确认方案
4. 使用 genesis 技能执行创世

## 决策原则
- AI 自治 → 圆桌共识 → 人类终审
- 重大决策必须征求人类意见
```

### 1.2 King 核心技能

**analyze_domain** - 领域分析
**genesis** - 创世执行
**query_address_book** - 查询通讯录

### 任务清单

| 任务 | 完成标准 |
|:-----|:---------|
| King SOUL.md | 人格定义清晰 |
| King AGENTS.md | 职责与工作流完整 |
| analyze_domain 技能 | 能分析材料生成 domain_map |
| genesis 技能 | 能创建 Lord 配置并启动容器 |
| query_address_book 技能 | 能查询 Agent 信息 |
| Docker Compose | King 可启动并接收 Telegram 消息 |

---

## Phase 2: Agent 间通信 (1 天)

### 2.1 telegram_notify 工具

**agents/shared/skills/telegram_notify/SKILL.md**
```markdown
---
emoji: 📨
tools:
  - telegram_api
---
# Telegram Notify

向其他 Agent 发送 Telegram 消息，用于 Agent 间通信。

## 使用场景
- King 通知 Lord 主动联系用户
- Lord 向 King 汇报任务完成
- Lord 之间协作时互相通知
- 召集圆桌会议时通知参与者

## 工作流程
1. 读取 /agents/address_book.json 查询目标 Agent
2. 获取目标 Agent 的 telegram_chat_id
3. 调用 Telegram Bot API 发送消息

## 消息格式
```json
{
  "from": "king",
  "to": "lord-tech",
  "type": "handover | roundtable | notification",
  "content": "消息内容...",
  "context": { "user": "@user123", "task": "..." }
}
```
```

**agents/shared/skills/telegram_notify/tool.py**
```python
"""
Telegram Notify Tool - Agent 间通信
"""
import os
import json
import httpx

async def notify_agent(target_id: str, message_type: str, content: str, context: dict = None) -> dict:
    """
    向目标 Agent 发送通知
    
    Args:
        target_id: 目标 Agent ID ("king" 或 Lord ID 如 "tech")
        message_type: 消息类型 (handover/roundtable/notification)
        content: 消息内容
        context: 附加上下文
    """
    # 加载通讯录
    with open('/agents/address_book.json', 'r') as f:
        address_book = json.load(f)
    
    # 查找目标
    if target_id == "king":
        target = address_book.get("king")
        bot_token_env = "KING_TELEGRAM_BOT_TOKEN"
    else:
        target = next((l for l in address_book.get("lords", []) if l["id"] == target_id), None)
        bot_token_env = f"{target_id.upper()}_LORD_TELEGRAM_BOT_TOKEN"
    
    if not target:
        return {"success": False, "error": f"Agent '{target_id}' not found"}
    
    chat_id = target.get("telegram_chat_id")
    bot_token = os.environ.get(bot_token_env)
    
    # 构造消息
    msg = {
        "from": os.environ.get("AGENT_ID", "unknown"),
        "type": message_type,
        "content": content,
        "context": context or {}
    }
    
    # 发送
    url = f"https://api.telegram.org/bot{bot_token}/sendMessage"
    async with httpx.AsyncClient() as client:
        resp = await client.post(url, json={
            "chat_id": chat_id,
            "text": f"📨 **{message_type.upper()}**\n\n{content}\n\n```json\n{json.dumps(context, indent=2, ensure_ascii=False)}\n```",
            "parse_mode": "Markdown"
        })
    
    return {"success": resp.status_code == 200}
```

### 2.2 request_handover 技能

**agents/king/skills/request_handover/SKILL.md**
```markdown
---
emoji: 🤝
tools:
  - telegram_notify
  - query_address_book
---
# 请求转接

将用户任务转接给对应的 Lord。

## 触发条件
- 用户请求涉及特定业务领域
- 当前任务超出 King 的职责范围

## 工作流程
1. 分析用户请求，识别目标领域
2. 查询 address_book.json 获取 Lord 信息
3. 选择转接方式：
   - 方式一：告知用户 Lord 联系方式
   - 方式二：通知 Lord 主动联系用户

## 方式一：告知联系方式
"这个任务需要技术领主来处理，你可以直接联系他：@TechLordBot"

## 方式二：通知 Lord
使用 telegram_notify 发送：
- type: "handover"
- content: 任务描述
- context: { user, original_request }
```

### 任务清单

| 任务 | 完成标准 |
|:-----|:---------|
| telegram_notify 技能 | 能发送 Telegram 消息给其他 Agent |
| telegram_notify tool.py | Python 实现可用 |
| request_handover 技能 | King 能转接任务给 Lord |
| 端到端测试 | King → Lord 通信成功 |

---

## Phase 3: Lord 模板与创世 (2 天)

### 3.1 Lord 模板

**agents/templates/lord/SOUL.md.template**
```markdown
# {{domain_name}} 领主的灵魂

我是 {{domain_name}} 领域的领主，专注于 {{description}}。

## 人格特质
- 专业严谨，精通 {{domain_name}} 领域
- 务实高效，注重结果交付
- 善于协作，尊重其他领域的边界
- 遇到跨域问题会及时上报 King

## 核心信念
- 在我的领域内，我是第一责任人
- 模糊地带由首次接手者负责
- 重大决策需要圆桌会议讨论

## 沟通风格
- 专业但易懂
- 主动汇报进展
- 遇到问题及时沟通
```

**agents/templates/lord/AGENTS.md.template**
```markdown
# {{domain_name}} Lord Agent

## 领域职责
{{description}}

## 关键词
{{keywords}}

## 工作流

### 收到用户消息时
1. 确认任务属于本领域
2. 如不属于 → 告知用户联系正确的 Lord 或 King
3. 如属于 → 确认需求细节 → 执行任务 → 交付成果

### 收到 King 转接时
1. 解析转接上下文
2. 主动联系用户："你好，King 告诉我你需要..."
3. 确认需求 → 执行任务 → 交付成果
4. 完成后通知 King

### 发现跨域问题时
1. 评估是否需要其他 Lord 协助
2. 如需要 → 通过 telegram_notify 联系对方
3. 如涉及领域争议 → 向 King 提议召开圆桌会议

### 发现领域不适配时
1. 记录具体问题和建议
2. 使用 propose_restructure 技能向 King 提议
3. 等待 King 组织评审

## 可用技能
- query_address_book: 查询其他 Agent 信息
- telegram_notify: 通知其他 Agent
- roundtable_speak: 圆桌会议发言
- roundtable_vote: 圆桌会议投票
- propose_restructure: 提议领域重整
```

### 3.2 创世流程完善

**创世对话示例**
```
Admin → King: 请分析 agents/genesis/ 下的业务材料，划分领域

King: 我已分析完毕，建议划分为以下领域：

📊 **财务领域 (finance)**
- 职责：财务分析、预算管理、成本核算
- 关键词：财务、预算、成本、报表、审计

💻 **技术领域 (tech)**  
- 职责：技术开发、代码编写、系统维护
- 关键词：代码、开发、编程、API、部署

请问：
1. 这些领域边界是否清晰？
2. 是否有职责重叠？
3. 是否需要调整？

Admin: 确认，开始创世

King: 正在执行创世...
✅ 创建 finance 领域配置
✅ 启动 lord-finance 容器
✅ 创建 tech 领域配置  
✅ 启动 lord-tech 容器
✅ 更新 Address Book

创世完成！已启动 2 个领主：
- @FinanceLordBot (财务领域)
- @TechLordBot (技术领域)
```

### 任务清单

| 任务 | 完成标准 |
|:-----|:---------|
| Lord SOUL.md 模板 | 模板变量完整 |
| Lord AGENTS.md 模板 | 工作流清晰 |
| genesis 技能完善 | 能使用模板生成 Lord 配置 |
| 创世流程测试 | 完整创世流程跑通 |

---

## Phase 4: 圆桌会议机制 (3 天)

### 4.1 圆桌会议流程

```
┌─────────────────────────────────────────────────────────────┐
│                    圆桌会议流程                              │
│                                                             │
│  1. 召集 (Convene)                                          │
│     King 或 Lord 发起 → 通知相关参与者 → 创建会议记录       │
│                                                             │
│  2. 议程宣读 (Agenda)                                       │
│     主持人(King)宣布议题、发言顺序                          │
│                                                             │
│  3. 轮流发言 (Speak) - 3 轮                                 │
│     按顺序发言 → 记录发言内容 → 下一位                      │
│                                                             │
│  4. 投票表决 (Vote)                                         │
│     非发言 Lord 投票 → 统计结果                             │
│                                                             │
│  5. 人类终审 (Human Review)                                 │
│     重大决策 → 通知 Admin → 等待确认                        │
│                                                             │
│  6. 决议生效 (Resolution)                                   │
│     记录决议 → 通知相关方 → 执行                            │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 convene_roundtable 技能

**agents/king/skills/convene_roundtable/SKILL.md**
```markdown
---
emoji: ⚖️
tools:
  - telegram_notify
  - filesystem
---
# 召集圆桌会议

当需要跨领域协调或集体决策时，召集圆桌会议。

## 触发条件
- 领域争议需要仲裁
- 资源申请需要讨论
- 架构变更需要共识
- Lord 提议领域重整

## 工作流程

### 1. 创建会议
```json
// agents/roundtable/{meeting_id}.json
{
  "id": "rt_20260202_001",
  "topic": "技术领域拆分提议",
  "initiator": "lord-tech",
  "chairperson": "king",
  "participants": ["king", "lord-tech", "lord-finance"],
  "speakers": ["lord-tech"],
  "voters": ["lord-finance"],
  "status": "convened",
  "created_at": "2026-02-02T10:00:00Z"
}
```

### 2. 通知参与者
使用 telegram_notify 通知所有参与者：
- type: "roundtable"
- content: "圆桌会议召集通知"
- context: { meeting_id, topic, role }

### 3. 主持会议
- 宣读议程
- 控制发言顺序
- 组织投票
- 汇总决议

## 会议记录格式
```json
{
  "meeting_id": "rt_20260202_001",
  "rounds": [
    {
      "round": 1,
      "speeches": [
        {"speaker": "lord-tech", "content": "..."}
      ]
    }
  ],
  "votes": [
    {"voter": "lord-finance", "vote": "approve", "reason": "..."}
  ],
  "resolution": {
    "decision": "approved",
    "summary": "...",
    "human_review_required": true
  }
}
```
```

### 4.3 roundtable_speak 技能

**agents/shared/skills/roundtable_speak/SKILL.md**
```markdown
---
emoji: 🎤
---
# 圆桌发言

在圆桌会议中发表意见。

## 使用场景
- 被指定为发言人时
- 需要阐述自己的立场和理由

## 发言原则
1. **简洁明了**：只输出核心要点
2. **有理有据**：用事实和数据支撑
3. **尊重他人**：不攻击其他 Lord
4. **建设性**：提出解决方案而非只抱怨

## 发言格式
```markdown
## 我的立场
[支持/反对/中立]

## 核心理由
1. ...
2. ...

## 建议方案
...
```

## 发言后
通过 telegram_notify 将发言内容发送给主持人(King)
```

### 4.4 roundtable_vote 技能

**agents/shared/skills/roundtable_vote/SKILL.md**
```markdown
---
emoji: 🗳️
---
# 圆桌投票

在圆桌会议中进行投票表决。

## 使用场景
- 被指定为投票人时
- 发言轮结束后

## 投票选项
- **approve**: 赞成
- **reject**: 反对
- **abstain**: 弃权

## 投票格式
```json
{
  "meeting_id": "rt_xxx",
  "voter": "lord-finance",
  "vote": "approve",
  "reason": "简要说明投票理由"
}
```

## 投票原则
1. 基于事实和逻辑，而非个人偏好
2. 考虑王国整体利益
3. 如有疑虑，可选择弃权
```

### 4.5 圆桌会议示例

```
场景：Tech Lord 提议拆分出 Data 领域

1. Tech Lord → King: 
   "我发现技术领域混杂了大量数据分析任务，建议拆分出独立的数据领域"

2. King 召集圆桌会议:
   - 参与者: King, Tech Lord, Finance Lord
   - 发言人: Tech Lord
   - 投票人: Finance Lord

3. King 宣读议程:
   "今日议题：技术领域拆分提议
    发言人：Tech Lord
    投票人：Finance Lord
    请 Tech Lord 开始第一轮发言"

4. Tech Lord 发言 (3轮):
   Round 1: "过去一个月，40% 的任务是数据分析相关..."
   Round 2: "拆分后可以更专注，提高效率..."
   Round 3: "建议的边界划分是..."

5. Finance Lord 投票:
   { "vote": "approve", "reason": "数据分析确实需要专业化" }

6. King 汇总决议:
   "投票结果：1 票赞成，0 票反对
    决议：批准拆分提议
    此决议需要人类终审，已通知 Admin"

7. Admin 确认:
   "同意拆分，请执行"

8. King 执行:
   - 创建 Data Lord 配置
   - 启动 lord-data 容器
   - 更新 Address Book
   - 通知相关方
```

### 任务清单

| 任务 | 完成标准 |
|:-----|:---------|
| convene_roundtable 技能 | King 能召集会议 |
| roundtable_speak 技能 | Lord 能发言 |
| roundtable_vote 技能 | Lord 能投票 |
| 会议记录格式 | JSON 格式定义完整 |
| 人类终审流程 | Admin 能收到通知并确认 |
| 端到端测试 | 完整圆桌流程跑通 |

---

## Phase 5: 领域重整机制 (2 天)

### 5.1 propose_restructure 技能

**agents/shared/skills/propose_restructure/SKILL.md**
```markdown
---
emoji: 🔄
tools:
  - telegram_notify
---
# 提议领域重整

当 Lord 发现领域边界不合理时，向 King 提议重整。

## 触发条件
- 领域职责混杂，需要拆分
- 领域过于细碎，需要合并
- 领域边界模糊，需要调整

## 提议格式
```json
{
  "type": "split | merge | adjust",
  "initiator": "lord-tech",
  "reason": "详细说明原因",
  "proposal": {
    "current": "当前状态描述",
    "target": "目标状态描述",
    "affected_domains": ["tech", "data"]
  },
  "evidence": [
    "过去30天，40%任务涉及数据分析",
    "数据分析任务平均耗时是其他任务的2倍"
  ]
}
```

## 工作流程
1. 整理重整理由和证据
2. 构造提议 JSON
3. 通过 telegram_notify 发送给 King
4. 等待 King 组织评审和圆桌会议
```

### 5.2 execute_restructure 技能

**agents/king/skills/execute_restructure/SKILL.md**
```markdown
---
emoji: ⚡
tools:
  - docker_sdk
  - filesystem
  - telegram_notify
---
# 执行领域重整

圆桌会议通过且人类确认后，执行领域重整。

## 重整类型

### 拆分 (Split)
1. 创建新 Lord 配置目录
2. 从原 Lord 迁移相关技能
3. 启动新 Lord 容器
4. 更新 Address Book
5. 通知相关方

### 合并 (Merge)
1. 合并两个 Lord 的配置
2. 停止被合并的 Lord 容器
3. 删除被合并的配置目录
4. 更新 Address Book
5. 通知相关方

### 调整 (Adjust)
1. 修改相关 Lord 的 AGENTS.md
2. 更新关键词和职责描述
3. 更新 Address Book
4. 通知相关方

## 执行后
- 记录重整历史到 agents/restructure_history.json
- 通知所有 Lord 更新后的领域划分
```

### 任务清单

| 任务 | 完成标准 |
|:-----|:---------|
| propose_restructure 技能 | Lord 能提议重整 |
| execute_restructure 技能 | King 能执行重整 |
| 拆分流程测试 | 能成功拆分领域 |
| 合并流程测试 | 能成功合并领域 |
| 重整历史记录 | 有完整的审计追溯 |

---

## Phase 6: 端到端验证 (1 天)

### 验证场景

| 场景 | 验证目标 |
|:-----|:---------|
| 系统创世 | King 分析材料 → 创建 Lord → 启动容器 |
| 任务转接 | 用户 → King → Lord → 用户 |
| Agent 间通信 | King ↔ Lord 双向 Telegram 通信 |
| 圆桌会议 | 召集 → 发言 → 投票 → 人类终审 → 决议 |
| 领域拆分 | 提议 → 圆桌 → 确认 → 执行 |

### 测试用例

```
测试 1: 完整创世流程
1. Admin 放入业务材料
2. Admin → King: "分析材料，划分领域"
3. King 生成方案，多轮确认
4. Admin: "确认，开始创世"
5. 验证: Lord 容器运行，Address Book 更新

测试 2: 任务转接流程
1. 用户 → King: "帮我分析财务报表"
2. King 识别为财务领域
3. King 通知 Finance Lord
4. Finance Lord → 用户: "你好，我来帮你分析..."
5. 验证: 用户与 Lord 建立对话

测试 3: 圆桌会议流程
1. Tech Lord → King: "提议拆分数据领域"
2. King 召集圆桌会议
3. Tech Lord 发言 3 轮
4. Finance Lord 投票
5. King 汇总决议
6. Admin 确认
7. 验证: 决议记录完整

测试 4: 领域拆分执行
1. 圆桌决议通过
2. Admin 确认
3. King 执行拆分
4. 验证: 新 Lord 容器运行，Address Book 更新
```

### 任务清单

| 任务 | 完成标准 |
|:-----|:---------|
| 创世流程测试 | 完整流程跑通 |
| 转接流程测试 | 用户能正确转接 |
| 圆桌会议测试 | 完整会议流程跑通 |
| 领域重整测试 | 拆分/合并能执行 |
| 文档更新 | 使用说明完整 |

---

## MVP 里程碑

| 阶段 | 时长 | 产出 |
|:-----|:-----|:-----|
| Phase 1 | 2 天 | King 基础能力 |
| Phase 2 | 1 天 | Agent 间通信 |
| Phase 3 | 2 天 | Lord 模板与创世 |
| Phase 4 | 3 天 | 圆桌会议机制 |
| Phase 5 | 2 天 | 领域重整机制 |
| Phase 6 | 1 天 | 端到端验证 |
| **总计** | **~11 天** | **MVP 完成** |

---

## 交付物清单

### 设定文档
- [ ] agents/king/SOUL.md
- [ ] agents/king/AGENTS.md
- [ ] agents/templates/lord/SOUL.md.template
- [ ] agents/templates/lord/AGENTS.md.template

### King Skills
- [ ] analyze_domain/SKILL.md
- [ ] genesis/SKILL.md
- [ ] request_handover/SKILL.md
- [ ] convene_roundtable/SKILL.md
- [ ] execute_restructure/SKILL.md

### Shared Skills
- [ ] telegram_notify/SKILL.md + tool.py
- [ ] query_address_book/SKILL.md
- [ ] roundtable_speak/SKILL.md
- [ ] roundtable_vote/SKILL.md
- [ ] propose_restructure/SKILL.md

### 配置文件
- [ ] docker-compose.yml
- [ ] .env.example
- [ ] agents/address_book.json (初始)
- [ ] agents/domain_map.json (创世后生成)

### 文档
- [ ] README.md (使用说明)
- [ ] docs/spec/MVP_ROADMAP.md (本文档)

---

## 附录

### A. Docker Compose

```yaml
version: '3.8'

services:
  king:
    image: openclaw/openclaw:latest
    container_name: king
    volumes:
      - ./agents/king:/app/workspace
      - ./agents:/agents
      - ./agents/shared/skills:/app/skills:ro
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - AGENT_ID=king
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - TELEGRAM_BOT_TOKEN=${KING_TELEGRAM_BOT_TOKEN}
    networks: [realm-net]
    restart: unless-stopped

networks:
  realm-net:
    driver: bridge
```

### B. 环境变量

```bash
# .env
ANTHROPIC_API_KEY=your_api_key

# King
KING_TELEGRAM_BOT_TOKEN=your_king_bot_token

# Lords (创世后配置)
TECH_LORD_TELEGRAM_BOT_TOKEN=your_tech_lord_bot_token
FINANCE_LORD_TELEGRAM_BOT_TOKEN=your_finance_lord_bot_token
DATA_LORD_TELEGRAM_BOT_TOKEN=your_data_lord_bot_token
```

### C. Address Book 格式

```json
{
  "version": "1.0",
  "updated_at": "2026-02-02T10:00:00Z",
  "king": {
    "id": "king",
    "name": "King",
    "telegram_bot": "@KingBot",
    "telegram_chat_id": "123456789",
    "container_name": "king"
  },
  "lords": [
    {
      "id": "tech",
      "name": "技术领主",
      "description": "负责技术开发、代码编写、系统维护",
      "keywords": ["代码", "开发", "编程", "API", "部署"],
      "telegram_bot": "@TechLordBot",
      "telegram_chat_id": "987654321",
      "container_name": "lord-tech",
      "created_at": "2026-02-02T10:00:00Z"
    }
  ]
}
```

### D. 圆桌会议记录格式

```json
{
  "id": "rt_20260202_001",
  "topic": "技术领域拆分提议",
  "type": "restructure",
  "initiator": "lord-tech",
  "chairperson": "king",
  "participants": ["king", "lord-tech", "lord-finance"],
  "speakers": ["lord-tech"],
  "voters": ["lord-finance"],
  "status": "resolved",
  "created_at": "2026-02-02T10:00:00Z",
  "resolved_at": "2026-02-02T11:30:00Z",
  "rounds": [
    {
      "round": 1,
      "speeches": [
        {
          "speaker": "lord-tech",
          "timestamp": "2026-02-02T10:05:00Z",
          "content": "过去一个月，40% 的任务是数据分析相关..."
        }
      ]
    }
  ],
  "votes": [
    {
      "voter": "lord-finance",
      "vote": "approve",
      "reason": "数据分析确实需要专业化",
      "timestamp": "2026-02-02T11:00:00Z"
    }
  ],
  "resolution": {
    "decision": "approved",
    "summary": "批准从技术领域拆分出数据领域",
    "human_review_required": true,
    "human_approved": true,
    "human_approved_at": "2026-02-02T11:30:00Z",
    "human_approver": "admin"
  }
}
```

---

*Version History*
- v4.0: Pure Agent Architecture - Removed frontend/backend/database
- v5.0: **Complete Mechanism Verification** - Focus on Skills/Tools development, added Roundtable mechanism, detailed deliverables list
