---
name: doc-review
description: 多维评审技能 - 对文档进行格式、架构、匹配性、创新四个维度的评审
---

# 多维评审技能 (Document Review)

## 功能说明

对需求文档进行多维度质量评审：
- **格式评审**: 语言规范、格式规范、完整性
- **架构评审**: 结构合理性、逻辑一致性、可行性
- **匹配评审**: 需求覆盖度、内容准确性
- **创新评审**: 功能增强、体验优化建议

## 使用方式

```bash
# 默认并行评审
/doc-review ./docs/prd/PRD-xxx.md

# 指定评审模式
/doc-review ./docs/prd/PRD-xxx.md --mode sequential

# 指定评分阈值
/doc-review ./docs/prd/PRD-xxx.md --threshold 4.5
```

## 参数说明

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `[doc_path]` | 待评审文档路径 | 必填 |
| `--mode` | 评审模式 | parallel |
| `--threshold` | 评分阈值 | 4.0 |

## 评审模式

### 完全并行 (parallel)

```
4 个评审 Agent 同时执行
├── 格式评审 Agent ──┐
├── 架构评审 Agent ──┼── 汇总 → 综合报告
├── 匹配评审 Agent ──┤
└── 创新评审 Agent ──┘
```

**适用场景**: 追求效率、文档较成熟

### 分组执行 (grouped)

```
第一组（并行）:
├── 格式评审 Agent ──┐
├── 架构评审 Agent ──┼── 汇总
└── 匹配评审 Agent ──┘
        ↓
第二组（基于第一组结果）:
└── 创新评审 Agent → 综合报告
```

**适用场景**: 希望创新建议基于基础评审结果

### 顺序执行 (sequential)

```
格式评审 → 架构评审 → 匹配评审 → 创新评审 → 综合报告
```

**适用场景**: 文档初稿、需要深度分析

## 执行流程

### 1. 模式选择

如果未指定模式，交互式询问用户：

```
请选择评审执行方式:
A) 完全并行 - 4个评审同时执行，速度最快
B) 分组执行 - 先基础评审，再创新扩展
C) 顺序执行 - 依次执行，深度分析
```

### 2. 启动评审 Agent

使用 Task 工具启动独立 Agent：

```yaml
格式评审:
  agent: "format-review-agent"
  input: { doc_path: "文档路径" }
  output: { score: number, issues: list }

架构评审:
  agent: "arch-review-agent"
  input: { doc_path: "文档路径" }
  output: { score: number, issues: list }

匹配评审:
  agent: "match-review-agent"
  input: { doc_path: "文档路径" }
  output: { score: number, matrix: list }

创新评审:
  agent: "innovation-review-agent"
  input: { doc_path: "文档路径", base_review: object }
  output: { suggestions: list }
```

### 3. 汇总评审结果

1. 收集各维度评分和问题
2. 计算综合评分
3. 生成评审报告
4. 判断是否通过阈值

### 4. 输出评审报告

保存到: `docs/reviews/REVIEW-{项目名}-{版本}-{日期}.md`

## 评分标准

| 维度 | 评分项 | 权重 |
|------|--------|------|
| 格式 | 语言规范、格式规范、完整性 | 25% |
| 架构 | 结构合理性、逻辑一致性、可行性 | 30% |
| 匹配 | 需求覆盖度、内容准确性 | 30% |
| 创新 | 建议质量、可行性 | 15% |

## 示例输出

```
评审模式: 完全并行
评审文档: docs/prd/PRD-Example-V1.0.md

评审进度:
✓ 格式评审完成: 4.5/5
✓ 架构评审完成: 4.0/5
✓ 匹配评审完成: 3.5/5
✓ 创新评审完成: 4.0/5

综合评分: 4.0/5
评审结论: 通过 ✓

问题统计:
- 高严重度: 0
- 中严重度: 2
- 低严重度: 5

评审报告已保存: docs/reviews/REVIEW-Example-V1.0-2026-02-05.md
```
