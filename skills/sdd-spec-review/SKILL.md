---
name: sdd-spec-review
description: "SDD 3.1 规格评审。对 spec.md 进行多维度评审，发现问题则回退到 PRD_TO_SPEC 阶段。"
---

# SDD Spec Review — 规格评审（阶段 3.1）

## 职责

对技术规格进行系统性评审，确保规格完整、准确、可实现。

## 路径约定
本阶段所有文件读写基于当前需求目录 `.sdd/{需求目录}/`：
- spec → `.sdd/{需求目录}/docs/spec.md`
- reviews → `.sdd/{需求目录}/docs/reviews/`
- state → `.sdd/{需求目录}/state.yaml`

## 输入

- `.sdd/{需求目录}/docs/spec.md`
- PRD 原文（交叉验证）
- API 设计文档（一致性验证）

## 评审维度

### 1. 完整性检查
- [ ] PRD 中每个功能点是否都有对应 spec 条目
- [ ] 每个接口是否有完整的请求/响应/错误码定义
- [ ] 是否覆盖了异常流程和边界条件
- [ ] 非功能需求（性能、安全、监控）是否明确

### 2. 一致性检查
- [ ] 接口定义与 Swagger/API 文档是否一致
- [ ] 数据模型与 DB 设计是否一致
- [ ] 字段命名是否统一（不出现同一含义多种命名）
- [ ] 错误码是否与现有系统错误码体系一致

### 3. 可实现性检查
- [ ] 是否依赖不存在的服务或接口
- [ ] 性能要求是否合理可达
- [ ] 技术方案是否与现有架构兼容
- [ ] 是否有循环依赖或死锁风险

### 4. 可测试性检查
- [ ] 每条业务规则是否可转为具体的测试用例
- [ ] 边界条件是否可构造测试数据
- [ ] 异常分支是否可模拟触发

## 评审输出格式

```markdown
# Spec Review Report

## 评审结论: PASS / NEEDS_REVISION

## 问题列表
### P0 — 阻塞性问题（必须修复才能继续）
1. [spec.md#2.1] 接口 /api/xxx 缺少分页参数定义
2. [spec.md#4.1] 并发扣款场景未定义幂等策略

### P1 — 重要问题（建议修复）
1. [spec.md#5] 未定义接口超时时间

### P2 — 建议改进
1. [spec.md#3.1] 建议为 xxx 字段添加索引

## 待确认项（需人工回答）
1. PRD 中 xxx 功能的优先级是否变更？
```

## Gate 行为

- **manual 模式**：展示 review report，等用户确认
  - 用户说通过 → 进入下一阶段
  - 用户说有问题 → 回退 PRD_TO_SPEC，附带问题列表
- **auto 模式**：
  - 无 P0 → 自动通过
  - 有 P0 → 自动回退 PRD_TO_SPEC
- **skip 模式**：跳过评审

## 回退时的行为

回退到 PRD_TO_SPEC 时，必须：
1. 将 review report 中的问题传递给 sdd-prd-to-spec
2. sdd-prd-to-spec 只修改有问题的部分，不重写整个 spec
3. 修改后重新进入 SPEC_REVIEW

最多回退 3 次。超过 3 次强制进入 manual gate，要求人工介入。

## 产出物

- `.sdd/{需求目录}/docs/reviews/spec-review-{timestamp}.md`
