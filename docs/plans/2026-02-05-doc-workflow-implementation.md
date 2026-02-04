# Doc-Workflow 技能包实现计划

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** 实现一个完整的 Claude Code 技能包，用于自动化文档工作流程（需求调查 → 多维评审 → 版本管理）

**Architecture:** 采用单一技能包架构，包含 5 个核心技能（doc-workflow, doc-investigate, doc-review, doc-version, doc-note）和 4 个输入解析器。技能之间通过 Task 工具调用独立 Agent 执行，保持上下文隔离。

**Tech Stack:** Claude Code Skills (Markdown), YAML 配置, Git 集成

**设计文档:** `docs/plans/2026-02-05-doc-workflow-design.md`

---

## 任务概览 (Task Overview)

| 任务 | 描述 | 预计步骤 |
|------|------|----------|
| Task 1 | 创建技能包基础目录结构 | 5 步 |
| Task 2 | 实现默认配置文件 | 4 步 |
| Task 3 | 实现 PRD 文档模板 | 4 步 |
| Task 4 | 实现技术设计文档模板 | 4 步 |
| Task 5 | 实现评审报告模板 | 4 步 |
| Task 6 | 实现 parse-url 解析器 | 4 步 |
| Task 7 | 实现 parse-doc 解析器 | 4 步 |
| Task 8 | 实现 parse-meeting 解析器 | 4 步 |
| Task 9 | 实现 parse-chat 解析器 | 4 步 |
| Task 10 | 实现 doc-investigate 技能 | 5 步 |
| Task 11 | 实现 doc-review 技能 | 5 步 |
| Task 12 | 实现 doc-version 技能 | 5 步 |
| Task 13 | 实现 doc-note 技能 | 4 步 |
| Task 14 | 实现 doc-workflow 主入口技能 | 5 步 |
| Task 15 | 集成测试与文档完善 | 5 步 |

---

## Task 1: 创建技能包基础目录结构

**目标:** 创建 doc-workflow 技能包的完整目录结构

**Files:**
- Create: `doc-workflow/skills/` (目录)
- Create: `doc-workflow/skills/parsers/` (目录)
- Create: `doc-workflow/templates/` (目录)
- Create: `doc-workflow/config/` (目录)
- Create: `doc-workflow/README.md`

**Step 1: 创建技能包根目录和子目录**

```bash
mkdir -p doc-workflow/skills/parsers
mkdir -p doc-workflow/templates
mkdir -p doc-workflow/config
```

**Step 2: 验证目录结构**

```bash
ls -la doc-workflow/
```

Expected: 显示 skills, templates, config 三个目录

**Step 3: 创建 README.md**

创建文件 `doc-workflow/README.md`:

```markdown
# Doc-Workflow 技能包

AI Agents 文档工作流程系统 - Claude Code 技能包

## 功能概述

- `/doc-workflow` - 主入口，协调全流程
- `/doc-investigate` - 需求调查，生成文档初稿
- `/doc-review` - 多维评审（格式、架构、匹配性、创新）
- `/doc-version` - 版本管理，追踪变更
- `/doc-note` - 备注记录

## 安装方式

将 `doc-workflow` 目录复制到 Claude Code 的 skills 目录中。

## 使用示例

```bash
# 完整工作流
/doc-workflow https://example.com

# 单独调用
/doc-investigate ./requirements.md
/doc-review ./docs/prd/PRD-xxx.md
```

## 配置

编辑 `config/default-config.yaml` 或在项目根目录创建 `.doc-workflow.yaml`。

## 版本

- v1.0.0 - 初始版本
```

**Step 4: 验证 README 创建成功**

```bash
cat doc-workflow/README.md
```

Expected: 显示 README 内容

**Step 5: 提交基础结构**

```bash
git add doc-workflow/
git commit -m "feat: 创建 doc-workflow 技能包基础目录结构"
```

---

## Task 2: 实现默认配置文件

**目标:** 创建技能包的默认配置文件，定义所有可配置项

**Files:**
- Create: `doc-workflow/config/default-config.yaml`

**Step 1: 创建默认配置文件**

创建文件 `doc-workflow/config/default-config.yaml`:

```yaml
# Doc-Workflow 默认配置文件
# Default Configuration for Doc-Workflow

# 配置版本 (Config Version)
config_version: "1.0"

# 基础设置 (Basic Settings)
basic:
  # 默认文档类型 (Default Document Type): prd | tech | custom
  default_doc_type: "prd"
  # 默认语言 (Default Language): zh-CN | en-US
  default_language: "zh-CN"
  # 输出目录 (Output Directory) - 相对于项目根目录
  output_dir: "docs"

# 评审设置 (Review Settings)
review:
  # 默认执行方式 (Default Mode): parallel | grouped | sequential
  default_mode: "parallel"
  # 格式评审阈值 (Format Threshold): 1.0 - 5.0
  format_threshold: 4.0
  # 架构评审阈值 (Architecture Threshold): 1.0 - 5.0
  arch_threshold: 4.0
  # 匹配评审阈值 (Matching Threshold): 1.0 - 5.0
  match_threshold: 4.0
  # 高严重度问题上限 (High Issues Max)
  high_issues_max: 0
  # 中严重度问题上限 (Medium Issues Max)
  medium_issues_max: 3

# 版本管理设置 (Version Management Settings)
version:
  # 自动触发 (Auto Trigger)
  auto_trigger: true
  # 自动提交 Git (Auto Git Commit)
  auto_git_commit: false
  # 最大迭代次数 (Max Iterations)
  max_iterations: 10
  # 迭代警告阈值 (Iteration Warning)
  iteration_warning: 5

# Agent 设置 (Agent Settings)
agents:
  # 最大并行数 (Max Parallel)
  max_parallel: 4
  # 超时时间毫秒 (Timeout MS)
  timeout_ms: 300000
  # 重试次数 (Retry Count)
  retry_count: 2

# 模板设置 (Template Settings)
templates:
  # PRD 模板路径 (PRD Template Path)
  prd_template: "templates/prd-template.md"
  # 技术设计模板路径 (Tech Design Template Path)
  tech_template: "templates/tech-design-template.md"
  # 评审报告模板路径 (Review Report Template Path)
  review_template: "templates/review-report-template.md"
```

**Step 2: 验证配置文件语法**

```bash
cat doc-workflow/config/default-config.yaml
```

Expected: 显示完整的 YAML 配置内容，无语法错误

**Step 3: 验证 YAML 格式正确**

```bash
python -c "import yaml; yaml.safe_load(open('doc-workflow/config/default-config.yaml'))" && echo "YAML valid"
```

Expected: 输出 "YAML valid"

**Step 4: 提交配置文件**

```bash
git add doc-workflow/config/default-config.yaml
git commit -m "feat: 添加默认配置文件"
```

---

## Task 3: 实现 PRD 文档模板

**目标:** 创建标准化的产品需求文档模板

**Files:**
- Create: `doc-workflow/templates/prd-template.md`

**Step 1: 创建 PRD 模板文件**

创建文件 `doc-workflow/templates/prd-template.md`:

```markdown
# {{PROJECT_NAME}} 产品需求文档 (PRD)

> 文档编号 (Document ID): PRD-{{PROJECT_CODE}}-{{VERSION}}
> 创建日期 (Created): {{DATE}}
> 版本 (Version): {{VERSION}}
> 状态 (Status): {{STATUS}}
> 作者 (Author): {{AUTHOR}}

---

## 1. 项目概述 (Project Overview)

### 1.1 项目背景 (Background)

{{PROJECT_BACKGROUND}}

### 1.2 项目目标 (Objectives)

{{PROJECT_OBJECTIVES}}

### 1.3 目标用户 (Target Users)

| 用户类型 | 描述 | 核心需求 |
|----------|------|----------|
| {{USER_TYPE_1}} | {{USER_DESC_1}} | {{USER_NEED_1}} |

---

## 2. 产品分析 (Product Analysis)

### 2.1 核心功能 (Core Features)

| 功能模块 | 功能描述 | 优先级 |
|----------|----------|--------|
| {{FEATURE_1}} | {{FEATURE_DESC_1}} | {{PRIORITY_1}} |

### 2.2 功能详情 (Feature Details)

#### 2.2.1 {{FEATURE_1}}

**功能描述:** {{FEATURE_DETAIL_1}}

**用户故事:**
- 作为 {{USER_TYPE}}，我希望 {{ACTION}}，以便 {{BENEFIT}}

**验收标准:**
- [ ] {{ACCEPTANCE_1}}

---

## 3. 界面设计 (Interface Design)

### 3.1 页面结构 (Page Structure)

{{PAGE_STRUCTURE}}

### 3.2 交互流程 (Interaction Flow)

{{INTERACTION_FLOW}}

---

## 4. 技术分析 (Technical Analysis)

### 4.1 技术架构 (Architecture)

{{TECH_ARCHITECTURE}}

### 4.2 接口设计 (API Design)

| 接口名称 | 方法 | 路径 | 描述 |
|----------|------|------|------|
| {{API_NAME}} | {{METHOD}} | {{PATH}} | {{API_DESC}} |

---

## 5. 非功能需求 (Non-Functional Requirements)

### 5.1 性能需求 (Performance)

- 响应时间: {{RESPONSE_TIME}}
- 并发用户: {{CONCURRENT_USERS}}

### 5.2 安全需求 (Security)

{{SECURITY_REQUIREMENTS}}

---

## 6. 附录 (Appendix)

### 6.1 术语表 (Glossary)

| 术语 | 定义 |
|------|------|
| {{TERM}} | {{DEFINITION}} |

### 6.2 参考资料 (References)

- {{REFERENCE}}

---

*本文档由 Doc-Workflow 技能包生成*
```

**Step 2: 验证模板文件创建成功**

```bash
wc -l doc-workflow/templates/prd-template.md
```

Expected: 显示约 100+ 行

**Step 3: 验证模板占位符格式**

```bash
grep -c "{{" doc-workflow/templates/prd-template.md
```

Expected: 显示占位符数量（约 30+）

**Step 4: 提交 PRD 模板**

```bash
git add doc-workflow/templates/prd-template.md
git commit -m "feat: 添加 PRD 文档模板"
```

---

## Task 4: 实现技术设计文档模板

**目标:** 创建标准化的技术设计文档模板

**Files:**
- Create: `doc-workflow/templates/tech-design-template.md`

**Step 1: 创建技术设计模板文件**

创建文件 `doc-workflow/templates/tech-design-template.md`:

```markdown
# {{PROJECT_NAME}} 技术设计文档

> 文档编号 (Document ID): TDD-{{PROJECT_CODE}}-{{VERSION}}
> 创建日期 (Created): {{DATE}}
> 版本 (Version): {{VERSION}}
> 状态 (Status): {{STATUS}}
> 作者 (Author): {{AUTHOR}}

---

## 1. 概述 (Overview)

### 1.1 背景 (Background)

{{BACKGROUND}}

### 1.2 目标 (Goals)

{{GOALS}}

### 1.3 非目标 (Non-Goals)

{{NON_GOALS}}

---

## 2. 系统架构 (System Architecture)

### 2.1 架构图 (Architecture Diagram)

```
{{ARCHITECTURE_DIAGRAM}}
```

### 2.2 组件说明 (Component Description)

| 组件 | 职责 | 技术栈 |
|------|------|--------|
| {{COMPONENT}} | {{RESPONSIBILITY}} | {{TECH_STACK}} |

---

## 3. 详细设计 (Detailed Design)

### 3.1 数据模型 (Data Model)

{{DATA_MODEL}}

### 3.2 接口设计 (API Design)

#### {{API_NAME}}

- **方法 (Method):** {{METHOD}}
- **路径 (Path):** {{PATH}}
- **请求 (Request):**
```json
{{REQUEST_BODY}}
```
- **响应 (Response):**
```json
{{RESPONSE_BODY}}
```

---

## 4. 技术选型 (Technology Choices)

| 领域 | 选择 | 理由 |
|------|------|------|
| {{DOMAIN}} | {{CHOICE}} | {{REASON}} |

---

## 5. 风险与缓解 (Risks & Mitigations)

| 风险 | 影响 | 缓解措施 |
|------|------|----------|
| {{RISK}} | {{IMPACT}} | {{MITIGATION}} |

---

## 6. 测试策略 (Testing Strategy)

{{TESTING_STRATEGY}}

---

## 7. 部署计划 (Deployment Plan)

{{DEPLOYMENT_PLAN}}

---

*本文档由 Doc-Workflow 技能包生成*
```

**Step 2: 验证模板文件创建成功**

```bash
wc -l doc-workflow/templates/tech-design-template.md
```

Expected: 显示约 90+ 行

**Step 3: 验证模板结构完整**

```bash
grep "^## " doc-workflow/templates/tech-design-template.md | wc -l
```

Expected: 显示 7（7 个主要章节）

**Step 4: 提交技术设计模板**

```bash
git add doc-workflow/templates/tech-design-template.md
git commit -m "feat: 添加技术设计文档模板"
```

---

## Task 5: 实现评审报告模板

**目标:** 创建标准化的评审报告模板

**Files:**
- Create: `doc-workflow/templates/review-report-template.md`

**Step 1: 创建评审报告模板文件**

创建文件 `doc-workflow/templates/review-report-template.md`:

```markdown
# 文档评审报告

> 评审文档 (Reviewed Document): {{DOC_PATH}}
> 评审日期 (Review Date): {{DATE}}
> 文档版本 (Document Version): {{DOC_VERSION}}
> 评审模式 (Review Mode): {{REVIEW_MODE}}

---

## 评审总结 (Review Summary)

| 维度 | 评分 | 状态 |
|------|------|------|
| 格式评审 (Format) | {{FORMAT_SCORE}}/5 | {{FORMAT_STATUS}} |
| 架构评审 (Architecture) | {{ARCH_SCORE}}/5 | {{ARCH_STATUS}} |
| 匹配评审 (Matching) | {{MATCH_SCORE}}/5 | {{MATCH_STATUS}} |
| 创新评审 (Innovation) | {{INNOVATION_SCORE}}/5 | {{INNOVATION_STATUS}} |

**综合评分 (Overall Score):** {{OVERALL_SCORE}}/5

**评审结论 (Conclusion):** {{CONCLUSION}}

---

## 1. 格式评审 (Format Review)

### 1.1 评分详情 (Score Details)

- 语言规范 (Language): {{LANG_SCORE}}/5
- 格式规范 (Formatting): {{FORMAT_DETAIL_SCORE}}/5
- 完整性 (Completeness): {{COMPLETENESS_SCORE}}/5

### 1.2 问题清单 (Issue List)

| 序号 | 位置 | 类型 | 严重度 | 问题描述 | 修改建议 |
|------|------|------|--------|----------|----------|
| {{ISSUE_NO}} | {{LOCATION}} | {{TYPE}} | {{SEVERITY}} | {{DESCRIPTION}} | {{SUGGESTION}} |

---

## 2. 架构评审 (Architecture Review)

### 2.1 评分详情 (Score Details)

- 结构合理性 (Structure): {{STRUCTURE_SCORE}}/5
- 逻辑一致性 (Consistency): {{CONSISTENCY_SCORE}}/5
- 可行性 (Feasibility): {{FEASIBILITY_SCORE}}/5

### 2.2 架构问题 (Architecture Issues)

{{ARCH_ISSUES}}

---

## 3. 匹配评审 (Matching Review)

### 3.1 需求追溯矩阵 (Requirement Traceability Matrix)

| 原始需求 | 文档章节 | 覆盖状态 | 备注 |
|----------|----------|----------|------|
| {{REQUIREMENT}} | {{SECTION}} | {{COVERAGE_STATUS}} | {{NOTE}} |

### 3.2 覆盖度统计 (Coverage Statistics)

- 完整覆盖 (Full): {{FULL_COUNT}}
- 部分覆盖 (Partial): {{PARTIAL_COUNT}}
- 未覆盖 (Missing): {{MISSING_COUNT}}

---

## 4. 创新评审 (Innovation Review)

### 4.1 创新建议清单 (Innovation Suggestions)

| 序号 | 类型 | 建议标题 | 描述 | 预期价值 | 实现难度 |
|------|------|----------|------|----------|----------|
| {{INNO_NO}} | {{INNO_TYPE}} | {{INNO_TITLE}} | {{INNO_DESC}} | {{INNO_VALUE}} | {{INNO_DIFFICULTY}} |

---

## 5. 修改建议优先级 (Suggested Fix Priority)

### 高优先级 (High Priority)
{{HIGH_PRIORITY_FIXES}}

### 中优先级 (Medium Priority)
{{MEDIUM_PRIORITY_FIXES}}

### 低优先级 (Low Priority)
{{LOW_PRIORITY_FIXES}}

---

*本报告由 Doc-Workflow 技能包生成*
```

**Step 2: 验证模板文件创建成功**

```bash
wc -l doc-workflow/templates/review-report-template.md
```

Expected: 显示约 100+ 行

**Step 3: 验证评审维度完整**

```bash
grep "评审" doc-workflow/templates/review-report-template.md | head -10
```

Expected: 显示格式、架构、匹配、创新四个评审维度

**Step 4: 提交评审报告模板**

```bash
git add doc-workflow/templates/review-report-template.md
git commit -m "feat: 添加评审报告模板"
```

---

## Task 6: 实现 parse-url 解析器

**目标:** 创建 URL 解析器技能，用于分析网站内容

**Files:**
- Create: `doc-workflow/skills/parsers/parse-url.md`

**Step 1: 创建 parse-url 技能文件**

创建文件 `doc-workflow/skills/parsers/parse-url.md`:

```markdown
---
name: parse-url
description: 解析网站 URL，提取产品信息和功能需求
---

# URL 解析器 (URL Parser)

## 功能说明

分析指定 URL 的网站内容，提取以下信息：
- 产品名称和描述
- 核心功能列表
- 目标用户群体
- 技术特点
- 交互流程

## 使用方式

此技能由 `/doc-investigate` 内部调用，不建议直接使用。

## 解析流程

1. **获取页面内容**
   - 使用 WebFetch 工具获取 URL 内容
   - 处理重定向和错误情况

2. **结构分析**
   - 识别页面标题和主要标题
   - 提取导航结构
   - 识别功能模块区域

3. **内容提取**
   - 提取产品描述文本
   - 识别功能列表和特性
   - 提取用户评价和案例

4. **信息整理**
   - 按 PRD 模板结构组织信息
   - 标记需要人工补充的部分

## 输出格式

```yaml
product_name: "产品名称"
product_description: "产品描述"
target_users:
  - type: "用户类型"
    description: "用户描述"
    needs: "核心需求"
features:
  - name: "功能名称"
    description: "功能描述"
    priority: "P0/P1/P2"
technical_points:
  - "技术特点1"
interaction_flows:
  - name: "流程名称"
    steps: ["步骤1", "步骤2"]
```

## 错误处理

- URL 无法访问：提示用户检查 URL 或网络
- 内容无法解析：返回原始内容供人工处理
- 超时：使用配置的超时时间，超时后提示重试
```

**Step 2: 验证技能文件格式**

```bash
head -5 doc-workflow/skills/parsers/parse-url.md
```

Expected: 显示 YAML frontmatter（name 和 description）

**Step 3: 验证技能内容完整**

```bash
grep "^## " doc-workflow/skills/parsers/parse-url.md
```

Expected: 显示功能说明、使用方式、解析流程、输出格式、错误处理等章节

**Step 4: 提交 parse-url 技能**

```bash
git add doc-workflow/skills/parsers/parse-url.md
git commit -m "feat: 添加 URL 解析器技能"
```

---

## Task 7: 实现 parse-doc 解析器

**目标:** 创建本地文档解析器技能

**Files:**
- Create: `doc-workflow/skills/parsers/parse-doc.md`

**Step 1: 创建 parse-doc 技能文件**

创建文件 `doc-workflow/skills/parsers/parse-doc.md`:

```markdown
---
name: parse-doc
description: 解析本地文档，提取需求信息
---

# 文档解析器 (Document Parser)

## 功能说明

解析本地文档文件，支持以下格式：
- Markdown (.md)
- 纯文本 (.txt)
- Word 文档描述

## 使用方式

此技能由 `/doc-investigate` 内部调用，不建议直接使用。

## 解析流程

1. **文件读取**
   - 使用 Read 工具读取文件内容
   - 检测文件编码和格式

2. **结构识别**
   - 识别标题层级
   - 提取列表和表格
   - 识别代码块和引用

3. **内容分类**
   - 识别需求描述
   - 提取功能列表
   - 识别约束条件

4. **信息整理**
   - 按 PRD 模板结构组织
   - 标记不确定的内容

## 输出格式

```yaml
source_file: "文件路径"
document_type: "markdown/text/other"
extracted_content:
  title: "文档标题"
  sections:
    - heading: "章节标题"
      content: "章节内容"
      type: "requirement/description/constraint"
  requirements:
    - id: "REQ-001"
      description: "需求描述"
      priority: "P0/P1/P2"
```

## 错误处理

- 文件不存在：提示用户检查路径
- 格式不支持：提示支持的格式列表
- 编码问题：尝试自动检测编码
```

**Step 2: 验证技能文件创建成功**

```bash
cat doc-workflow/skills/parsers/parse-doc.md | head -20
```

Expected: 显示 frontmatter 和功能说明

**Step 3: 验证支持的格式列表**

```bash
grep -A3 "支持以下格式" doc-workflow/skills/parsers/parse-doc.md
```

Expected: 显示 Markdown, 纯文本, Word 文档

**Step 4: 提交 parse-doc 技能**

```bash
git add doc-workflow/skills/parsers/parse-doc.md
git commit -m "feat: 添加文档解析器技能"
```

---

## Task 8: 实现 parse-meeting 解析器

**目标:** 创建会议记录解析器技能

**Files:**
- Create: `doc-workflow/skills/parsers/parse-meeting.md`

**Step 1: 创建 parse-meeting 技能文件**

创建文件 `doc-workflow/skills/parsers/parse-meeting.md`:

```markdown
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
```

**Step 2: 验证技能文件创建成功**

```bash
wc -l doc-workflow/skills/parsers/parse-meeting.md
```

Expected: 显示约 90+ 行

**Step 3: 验证输出格式定义**

```bash
grep -A5 "meeting_info:" doc-workflow/skills/parsers/parse-meeting.md
```

Expected: 显示会议信息的 YAML 结构

**Step 4: 提交 parse-meeting 技能**

```bash
git add doc-workflow/skills/parsers/parse-meeting.md
git commit -m "feat: 添加会议记录解析器技能"
```

---

## Task 9: 实现 parse-chat 解析器

**目标:** 创建对话/口述需求解析器技能

**Files:**
- Create: `doc-workflow/skills/parsers/parse-chat.md`

**Step 1: 创建 parse-chat 技能文件**

创建文件 `doc-workflow/skills/parsers/parse-chat.md`:

```markdown
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
```

**Step 2: 验证技能文件创建成功**

```bash
cat doc-workflow/skills/parsers/parse-chat.md | head -30
```

Expected: 显示 frontmatter 和功能说明

**Step 3: 验证交互流程定义**

```bash
grep "^### 阶段" doc-workflow/skills/parsers/parse-chat.md
```

Expected: 显示 4 个阶段

**Step 4: 提交 parse-chat 技能**

```bash
git add doc-workflow/skills/parsers/parse-chat.md
git commit -m "feat: 添加对话解析器技能"
```

---

## Task 10: 实现 doc-investigate 技能

**目标:** 创建需求调查技能，协调各解析器生成文档初稿

**Files:**
- Create: `doc-workflow/skills/doc-investigate.md`

**Step 1: 创建 doc-investigate 技能文件**

创建文件 `doc-workflow/skills/doc-investigate.md`:

```markdown
---
name: doc-investigate
description: 需求调查技能 - 分析输入源生成标准化需求文档初稿
---

# 需求调查技能 (Document Investigation)

## 功能说明

分析各种输入源，生成标准化的需求文档初稿。支持：
- URL 网站分析
- 本地文档解析
- 会议记录整理
- 交互式需求收集

## 使用方式

```bash
# 分析网站
/doc-investigate https://example.com

# 解析本地文档
/doc-investigate ./requirements.md

# 解析会议记录
/doc-investigate --meeting ./meeting-notes.md

# 交互式收集
/doc-investigate --chat
```

## 参数说明

| 参数 | 说明 | 示例 |
|------|------|------|
| `[source]` | 输入源（URL 或文件路径） | `https://example.com` |
| `--meeting` | 指定为会议记录 | `--meeting ./notes.md` |
| `--chat` | 进入交互模式 | `--chat` |
| `--type` | 文档类型 | `--type prd` |
| `--output` | 输出路径 | `--output ./docs/prd/` |

## 执行流程

### 1. 输入源识别

```
判断输入类型：
- 以 http:// 或 https:// 开头 → URL 类型
- 使用 --meeting 参数 → 会议记录类型
- 使用 --chat 参数 → 交互模式
- 其他 → 本地文档类型
```

### 2. 调用对应解析器

使用 Task 工具启动独立 Agent 执行解析：

```yaml
URL 类型:
  agent: "parse-url-agent"
  input: { url: "输入的URL" }

文档类型:
  agent: "parse-doc-agent"
  input: { file_path: "文件路径" }

会议类型:
  agent: "parse-meeting-agent"
  input: { file_path: "文件路径" }

交互类型:
  agent: "parse-chat-agent"
  input: { mode: "interactive" }
```

### 3. 生成文档初稿

1. 加载对应模板（PRD/技术设计）
2. 填充解析得到的信息
3. 标记需要人工补充的部分
4. 生成文档文件

### 4. 版本记录

自动调用 `/doc-version` 记录 V1.0 版本

## 输出

- 文档路径: `docs/prd/PRD-{项目名}-V1.0.md`
- 版本记录: `docs/versions/VERSION-{项目名}-history.md`

## 错误处理

| 错误 | 处理 |
|------|------|
| 输入源无法访问 | 提示检查 URL 或文件路径 |
| 解析失败 | 返回原始内容供人工处理 |
| 模板不存在 | 使用默认模板 |

## 示例输出

```
✓ 输入源识别: URL 类型
✓ 解析完成: 提取了 5 个功能模块
✓ 文档生成: docs/prd/PRD-Example-V1.0.md
✓ 版本记录: V1.0 已记录

下一步建议:
1. 查看生成的文档: cat docs/prd/PRD-Example-V1.0.md
2. 进行评审: /doc-review docs/prd/PRD-Example-V1.0.md
```
```

**Step 2: 验证技能文件格式正确**

```bash
head -10 doc-workflow/skills/doc-investigate.md
```

Expected: 显示 YAML frontmatter 和标题

**Step 3: 验证使用方式完整**

```bash
grep -A10 "## 使用方式" doc-workflow/skills/doc-investigate.md
```

Expected: 显示 4 种使用方式示例

**Step 4: 验证执行流程定义**

```bash
grep "^### " doc-workflow/skills/doc-investigate.md
```

Expected: 显示 4 个执行步骤

**Step 5: 提交 doc-investigate 技能**

```bash
git add doc-workflow/skills/doc-investigate.md
git commit -m "feat: 添加需求调查技能"
```

---

## Task 11: 实现 doc-review 技能

**目标:** 创建多维评审技能，实现 4 个评审维度

**Files:**
- Create: `doc-workflow/skills/doc-review.md`

**Step 1: 创建 doc-review 技能文件**

创建文件 `doc-workflow/skills/doc-review.md`:

```markdown
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
```

**Step 2: 验证技能文件格式正确**

```bash
head -10 doc-workflow/skills/doc-review.md
```

Expected: 显示 YAML frontmatter 和标题

**Step 3: 验证评审模式定义**

```bash
grep "^### " doc-workflow/skills/doc-review.md | head -10
```

Expected: 显示完全并行、分组执行、顺序执行等模式

**Step 4: 验证评分标准定义**

```bash
grep -A5 "## 评分标准" doc-workflow/skills/doc-review.md
```

Expected: 显示 4 个维度的评分权重

**Step 5: 提交 doc-review 技能**

```bash
git add doc-workflow/skills/doc-review.md
git commit -m "feat: 添加多维评审技能"
```

---

## Task 12: 实现 doc-version 技能

**目标:** 创建版本管理技能，实现自动版本追踪和差异分析

**Files:**
- Create: `doc-workflow/skills/doc-version.md`

**Step 1: 创建 doc-version 技能文件**

创建文件 `doc-workflow/skills/doc-version.md`:

```markdown
---
name: doc-version
description: 版本管理技能 - 追踪文档变更，生成差异报告
---

# 版本管理技能 (Document Version)

## 功能说明

追踪文档版本变更，提供：
- 版本历史记录
- 差异分析报告
- 修改模式分析
- Git 集成（可选）

## 使用方式

```bash
# 手动触发版本记录
/doc-version ./docs/prd/PRD-xxx.md

# 查看版本历史
/doc-version ./docs/prd/PRD-xxx.md --history

# 比较两个版本
/doc-version ./docs/prd/PRD-xxx.md --compare V1.0 V1.2
```

## 参数说明

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `[doc_path]` | 文档路径 | 必填 |
| `--history` | 显示版本历史 | false |
| `--compare` | 比较两个版本 | - |
| `--git` | 同时提交 Git | 配置决定 |

## 自动触发

当配置 `version.auto_trigger: true` 时，以下情况自动触发：
- 文档首次生成（V1.0）
- 用户通过 Claude Code 修改文档后
- 评审完成并应用修改后

## 执行流程

### 1. 检测文档变更

```yaml
检测方式:
  - 计算文件 hash
  - 与上一版本 hash 比较
  - 如果不同，触发版本记录
```

### 2. 确定版本号

```yaml
版本号规则:
  V{主版本}.{次版本}

自动递增:
  - 普通修改: 次版本 +1 (V1.0 → V1.1)
  - 重大重构: 主版本 +1 (V1.x → V2.0)

重大重构判断:
  - 章节结构变化 > 50%
  - 内容变化 > 70%
  - 用户显式指定
```

### 3. 生成差异报告

```yaml
差异分析内容:
  变更统计:
    - added: 新增数量
    - modified: 修改数量
    - deleted: 删除数量

  详细变更:
    - location: 位置
    - type: 类型
    - original: 原内容
    - new: 新内容
    - reason: 变更原因（如有）

  修改模式:
    - requirement_expansion: 需求扩展占比
    - detail_refinement: 细节完善占比
    - error_correction: 错误修正占比
```

### 4. 更新版本历史

更新文件: `docs/versions/VERSION-{项目名}-history.md`

```markdown
# 版本历史: PRD-项目名

## 版本链
V1.0 → V1.1 → V1.2 (当前)

## V1.2 (2026-02-05)
- 变更统计: +3 ~8 -1
- 主要变更: 完善功能需求章节
- 触发来源: 评审修改

## V1.1 (2026-02-04)
...
```

### 5. Git 集成（可选）

如果配置 `version.auto_git_commit: true`:

```bash
git add docs/prd/PRD-xxx-V1.2.md
git add docs/versions/VERSION-xxx-history.md
git commit -m "docs: 更新 PRD-xxx 至 V1.2"
```

## 输出

```
版本管理: PRD-Example

检测到文档变更
├── 文件 hash: abc123 → def456
├── 变更统计: +3 ~8 -1
└── 版本更新: V1.1 → V1.2

差异报告已生成: docs/versions/DIFF-Example-V1.1-V1.2.md
版本历史已更新: docs/versions/VERSION-Example-history.md

Git 提交: ✓ (commit: 1a2b3c4)
```

## 错误处理

| 错误 | 处理 |
|------|------|
| 文档不存在 | 提示检查路径 |
| 无前一版本 | 创建 V1.0 初始版本 |
| Git 操作失败 | 继续版本记录，提示手动处理 Git |
```

**Step 2: 验证技能文件格式正确**

```bash
head -10 doc-workflow/skills/doc-version.md
```

Expected: 显示 YAML frontmatter 和标题

**Step 3: 验证版本号规则定义**

```bash
grep -A10 "版本号规则" doc-workflow/skills/doc-version.md
```

Expected: 显示版本号格式和递增规则

**Step 4: 验证差异分析内容定义**

```bash
grep -A15 "差异分析内容" doc-workflow/skills/doc-version.md
```

Expected: 显示变更统计、详细变更、修改模式

**Step 5: 提交 doc-version 技能**

```bash
git add doc-workflow/skills/doc-version.md
git commit -m "feat: 添加版本管理技能"
```

---

## Task 13: 实现 doc-note 技能

**目标:** 创建备注记录技能，支持手动添加讨论要点

**Files:**
- Create: `doc-workflow/skills/doc-note.md`

**Step 1: 创建 doc-note 技能文件**

创建文件 `doc-workflow/skills/doc-note.md`:

```markdown
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
```

**Step 2: 验证技能文件格式正确**

```bash
head -10 doc-workflow/skills/doc-note.md
```

Expected: 显示 YAML frontmatter 和标题

**Step 3: 验证备注类型定义**

```bash
grep "^### " doc-workflow/skills/doc-note.md
```

Expected: 显示普通备注、会议记录、决策记录三种类型

**Step 4: 提交 doc-note 技能**

```bash
git add doc-workflow/skills/doc-note.md
git commit -m "feat: 添加备注记录技能"
```

---

## Task 14: 实现 doc-workflow 主入口技能

**目标:** 创建主入口技能，协调全流程

**Files:**
- Create: `doc-workflow/skills/doc-workflow.md`

**Step 1: 创建 doc-workflow 技能文件**

创建文件 `doc-workflow/skills/doc-workflow.md`:

```markdown
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
```

**Step 2: 验证技能文件格式正确**

```bash
head -10 doc-workflow/skills/doc-workflow.md
```

Expected: 显示 YAML frontmatter 和标题

**Step 3: 验证工作流程定义**

```bash
grep "^### Step" doc-workflow/skills/doc-workflow.md
```

Expected: 显示 5 个步骤

**Step 4: 验证状态保存定义**

```bash
grep -A10 "## 状态保存" doc-workflow/skills/doc-workflow.md
```

Expected: 显示状态文件路径和 JSON 结构

**Step 5: 提交 doc-workflow 技能**

```bash
git add doc-workflow/skills/doc-workflow.md
git commit -m "feat: 添加文档工作流程主入口技能"
```

---

## Task 15: 集成测试与文档完善

**目标:** 验证技能包完整性，完善文档

**Files:**
- Verify: 所有技能文件
- Update: `doc-workflow/README.md`

**Step 1: 验证目录结构完整**

```bash
find doc-workflow -type f -name "*.md" -o -name "*.yaml" | sort
```

Expected:
```
doc-workflow/README.md
doc-workflow/config/default-config.yaml
doc-workflow/skills/doc-investigate.md
doc-workflow/skills/doc-note.md
doc-workflow/skills/doc-review.md
doc-workflow/skills/doc-version.md
doc-workflow/skills/doc-workflow.md
doc-workflow/skills/parsers/parse-chat.md
doc-workflow/skills/parsers/parse-doc.md
doc-workflow/skills/parsers/parse-meeting.md
doc-workflow/skills/parsers/parse-url.md
doc-workflow/templates/prd-template.md
doc-workflow/templates/review-report-template.md
doc-workflow/templates/tech-design-template.md
```

**Step 2: 验证所有技能文件有 frontmatter**

```bash
for f in doc-workflow/skills/*.md doc-workflow/skills/parsers/*.md; do
  echo "=== $f ==="
  head -3 "$f"
done
```

Expected: 每个文件都显示 `---` 开头的 frontmatter

**Step 3: 更新 README.md 添加完整文件列表**

更新 `doc-workflow/README.md` 添加文件清单章节：

```markdown
## 文件清单

### 核心技能
- `skills/doc-workflow.md` - 主入口
- `skills/doc-investigate.md` - 需求调查
- `skills/doc-review.md` - 多维评审
- `skills/doc-version.md` - 版本管理
- `skills/doc-note.md` - 备注记录

### 解析器
- `skills/parsers/parse-url.md` - URL 解析
- `skills/parsers/parse-doc.md` - 文档解析
- `skills/parsers/parse-meeting.md` - 会议记录解析
- `skills/parsers/parse-chat.md` - 对话解析

### 模板
- `templates/prd-template.md` - PRD 模板
- `templates/tech-design-template.md` - 技术设计模板
- `templates/review-report-template.md` - 评审报告模板

### 配置
- `config/default-config.yaml` - 默认配置
```

**Step 4: 统计技能包规模**

```bash
echo "文件数量:" && find doc-workflow -type f | wc -l
echo "总行数:" && find doc-workflow -type f -exec cat {} \; | wc -l
```

Expected: 约 15 个文件，约 1500+ 行

**Step 5: 最终提交**

```bash
git add doc-workflow/
git commit -m "feat: 完成 doc-workflow 技能包 v1.0

包含:
- 5 个核心技能
- 4 个输入解析器
- 3 个文档模板
- 1 个配置文件

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"
```

---

## 实现计划完成

### 文件清单

| 类别 | 文件 | 描述 |
|------|------|------|
| 根目录 | `README.md` | 技能包说明 |
| 配置 | `config/default-config.yaml` | 默认配置 |
| 模板 | `templates/prd-template.md` | PRD 模板 |
| 模板 | `templates/tech-design-template.md` | 技术设计模板 |
| 模板 | `templates/review-report-template.md` | 评审报告模板 |
| 解析器 | `skills/parsers/parse-url.md` | URL 解析 |
| 解析器 | `skills/parsers/parse-doc.md` | 文档解析 |
| 解析器 | `skills/parsers/parse-meeting.md` | 会议记录解析 |
| 解析器 | `skills/parsers/parse-chat.md` | 对话解析 |
| 核心技能 | `skills/doc-investigate.md` | 需求调查 |
| 核心技能 | `skills/doc-review.md` | 多维评审 |
| 核心技能 | `skills/doc-version.md` | 版本管理 |
| 核心技能 | `skills/doc-note.md` | 备注记录 |
| 核心技能 | `skills/doc-workflow.md` | 主入口 |

### 预计提交历史

1. `feat: 创建 doc-workflow 技能包基础目录结构`
2. `feat: 添加默认配置文件`
3. `feat: 添加 PRD 文档模板`
4. `feat: 添加技术设计文档模板`
5. `feat: 添加评审报告模板`
6. `feat: 添加 URL 解析器技能`
7. `feat: 添加文档解析器技能`
8. `feat: 添加会议记录解析器技能`
9. `feat: 添加对话解析器技能`
10. `feat: 添加需求调查技能`
11. `feat: 添加多维评审技能`
12. `feat: 添加版本管理技能`
13. `feat: 添加备注记录技能`
14. `feat: 添加文档工作流程主入口技能`
15. `feat: 完成 doc-workflow 技能包 v1.0`

---

*本计划由 superpowers:writing-plans 技能生成*
