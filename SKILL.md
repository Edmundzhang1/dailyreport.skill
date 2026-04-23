---
name: daily-work
description: 基于 Codex CLI 对话日志生成中文工作总结。用于在 Codex 环境中按当天自然日或最近 N 个自然日读取本地会话日志，提取已完成且有意义的工作结果，并输出可直接汇报的纯文本总结。用户提到“总结今天的工作”“帮我整理这两天在 Codex 做了什么”“从 Codex 日志生成日报/工作总结”等场景时使用。
---

# Daily Work

生成基于 `Codex` 对话日志的纯文本工作总结。

## 固定边界

- 只支持 `Codex CLI`
- 只使用 `Codex` 对话日志作为事实来源
- 默认汇总当天自然日
- 支持最近 `N` 个自然日的窗口表达
- 只总结已完成且有意义的工作结果
- 不输出内部 `Work Unit`
- 不将 `Git / diff / 文件修改 / shell / test` 记录纳入事实来源

## 执行流程

1. 先阅读 [prompts/intake.md](prompts/intake.md)，收口本次总结请求的时间窗口、输出口径和固定约束。
2. 再阅读 [references/codex_log_workflow.md](references/codex_log_workflow.md)，只按该文档定义的方式定位并读取 `Codex` 会话日志。
3. 接着阅读 [references/judging_rules.md](references/judging_rules.md) 与 [prompts/result_extractor.md](prompts/result_extractor.md)，识别已完成且有意义的结果。
4. 然后阅读 [prompts/work_unit_merger.md](prompts/work_unit_merger.md)，将共同导向同一个完成结果的对话组织为内部 `Work Unit`。
5. 最后阅读 [references/output_rules.md](references/output_rules.md) 与 [prompts/report_writer.md](prompts/report_writer.md)，输出最终纯文本总结。

## 输出要求

- `days=1` 时使用“今日完成”口径
- `days>1` 时使用“周期内完成”口径
- 只写结果，不写过程流水
- 不补写日志中不存在的新任务、新结论和新计划

## 失败条件

遇到以下情况时，不要猜测结果，应直接说明原因并停止生成总结：

- 无法访问所需的 `Codex` 会话日志
- 用户要求超出本 Skill 的事实边界
- 在目标时间窗口内没有足够证据支撑任何已完成且有意义的结果

如果日志可读，但在目标时间窗口内未识别到符合准入条件的结果，明确输出：

`在所选时间窗口内，未识别到已完成且有意义的工作结果。`
