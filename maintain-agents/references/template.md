# AGENTS.md 推荐模板

基于实战验证的成熟案例提炼。不是每个项目都需要全部 section，按需选用。

---

## Section 清单（按推荐优先级排序）

### 1. Compact instructions — 上下文压缩优先级
**优先级：高（长 session 项目必备）**
**解决问题：** 长对话被压缩/handoff 时，关键决策信息被丢弃，下一个 agent 重新讨论已决事项。

定义压缩时保留什么、丢弃什么的优先级排序。

✅ 好的写法：
```markdown
## Compact instructions — retention priority
1. 架构决策 — never summarize away
2. 已修改文件 + 关键变更 — file:line + verbatim contract change
3. 验证状态 — pass/fail summary per suite
4. 未解决 TODO + 回滚笔记
5. 工具输出 — discard
```

❌ 差的写法：
```markdown
## 压缩规则
请保留重要信息，丢弃不重要的。
```

---

### 2. Operating mode — 自主决策规则
**优先级：高（所有项目）**
**解决问题：** Agent 不知道何时该自主决策、何时该停下来问。

定义"什么时候问"的边界条件，默认偏向不问。

✅ 好的写法：列出 2-3 个具体的"停下来问"触发条件 + 明确的默认行为
❌ 差的写法："有问题就问用户" 或 "自己判断"

---

### 3. Never do (hard red lines) — 范围边界 / 硬禁令
**优先级：高（自主模式项目必备）**
**解决问题：** Agent 在自主模式下做出不可逆的危险操作。

明确"即使用户说 just do it 也要先确认"的操作。

✅ 好的写法：
```markdown
## Never do
- 不要直接写生产数据库 — 所有改动只走 memory 或本地 dev DB
- 不要 git push --force 到 main
- 不要修改 CI 配置 / 根 pyproject.toml 的工具链段
- 不要为绕过测试而改测试
```

常见红线类别（按需选用）：
- 数据安全：生产 DB、用户数据、密钥
- Git 操作：force-push、删 branch、改 protected
- 基础设施：CI/CD、部署配置、环境变量
- 测试完整性：改测试绕过失败
- API 契约：改 schema 不改 consumer

---

### 4. Coding discipline — 编码纪律
**优先级：高（编码项目）**
**解决问题：** LLM 常见编码毛病——过度工程、改无关代码、不验证就交付。

推荐四条纪律（可增减）：

| 纪律 | 核心约束 |
|------|----------|
| Think before coding | 先说假设再写代码，不确定就声明 |
| Simplicity first | 最少代码解决问题，不写没被要求的功能 |
| Surgical changes | 只改必须改的，不"顺便改进"相邻代码 |
| Goal-driven | 任务转化为可验证目标，循环到验证通过 |

---

### 5. Read first — 必读文档路由
**优先级：高（有多种文档的项目）**
**解决问题：** Agent 不知道不同任务类型该先读哪些文件。

最好用 task-type → file 路由表，而非 flat 列表。

✅ 好的写法：
```markdown
## Read first
MANDATORY: 先读 docs/ARCHITECTURE.md，然后按任务类型：

| Task type | Read before editing |
|---|---|
| Frontend / UI | .impeccable.md + copilot-instructions.md |
| SSE / wire | sse-events.schema.json + wire/events.py |
| Persistence | db.py + *_store.py |
```

❌ 差的写法：
```markdown
## Read first
读 ARCHITECTURE.md
```

---

### 6. Project shape — 项目结构概述
**优先级：中**
**解决问题：** Agent 不了解项目是什么、怎么跑。

用 one-liner 描述每个进程/组件，不超过 5 行。

---

### 7. Commands agent will guess wrong — 防猜错命令
**优先级：高（有自定义 CLI 的项目）**
**解决问题：** Agent 猜错入口命令、安装方式、测试命令。

直接给可复制的命令块，加注释说明"不是 XX"。

✅ 好的写法：
```markdown
pip install -e ".[dev]"   # NOT requirements.txt
voyage-api                 # entry point; NOT python app.py
pytest -q                  # full suite; no Azure required
```

---

### 8. Non-obvious gotchas — 非显而易见的坑
**优先级：高（成熟项目）**
**解决问题：** 项目特有的坑，不写进来 agent 一定会踩。

**关键结构规则：** 超过 10 条必须按关注点分组（如 Wire / Runtime / Persistence / Frontend / Env）。Flat list 会在 context 衰减时导致早期条目被遗忘。

每条 gotcha 应包含：
- 什么被改了/删了
- 旧的路径/API 为什么不能用
- 新的正确做法是什么

❌ 差的做法：把 UI 规范从 .impeccable.md 复制到 gotchas 里 → 双源同步问题。改为引用。

---

### 9. Speckit / workflow — 工作流集成
**优先级：低（使用 speckit 或类似工具的项目）**

用 HTML 注释标记块存放动态状态（active spec、prior specs），保持可机器解析。

---

## 按项目类型的推荐组合

| 项目类型 | 必备 section | 推荐 section | 可选 section |
|----------|-------------|-------------|-------------|
| **编码项目** | Operating mode, Coding discipline, Commands, Gotchas, Never do | Read first, Project shape, Compact instructions | Speckit |
| **知识库/文档** | Operating mode, Compact instructions, Read first | Project shape | Coding discipline |
| **多人协作** | Operating mode, Never do, Gotchas, Read first | Compact instructions, Coding discipline | Commands |
| **有 CI/CD** | Commands, Gotchas, Never do | Coding discipline, Read first | Compact instructions |

---

## 反模式（审查时检查）

1. **Gotchas flat list** — 超过 10 条未分组 → context 衰减风险
2. **双源同步** — 同一规范在 AGENTS.md 和其他文件都有一份 → 改为引用
3. **说明文字过多** — section 里 agent 自己能发现的信息占比 > 50% → 精简
4. **缺路由表** — Read first 是 flat 列表而非 task-type 路由 → 加路由
5. **无红线** — 有 Operating mode 但没 Never do → 自主模式下风险大
6. **超过 400 行** — 考虑拆详细规范到独立文件，AGENTS.md 保留指针
