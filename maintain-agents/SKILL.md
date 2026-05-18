---
name: maintain-agents
description: 审查并维护 AGENTS.md——支持 session 收工时增量更新，也支持对现有内容做完整性审查。触发词："收工"、"结束"、"更新规则"、"sync agents"、"审查 agents"、"review agents"、"agents 健康检查"。
---

## 触发时机

**模式 A — Session 收工更新：**
- "收工"、"结束"、"更新规则"、"sync agents"
- 完成了重大架构变更、引入新依赖、或修改了代码规范后

**模式 B — 完整性审查：**
- "审查 agents"、"review agents"、"agents 健康检查"
- 首次为项目创建 AGENTS.md 时
- 用户主动要求检查 AGENTS.md 是否有遗漏

---

## 模式 A: Session 收工更新

### Step 1: 读取并理解当前 AGENTS.md

读取项目根目录的 AGENTS.md。如果不存在，提醒用户先创建。

**关键：不要假设任何固定的 section 结构。** 每个项目的 AGENTS.md 组织方式不同。你必须：
1. 列出当前所有 `##` level 的 section 及其用途
2. 理解每个 section 的写作风格（命令式？叙述式？列表？）
3. 后续所有修改必须匹配已有风格和结构

### Step 2: 回顾本次 session

检查本次 session 中是否发生了以下任何一项：

- [ ] 新增或修改了架构决策（如换了依赖、改了数据模型、调整了 API 设计）
- [ ] 引入了新的依赖或工具
- [ ] 发现了 agent 容易猜错的命令或行为
- [ ] 修改或新增了代码规范 / 编码纪律
- [ ] 发现了新的 gotcha（非显而易见的坑）
- [ ] 删除或重命名了重要模块/文件/API
- [ ] 范围边界发生变化

如果以上全部为否，告知用户"本次 session 无需更新 AGENTS.md"，流程结束。

### Step 3: 定位变更归属

对每个需要记录的变更，在 **现有 section 结构** 中找到最合适的归属位置：

1. 扫描已有 section，判断变更内容属于哪个 section
2. 如果没有合适的 section，**建议新增一个**，但命名和层级必须与已有结构一致
3. 绝不往不相关的 section 塞内容

### Step 4: 生成变更提案

对每个变更条目，生成具体的修改内容：

- **追加到已有 section**：匹配该 section 现有的格式（bullet style、日期格式、语气）
- **新增 section**：给出完整的 section 标题和初始内容，放在逻辑上合理的位置
- **修改已有条目**：如果新信息使旧条目过时（如模块被删除/重命名），更新旧条目而不是追加矛盾信息

### Step 5: 检查一致性

在提交变更前，检查：

- **矛盾检测**：新内容是否与 AGENTS.md 中已有内容矛盾？如果是，标记出来并建议同时更新旧内容
- **过时引用**：是否有引用了已删除文件/模块/API 的条目？标记建议清理
- **文件健康度**：如果 AGENTS.md 超过 400 行，建议将详细规范拆到独立文件

### Step 6: 输出并确认

将所有变更以 diff 格式展示给用户：

```
📋 AGENTS.md 变更提案：

[Section名] + 新增内容概述
[Section名] ~ 修改: 旧内容 → 新内容
[新Section] + 建议新增 "## Section名"
[矛盾]      ! 第XX行与新变更矛盾，建议同时更新

确认更新？(y/n)
```

用户确认后执行写入。

---

## 模式 B: 完整性审查

对现有 AGENTS.md 做全面体检，检查是否缺少对 agent 有效工作至关重要的信息类别。

### Step 1: 读取现有内容

读取 AGENTS.md，列出所有现有 section。如果文件不存在，进入"从零创建"流程——用推荐模板（见 references/template.md）引导用户逐项填写。

### Step 2: 对照推荐模板

读取 references/template.md 获取推荐的 section 结构和检查清单。模板基于实战验证的成熟 AGENTS.md 案例，包含：

- 每个推荐 section 的用途说明
- 好 vs 坏的写法示例
- 按项目类型（编码/知识库/多人协作）的优先级建议

对比现有 AGENTS.md 与模板，识别：
1. 哪些 section 已覆盖（可能名字不同但内容等价）
2. 哪些 section 完全缺失
3. 哪些 section 有但内容过于空泛或存在结构问题

### Step 3: 结构质量检查

除了内容缺失，还要检查已有 section 的结构健康度：

- **Gotchas 是否分组？** 如果 gotchas 超过 10 条且未按关注点分组（如 wire / runtime / persistence / frontend），建议分组。长 flat list 会导致 agent 在 context 衰减时遗忘早期条目。
- **是否有双源同步问题？** 如果某个 section 的内容在其他文件中也有一份（如 UI 规范同时写在 AGENTS.md 和 .impeccable.md），建议改为引用而非复制。
- **Read first 是否有路由？** 如果项目有多种任务类型（前端/后端/SSE/持久层），建议加 task-type → file 路由表，而不是一个 flat 列表。
- **说明文字 vs 可执行指令比例** — 如果某个 section 的说明文字（agent 自己能发现的）占比超过内容的 50%，建议精简。

### Step 4: 生成审查报告

输出格式：

```
📋 AGENTS.md 完整性审查：

✅ 已覆盖：
  - 自主决策规则 ("## Operating mode")
  - 编码纪律 ("## Coding discipline")
  - ...

⚠️ 建议补充（按优先级排序）：
  1. [高] 防猜错命令 — 项目有自定义 CLI，agent 容易猜错入口
  2. [中] 上下文压缩优先级 — 长 session 后容易丢失关键架构信息
  3. [低] 范围边界 — 目前项目范围比较明确

🔧 结构改进建议：
  1. Gotchas 分组 — 当前 15 条 flat list，建议按 wire/runtime/persistence 分组
  2. Frontend 规范去重 — 改为引用 .impeccable.md 而非复制

✅ 不需要：
  - 工具硬性禁止 — 项目无特殊禁令

要我帮你生成缺失 section 的草稿吗？
```

### Step 5: 按需生成草稿

用户确认需要补充的类别后：
1. 扫描项目代码和现有文档，提取相关信息
2. 参考 references/template.md 中对应 section 的写法示例
3. 生成 section 草稿，匹配已有 AGENTS.md 的风格
4. 展示给用户确认后写入

---

## 写入规范

- 匹配已有 AGENTS.md 的日期格式、语气、列表风格
- 如果已有文件没有日期标注习惯，不要强加日期
- 不要删除任何现有内容，除非它与新变更直接矛盾且用户确认
- 架构决策类变更必须包含原因

## 不要做的事

- 不要假设固定的 section 结构——每个项目不同
- 不要自动执行写入，必须等用户确认
- 不要修改代码文件，只维护 AGENTS.md
- 不要编造决策，只记录本次 session 中实际发生的事
- 不要把临时调试信息写入 AGENTS.md
- 不要重新组织已有 section 的顺序或命名
- 审查模式下不要强制要求补全所有类别——按项目实际需要建议
