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
