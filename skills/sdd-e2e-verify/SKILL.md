---
name: sdd-e2e-verify
description: "SDD 9.1 E2E 执行与验证。执行 E2E 测试，验证通过后进入 MR 创建阶段。"
---

# SDD E2E Verify — E2E 执行与验证（阶段 9.1）

## 职责

执行 E2E 测试，收集结果，验证功能完整性。

## 路径约定
本阶段所有文件读写基于当前需求目录 `.sdd/{需求目录}/`：
- e2e-test-plan → `.sdd/{需求目录}/docs/e2e-test-plan.md`
- reviews → `.sdd/{需求目录}/docs/reviews/`
- state → `.sdd/{需求目录}/state.yaml`

## 输入

- E2E 测试代码
- `.sdd/{需求目录}/docs/e2e-test-plan.md`
- 泳道环境信息（如需要）

## 执行步骤

### 1. 环境检查

```
检查：
  → 项目是否可以编译通过
  → 测试依赖是否就绪（数据库、中间件）
  → 配置是否正确
```

### 2. 执行测试

```
运行全部测试（单元测试 + 集成测试 + E2E 测试）
  → 收集测试结果
  → 收集覆盖率数据
  → 记录失败的测试及错误信息
```

### 3. 结果分析

```markdown
# E2E 验证报告

## 结论: PASS / FAIL

## 测试统计
- 单元测试: X passed / Y total
- 集成测试: X passed / Y total
- E2E 测试: X passed / Y total
- 覆盖率: X%

## 失败测试（如有）
### {测试名}
- 错误信息: xxx
- 可能原因: xxx
- 建议修复: xxx
```

### 4. 验证循环

如果有测试失败：
1. 分析失败原因
2. 如果是代码 bug → 回退 TDD_IMPL 修复对应 task
3. 如果是测试本身的问题 → 修复测试后重新执行
4. 如果是环境问题 → 标记，等人工处理

最多自动重试 2 轮。超过后进入 manual gate。

## Gate 行为

- **manual**: 展示验证报告，等用户确认
- **auto**: 全部通过则继续，否则回退
- **skip**: 跳过（不推荐）

## 完成条件

- [x] 所有测试通过
- [x] 覆盖率达到 config 中的阈值
- [x] 验证报告已生成

## 产出物

- `.sdd/{需求目录}/docs/reviews/e2e-verify-{timestamp}.md`
