---
name: sdd-spec-to-tasks
description: "SDD 5.1 规格 → 任务拆分。将 spec 和 plan 拆分为可独立执行的开发任务列表 .sdd/{需求目录}/docs/tasks.md。"
---

# SDD Spec to Tasks — 任务拆分（阶段 5.1）

## 职责

将 spec 和 plan 拆分为粒度合适、可独立执行的开发任务。每个任务应该是 TDD 实现阶段可以独立完成的最小单元。

## 路径约定
本阶段所有文件读写基于当前需求目录 `.sdd/{需求目录}/`：
- spec → `.sdd/{需求目录}/docs/spec.md`
- plan → `.sdd/{需求目录}/docs/plan.md`
- tasks → `.sdd/{需求目录}/docs/tasks.md`
- state → `.sdd/{需求目录}/state.yaml`

## 输入

- `.sdd/{需求目录}/docs/spec.md`
- `.sdd/{需求目录}/docs/plan.md`

## 拆分原则

1. **单一职责** — 一个 task 只做一件事
2. **可独立测试** — 每个 task 完成后可以跑测试验证
3. **有明确输入输出** — 依赖什么、产出什么
4. **按依赖排序** — 被依赖的 task 排在前面
5. **粒度适中** — 一个 task 预期 1-3 个文件变更

## 任务格式

生成 `.sdd/{需求目录}/docs/tasks.md`：

```markdown
# 任务列表

## 概览
- 总任务数: N
- 预计涉及文件数: M

## Tasks

### Task 1: {简短描述}
- **类型**: model | dao | service | controller | config | test
- **文件**:
  - 新增: `src/.../XxxEntity.java`
  - 新增: `src/.../XxxMapper.java`
  - 新增: `src/.../XxxMapper.xml`
- **依赖**: 无
- **验收标准**:
  - [ ] Entity 字段与 DB 设计一致
  - [ ] Mapper 接口定义完整
  - [ ] 单元测试通过
- **关联 spec**: spec.md#3.1

### Task 2: {简短描述}
- **类型**: service
- **文件**:
  - 新增: `src/.../XxxService.java`
  - 新增: `src/.../XxxServiceImpl.java`
- **依赖**: Task 1
- **验收标准**:
  - [ ] 实现 spec.md#4.1 中定义的业务逻辑
  - [ ] 异常分支覆盖
  - [ ] 单元测试覆盖率 >= 80%
- **关联 spec**: spec.md#4.1

### Task N: ...
```

## 自检

- [ ] 所有 spec 功能点都被 task 覆盖
- [ ] 所有 plan 中的文件都在某个 task 中
- [ ] 依赖顺序无环
- [ ] 每个 task 有明确的验收标准

## 完成后

更新 `.sdd/{需求目录}/state.yaml`：

```yaml
current_stage: SPEC_TO_TASKS
total_tasks: {N}
```

## 产出物

- `.sdd/{需求目录}/docs/tasks.md`
