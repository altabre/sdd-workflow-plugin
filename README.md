# SDD Workflow Plugin

Spec-Driven Development (SDD) 规格驱动开发流程 Claude Code 插件。

## 流程概览

```
PRD + API Doc + DB Design
        ↓
  1.  Project Onboard       — 初始化项目环境，生成项目概览
  2.  PRD to Spec            — 产品需求 → 技术规格
  3.  Spec Review            — 规格评审（可回退 2）
  4.  Spec to E2E Plan       — 规格 → 端到端方案
  5.  Spec to Tasks          — 规格 → 任务拆分
  6.  TDD Implementation     — 测试驱动开发（循环）
  7.  Code Review            — 代码评审（可回退 6）
  8.  E2E Tests              — E2E 测试编写
  9.  E2E Verify             — E2E 执行验证（可回退 6）
 10.  Create MR              — 创建合并请求
```

## 安装

```bash
# 方式一：从 GitHub marketplace
/plugin marketplace add yourorg/sdd-workflow
/plugin install sdd-workflow

# 方式二：手动安装
git clone https://github.com/yourorg/sdd-workflow-plugin ~/.claude/plugins/sdd-workflow
```

## 使用

```bash
# 启动新需求或恢复未完成的需求
/sdd

# 切换到指定需求
/sdd SEC-1234

# 列出所有需求及状态
/sdd list

# 查看当前需求进度
/sdd status

# 切换需求
/sdd switch SEC-1234

# 跳过当前阶段
/sdd skip

# 回退到上一阶段
/sdd back

# 重置当前需求
/sdd reset

# 归档已完成的需求
/sdd archive SEC-1234
```

## 配置

在项目根目录创建 `.sdd/config.yaml`，或在首次 `/sdd` 时自动生成。

关键配置项：

```yaml
gates:
  spec_review: manual        # manual | auto | skip
  e2e_plan_review: manual
  code_review: auto
  e2e_verify: manual
  create_mr: manual
```

## 目录结构

```
.sdd/
  ├── config.yaml                          — 项目级配置（共享）
  ├── onboard.md                           — 项目概览（共享）
  ├── 20260311-SEC-1234-pause-contract/    — 需求 1
  │   ├── state.yaml
  │   ├── inputs/
  │   └── docs/
  │       ├── spec.md
  │       ├── plan.md
  │       ├── tasks.md
  │       ├── e2e-test-plan.md
  │       └── reviews/
  └── 20260315-SEC-1235-alert-display/     — 需求 2
```

## 设计原则

1. **Spec 是唯一真相源** — 所有代码必须有 spec 对应
2. **TDD 强制** — 先测试后实现
3. **产出物全部落文件** — 不依赖对话上下文
4. **可配置自主程度** — gate 控制人机协作边界
5. **可回退** — 发现问题随时回退，不带着问题往下走
