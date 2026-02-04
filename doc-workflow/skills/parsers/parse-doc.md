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
