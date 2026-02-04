# AI Agents 文档工作流程系统 - 完整设计文档

> 创建日期 (Created): 2026-02-05
> 版本 (Version): 1.0
> 状态 (Status): 设计完成，待实现

---

## 目录 (Table of Contents)

1. [设计决策汇总](#设计决策汇总)
2. [第 1 部分：技能包结构](#第-1-部分技能包结构)
3. [第 2 部分：核心技能定义](#第-2-部分核心技能定义)
4. [第 3 部分：多 Agent 架构](#第-3-部分多-agent-架构)
5. [第 4 部分：数据流与存储](#第-4-部分数据流与存储)
6. [第 5 部分：版本管理与自动触发](#第-5-部分版本管理与自动触发)
7. [第 6 部分：达标判断与迭代循环](#第-6-部分达标判断与迭代循环)
8. [第 7 部分：修改记录整合](#第-7-部分修改记录整合)
9. [第 8 部分：配置系统](#第-8-部分配置系统)
10. [第 9 部分：错误处理与异常情况](#第-9-部分错误处理与异常情况)
11. [第 10 部分：扩展性设计](#第-10-部分扩展性设计)

---

## 设计决策汇总

| 决策项 | 选择 | 说明 |
|--------|------|------|
| 实现形态 | Claude Code 技能包 | 保留扩展到独立应用和 MCP 服务器的能力 |
| 主要场景 | PRD 为主 | 同时支持技术设计文档等其他类型 |
| 技能组织 | 混合模式 | `/doc-workflow` 主入口 + 独立子技能可单独调用 |
| 评审维度 | 4 个 | 格式、架构、匹配性、创新 |
| 存储方式 | 项目目录结构 | `docs/prd/`、`docs/reviews/`、`docs/versions/` |
| 输入来源 | 多种输入源 | 通过加载不同 skills 解析（URL、文档、会议、对话） |
| 评审执行 | 用户可配置 | 支持并行、分组、顺序三种方式 |
| 版本管理 | 自动触发 | 每次文档修改后自动生成版本记录 |
| 修改记录 | 全部支持 | Git 集成 + 对话记录 + 显式备注 |
| 达标判断 | 混合标准 | 自动评分 + 人工确认双重机制 |
| 包结构 | 单一技能包 | 统一管理，便于安装和版本控制 |

---

## 第 1 部分：技能包结构

### 设计说明 (Design Notes)
本部分定义了技能包的整体目录结构和文件组织方式。采用单一技能包架构，将所有功能模块整合在一起，便于统一管理、版本控制和用户安装。

### 目录结构 (Directory Structure)

```
doc-workflow/
├── skills/                          # 技能定义目录
│   ├── doc-workflow.md              # 主入口：协调全流程
│   ├── doc-investigate.md           # 需求调查：生成文档初稿
│   ├── doc-review.md                # 多维评审：4维度质量检查
│   ├── doc-version.md               # 版本管理：追踪变更
│   ├── doc-note.md                  # 备注记录：手动添加讨论要点
│   └── parsers/                     # 输入源解析器目录
│       ├── parse-url.md             # 解析网站 URL
│       ├── parse-doc.md             # 解析本地文档
│       ├── parse-meeting.md         # 解析会议记录
│       └── parse-chat.md            # 解析对话/口述需求
├── templates/                       # 文档模板目录
│   ├── prd-template.md              # PRD 模板
│   ├── tech-design-template.md      # 技术设计模板
│   └── review-report-template.md    # 评审报告模板
└── config/
    └── default-config.yaml          # 默认配置文件
```

### 文件职责说明 (File Responsibilities)

| 文件 | 职责 | 调用方式 |
|------|------|----------|
| doc-workflow.md | 主入口，协调全流程 | `/doc-workflow` |
| doc-investigate.md | 需求调查，生成初稿 | `/doc-investigate` 或由主入口调用 |
| doc-review.md | 多维评审，质量检查 | `/doc-review` 或由主入口调用 |
| doc-version.md | 版本管理，追踪变更 | `/doc-version` 或自动触发 |
| doc-note.md | 备注记录，手动添加 | `/doc-note` |
| parse-*.md | 输入源解析器 | 由 doc-investigate 内部调用 |

---

## 第 2 部分：核心技能定义

### 设计说明 (Design Notes)
本部分定义了主入口技能 `/doc-workflow` 的详细功能、参数和工作流程。主入口技能负责协调整个文档工作流程，自动调度各子技能完成从需求调查到最终归档的全过程。

### `/doc-workflow` - 主入口技能

**触发方式 (Invocation)**：`/doc-workflow [输入源]`

**功能描述 (Description)**：协调全流程，自动调度各子技能

**工作流程 (Workflow)**：
```
1. 解析输入源 → 调用对应 parser
2. 生成初稿 → 调用 /doc-investigate
3. 用户确认/修改 → 等待人工介入
4. 多维评审 → 调用 /doc-review（用户选择执行方式）
5. 自动版本记录 → 调用 /doc-version
6. 达标判断 → 自动评分 + 人工确认
7. 未达标则循环 3-6，达标则归档
```

**参数说明 (Parameters)**：

| 参数 | 说明 | 默认值 | 可选值 |
|------|------|--------|--------|
| `--type` | 文档类型 | prd | prd / tech / custom |
| `--review-mode` | 评审方式 | parallel | parallel / grouped / sequential |
| `--threshold` | 评分阈值 | 4/5 | 1.0 ~ 5.0 |
| `--auto-commit` | 自动 Git 提交 | false | true / false |

**输出目录结构 (Output Structure)**：
```
docs/
├── prd/
│   └── PRD-{项目名}-{日期}.md
├── reviews/
│   └── REVIEW-{项目名}-{版本}-{日期}.md
└── versions/
    └── VERSION-{项目名}-history.md
```

### `/doc-investigate` - 需求调查技能

**触发方式 (Invocation)**：`/doc-investigate [输入源]`

**功能描述 (Description)**：分析输入源，生成标准化需求文档初稿

**支持的输入源 (Supported Input Sources)**：

| 输入类型 | 示例 | 调用的 Parser |
|----------|------|---------------|
| URL | `https://example.com` | parse-url |
| 本地文件 | `./meeting-notes.md` | parse-doc |
| 会议记录 | `--meeting "2024-01-15会议"` | parse-meeting |
| 口述需求 | `--chat` (进入交互模式) | parse-chat |

**输出 (Output)**：
- 生成 `docs/prd/PRD-{项目名}-{日期}.md`
- 自动调用 `/doc-version` 记录 V1.0

### `/doc-review` - 多维评审技能

**触发方式 (Invocation)**：`/doc-review [文档路径] [--mode parallel|grouped|sequential]`

**4 个评审维度 (4 Review Dimensions)**：

| 维度 | 检查内容 | 输出 |
|------|----------|------|
| 格式评审 (Format) | 语言规范、格式规范、完整性 | 问题清单（位置、类型、严重度） |
| 架构评审 (Architecture) | 结构合理性、逻辑一致性、可行性 | 评分报告（1-5分） |
| 匹配评审 (Matching) | 需求覆盖度、内容准确性 | 追溯矩阵（✅/⚠️/❌） |
| 创新评审 (Innovation) | 功能增强、体验优化建议 | 创新建议清单 |

**执行方式选择（交互式）(Execution Mode Selection)**：
```
请选择评审执行方式 (Please select review execution mode)：
A) 完全并行 (Parallel) - 4个评审同时执行，速度最快，适合成熟文档
B) 分组执行 (Grouped) - 先基础评审，再创新扩展，创新建议更有针对性
C) 顺序执行 (Sequential) - 依次执行，深度分析，适合初稿
```

### `/doc-version` - 版本管理技能

**触发方式 (Invocation)**：`/doc-version [文档路径]` 或自动触发

**功能描述 (Description)**：追踪文档变更，生成差异报告

### `/doc-note` - 备注记录技能

**触发方式 (Invocation)**：`/doc-note "备注内容"` 或 `/doc-note --meeting "会议主题"`

**功能描述 (Description)**：手动添加备注和讨论要点

---

## 第 3 部分：多 Agent 架构

### 设计说明 (Design Notes)
本部分定义了多 Agent 协作架构。为了保持主 Agent 上下文精简，支持大型项目的持续迭代，每个子技能由独立的 Agent 执行。主 Agent 只负责协调和调度，子 Agent 专注于单一任务，完成后返回结果给主 Agent。

### Agent 调度架构 (Agent Scheduling Architecture)

```
┌─────────────────────────────────────────────────────────┐
│              主 Agent (doc-workflow)                     │
│  - 协调调度 (Coordination)                               │
│  - 状态管理 (State Management)                           │
│  - 用户交互 (User Interaction)                           │
│  - 结果汇总 (Result Aggregation)                         │
└─────────────────────────────────────────────────────────┘
         │              │              │              │
         ▼              ▼              ▼              ▼
    ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐
    │ Agent 1 │   │ Agent 2 │   │ Agent 3 │   │ Agent 4 │
    │调查Agent│   │评审Agent│   │版本Agent│   │备注Agent│
    │Investigate│ │ Review  │   │ Version │   │  Note   │
    └─────────┘   └─────────┘   └─────────┘   └─────────┘
                       │
         ┌─────────────┼─────────────┐
         ▼             ▼             ▼
    ┌─────────┐   ┌─────────┐   ┌─────────┐
    │格式评审 │   │架构评审 │   │匹配评审 │  (可并行 Parallelizable)
    │Format   │   │Arch     │   │Match    │
    │SubAgent │   │SubAgent │   │SubAgent │
    └─────────┘   └─────────┘   └─────────┘
```

### 技能调用方式 (Skill Invocation Method)

每个子技能通过 `Task` 工具启动独立 agent：

```yaml
主 Agent → 调查 Agent (Main → Investigate):
  input:
    - source_type: "url" | "file" | "meeting" | "chat"  # 输入源类型
    - source_content: string | file_path                 # 输入源内容
    - template: "prd" | "tech" | "custom"               # 模板类型
    - output_dir: "docs/prd/"                           # 输出目录
  output:
    - doc_path: "docs/prd/PRD-xxx-V1.0.md"              # 文档路径
    - version: "V1.0"                                    # 版本号
    - summary: "生成了包含 X 个功能模块的 PRD"           # 摘要

主 Agent → 评审 Agent (Main → Review):
  input:
    - doc_path: string                                   # 文档路径
    - review_mode: "parallel" | "grouped" | "sequential" # 评审模式
    - thresholds: { format: 4, arch: 4, match: 4 }      # 阈值配置
  output:
    - report_path: "docs/reviews/REVIEW-xxx.md"         # 报告路径
    - scores: { format: 4.5, arch: 4.0, match: 3.5, innovation: 4.0 }  # 评分
    - issues: [{ severity: "high", location: "2.3", desc: "..." }]     # 问题列表
    - passed: boolean                                    # 是否通过

主 Agent → 版本 Agent (Main → Version):
  input:
    - current_doc: string                               # 当前文档路径
    - previous_doc: string | null                       # 前一版本路径
  output:
    - diff_report: "docs/versions/VERSION-xxx.md"       # 差异报告路径
    - stats: { added: 5, modified: 12, deleted: 2 }     # 变更统计
```

### 上下文隔离策略 (Context Isolation Strategy)

| Agent | 上下文内容 (Context Content) | 预估 Token (Est. Tokens) |
|-------|------------------------------|--------------------------|
| 主 Agent (Main) | 流程状态、用户交互、结果摘要 | ~2K |
| 调查 Agent (Investigate) | 输入源内容、模板、生成逻辑 | ~10K |
| 评审 Agent (Review) | 文档内容、评审规则、报告生成 | ~8K |
| 版本 Agent (Version) | 两版本内容、差异算法 | ~6K |

---

## 第 4 部分：数据流与存储

### 设计说明 (Design Notes)
本部分定义了项目目录结构和 Agent 间的数据传递格式。采用标准化的目录结构存储所有文档和报告，确保数据的可追溯性和一致性。

### 项目目录结构 (Project Directory Structure)

当用户在项目中使用 `/doc-workflow` 时，自动创建以下目录：

```
{项目根目录}/
└── docs/
    ├── prd/                          # 需求文档 (PRD Documents)
    │   ├── PRD-{项目名}-V1.0.md      # 初稿 (Initial Draft)
    │   ├── PRD-{项目名}-V1.1.md      # 迭代版本 (Iteration)
    │   └── PRD-{项目名}-V1.2.md      # 当前版本 (Current)
    ├── reviews/                      # 评审报告 (Review Reports)
    │   ├── REVIEW-{项目名}-V1.0-{日期}.md
    │   └── REVIEW-{项目名}-V1.1-{日期}.md
    ├── versions/                     # 版本历史 (Version History)
    │   └── VERSION-{项目名}-history.md  # 完整版本链
    └── notes/                        # 讨论备注 (Discussion Notes)
        ├── NOTE-{项目名}-{日期}-{序号}.md
        └── MEETING-{项目名}-{日期}.md
```

### Agent 间数据传递格式 (Inter-Agent Data Format)

```yaml
主 Agent → 调查 Agent (Main → Investigate Agent):
  input:
    source_type: "url" | "file" | "meeting" | "chat"  # 输入源类型
    source_content: string | file_path                 # 输入源内容或路径
    template: "prd" | "tech" | "custom"               # 使用的模板类型
    output_dir: "docs/prd/"                           # 输出目录
    project_name: string                               # 项目名称
  output:
    doc_path: "docs/prd/PRD-xxx-V1.0.md"              # 生成的文档路径
    version: "V1.0"                                    # 版本号
    summary: string                                    # 生成摘要
    sections_count: number                             # 章节数量
    features_count: number                             # 功能点数量

主 Agent → 评审 Agent (Main → Review Agent):
  input:
    doc_path: string                                   # 待评审文档路径
    review_mode: "parallel" | "grouped" | "sequential" # 评审执行模式
    thresholds:                                        # 评分阈值
      format_threshold: 4.0                            # 格式评审阈值
      arch_threshold: 4.0                              # 架构评审阈值
      match_threshold: 4.0                             # 匹配评审阈值
  output:
    report_path: "docs/reviews/REVIEW-xxx.md"         # 评审报告路径
    scores:                                            # 各维度评分
      format: 4.5
      arch: 4.0
      match: 3.5
      innovation: 4.0
    issues:                                            # 问题列表
      - severity: "high" | "medium" | "low"
        location: "2.3.1"
        description: string
        suggestion: string
    passed: boolean                                    # 是否通过自动评分

主 Agent → 版本 Agent (Main → Version Agent):
  input:
    current_doc: string                               # 当前版本文档路径
    previous_doc: string | null                       # 前一版本文档路径
    change_source: "review" | "manual" | "auto"       # 变更来源
  output:
    diff_report: "docs/versions/VERSION-xxx.md"       # 差异报告路径
    stats:                                             # 变更统计
      added: number                                    # 新增数量
      modified: number                                 # 修改数量
      deleted: number                                  # 删除数量
    change_patterns:                                   # 修改模式分析
      requirement_expansion: "40%"                     # 需求扩展占比
      detail_refinement: "35%"                         # 细节完善占比
      error_correction: "25%"                          # 错误修正占比
```

---

## 第 5 部分：版本管理与自动触发

### 设计说明 (Design Notes)
本部分定义了版本管理的自动触发机制、版本号规则和差异报告格式。版本管理采用自动触发方式，每次文档修改后自动生成版本记录和差异报告，确保完整的变更追溯。

### 自动触发机制 (Auto-Trigger Mechanism)

```yaml
触发条件 (Trigger Conditions):
  - 文档首次生成（V1.0）(Initial document generation)
  - 用户通过 Claude Code 修改文档后 (After user edits via Claude Code)
  - 评审完成并应用修改后 (After review and applying changes)
  - 用户显式调用 /doc-version (Explicit invocation)

触发流程 (Trigger Flow):
  1. 检测文档变更（文件 hash 对比）(Detect changes via file hash)
  2. 启动 version-agent (Launch version agent)
  3. 生成差异报告 (Generate diff report)
  4. 更新版本历史文件 (Update version history)
  5. Git commit（可选，用户配置）(Optional Git commit)
```

### 版本号规则 (Version Numbering Rules)

```yaml
格式 (Format): V{主版本}.{次版本} (V{major}.{minor})

版本说明 (Version Description):
  V1.0 - 初稿（调查 Agent 生成）(Initial draft from Investigate Agent)
  V1.1 - 首次评审后修改 (After first review)
  V1.2 - 二次评审后修改 (After second review)
  V2.0 - 重大重构（用户手动升级主版本）(Major restructure, manual upgrade)

主版本升级条件 (Major Version Upgrade Conditions):
  - 需求范围重大变更 (Significant scope change)
  - 架构重新设计 (Architecture redesign)
  - 用户显式请求 (Explicit user request)
```

### 差异报告格式 (Diff Report Format)

```markdown
# 版本差异报告 (Version Diff Report): V1.1 → V1.2

## 变更统计 (Change Statistics)
- 新增 (Added): 3 处
- 修改 (Modified): 8 处
- 删除 (Deleted): 1 处

## 详细变更 (Detailed Changes)
| 位置 (Location) | 类型 (Type) | 原内容 (Original) | 新内容 (New) | 变更原因 (Reason) |
|-----------------|-------------|-------------------|--------------|-------------------|
| 2.3.1 | 修改 | "支持3种格式" | "支持5种格式" | 评审建议 |
| 3.1 | 新增 | - | "新增安全模块" | 需求扩展 |

## 修改模式分析 (Change Pattern Analysis)
- 需求扩展 (Requirement Expansion): 40%
- 细节完善 (Detail Refinement): 35%
- 错误修正 (Error Correction): 25%

## 修改热区 (Change Hotspots)
- 第2章 功能需求：集中度最高 (Highest concentration)
- 第3章 技术方案：中等集中度 (Medium concentration)
```

---

## 第 6 部分：达标判断与迭代循环

### 设计说明 (Design Notes)
本部分定义了文档质量达标的混合判断机制，结合自动评分和人工确认双重标准。自动评分提供客观的质量指标，人工确认确保最终决策权在用户手中。

### 混合标准机制 (Hybrid Criteria Mechanism)

```yaml
自动评分标准 (Auto Scoring Criteria):
  格式评审阈值 (format_threshold): 4.0      # 格式规范最低分
  架构评审阈值 (arch_threshold): 4.0        # 架构逻辑最低分
  匹配评审阈值 (match_threshold): 4.0       # 需求覆盖最低分
  高严重度问题上限 (high_issues_max): 0     # 高严重度问题最大数量
  中严重度问题上限 (medium_issues_max): 3   # 中严重度问题最大数量

自动判断逻辑 (Auto Judgment Logic):
  通过 (passed) = (
    格式评分 (format_score) >= 格式评审阈值 (format_threshold) AND
    架构评分 (arch_score) >= 架构评审阈值 (arch_threshold) AND
    匹配评分 (match_score) >= 匹配评审阈值 (match_threshold) AND
    高严重度问题数 (high_issues_count) <= 高严重度问题上限 (high_issues_max) AND
    中严重度问题数 (medium_issues_count) <= 中严重度问题上限 (medium_issues_max)
  )
```

### 人工确认流程 (Manual Confirmation Flow)

```yaml
自动评分通过后 (After Auto Scoring Passed):
  提示用户 (Prompt User):
    "文档已通过自动评分标准，是否确认达标？"
    "(Document passed auto scoring. Confirm as qualified?)"
    - A) 确认达标，归档文档 (Confirm & Archive)
    - B) 继续迭代，进入下一轮评审 (Continue Iteration)
    - C) 查看详细评审报告 (View Detailed Report)

自动评分未通过时 (When Auto Scoring Failed):
  提示用户 (Prompt User):
    "文档未通过自动评分，存在以下问题："
    "(Document failed auto scoring. Issues found:)"
    [问题清单 (Issue List)]
    - A) 自动修复可修复的问题 (Auto Fix)
    - B) 手动修改后重新评审 (Manual Edit & Re-review)
    - C) 忽略问题，强制通过 (Force Pass)
```

### 迭代循环控制 (Iteration Loop Control)

```yaml
最大迭代次数 (max_iterations): 10          # 防止无限循环
迭代超时提醒 (iteration_warning): 5        # 第5次迭代时提醒用户

迭代记录格式 (Iteration Log Format):
  - 迭代轮次 (round): 1
    评分 (scores): { 格式: 3.5, 架构: 4.0, 匹配: 3.0 }
    问题数 (issues): { 高: 2, 中: 5, 低: 8 }
    状态 (status): "未通过 (Not Passed)"
  - 迭代轮次 (round): 2
    评分 (scores): { 格式: 4.5, 架构: 4.5, 匹配: 4.0 }
    问题数 (issues): { 高: 0, 中: 2, 低: 3 }
    状态 (status): "通过 (Passed)"

迭代超限处理 (Iteration Limit Handling):
  当达到最大迭代次数时 (When max iterations reached):
    - 强制暂停工作流 (Force pause workflow)
    - 生成迭代分析报告 (Generate iteration analysis report)
    - 提示用户：重新评估需求或调整阈值 (Prompt: re-evaluate requirements or adjust thresholds)
```

---

## 第 7 部分：修改记录整合

### 设计说明 (Design Notes)
本部分定义了三种修改记录来源的整合机制：Git 集成、对话记录和显式备注。通过整合多种来源的变更信息，确保所有修改都有完整的追溯链，便于后期审计和知识沉淀。

### Git 集成 (Git Integration)

```yaml
功能描述 (Description):
  自动从 Git 历史中提取文档相关的变更记录
  (Automatically extract document-related changes from Git history)

触发时机 (Trigger):
  - 版本 Agent 执行时自动调用 (Auto-invoked by Version Agent)
  - 用户显式调用 /doc-version 时 (When user invokes /doc-version)

提取内容 (Extracted Content):
  提交哈希 (commit_hash): string           # Git commit hash
  提交信息 (commit_message): string        # Commit message
  提交时间 (commit_time): datetime         # Commit timestamp
  变更文件 (changed_files): list           # List of changed files
  差异内容 (diff_content): string          # Diff content
  提交作者 (author): string                # Commit author

整合格式 (Integration Format):
  ## Git 变更记录 (Git Change Log)
  | 时间 (Time) | 提交 (Commit) | 说明 (Message) | 作者 (Author) |
  |-------------|---------------|----------------|---------------|
  | 2026-02-04 10:30 | a1b2c3d | 完善功能需求章节 | user |
```

### 对话记录 (Conversation Log)

```yaml
功能描述 (Description):
  保存 Claude Code 对话中与文档相关的讨论要点
  (Save document-related discussion points from Claude Code conversations)

触发时机 (Trigger):
  - 每轮评审结束时自动提取 (Auto-extract after each review round)
  - 用户显式请求保存对话时 (When user requests to save conversation)

提取内容 (Extracted Content):
  对话时间 (conversation_time): datetime   # Conversation timestamp
  讨论主题 (topic): string                 # Discussion topic
  关键决策 (key_decisions): list           # Key decisions made
  待办事项 (action_items): list            # Action items identified
  相关文档 (related_docs): list            # Related documents

整合格式 (Integration Format):
  ## 对话记录 (Conversation Log)
  ### 2026-02-04 讨论 (Discussion)
  **主题 (Topic)**: 功能优先级调整
  **决策 (Decisions)**:
  - 将语音克隆功能优先级提升至 P0
  - 延迟实现批量处理功能
  **待办 (Action Items)**:
  - [ ] 更新 2.3 节功能优先级
  - [ ] 补充语音克隆技术方案
```

### 显式备注 (Explicit Notes)

```yaml
功能描述 (Description):
  用户通过 /doc-note 手动添加的备注和讨论要点
  (User-added notes and discussion points via /doc-note)

触发方式 (Trigger):
  /doc-note "备注内容" (Note content)
  /doc-note --meeting "会议主题" (Meeting topic)

参数说明 (Parameters):
  内容 (content): string                   # 备注正文 (Note body)
  类型 (type): "note" | "meeting" | "decision"  # 备注类型 (Note type)
  关联文档 (related_doc): string           # 关联的文档路径 (Related document path)
  标签 (tags): list                        # 分类标签 (Classification tags)

存储位置 (Storage Location):
  docs/notes/NOTE-{项目名}-{日期}-{序号}.md
  docs/notes/MEETING-{项目名}-{日期}.md

备注模板 (Note Template):
  ---
  类型 (type): note
  创建时间 (created): 2026-02-04 10:30
  关联文档 (related): docs/prd/PRD-xxx-V1.2.md
  标签 (tags): [需求变更, 优先级]
  ---
  # 备注标题 (Note Title)
  备注正文内容... (Note body content...)
```

### 记录汇总视图 (Consolidated View)

```yaml
功能描述 (Description):
  将三种来源的记录整合到版本历史文件中
  (Consolidate records from all three sources into version history)

汇总结构 (Structure):
  # 版本历史 (Version History): PRD-项目名

  ## 版本链 (Version Chain)
  V1.0 → V1.1 → V1.2 (当前 Current)

  ## V1.2 变更详情 (V1.2 Change Details)
  ### 来源: Git (Source: Git)
  [Git 变更记录]

  ### 来源: 对话 (Source: Conversation)
  [对话记录摘要]

  ### 来源: 备注 (Source: Notes)
  [相关备注列表]
```

---

## 第 8 部分：配置系统

### 设计说明 (Design Notes)
本部分定义了技能包的配置机制，支持全局默认配置、用户级配置和项目级自定义配置。配置采用分层覆盖机制，确保灵活性和一致性，同时提供配置验证确保配置的正确性。

### 配置文件结构 (Configuration Structure)

```yaml
# config/default-config.yaml
# 默认配置文件 (Default Configuration)

# 基础设置 (Basic Settings)
基础设置 (basic):
  默认文档类型 (default_doc_type): "prd"        # prd | tech | custom
  默认语言 (default_language): "zh-CN"          # zh-CN | en-US
  输出目录 (output_dir): "docs"                 # 相对于项目根目录

# 评审设置 (Review Settings)
评审设置 (review):
  默认执行方式 (default_mode): "parallel"       # parallel | grouped | sequential
  格式评审阈值 (format_threshold): 4.0          # 格式规范最低分 (1.0-5.0)
  架构评审阈值 (arch_threshold): 4.0            # 架构逻辑最低分 (1.0-5.0)
  匹配评审阈值 (match_threshold): 4.0           # 需求覆盖最低分 (1.0-5.0)
  高严重度问题上限 (high_issues_max): 0         # 高严重度问题最大数量
  中严重度问题上限 (medium_issues_max): 3       # 中严重度问题最大数量

# 版本管理设置 (Version Management Settings)
版本管理 (version):
  自动触发 (auto_trigger): true                 # 是否自动触发版本记录
  自动提交Git (auto_git_commit): false          # 是否自动 Git 提交
  最大迭代次数 (max_iterations): 10             # 最大迭代轮数
  迭代警告阈值 (iteration_warning): 5           # 迭代警告触发轮数

# Agent 设置 (Agent Settings)
Agent设置 (agents):
  最大并行数 (max_parallel): 4                  # 最大并行 Agent 数量
  超时时间 (timeout_ms): 300000                 # Agent 执行超时（毫秒）
  重试次数 (retry_count): 2                     # 失败重试次数

# 模板设置 (Template Settings)
模板设置 (templates):
  PRD模板路径 (prd_template): "templates/prd-template.md"
  技术设计模板路径 (tech_template): "templates/tech-design-template.md"
  评审报告模板路径 (review_template): "templates/review-report-template.md"
```

### 项目级配置覆盖 (Project-Level Override)

```yaml
# 项目根目录/.doc-workflow.yaml
# 项目级配置 (Project Configuration)

# 覆盖默认设置 (Override Defaults)
评审设置 (review):
  默认执行方式 (default_mode): "sequential"     # 本项目使用顺序评审
  格式评审阈值 (format_threshold): 4.5          # 提高格式要求

# 项目特定设置 (Project-Specific Settings)
项目信息 (project):
  项目代码 (project_code): "MYAPP"              # 用于文档编号
  项目名称 (project_name): "我的应用"           # 项目显示名称
  负责人 (owner): "张三"                        # 项目负责人

# 自定义模板 (Custom Templates)
模板设置 (templates):
  PRD模板路径 (prd_template): "my-templates/custom-prd.md"
```

### 配置加载优先级 (Configuration Priority)

```yaml
优先级顺序 (Priority Order) - 从高到低:
  1. 命令行参数 (Command Line Args)      # 最高优先级
  2. 项目级配置 (.doc-workflow.yaml)     # 项目目录
  3. 用户级配置 (~/.doc-workflow.yaml)   # 用户主目录
  4. 默认配置 (default-config.yaml)      # 技能包内置

示例 (Example):
  # 命令行指定评审方式，覆盖所有配置
  /doc-review ./docs/prd/PRD-xxx.md --mode sequential
```

### 配置验证 (Configuration Validation)

```yaml
验证规则 (Validation Rules):
  阈值范围 (threshold_range):
    - 所有阈值必须在 1.0 ~ 5.0 之间
    - All thresholds must be between 1.0 and 5.0
  路径存在性 (path_exists):
    - 模板路径必须存在
    - Template paths must exist
  类型检查 (type_check):
    - 确保值类型正确（数字、字符串、布尔等）
    - Ensure correct value types (number, string, boolean, etc.)
  必填项检查 (required_check):
    - 关键配置不能为空
    - Critical configurations cannot be empty

错误处理 (Error Handling):
  配置无效时 (On Invalid Config):
    - 输出详细错误信息 (Output detailed error message)
    - 提示正确的配置格式 (Show correct configuration format)
    - 回退到默认配置（可选）(Fallback to default config - optional)
```

---

## 第 9 部分：错误处理与异常情况

### 设计说明 (Design Notes)
本部分定义了系统在各种异常情况下的处理机制，包括 Agent 执行错误、评审异常、版本管理异常和用户中断处理。完善的错误处理确保工作流的健壮性和良好的用户体验。

### Agent 执行错误 (Agent Execution Errors)

```yaml
超时错误 (Timeout Error):
  描述 (Description): Agent 执行超过配置的超时时间
  处理策略 (Strategy):
    - 记录当前进度 (Save current progress)
    - 提示用户选择 (Prompt user to choose):
      A) 重试 (Retry)
      B) 跳过 (Skip)
      C) 终止 (Terminate)
    - 保存部分结果（如有）(Save partial results if available)

解析错误 (Parse Error):
  描述 (Description): 输入源无法正确解析
  处理策略 (Strategy):
    - 输出详细错误位置和原因 (Output detailed error location and reason)
    - 建议用户检查输入格式 (Suggest user check input format)
    - 提供支持的格式说明 (Provide supported format documentation)

网络错误 (Network Error):
  描述 (Description): URL 抓取失败或网络不可用
  处理策略 (Strategy):
    - 自动重试（最多 retry_count 次）(Auto retry up to retry_count times)
    - 提示用户检查网络连接 (Prompt user to check network connection)
    - 建议使用本地文件替代 (Suggest using local file instead)

模板错误 (Template Error):
  描述 (Description): 模板文件不存在或格式错误
  处理策略 (Strategy):
    - 回退到默认模板 (Fallback to default template)
    - 提示用户修复自定义模板 (Prompt user to fix custom template)
    - 输出模板格式要求 (Output template format requirements)
```

### 评审异常 (Review Exceptions)

```yaml
并行评审部分失败 (Partial Failure in Parallel Review):
  描述 (Description): 4 个评审 Agent 中部分失败
  处理策略 (Strategy):
    - 收集成功的评审结果 (Collect successful review results)
    - 标记失败的评审维度 (Mark failed review dimensions)
    - 提示用户选择 (Prompt user to choose):
      A) 仅使用成功的评审结果继续 (Continue with successful results only)
      B) 重试失败的评审 (Retry failed reviews)
      C) 切换到顺序执行模式重新评审 (Switch to sequential mode and re-review)

评分异常 (Scoring Anomaly):
  描述 (Description): 评分结果异常（如全 5 分或全 1 分）
  处理策略 (Strategy):
    - 触发二次验证 (Trigger secondary validation)
    - 输出评分依据供用户审核 (Output scoring basis for user review)
    - 建议人工复核 (Suggest manual review)

评审冲突 (Review Conflict):
  描述 (Description): 不同评审维度给出矛盾的建议
  处理策略 (Strategy):
    - 高亮显示冲突点 (Highlight conflict points)
    - 提供各维度的理由 (Provide reasoning from each dimension)
    - 由用户决定采纳哪个建议 (Let user decide which suggestion to adopt)
```

### 版本管理异常 (Version Management Exceptions)

```yaml
版本冲突 (Version Conflict):
  描述 (Description): 检测到文档被外部修改
  处理策略 (Strategy):
    - 暂停自动版本记录 (Pause auto version recording)
    - 提示用户确认外部修改 (Prompt user to confirm external changes)
    - 选择 (Choose):
      A) 合并 (Merge)
      B) 覆盖 (Overwrite)
      C) 创建分支版本 (Create branch version)

Git 操作失败 (Git Operation Failure):
  描述 (Description): Git commit 或其他操作失败
  处理策略 (Strategy):
    - 版本记录仍保存到 VERSION-history.md (Still save to VERSION-history.md)
    - 提示用户手动处理 Git 问题 (Prompt user to handle Git issue manually)
    - 不阻塞主工作流 (Don't block main workflow)

差异计算失败 (Diff Calculation Failure):
  描述 (Description): 无法计算两版本差异
  处理策略 (Strategy):
    - 保存两个版本的完整内容 (Save complete content of both versions)
    - 标记为"差异计算失败" (Mark as "diff calculation failed")
    - 建议用户使用外部 diff 工具 (Suggest using external diff tool)
```

### 用户中断处理 (User Interruption Handling)

```yaml
中断恢复机制 (Interruption Recovery Mechanism):
  状态保存点 (Checkpoints):
    - 每个 Agent 完成后保存状态 (Save state after each Agent completes)
    - 记录当前工作流阶段 (Record current workflow stage)
    - 保存中间产物路径 (Save intermediate artifact paths)

  恢复命令 (Recovery Command):
    /doc-workflow --resume

  恢复流程 (Recovery Flow):
    1. 读取最近的状态保存点 (Read latest checkpoint)
    2. 显示中断时的进度 (Display progress at interruption)
    3. 提示用户选择 (Prompt user to choose):
       A) 从中断点继续 (Continue from checkpoint)
       B) 从头开始 (Start from beginning)
       C) 从指定阶段开始 (Start from specified stage)

状态文件 (State File):
  位置 (Location): docs/.doc-workflow-state.json
  内容 (Content):
    当前阶段 (current_stage): "review"
    已完成阶段 (completed_stages): ["investigate"]
    当前文档 (current_doc): "docs/prd/PRD-xxx-V1.1.md"
    评审进度 (review_progress): { format: "done", arch: "pending" }
    最后更新 (last_updated): "2026-02-04T10:30:00Z"
```

---

## 第 10 部分：扩展性设计

### 设计说明 (Design Notes)
本部分定义了系统的扩展性架构，为未来支持独立应用和 MCP 服务器预留接口。通过核心抽象层将业务逻辑与执行环境解耦，确保技能包的长期演进能力和跨平台兼容性。

### 核心抽象层 (Core Abstraction Layer)

```yaml
设计目标 (Design Goal):
  将业务逻辑与执行环境解耦，使同一套核心逻辑可运行在不同环境中
  (Decouple business logic from execution environment for cross-platform compatibility)

抽象接口 (Abstract Interfaces):

  输入适配器 (Input Adapter):
    描述 (Description): 统一不同来源的输入处理
    接口定义 (Interface):
      - parseInput(source: InputSource): ParsedContent
      - validateInput(content: ParsedContent): ValidationResult
    实现 (Implementations):
      - ClaudeCodeInputAdapter   # Claude Code 技能包
      - WebUIInputAdapter       # 未来 Web 应用
      - MCPInputAdapter         # 未来 MCP 服务器

  输出适配器 (Output Adapter):
    描述 (Description): 统一不同环境的输出处理
    接口定义 (Interface):
      - writeDocument(content: Document, path: string): Result
      - displayProgress(stage: string, progress: number): void
      - promptUser(question: Question): UserResponse
    实现 (Implementations):
      - ClaudeCodeOutputAdapter  # 命令行输出
      - WebUIOutputAdapter      # Web 界面渲染
      - MCPOutputAdapter        # MCP 协议响应

  存储适配器 (Storage Adapter):
    描述 (Description): 统一不同存储后端的操作
    接口定义 (Interface):
      - save(key: string, data: any): Result
      - load(key: string): any
      - list(prefix: string): string[]
    实现 (Implementations):
      - FileSystemStorage       # 本地文件系统
      - ObsidianStorage        # Obsidian 知识库
      - DatabaseStorage        # 未来数据库支持
```

### 未来形态预留 (Future Form Reservations)

```yaml
独立应用形态 (Standalone Application):
  架构预留 (Architecture Reservation):
    - 核心逻辑模块可独立打包 (Core logic can be packaged independently)
    - UI 层与逻辑层分离 (UI layer separated from logic layer)
    - 支持 Electron / Tauri 桌面应用 (Support Electron / Tauri desktop apps)
    - 支持 Web 应用部署 (Support Web application deployment)

  接口预留 (Interface Reservation):
    - REST API 端点定义 (REST API endpoint definitions)
    - WebSocket 实时通信 (WebSocket real-time communication)
    - 文件上传/下载接口 (File upload/download interfaces)

MCP 服务器形态 (MCP Server):
  协议支持 (Protocol Support):
    - 实现 MCP 标准工具接口 (Implement MCP standard tool interface)
    - 支持 resources 暴露文档和模板 (Support resources for documents and templates)
    - 支持 prompts 提供预设工作流 (Support prompts for preset workflows)

  工具定义预留 (Tool Definition Reservation):
    tools:
      - name: "doc_investigate"
        description: "分析输入源生成需求文档 (Analyze input source to generate PRD)"
        inputSchema: { source: string, type: string }
      - name: "doc_review"
        description: "多维度评审文档质量 (Multi-dimensional document quality review)"
        inputSchema: { docPath: string, mode: string }
      - name: "doc_version"
        description: "生成版本差异报告 (Generate version diff report)"
        inputSchema: { currentDoc: string, previousDoc: string }
```

### 插件扩展机制 (Plugin Extension Mechanism)

```yaml
设计目标 (Design Goal):
  允许用户和第三方扩展系统功能，无需修改核心代码
  (Allow users and third parties to extend functionality without modifying core code)

扩展点 (Extension Points):

  自定义解析器 (Custom Parsers):
    描述 (Description): 添加新的输入源解析器
    注册方式 (Registration):
      # parsers/custom-parser.md
      ---
      name: parse-figma
      description: 解析 Figma 设计稿 (Parse Figma design files)
      input_types: ["figma_url", "figma_file"]
      ---
    调用方式 (Invocation):
      /doc-investigate --parser figma https://figma.com/xxx

  自定义评审维度 (Custom Review Dimensions):
    描述 (Description): 添加新的评审维度
    注册方式 (Registration):
      # reviewers/custom-reviewer.md
      ---
      name: security-review
      description: 安全合规评审 (Security compliance review)
      priority: 2
      ---
    配置启用 (Enable in Config):
      评审设置:
        额外评审维度 (additional_dimensions): ["security-review"]

  自定义模板 (Custom Templates):
    描述 (Description): 添加或覆盖文档模板
    存放位置 (Location):
      项目目录/.doc-workflow/templates/
    优先级 (Priority):
      项目模板 > 用户模板 > 默认模板
      (Project templates > User templates > Default templates)
```

### 版本兼容性策略 (Version Compatibility Strategy)

```yaml
语义化版本 (Semantic Versioning):
  格式 (Format): MAJOR.MINOR.PATCH
  规则 (Rules):
    - MAJOR: 不兼容的 API 变更 (Incompatible API changes)
    - MINOR: 向后兼容的功能新增 (Backward-compatible new features)
    - PATCH: 向后兼容的问题修复 (Backward-compatible bug fixes)

配置迁移 (Configuration Migration):
  描述 (Description): 自动迁移旧版本配置到新版本
  机制 (Mechanism):
    - 检测配置文件版本 (Detect config file version)
    - 应用迁移脚本 (Apply migration scripts)
    - 备份旧配置 (Backup old config)
    - 生成新配置 (Generate new config)

向后兼容承诺 (Backward Compatibility Promise):
  - 技能命令名称保持稳定 (Skill command names remain stable)
  - 输出目录结构保持稳定 (Output directory structure remains stable)
  - 配置项只增不删（废弃项标记 deprecated）(Config items only added, deprecated items marked)
```

---

## 附录 A：技能命令速查表 (Appendix A: Skill Command Quick Reference)

| 命令 (Command) | 说明 (Description) | 示例 (Example) |
|----------------|---------------------|----------------|
| `/doc-workflow` | 主入口，协调全流程 | `/doc-workflow https://example.com` |
| `/doc-investigate` | 需求调查，生成初稿 | `/doc-investigate ./notes.md` |
| `/doc-review` | 多维评审 | `/doc-review ./docs/prd/PRD-xxx.md --mode parallel` |
| `/doc-version` | 版本管理 | `/doc-version ./docs/prd/PRD-xxx.md` |
| `/doc-note` | 添加备注 | `/doc-note "功能优先级调整"` |

---

## 附录 B：配置项速查表 (Appendix B: Configuration Quick Reference)

| 配置项 (Config Item) | 默认值 (Default) | 说明 (Description) |
|----------------------|------------------|---------------------|
| `default_doc_type` | prd | 默认文档类型 |
| `default_mode` | parallel | 默认评审执行方式 |
| `format_threshold` | 4.0 | 格式评审阈值 |
| `arch_threshold` | 4.0 | 架构评审阈值 |
| `match_threshold` | 4.0 | 匹配评审阈值 |
| `auto_trigger` | true | 自动触发版本记录 |
| `auto_git_commit` | false | 自动 Git 提交 |
| `max_iterations` | 10 | 最大迭代次数 |
| `max_parallel` | 4 | 最大并行 Agent 数 |
| `timeout_ms` | 300000 | Agent 超时时间（毫秒） |

---

## 附录 C：错误代码表 (Appendix C: Error Code Reference)

| 错误代码 (Error Code) | 说明 (Description) | 处理建议 (Suggested Action) |
|-----------------------|---------------------|----------------------------|
| E001 | Agent 执行超时 | 重试或增加超时时间 |
| E002 | 输入源解析失败 | 检查输入格式 |
| E003 | 网络请求失败 | 检查网络连接 |
| E004 | 模板文件不存在 | 检查模板路径 |
| E005 | 配置验证失败 | 检查配置文件格式 |
| E006 | 版本冲突 | 手动解决冲突 |
| E007 | Git 操作失败 | 检查 Git 状态 |
| E008 | 评审部分失败 | 重试或切换模式 |

---

## 文档修订历史 (Document Revision History)

| 版本 (Version) | 日期 (Date) | 作者 (Author) | 说明 (Description) |
|----------------|-------------|---------------|---------------------|
| 1.0 | 2026-02-05 | Claude | 初始设计文档 |

---

*本文档由 Claude Code brainstorming skill 生成*
