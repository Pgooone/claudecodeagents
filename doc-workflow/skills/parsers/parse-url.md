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