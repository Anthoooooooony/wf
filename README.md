# wf

一个聚焦的四阶段工作流插件：**brainstorm（脑暴） → plan（规划） → code（编码） → review（审查）**。

仅支持 slash 命令显式调用（不会被模型自动触发）。所有过程产物写到项目根下的 `.wf/`（默认进 `.gitignore`）。当 review 阶段发现 confidence ≥ 80 的问题时，强制把状态切回 `coding` 并结束当前会话。

## 安装

```
/plugin marketplace add Anthoooooooony/wf
/plugin install wf@wf-marketplace
```

第一条命令把这个仓库注册为名为 `wf-marketplace` 的 marketplace（名字来自 `.claude-plugin/marketplace.json` 的 `name` 字段，与 GitHub 仓库名 `wf` 不同）。第二条命令从该 marketplace 安装名为 `wf` 的 plugin。

如果在 marketplace 添加后想刷新到最新提交，运行：

```
/plugin marketplace update wf-marketplace
```

## 命令

| 命令 | 阶段 | 主要产物 |
|---|---|---|
| `/wf:brainstorm` | 阶段 1：探明意图，提出 2-3 个产品/UX 层方案，写设计文档 | `.wf/brainstorm/<feature_id>.md`（视觉问题另存自包含 HTML mockup） |
| `/wf:plan` | 阶段 2：架构决策，撰写 PRD 和 ADR | `.wf/plans/PRD-<feature_id>.md`、`.wf/adr/ADR-NNN-*.md` |
| `/wf:code` | 阶段 3：实现；并行调度 explorer / architect / reviewer 三个子代理协作 | 源代码文件；阶段保持 `coding` |
| `/wf:review` | 阶段 4：基于五档 confidence scoring 的独立审查 | `.wf/reviews/<feature_id>-<round>.md`；只有零个 ≥80 issue 才能进 `done` |
| `/wf:archive` | （可选）清理：把当前已完成 feature 的产物收拢到 `.wf/archive/<feature_id>/` 并重置 STATE | `.wf/archive/<feature_id>/` 子目录；ADR 留在 `.wf/adr/` 不动 |

## 状态机

```
brainstorm → planning → coding ⇄ reviewing → done →（可选 /wf:archive）→ 重置
                          ↑          │
                          └──────────┘
                  （任意 issue confidence ≥ 80）
```

`.wf/STATE.md` 是当前阶段的唯一事实源。每个 skill 进入时校验 phase，不匹配就拒绝执行并提示运行哪个命令。

## 产物布局

```
.wf/
├── STATE.md
├── brainstorm/
│   ├── <feature_id>.md
│   └── <feature_id>-visual-<n>.html        # 可选，自包含
├── plans/
│   └── PRD-<feature_id>.md
├── adr/
│   └── ADR-NNN-<slug>.md
├── reviews/
│   └── <feature_id>-<round>.md
└── archive/                                 # 由 /wf:archive 创建（可选）
    └── <feature_id>/
        ├── brainstorm.md
        ├── brainstorm-visual-<n>.html
        ├── PRD.md
        └── reviews/
            └── round-<N>.md
```

首次运行时，插件会自动把 `.wf/` 追加到项目的 `.gitignore`（如果还没有的话）。如果你想让 PRD/ADR 进入 git 历史，手动 `mv` 到 `docs/` 即可——插件不会替你做这个决定。

## 设计来源（吸收 vs 修改）

本插件直接吸收了若干上游 skill 的英文原文，再做针对性修改：

| 来源 | 吸收 | 修改 |
|---|---|---|
| Superpowers `brainstorming` | HARD-GATE 子句、9 步 checklist、spec self-review 四象限、"This Is Too Simple" 反模式、key principles | 移除自动触发、移除自动 git commit、移除 `writing-plans` 强绑定；spec 路径从 `docs/superpowers/specs/` 改为 `.wf/brainstorm/`；可视化从浏览器服务器（visual-companion）替换为自包含 HTML + `open` 自动打开 |
| mattpocock `zoom-out` | `disable-model-invocation: true` 字段约定、"使用项目的领域词汇" 原则 | 无 |
| mattpocock `improve-codebase-architecture` | 8 个架构术语 glossary（Module / Interface / Implementation / Depth / Seam / Adapter / Leverage / Locality）、三条核心原则（deletion test、interface = test surface、one vs two adapters）、ADR offer 措辞 | ADR 路径从 `docs/adr/` 改为 `.wf/adr/`；不再 inline 改写 `CONTEXT.md`，而是询问用户 |
| Anthropic `feature-dev` | 7-phase 骨架、并行子代理调度模式、`code-architect` / `code-explorer` / `code-reviewer` 三个 agent 定义、五档 confidence scoring（0/25/50/75/100） | Phase 6 软 gate（fix now/later/proceed）替换为硬规则：任何 ≥80 的 issue 强制把 phase 切回 `coding` 并结束会话 |
| Anthropic `frontend-design` | UI 美学清单（NEVER 列表、intentionality 原则） | 在 `wf:code` 末尾以**软引用**方式调用；如果没装就 fallback 到内嵌的 inline checklist |

## 这个插件为什么存在

现有 skill 倾向于把整套工作流（brainstorm → 实现 → 完成）打包到一个 skill 里、自动触发自己、强绑定其他 skill、自动提交产物到 git。本插件把四个阶段拆成可独立调用的单元，所有产物本地化到 `.wf/`，并通过 `STATE.md` 显式管理阶段流转——让 brainstorm/plan/code/review 这个循环对"自己当前在哪一步"保持诚实。
