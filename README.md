# j-code-review

一个 Claude Code 技能，按 Google 工程实践的标准审查代码改动。它只接受让整体代码健康变好的改动，发现按 P0–P3 定级。下结论前，必须先说出反对理由，并标注证据强度。

## 它做什么

审查 PR、diff 或单个文件的改动。整个过程只问一件事：这次改动有没有让系统的整体代码健康变好。不存在完美代码，只存在更好的代码。改动只要让代码健康明确变好，即使不完美，也该倾向批准。改动让代码健康明确变差，一律不接受。

检查按维度逐个进行，顺序就是优先级：Design（含模块深度）→ Functionality → Complexity → Errors/Contracts → Tests → Naming → Comments → Style/Consistency → Documentation。另设一节「结构与抽象红旗」，专门找能删掉复杂度的机会。把代码整理干净不算过关。

判定标准、失败分支和输出契约，以 [j-code-review/SKILL.md](./j-code-review/SKILL.md) 为唯一权威。本文件只说明去哪找规则，不重复规则。

## 安装

把仓库里的 `j-code-review/` 子目录整个拷到 skills 目录，让 `SKILL.md` 位于该目录根部。要拷的是内层目录，不是仓库根：

```
j-code-review/            仓库根
├── README.md
├── LICENSE
├── docs/
│   └── sources.md        标准来源的逐章对照
└── j-code-review/        技能本体
    └── SKILL.md
```

- 用户级，对所有项目生效：Windows `C:\Users\<用户名>\.claude\skills\j-code-review\`；macOS / Linux `~/.claude/skills/j-code-review/`
- 项目级，只对该项目生效：`<项目根>/.claude/skills/j-code-review/`

## 使用

技能在 frontmatter 里设了 `disable-model-invocation: true`。模型不会自动触发这个技能，需要你手动调用：

```
/j-code-review 审查当前分支相对 main 的改动
/j-code-review 帮我看看 src/auth/session.ts 这个文件
```

description 里列了触发词，用来手动检索和识别：代码审查、code review、帮我审查这个文件/PR、这个 PR 能不能过、帮忙看看这段改动、cr。

## 输出长什么样

每条发现按「上线后会发生什么」定级，不按修复难度定级。下面列出输出形状的速览，权威定义在 SKILL.md 的「输出」一节：

| 级别 | 含义 | 对结论的影响 |
| --- | --- | --- |
| P0 | 崩溃、死锁、数据丢失/损坏、安全漏洞、核心功能不可用 | 阻止项：拒绝，或需修改后批准 |
| P1 | 确定是 bug，但有绕行办法或影响范围有限；或明确违反验收标准 | 同上 |
| P2 | 行为正确，但存在真实的健壮性、性能、一致性或可维护性风险 | 需修改后批准，或带强烈建议批准 |
| P3 | 非功能性建议：风格、命名、注释、结构调整 | `Nit:` / `Optional:` |

结论三选一：批准（含附带评论）/ 需修改后批准 / 拒绝。下结论前，先列出最强的反面理由，并给每条理由标注证据强度：`代码证据` / `测试证据` / `推测`。然后声明实际跑过哪些验证，以及本次没覆盖的范围。零发现时也要声明。证据不足，或疑虑无法消除时，不给批准结论。

## 适用边界

- 只审一次，看完就给结论。审查过程不回看历史轮次，也不维护跨轮状态。如果要修一轮审一轮、最后判收敛，改用 review-bounded 技能；它做三轮有界收敛审查，输出 CONVERGED / NOT CONVERGED 判定。
- 没有改动意图或 diff 时不开审，先问清审什么、范围多大。
- 改动大于约 500 行，或文件超过 1000 行时，技能会建议按模块拆成多轮，第一轮只审核心路径。
- 作者明确只要语法 / 风格速扫时，只做该范围，并在开头注明。

## 背景：标准从哪来

规则不是自拟的，拼装自六份来源。SKILL.md 里每条规则都带行内引用，可以逐条回溯。逐章对照见 [docs/sources.md](./docs/sources.md)。

| 来源 | 借用了什么 |
| --- | --- |
| [Google Engineering Practices](https://google.github.io/eng-practices/) | 整体代码健康这个唯一门槛；`Nit:` / `Optional:` / `FYI:` 评论前缀；冲突处理与升级路径 |
| A Philosophy of Software Design, 2e（John Ousterhout，技能中简称 APoSD） | 复杂度与模块深度理论：深浅模块、信息泄漏、透传、合与分、让错误情形消失 |
| Code Complete, 2e（Steve McConnell） | 命名、注释（含 document surprises）、典型错误类别清单，以及「读代码发现的缺陷多于跑测试」这条依据 |
| [review-bounded](https://github.com/ZM-BAD/review-bounded)（外部技能，MIT） | P0–P3 严重度标尺、按上线后果定级、结论前先陈述反对理由与证据强度标签、复审重读最新代码；不借它的三轮编排 |
| [precision-code-review](https://github.com/ydpzg/precision-code-review)（外部技能） | 行为链路追踪、本次变更与基线问题分离、发现名单过滤、待确认事项分区、四种验证状态 |
| thermo-nuclear 系列（外部技能） | 激进简化维度：1000 行红线、面条式增长、优先删复杂度 |

后三行是技能，不是公开规范。它们的权威性低于前三份，也不凌驾于项目授权。各借鉴来源的授权条件，以各自仓库的声明为准。

## 许可证

MIT，见 [LICENSE](./LICENSE)。
