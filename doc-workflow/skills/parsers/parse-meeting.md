---
name: parse-meeting
description: 解析会议记录，提取需求和决策
---

# 会议记录解析器 (Meeting Parser)

## 功能说明

解析会议记录文档，提取：
- 会议主题和参与者
- 讨论要点
- 决策事项
- 待办任务
- 需求变更

## 使用方式

此技能由 `/doc-investigate` 内部调用，不建议直接使用。

调用参数：`--meeting "会议记录文件路径或会议主题"`

## 解析流程

1. **会议信息识别**
   - 提取会议日期、时间
   - 识别参与者列表
   - 确定会议主题

2. **内容结构化**
   - 识别议题和讨论点
   - 提取决策和结论
   - 识别待办事项

3. **需求提取**
   - 从讨论中提取需求
   - 识别需求变更
   - 确定优先级调整

4. **输出整理**
   - 按时间线组织
   - 关联相关文档

## 输出格式

```yaml
meeting_info:
  date: "2026-02-05"
  topic: "会议主题"
  participants: ["参与者1", "参与者2"]
discussions:
  - topic: "讨论主题"
    points: ["要点1", "要点2"]
    conclusion: "结论"
decisions:
  - id: "DEC-001"
    description: "决策描述"
    owner: "负责人"
action_items:
  - id: "ACT-001"
    task: "任务描述"
    assignee: "负责人"
    due_date: "截止日期"
requirements:
  - id: "REQ-001"
    description: "需求描述"
    source: "来源讨论"
    priority: "P0/P1/P2"
```

## 错误处理

- 格式不规范：尝试智能识别结构
- 信息不完整：标记缺失字段
- 多次会议混合：按日期分离