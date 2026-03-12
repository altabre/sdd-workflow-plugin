---
name: sdd-tdd-impl
description: "SDD 6.1 TDD 实现。按任务列表逐个执行 TDD 循环：写测试 → 实现 → 重构 → 自审。遇到 blocker 可回退到 spec 阶段。"
---

# SDD TDD Implementation — 测试驱动开发实现（阶段 6.1）

## 职责

按 `.sdd/{需求目录}/docs/tasks.md` 逐个任务执行 TDD 开发循环。这是核心编码阶段。

## 路径约定
本阶段所有文件读写基于当前需求目录 `.sdd/{需求目录}/`：
- tasks → `.sdd/{需求目录}/docs/tasks.md`
- spec → `.sdd/{需求目录}/docs/spec.md`
- plan → `.sdd/{需求目录}/docs/plan.md`
- blocker → `.sdd/{需求目录}/docs/reviews/blocker-{timestamp}.md`
- state → `.sdd/{需求目录}/state.yaml`

## 输入

- `.sdd/onboard.md`（项目概览 — 了解架构和规范）
- `.sdd/{需求目录}/docs/tasks.md`（任务列表）
- `.sdd/{需求目录}/docs/spec.md`（技术规格，作为实现参考）
- `.sdd/{需求目录}/docs/plan.md`（E2E 方案，作为架构参考）
- `.sdd/{需求目录}/state.yaml`（当前 task 编号）

## TDD 循环（对每个 Task）

### 第一步：Red — 写失败的测试

```
读取 task 的验收标准
  → 为每个验收标准写一个测试方法
  → 测试必须能编译但执行失败（Red）
  → 如果是接口测试，mock 依赖
```

规则：
- 测试类命名: `{ClassName}Test.java`
- 测试方法命名: `test_{场景}_{期望结果}`
- 每个验收标准至少一个测试
- 包含正常流程 + 异常流程 + 边界条件

### 第二步：Green — 写最少代码让测试通过

```
实现生产代码
  → 只写让测试通过的最少代码
  → 不做提前优化
  → 不做超出 task 范围的事
  → 运行测试，确认全部 Green
```

规则：
- 严格按照 spec 定义实现，不自行假设
- 命名与 spec 一致
- 如果发现 spec 不完整或有矛盾，标记为 blocker

### 第三步：Refactor — 重构

```
测试全部通过后
  → 消除重复代码
  → 优化命名和结构
  → 确保符合项目代码规范
  → 重新运行测试，确认仍然 Green
```

### 第四步：Spec Compliance Review — 规格一致性审查

以 spec-reviewer 身份检查：
- [ ] 实现是否严格符合 spec 定义的接口规格
- [ ] 业务逻辑是否与 spec 描述一致
- [ ] 数据模型是否与 spec 定义一致
- [ ] 错误码和异常处理是否与 spec 一致

审查不通过 → 修复后重新审查

### 第五步：Quality Review — 代码质量审查

以 quality-reviewer 身份检查：
- [ ] 是否有硬编码的值应该配置化
- [ ] 是否有安全隐患（SQL 注入、XSS、敏感信息泄露）
- [ ] 是否有性能隐患（N+1 查询、大量内存分配）
- [ ] 日志是否合理
- [ ] 异常处理是否完善
- [ ] 命名是否清晰

审查通过 → 进入下一个 task
审查不通过 → 修复后重新审查

## Blocker 处理

如果实现过程中发现：
- spec 定义不完整或有矛盾
- API 设计与 spec 不一致
- DB 设计无法支撑业务逻辑
- 依赖的外部服务接口不存在

处理流程：
1. 在 `.sdd/{需求目录}/docs/reviews/blocker-{timestamp}.md` 记录问题
2. 根据 gate 配置：
   - manual: 暂停，展示 blocker，等用户决定回退到哪个阶段
   - auto: 自动回退到 PRD_TO_SPEC
   - flow: 与 manual 相同，始终暂停（blocker 无法自动解决）

## 状态更新

每完成一个 task，更新 `.sdd/{需求目录}/state.yaml`：

```yaml
current_stage: TDD_IMPL
current_task: {N+1}
```

## 进度报告

每完成一个 task，输出：

```
✅ Task {N}/{Total}: {描述}
   - 新增文件: 3
   - 测试数: 5 (5 passed)
   - 下一个: Task {N+1}: {描述}
```

## 完成条件

- [x] 所有 task 已完成
- [x] 所有测试通过
- [x] 无未解决的 blocker

## 产出物

- 生产代码
- 测试代码
- 可能的 blocker 报告
