---
name: sdd-onboard
description: "SDD 1.0 项目初始化。创建需求目录，生成 config.yaml，收集输入物料，建立项目上下文。"
---

# SDD Onboard — 项目初始化（阶段 1.0）

## 职责

初始化 SDD 工作环境，收集必要输入，建立项目上下文。

## 目录命名规范

每个需求一个独立目录，格式：

```
.sdd/{YYYYMMDD}-{slug}/
```

- `YYYYMMDD` — 需求启动日期
- `slug` — 需求简短标识，小写英文+连字符，来源优先级：
  1. JIRA ticket 编号（如 `SEC-1234`）
  2. 用户指定的名称（如 `pause-contract`）
  3. 从 PRD 标题自动提取（如 `user-auth-refactor`）

示例：
```
.sdd/
  ├── config.yaml                          — 项目级全局配置（只需创建一次）
  ├── 20260311-SEC-1234-pause-contract/    — 需求 1
  │   ├── state.yaml
  │   ├── inputs/
  │   └── docs/
  │       ├── spec.md
  │       ├── plan.md
  │       ├── tasks.md
  │       └── reviews/
  ├── 20260315-SEC-1235-alert-display/     — 需求 2
  │   ├── state.yaml
  │   ├── inputs/
  │   └── docs/
  └── 20260320-ad-api-refactor/            — 需求 3（无 JIRA）
      ├── state.yaml
      ├── inputs/
      └── docs/
```

## 执行步骤

### 1. 检查全局配置

检查 `.sdd/config.yaml` 是否存在：
- 存在 → 读取，跳到步骤 3
- 不存在 → 首次使用，执行步骤 2

### 2. 创建全局配置 & 项目概览（仅首次）

#### 2a. 生成项目概览 `.sdd/onboard.md`

自动扫描项目，回答四个问题：

1. **项目是什么** — 读 README / CLAUDE.md / 依赖声明，一段话说清楚项目做什么、核心功能、技术栈
2. **如何 Setup** — 读 README / Makefile / Dockerfile / package.json，提取环境依赖和安装步骤
3. **如何启动** — 读启动脚本 / application.yml / .env / docker-compose，提取本地启动命令和所需配置
4. **如何跑 E2E** — 读测试目录 / CI 配置 / Makefile，提取 E2E 测试运行方式和环境依赖

将结果写入 `.sdd/onboard.md`。这个文件是所有需求的**共享上下文**，后续每个子 Skill 执行前都应先读取它。

#### 2b. 自动检测技术栈并生成 config.yaml

```yaml
# .sdd/config.yaml — 项目级配置，所有需求共享
project:
  name: "项目名"
  repo: "仓库地址"
  tech_stack: "auto-detected"
  test_framework: "auto-detected"
  build_tool: "auto-detected"

inputs:
  prd: required
  api_design: required
  db_design: required

gates:
  spec_review: manual
  e2e_plan_review: manual
  code_review: auto
  e2e_verify: manual
  create_mr: manual

defaults:
  branch_prefix: "feat/"
  commit_convention: "conventional"
  test_coverage_threshold: 80
  max_loop_backs: 3
```

自动检测：
- 技术栈（读 pom.xml / go.mod / Cargo.toml / package.json）
- 测试框架（JUnit / TestNG / Go test / pytest）
- 构建工具（Maven / Gradle / Make）
- 代码规范（.editorconfig / lint 配置）

### 3. 创建需求目录

向用户确认需求标识：

```
请提供需求信息：
1. 需求名称/简述？
2. JIRA ticket？（可选）
```

根据回答生成目录名，如 `.sdd/20260311-SEC-1234-pause-contract/`

### 4. 收集输入物料

按全局 config 中的 inputs 配置收集：

**必选（required 的项）：**
- PRD 文档路径或链接（飞书/Confluence/本地文件）
- API 设计文档 / Swagger 链接
- DB 设计文档（表结构、ER 图）

**可选（optional 的项）：**
- JIRA ticket 链接
- Figma 设计稿
- 技术约束说明

写入 `.sdd/{需求目录}/inputs/README.md`

### 5. 初始化状态

```yaml
# .sdd/{需求目录}/state.yaml
requirement: "SEC-1234-pause-contract"
created_at: "2026-03-11T10:00:00Z"
current_stage: ONBOARD
current_task: 0
total_tasks: 0
history: []
loop_backs: 0
```

## 列出所有需求

`/sdd list` 扫描 `.sdd/` 下所有子目录，输出：

```
# SDD 需求列表
| # | 需求 | 阶段 | 进度 | 创建时间 |
|---|------|------|------|---------|
| 1 | SEC-1234-pause-contract | TDD_IMPL | 5/8 | 2026-03-11 |
| 2 | SEC-1235-alert-display | PRD_TO_SPEC | - | 2026-03-15 |
| 3 | ad-api-refactor | DONE | 8/8 | 2026-03-20 |
```

## 切换需求

`/sdd switch {slug}` — 切换到指定需求的上下文

## 完成条件

- [x] 全局 config.yaml 存在
- [x] 需求目录已创建
- [x] 输入物料已收集
- [x] state.yaml 已初始化

## 产出物

- `.sdd/config.yaml`（如果首次）
- `.sdd/onboard.md`（如果首次 — 项目概览，所有需求共享）
- `.sdd/{需求目录}/state.yaml`
- `.sdd/{需求目录}/inputs/README.md`
