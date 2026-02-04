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
