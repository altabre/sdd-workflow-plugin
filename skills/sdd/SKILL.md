---
name: sdd
description: "SDD 主编排器。当用户说 /sdd、开始开发、新需求 等触发。管理多需求目录，按阶段顺序调度子 Skill，管理状态流转和反馈环路。"
---

# SDD Workflow — Spec-Driven Development 主编排

你是 SDD 流程的编排器。你的职责是管理需求目录、按顺序调度各阶段 Skill、管理状态、处理反馈环路。

## 目录结构

```
.sdd/
  ├── config.yaml                          — 项目级全局配置（共享）
  ├── 20260311-SEC-1234-pause-contract/    — 需求 1
  │   ├── state.yaml
  │   ├── inputs/
  │   └── docs/
  ├── 20260315-SEC-1235-alert-display/     — 需求 2
  └── ...
```

- 全局配置：`.sdd/config.yaml`
- 项目概览：`.sdd/onboard.md`（所有需求共享的项目上下文）
- 需求状态：`.sdd/{需求目录}/state.yaml`
- 需求产出：`.sdd/{需求目录}/docs/`

## 状态机

```
INIT → ONBOARD → [IDEATE →] PRD_TO_SPEC → SPEC_REVIEW → SPEC_TO_PLAN → SPEC_TO_TASKS → TDD_IMPL → CODE_REVIEW → E2E_TESTS → E2E_VERIFY → CREATE_MR → DONE
```

IDEATE 为可选阶段：
- `inputs/prd.md` 或用户提供了 PRD → 跳过 IDEATE，直接进入 PRD_TO_SPEC
- 没有 PRD 且 `config.yaml` 中 `inputs.prd: optional` → 进入 IDEATE

回退路径：
- SPEC_REVIEW 不通过 → PRD_TO_SPEC
- TDD_IMPL 遇到 blocker → PRD_TO_SPEC 或 SPEC_TO_PLAN
- CODE_REVIEW 不通过 → TDD_IMPL
- E2E_VERIFY 不通过 → TDD_IMPL

## 启动流程

1. 读取 `.sdd/config.yaml`，不存在则调用 `sdd-onboard` 创建
2. 确定当前需求：
   - 如果用户指定了需求（`/sdd SEC-1234`）→ 切换到该需求
   - 如果只有一个未完成需求 → 自动选择
   - 如果有多个未完成需求 → 列出，让用户选
   - 如果没有未完成需求 → 调用 `sdd-onboard` 创建新需求
3. 读取需求的 `state.yaml` 恢复当前阶段
4. 如果当前阶段是 `ONBOARD` 完成后，检查是否需要进入 IDEATE：
   - `inputs/prd.md` 存在 → 跳过，进入 PRD_TO_SPEC
   - 用户在对话中提供了 PRD 内容 → 保存到 `inputs/prd.md`，进入 PRD_TO_SPEC
   - 两者都没有 且 `config.yaml` 中 `inputs.prd: optional` → 调用 `sdd-ideate`
   - 两者都没有 且 `config.yaml` 中 `inputs.prd: required` → 提示用户提供 PRD
5. 从当前阶段开始，按顺序调度子 Skill

## 子 Skill 调度时的路径规则

所有子 Skill 的文件读写路径基于当前需求目录：
- spec → `.sdd/{需求目录}/docs/spec.md`
- plan → `.sdd/{需求目录}/docs/plan.md`
- tasks → `.sdd/{需求目录}/docs/tasks.md`
- e2e-test-plan → `.sdd/{需求目录}/docs/e2e-test-plan.md`
- reviews → `.sdd/{需求目录}/docs/reviews/`
- state → `.sdd/{需求目录}/state.yaml`

调度子 Skill 时，必须将当前需求目录路径传递给子 Skill。

## Gate 机制

每个阶段完成后，根据 config.yaml 中的 gates 配置决定：
- `manual` — 暂停，展示产出物，等用户确认后继续
- `auto` — AI 自动评审，通过则继续，不通过则回退
- `flow` — 智能自动流转：有卡点（P0问题/待确认项/架构决策）时暂停，否则自动推进
- `skip` — 跳过该阶段

### Gate → 阶段映射

| config gate 键 | 触发阶段 | 不通过回退到 |
|---------------|---------|------------|
| `spec_review` | SPEC_REVIEW (3) | PRD_TO_SPEC |
| `e2e_plan_review` | SPEC_TO_PLAN (4) | PRD_TO_SPEC |
| `code_review` | CODE_REVIEW (7) | TDD_IMPL |
| `e2e_verify` | E2E_VERIFY (9) | TDD_IMPL |
| `create_mr` | CREATE_MR (10) | — |

无 gate 的阶段（ONBOARD, PRD_TO_SPEC, SPEC_TO_TASKS, TDD_IMPL, E2E_TESTS）自动进入下一阶段。

### `flow` 模式暂停条件

| 阶段 | 暂停条件 | 自动通过条件 |
|------|---------|------------|
| SPEC_REVIEW | 有 P0 问题 或 待确认项 > 0 | 无 P0，无待确认项 |
| SPEC_TO_PLAN | 有重大架构分歧需用户选择 | 方案清晰，AI 可自行决策 |
| CODE_REVIEW | 有 MUST FIX 问题 | 无 MUST FIX（SHOULD FIX 自动记录） |
| E2E_VERIFY | 有测试失败 | 所有测试通过 |
| CREATE_MR | 从不暂停 | 始终自动创建 |
| TDD_IMPL blocker | 始终暂停 | — |

## 状态文件格式

```yaml
# .sdd/{需求目录}/state.yaml
requirement: "SEC-1234-pause-contract"
created_at: "2026-03-11T10:00:00Z"
current_stage: TDD_IMPL
current_task: 3
total_tasks: 8
history:
  - stage: ONBOARD
    status: completed
    timestamp: "2026-03-11T10:00:00Z"
  - stage: PRD_TO_SPEC
    status: completed
    timestamp: "2026-03-11T10:15:00Z"
loop_backs: 0
```

## 调度规则

```
阶段完成 → 写入产出物到 {需求目录}/docs/ → 更新 state.yaml → 检查 gate
  → manual: 展示产出物，问用户 "是否通过？"
    → 通过: 进入下一阶段
    → 不通过: 用户说明问题 → 回退到指定阶段
  → auto: AI 自评
    → 通过: 进入下一阶段
    → 不通过: 自动回退
  → skip: 直接进入下一阶段
```

## 命令

- `/sdd` — 启动新需求或恢复未完成的需求
- `/sdd {slug}` — 切换到指定需求（如 `/sdd SEC-1234`）
- `/sdd list` — 列出所有需求及状态
- `/sdd status` — 查看当前需求的阶段和进度
- `/sdd switch {slug}` — 切换需求
- `/sdd skip` — 跳过当前阶段
- `/sdd back` — 回退到上一阶段
- `/sdd reset` — 重置当前需求（需确认）
- `/sdd archive {slug}` — 归档已完成的需求

### `/sdd status`
读取当前需求的 state.yaml，输出：阶段、task 进度、回退次数、历史记录。

### `/sdd skip`
将当前阶段标记为 skipped 写入 history，current_stage 前进到下一阶段。TDD_IMPL 阶段不允许 skip。

### `/sdd back`
将 current_stage 回退到上一阶段。ONBOARD 阶段不允许 back。回退时不删除已有产出物（允许修改）。

### `/sdd reset`
询问用户确认，重置当前需求的 state.yaml 为 ONBOARD 阶段。不删除已有产出物。加 `--hard` 参数则同时删除产出物。

### `/sdd archive {slug}`
将已完成（DONE）的需求目录移动到 `.sdd/_archived/` 下。未完成的需求不允许 archive。
