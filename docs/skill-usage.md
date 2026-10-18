# j-code-review 使用说明

单次代码审查技能。只接受改善整体代码健康的改动；发现按 P0–P3 定级；结论三选一：批准、需修改后批准、拒绝。

## 安装

把仓库里的 `j-code-review/` 子目录整个拷到 skills 目录，让 `SKILL.md` 位于该目录根部。

- 用户级：Windows `C:\Users\<用户名>\.claude\skills\j-code-review\`；macOS / Linux `~/.claude/skills/j-code-review/`
- 项目级：`<项目根>/.claude/skills/j-code-review/`

## 使用

技能设置了 `disable-model-invocation: true`，需要手动调用：

```
/j-code-review 审查当前分支相对 main 的改动
/j-code-review 帮我看看 src/auth/session.ts 这个文件
/j-code-review 发版前全量审查 v1.4.0 以来的改动
```

## 文件结构

```
j-code-review/
├── SKILL.md                         触发边界、工作流、输出契约
├── references/
│   ├── review-checklist.md          九个维度的检查项、行为链路、错误类别
│   ├── language-hotspots.md         语言专项高信号检查
│   ├── test-quality.md              测试腐化 T1-T6
│   └── structure-red-flags.md       结构红旗的判定信号与改法
├── assets/
│   └── review-report-template.md    报告字段与顺序
└── evals/
    ├── evals.json                   测例清单
    ├── trigger-evals.json           正向、负向、边界触发
    ├── output-quality-evals.json    输出质量
    ├── capability-evals.json        语言、测试、语义归属、安全能力
    ├── mode-selection-evals.json    模式选择
    ├── real-sample-regression-evals.json  真实样例回归
    └── scoring-boundary-evals.json  P0–P3 定级边界
```

## 审查模式

| 模式 | 何时用 | 输出 |
| --- | --- | --- |
| 快速扫描 | 用户只要速扫；或改动小于 50 行、单文件、低风险 | 结论 + 发现 + 验证与范围 |
| 标准审查 | 默认 | 完整报告 |
| 发布前全量 | 发版前、超大改动、跨模块 | 按子系统分节 + 整体结论 |

高风险区（认证、权限、支付、数据删除、迁移、并发、事务）至少用标准审查。

## 与其他审查技能的分工

- `review-bounded`：做三轮「审查—修复」循环和收敛判定。本技能只做单次审查。
- `frontend-code-review`：提供前端专项检查表评分。可以作为本技能的补充输入，结论仍由本技能给。
- `yotta-code-quality`：输出 0–100 的健康度评分。本技能不产出健康分，只给合并结论。
- `security-review`：做专项安全审计。本技能只把安全问题计入 P0/P1。
- `simplify`：直接改代码做清理，不产出审查结论。
- 内置 `code-review`：后台多角度找 bug。本技能可以作为裁决层，但要重新核实它的发现。

## 测例

`evals/` 下的 JSON 是回归测例，不是运行时依赖。修改 SKILL.md 的触发边界、输出契约或定级规则后，应同步更新对应测例。

真实样例回归测例取自实际审查记录，已脱敏。
