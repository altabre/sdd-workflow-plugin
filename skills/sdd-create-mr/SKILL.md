---
name: sdd-create-mr
description: "SDD 10.1 创建 MR。所有验证通过后，整理提交记录，创建 Merge Request。"
---

# SDD Create MR — 创建合并请求（阶段 10.1）

## 职责

整理代码变更，创建规范的 Merge Request。

## 路径约定
本阶段所有文件读写基于当前需求目录 `.sdd/{需求目录}/`：
- spec → `.sdd/{需求目录}/docs/spec.md`
- reviews → `.sdd/{需求目录}/docs/reviews/`
- state → `.sdd/{需求目录}/state.yaml`

## 输入

- 所有代码变更（git diff）
- `.sdd/{需求目录}/docs/spec.md`
- `.sdd/{需求目录}/docs/reviews/e2e-verify-*.md`（最新的验证报告）
- `.sdd/config.yaml`（commit 规范、分支命名）

## 执行步骤

### 1. 整理提交

```
检查 git status
  → 确保没有遗漏的文件
  → 确保 .sdd/ 目录不包含在提交中（加入 .gitignore）
  → 按 config 中的 commit_convention 整理提交信息
```

### 2. 推送分支

```
确认分支命名符合 config 中的 branch_prefix
  → git push -u origin {branch}
```

### 3. 创建 MR

使用 `gh pr create` 或 `git push -o merge_request.create`，内容包含：

```markdown
## Summary
{基于 spec 的一句话描述}

## Changes
{基于 plan 的变更清单}

## Test Results
- Unit Tests: ✅ X passed
- Integration Tests: ✅ X passed
- E2E Tests: ✅ X passed
- Coverage: X%

## Spec Reference
{spec.md 的关键章节摘要或链接}

## Checklist
- [x] 代码符合技术规格
- [x] 单元测试通过
- [x] E2E 测试通过
- [x] Code Review 通过
- [x] 无安全隐患
```

### 4. 清理

- 更新 state.yaml 为 DONE
- 输出 MR 链接

## Gate 行为

- **manual**: 展示 MR 内容预览，等用户确认后创建
- **auto**: 直接创建
- **flow**: 自动创建 MR，无需用户确认（与 `auto` 行为相同）
- **skip**: 不创建 MR，只推送代码

## 完成条件

- [x] MR 已创建
- [x] state.yaml 标记为 DONE

## 产出物

- Merge Request
- 更新后的 `.sdd/{需求目录}/state.yaml`
