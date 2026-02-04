---
name: parse-chat
description: 通过交互式对话收集需求信息
---

# 对话解析器 (Chat Parser)

## 功能说明

通过交互式问答收集用户需求，适用于：
- 口述需求整理
- 需求访谈记录
- 头脑风暴整理

## 使用方式

此技能由 `/doc-investigate` 内部调用。

调用参数：`--chat` 进入交互模式

## 交互流程

### 阶段 1: 项目基础信息

```
Q1: 请简要描述您的项目或产品是什么？
Q2: 这个项目主要解决什么问题？
Q3: 目标用户是谁？
```

### 阶段 2: 功能需求

```
Q4: 核心功能有哪些？请列举主要功能点。
Q5: 这些功能的优先级如何排序？
Q6: 有没有参考的竞品或类似产品？
```

### 阶段 3: 技术约束

```
Q7: 有没有特定的技术要求或限制？
Q8: 对性能有什么要求？
Q9: 有没有安全或合规方面的考虑？
```

### 阶段 4: 确认与补充

```
Q10: 还有什么需要补充的信息吗？
```

## 输出格式

```yaml
chat_session:
  start_time: "开始时间"
  end_time: "结束时间"
  questions_answered: 10
collected_info:
  project:
    name: "项目名称"
    description: "项目描述"
    problem_solved: "解决的问题"
  users:
    - type: "用户类型"
      description: "用户描述"
  features:
    - name: "功能名称"
      description: "功能描述"
      priority: "P0/P1/P2"
  constraints:
    technical: ["技术约束"]
    performance: ["性能要求"]
    security: ["安全要求"]
  references:
    - name: "参考产品"
      url: "URL"
  additional_notes: "补充信息"
```

## 交互技巧

- 使用 AskUserQuestion 工具进行多选题
- 对模糊回答进行追问澄清
- 自动总结并确认理解是否正确

## 错误处理

- 用户中断对话：保存已收集的信息，提示可继续
- 回答过于简短：自动追问获取更多细节
- 跳过问题：标记为待补充，继续下一问题