---
name: daily-work
description: 基于当前 Agent 宿主的本地对话日志生成中文工作总结。用于在 Codex、Claude Code 或 OpenClaw 环境中按当天自然日或最近 N 个自然日读取本地会话日志，提取已完成且有意义的工作结果，并输出可直接汇报的纯文本总结。用户提到“总结今天的工作”“帮我整理这两天完成了什么”“从对话日志生成日报/工作总结”等场景时使用。
---

# Daily Work

生成基于当前宿主对话日志的纯文本工作总结。

## 固定边界

- 支持 `Codex`
- 支持 `Claude Code`
- 支持 `OpenClaw`
- 只使用当前宿主的本地对话日志作为事实来源
- 默认汇总当天自然日
- 支持最近 `N` 个自然日的窗口表达
- 只总结已完成且有意义的工作结果
- 不输出内部 `Work Unit`
- 不将 `Git / diff / 文件修改 / shell / test` 记录纳入事实来源
- 不跨宿主混用日志

## 执行流程

1. 先阅读 [prompts/intake.md](prompts/intake.md)，收口本次总结请求的时间窗口、输出口径和固定约束。
2. 再识别当前宿主，并只读取对应的日志工作流说明：
   - `Codex`：阅读 [references/codex_log_workflow.md](references/codex_log_workflow.md)
   - `Claude Code`：阅读 [references/claude_code_log_workflow.md](references/claude_code_log_workflow.md)
   - `OpenClaw`：阅读 [references/openclaw_log_workflow.md](references/openclaw_log_workflow.md)
3. 接着阅读 [references/judging_rules.md](references/judging_rules.md) 与 [prompts/result_extractor.md](prompts/result_extractor.md)，识别已完成且有意义的结果。
4. 然后阅读 [prompts/work_unit_merger.md](prompts/work_unit_merger.md)，将共同导向同一个完成结果的对话组织为内部 `Work Unit`。
5. 最后阅读 [references/output_rules.md](references/output_rules.md) 与 [prompts/report_writer.md](prompts/report_writer.md)，输出最终纯文本总结。

## 宿主识别规则

优先按当前 Agent 运行时判断宿主，而不是按机器上安装了哪些工具判断。

判断原则如下：

- 如果当前运行时是 `Codex`，只读取 `Codex` 日志
- 如果当前运行时是 `Claude Code`，只读取 `Claude Code` 日志
- 如果当前运行时是 `OpenClaw`，只读取 `OpenClaw` 日志

不要因为同一台机器上同时存在多个宿主目录，就把它们混在一起。

## 输出要求

- `days=1` 时使用“今日完成”口径
- `days>1` 时使用“周期内完成”口径
- 只写结果，不写过程流水
- 不补写日志中不存在的新任务、新结论和新计划

## 失败条件

遇到以下情况时，不要猜测结果，应直接说明原因并停止生成总结：

- 当前宿主不是 `Codex`、`Claude Code` 或 `OpenClaw`
- 无法访问当前宿主所需的会话日志
- 用户要求超出本 Skill 的事实边界
- 在目标时间窗口内没有足够证据支撑任何已完成且有意义的结果

如果日志可读，但在目标时间窗口内未识别到符合准入条件的结果，明确输出：

`在所选时间窗口内，未识别到已完成且有意义的工作结果。`
