---
name: doc-workflow
description: 文档工作流程主入口 - 协调需求调查、多维评审、版本管理全流程
---

# 文档工作流程 (Document Workflow)

## 功能说明

协调完整的文档工作流程：
1. **需求调查** - 分析输入源，生成文档初稿
2. **多维评审** - 4 维度质量检查
3. **版本管理** - 追踪变更，生成差异报告
4. **迭代循环** - 直到文档达标

## 使用方式

```bash
# 完整工作流（从 URL）
/doc-workflow https://example.com

# 完整工作流（从本地文档）
/doc-workflow ./requirements.md

# 指定文档类型
/doc-workflow https://example.com --type tech

# 指定评审模式
/doc-workflow https://example.com --review-mode sequential
```

## 参数说明

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `[source]` | 输入源 | 必填 |
| `--type` | 文档类型 | prd |
| `--review-mode` | 评审模式 | parallel |
| `--threshold` | 评分阈值 | 4.0 |
| `--resume` | 从中断点恢复 | false |

## 工作流程

```
┌─────────────────────────────────────────────────────────┐
│                    用户输入源                            │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  Step 1: 需求调查                                        │
│  调用 /doc-investigate 生成初稿 V1.0                     │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  Step 2: 用户确认/修改                                   │
│  等待用户查看并修改文档                                   │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  Step 3: 多维评审                                        │
│  调用 /doc-review 进行 4 维度评审                        │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  Step 4: 版本记录                                        │
│  调用 /doc-version 记录变更                              │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  Step 5: 达标判断                                        │
│  自动评分 + 人工确认                                     │
└─────────────────────────────────────────────────────────┘
                          │
              ┌───────────┴───────────┐
              │                       │
              ▼                       ▼
        ┌──────────┐           ┌──────────┐
        │  未达标   │           │   达标    │
        │ 返回 Step 2│          │  归档完成  │
        └──────────┘           └──────────┘
```

## 执行流程详解

### Step 1: 需求调查

使用 Task 工具启动 investigate-agent：

```yaml
agent: "investigate-agent"
skill: "/doc-investigate"
input:
  source: "用户输入源"
  type: "prd"
output:
  doc_path: "docs/prd/PRD-xxx-V1.0.md"
  version: "V1.0"
```

### Step 2: 用户确认/修改

```
文档初稿已生成: docs/prd/PRD-xxx-V1.0.md

请查看文档并进行必要的修改。完成后请选择:
A) 继续评审 - 文档已准备好
B) 添加备注 - 记录修改说明
C) 暂停工作流 - 稍后继续
```

### Step 3: 多维评审

使用 Task 工具启动 review-agent：

```yaml
agent: "review-agent"
skill: "/doc-review"
input:
  doc_path: "docs/prd/PRD-xxx-V1.1.md"
  mode: "parallel"
  threshold: 4.0
output:
  report_path: "docs/reviews/REVIEW-xxx.md"
  scores: { format: 4.5, arch: 4.0, match: 3.5, innovation: 4.0 }
  passed: true/false
```

### Step 4: 版本记录

自动调用 /doc-version 记录变更。

### Step 5: 达标判断

```yaml
自动评分:
  passed = (
    scores.format >= threshold AND
    scores.arch >= threshold AND
    scores.match >= threshold AND
    high_issues == 0 AND
    medium_issues <= 3
  )

人工确认:
  如果自动评分通过:
    "文档已通过自动评分，是否确认达标？"
    A) 确认达标，归档文档
    B) 继续迭代
    C) 查看详细报告
```

## 状态保存

工作流状态保存到: `docs/.doc-workflow-state.json`

```json
{
  "current_stage": "review",
  "completed_stages": ["investigate"],
  "current_doc": "docs/prd/PRD-xxx-V1.1.md",
  "iteration": 2,
  "last_updated": "2026-02-05T10:30:00Z"
}
```

## 恢复中断

```bash
/doc-workflow --resume
```

读取状态文件，从中断点继续。

## 示例输出

```
=== 文档工作流程 ===

输入源: https://example.com
文档类型: PRD
评审模式: parallel

Step 1: 需求调查
├── 输入源类型: URL
├── 解析中...
├── ✓ 解析完成: 提取了 5 个功能模块
└── ✓ 初稿生成: docs/prd/PRD-Example-V1.0.md

Step 2: 用户确认
└── 等待用户操作...

[用户选择: 继续评审]

Step 3: 多维评审
├── 评审模式: parallel
├── ✓ 格式评审: 4.5/5
├── ✓ 架构评审: 4.0/5
├── ✓ 匹配评审: 4.0/5
└── ✓ 创新评审: 4.0/5

Step 4: 版本记录
└── ✓ V1.0 → V1.1 已记录

Step 5: 达标判断
├── 综合评分: 4.1/5
├── 自动评分: 通过 ✓
└── 等待人工确认...

[用户选择: 确认达标]

=== 工作流程完成 ===
最终文档: docs/prd/PRD-Example-V1.1.md
评审报告: docs/reviews/REVIEW-Example-V1.1-2026-02-05.md
版本历史: docs/versions/VERSION-Example-history.md
```
