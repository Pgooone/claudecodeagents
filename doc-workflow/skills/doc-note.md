---
name: doc-note
description: 备注记录技能 - 手动添加备注和讨论要点
---

# 备注记录技能 (Document Note)

## 功能说明

手动添加备注和讨论要点，支持：
- 普通备注
- 会议记录
- 决策记录
- 关联文档

## 使用方式

```bash
# 添加普通备注
/doc-note "功能优先级调整：语音克隆提升至 P0"

# 添加会议记录
/doc-note --meeting "2026-02-05 需求评审会"

# 添加决策记录
/doc-note --decision "采用 WebSocket 实现实时通信"

# 关联到特定文档
/doc-note "补充安全需求" --related ./docs/prd/PRD-xxx.md
```

## 参数说明

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `[content]` | 备注内容 | 必填 |
| `--meeting` | 会议主题 | - |
| `--decision` | 决策主题 | - |
| `--related` | 关联文档 | - |
| `--tags` | 标签列表 | - |

## 备注类型

### 普通备注 (note)

```markdown
---
type: note
created: 2026-02-05 10:30
related: docs/prd/PRD-xxx-V1.2.md
tags: [需求变更, 优先级]
---

# 功能优先级调整

将语音克隆功能优先级从 P1 提升至 P0。

原因：
- 用户反馈强烈
- 竞品已上线类似功能
```

### 会议记录 (meeting)

```markdown
---
type: meeting
date: 2026-02-05
topic: 需求评审会
participants: [张三, 李四, 王五]
related: docs/prd/PRD-xxx-V1.2.md
---

# 2026-02-05 需求评审会

## 议题

1. 功能优先级调整
2. 技术方案确认

## 决策

- 语音克隆提升至 P0
- 采用 WebSocket 方案

## 待办

- [ ] 更新 PRD 文档
- [ ] 补充技术方案
```

### 决策记录 (decision)

```markdown
---
type: decision
date: 2026-02-05
topic: 实时通信方案
decision_maker: 技术负责人
related: docs/prd/PRD-xxx-V1.2.md
---

# 决策：采用 WebSocket 实现实时通信

## 背景

需要实现实时语音预览功能。

## 方案对比

| 方案 | 优点 | 缺点 |
|------|------|------|
| WebSocket | 双向通信、低延迟 | 需要维护连接 |
| SSE | 简单、单向 | 不支持双向 |
| 轮询 | 兼容性好 | 延迟高、资源消耗 |

## 决策

采用 WebSocket 方案。

## 理由

- 需要双向通信
- 对延迟要求高
- 团队有相关经验
```

## 存储位置

```
docs/notes/
├── NOTE-{项目名}-{日期}-{序号}.md
├── MEETING-{项目名}-{日期}.md
└── DECISION-{项目名}-{日期}-{序号}.md
```

## 示例输出

```
备注类型: 普通备注
备注内容: 功能优先级调整：语音克隆提升至 P0

✓ 备注已保存: docs/notes/NOTE-Example-2026-02-05-001.md
✓ 已关联文档: docs/prd/PRD-Example-V1.2.md
```
