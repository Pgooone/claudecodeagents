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
