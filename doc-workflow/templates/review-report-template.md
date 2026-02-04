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
