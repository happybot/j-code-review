# j-code-review

一个 Claude Code 技能，按 Google 工程实践的标准审查代码改动。它只接受让整体代码健康变好的改动，发现按 P0–P3 定级。下结论前，必须先说出反对理由，并标注证据强度。

## 它做什么

审查 PR、diff、分支或单个文件的改动。整个过程只问一件事：这次改动有没有让系统的整体代码健康变好。不存在完美代码，只存在更好的代码。改动只要让代码健康明确变好，即使不完美，也该倾向批准。改动让代码健康明确变差，一律不接受。

检查按维度逐个进行，顺序就是优先级：Design（含模块深度）→ Functionality → Complexity → Errors/Contracts → Tests → Naming → Comments → Style/Consistency → Documentation。另设一节「结构与抽象红旗」，专门找能删掉复杂度的机会。把代码整理干净不算过关。

## 安装

把仓库里的 `j-code-review/` 子目录整个拷到 skills 目录，让 `SKILL.md` 位于该目录根部。要拷的是内层目录，不是仓库根：

```
j-code-review/            仓库根
├── README.md
├── LICENSE
├── docs/
│   ├── sources.md        标准来源的逐章对照
│   └── skill-usage.md    技能的安装、使用与文件结构说明
└── j-code-review/        技能本体
    ├── SKILL.md
    ├── references/
    │   ├── review-checklist.md
    │   ├── language-hotspots.md
    │   ├── test-quality.md
    │   └── structure-red-flags.md
    ├── assets/
    │   └── review-report-template.md
    └── evals/
        ├── evals.json
        ├── trigger-evals.json
        ├── output-quality-evals.json
        ├── capability-evals.json
        ├── mode-selection-evals.json
        ├── real-sample-regression-evals.json
        └── scoring-boundary-evals.json
```

- 用户级，对所有项目生效：Windows `C:\Users\<用户名>\.claude\skills\j-code-review\`；macOS / Linux `~/.claude/skills/j-code-review/`
- 项目级，只对该项目生效：`<项目根>/.claude/skills/j-code-review/`

技能目录的详细说明见 [docs/skill-usage.md](./docs/skill-usage.md)。

## 使用

技能在 frontmatter 里设了 `disable-model-invocation: true`。模型不会自动触发这个技能，需要你手动调用：

```
/j-code-review 审查当前分支相对 main 的改动
/j-code-review 帮我看看 src/auth/session.ts 这个文件
/j-code-review 发版前全量审查 v1.4.0 以来的改动
```

description 里列了触发词，用来手动检索和识别：代码审查、code review、帮我审查这个文件/PR、这个 PR 能不能过、帮忙看看这段改动、cr。

## 审查模式

| 模式 | 何时用 | 输出 |
| --- | --- | --- |
| 快速扫描 | 用户只要速扫；或改动小于 50 行、单文件、低风险 | 结论 + 发现 + 验证与范围 |
| 标准审查 | 默认；改动 50–500 行，或规模未知 | 完整报告 |
| 发布前全量 | 发版前；改动大于 500 行或跨多个模块 | 按子系统分节 + 整体结论 |

高风险区包括认证、权限、支付、数据删除、数据迁移、并发与事务。改动落在高风险区时，至少用标准审查。

## 输出长什么样

每条发现按「上线后会发生什么」定级，不按修复难度定级。下面列出输出形状的速览，权威定义在 SKILL.md 的「输出格式」一节，字段顺序见 `assets/review-report-template.md`：

| 级别 | 含义 | 对结论的影响 |
| --- | --- | --- |
| P0 | 崩溃、死锁、数据丢失/损坏、安全漏洞、核心功能不可用 | 阻止项：拒绝，或需修改后批准 |
| P1 | 确定是 bug，但有绕行办法或影响范围有限；或明确违反验收标准 | 同上 |
| P2 | 行为正确，但存在真实的健壮性、性能、一致性或可维护性风险 | 需修改后批准，或带强烈建议批准 |
| P3 | 非功能性建议：风格、命名、注释、结构调整 | `Nit:` / `Optional:` |

结论三选一：批准（含附带评论）/ 需修改后批准 / 拒绝。下结论前，先列出最强的反面理由，并给每条理由标注证据强度：`代码证据` / `测试证据` / `推测`。然后声明实际跑过哪些验证，以及本次没覆盖的范围。零发现时也要声明。证据不足，或疑虑无法消除时，不给批准结论。

报告固定包含九节：结论、审查对象与范围、变更判定、发现、待确认事项、反对理由与证据强度、已核对但不列出、验证与搜索空间、下一步。

## 适用边界

- 只审一次，看完就给结论。审查过程不回看历史轮次，也不维护跨轮状态。如果要修一轮审一轮、最后判收敛，改用 review-bounded 技能；它做三轮有界收敛审查，输出 CONVERGED / NOT CONVERGED 判定。
- 没有改动意图或 diff 时不开审，先问清审什么、范围多大。
- 改动大于约 500 行，或文件超过 1000 行时，技能会建议按模块拆成多轮，第一轮只审核心路径。用户要求一次审完时，改为按子系统分节输出。
- 作者明确只要语法 / 风格速扫时，只做该范围，并在开头注明。
- `frontend-code-review` 可以提供前端专项检查表，作为补充输入；结论仍由本技能给。
- `yotta-code-quality` 输出 0–100 的健康度评分；本技能不产出健康分，只给合并结论。
- `security-review` 做专项安全审计；本技能只把安全问题计入 P0/P1。
- `simplify` 直接改代码做清理，不产出审查结论。

## 测例与回归

`j-code-review/evals/` 下有 43 条测例，覆盖六类：

| 文件 | 覆盖内容 |
| --- | --- |
| `trigger-evals.json` | 5 条正向、5 条负向、4 条边界触发 |
| `output-quality-evals.json` | 报告字段、三要素、反对理由、验证状态、待确认分区、过滤名单、零发现声明、快速扫描简版 |
| `capability-evals.json` | JS 异步、Python 可变默认参数、mock 滥用、语义归属、SQL 注入与对象级授权、误导性命名 |
| `mode-selection-evals.json` | 快速扫描、标准审查、高风险区、发布前全量、用户限定范围 |
| `real-sample-regression-evals.json` | 真实审查记录（已脱敏） |
| `scoring-boundary-evals.json` | P1/P2 边界、文档问题、无失败场景、纯偏好、测试同错、禁止批准条件 |

真实样例取自实际审查记录，已脱敏。修改触发边界、输出契约或定级规则后，同步更新对应测例。

## 背景：标准从哪来

规则不是自拟的，核心来源六份，2026-10 又增补四份。来源与借用条目记录在 [docs/sources.md](./docs/sources.md)，可以逐条回溯。

| 来源 | 借用了什么 |
| --- | --- |
| [Google Engineering Practices](https://google.github.io/eng-practices/) | 整体代码健康这个唯一门槛；`Nit:` / `Optional:` / `FYI:` 评论前缀；冲突处理与升级路径 |
| A Philosophy of Software Design, 2e（John Ousterhout，技能中简称 APoSD） | 复杂度与模块深度理论：深浅模块、信息泄漏、透传、合与分、让错误情形消失 |
| Code Complete, 2e（Steve McConnell） | 命名、注释（含 document surprises）、典型错误类别清单，以及「读代码发现的缺陷多于跑测试」这条依据 |
| [review-bounded](https://github.com/ZM-BAD/review-bounded)（外部技能，MIT） | P0–P3 严重度标尺、按上线后果定级、结论前先陈述反对理由与证据强度标签、复审重读最新代码；不借它的三轮编排 |
| [precision-code-review](https://github.com/ydpzg/precision-code-review)（外部技能） | 行为链路追踪、本次变更与基线问题分离、发现名单过滤、待确认事项分区、四种验证状态 |
| thermo-nuclear 系列（外部技能） | 激进简化维度：1000 行红线、面条式增长、优先删复杂度 |

后三行是技能，不是公开规范。它们的权威性低于前三份，也不凌驾于项目授权。各借鉴来源的授权条件，以各自仓库的声明为准。

2026-10 增补来源：

- [code-inspector-zhiai](https://github.com/ZhiHui6/code-inspector-zhiai)：语言热点、验证命令发现、误报控制
- [cuckoo-skills-pack](https://github.com/jiangchengnay/cuckoo-skills-pack)：安全快速检查
- [dev-skeleton](https://github.com/glenzli/dev-skeleton)：语义归属、真源分散、范围蔓延、地图漂移
- [yotta-code-quality](https://github.com/YottaMeta/yotta-code-quality)：测试腐化 T1-T6、发现要写清后果

逐项借用与未借用条目见 [docs/sources.md](./docs/sources.md#2026-10-增补来源)。

## 许可证

MIT，见 [LICENSE](./LICENSE)。
