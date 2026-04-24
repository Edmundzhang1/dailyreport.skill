# Daily Work Skill

一个运行在 `Codex CLI` 中的日报蒸馏 Skill，用于从 `Codex` 本地对话日志中提取`已完成且有意义`的工作结果，并生成可直接汇报的中文纯文本总结。

## 1. 项目目标

这个 Skill 要解决的问题不是“把对话简单摘要一下”，而是：

从一天或最近若干自然日的 `Codex` 对话日志中，识别真正完成了什么工作，并把这些结果整理成可直接汇报的中文工作总结。

它关注的是：

- 工作结果
- 结果是否完成
- 结果是否有独立汇报价值

它不关注的是：

- 所有聊天过程
- 所有思考痕迹
- 所有运行事件
- Git、diff、shell、test 等其他数据源

## 2. 当前版本边界

当前版本是 `V1`，边界已经明确固定：

- 只支持 `Codex CLI`
- 只使用 `Codex` 对话日志作为事实来源
- 默认按当天自然日汇总
- 支持“最近 N 个自然日”的时间窗口表达
- 只输出中文纯文本总结
- 只纳入`已完成且有意义`的结果
- 不输出内部 `Work Unit`

以下内容明确不属于当前版本：

- `Claude Code`、`OpenCode` 或其他 agent
- `Git commit / diff`
- 文件修改记录
- `shell / test` 记录
- 多源融合
- 周报、绩效分析、长期画像
- 结构化 JSON 导出

## 3. 核心设计原则

这个 Skill 的设计遵循以下原则：

- 只基于对话日志，不引入额外事实源
- 只写结果，不写过程流水
- 以“完成结果”为最小汇报单位
- “有意义”不做固定枚举，但必须按统一原则判断
- 没有证据支撑的内容不补写
- 未完成事项不写入最终总结

这意味着最终输出不是“今天聊了什么”，而是“今天完成了什么”。

## 4. 目录结构

当前仓库结构如下：

```text
skill/
├── SKILL.md
├── README.md
├── prompts/
│   ├── intake.md
│   ├── result_extractor.md
│   ├── work_unit_merger.md
│   └── report_writer.md
└── references/
    ├── codex_log_workflow.md
    ├── judging_rules.md
    └── output_rules.md
```

各文件职责如下。

### `SKILL.md`

这是运行时主入口，供 `Codex` 触发和执行使用。

它负责：

- 描述 Skill 做什么
- 描述何时使用
- 指定执行主流程
- 限定当前版本边界

它不负责：

- 展开所有细节规则
- 承担仓库说明文档职责

### `README.md`

这是面向人类开发者和维护者的仓库说明文档。

它负责：

- 解释这个 Skill 的目标与边界
- 解释目录结构
- 说明如何安装、使用和测试
- 说明当前限制和迁移方式

### `prompts/`

这一层用于把蒸馏链路拆成多个职责清晰的阶段。

#### `prompts/intake.md`

用于收口输入请求，明确：

- 时间窗口
- 输出口径
- 事实边界

#### `prompts/result_extractor.md`

用于从日志中识别`已完成且有意义`的完成结果。

这是整个 Skill 的核心提取层。

#### `prompts/work_unit_merger.md`

用于将共同导向同一个完成结果的多轮对话组织为内部 `Work Unit`，并在必要时拆分多个独立结果。

#### `prompts/report_writer.md`

用于把筛选后的 `Work Unit` 转写为最终中文纯文本总结。

### `references/`

这一层用于存放规则和宿主相关说明。

#### `references/codex_log_workflow.md`

定义日志读取主链路，只允许从 `Codex` 本地会话日志中读取事实。

#### `references/judging_rules.md`

定义完成结果、准入规则、`Work Unit` 合并与拆分规则，以及判定原则。

#### `references/output_rules.md`

定义最终总结的输出口径与表达要求。

## 5. 工作方式

这个 Skill 的主流程可以概括为：

1. 收口用户请求
2. 读取目标时间窗口内的 `Codex` 会话日志
3. 识别已完成且有意义的结果
4. 将碎片对话组织成内部 `Work Unit`
5. 输出最终中文纯文本总结

从概念上看，整体链路是：

`Codex 对话日志 -> 完成结果 -> Work Unit -> 中文工作总结`

## 6. 日志来源

当前版本只使用这一条日志主链路：

```text
~/.codex/sessions/YYYY/MM/DD/rollout-*.jsonl
```

这是当前版本唯一认可的事实源。

以下内容不会被纳入主链路：

- `~/.codex/history.jsonl`
- `~/.codex/logs_2.sqlite`
- `state_*.sqlite`
- `Git / diff / shell / test`

原因是当前 `V1` 的目标是验证“仅靠对话日志能否稳定生成可信总结”，而不是构建多源行为分析系统。

## 7. 如何判定“能进总结”

只有同时满足以下两个条件的结果才会进入最终总结：

- 已完成
- 有意义

### 什么是“已完成”

某个任务目标已经闭环，并形成了明确结果、结论或交付物。

例如：

- 完成功能实现
- 完成问题定位
- 完成规则定义
- 完成方案选型
- 完成文档或规划书定稿
- 完成验证并形成明确结论

### 什么是“有意义”

不采用固定类型白名单，而由 agent 按统一原则判断。

通常应满足至少一项：

- 对工作推进有明确贡献
- 对结果产出有独立价值
- 对决策收敛形成了明确结论
- 可以被单独汇报

以下内容默认排除：

- 纯过程噪声
- 重复确认
- 礼貌性往返
- 无结论讨论
- 纯措辞微调

## 8. 如何使用

### 当前 Codex 中使用

在 `Codex CLI` 中直接点名使用：

```text
请使用 daily-work skill，总结今天已完成且有意义的工作结果。
```

总结最近两天：

```text
请使用 daily-work skill，总结最近两天已完成且有意义的工作结果。
```

也可以用更自然的表达，只要语义足够接近：

```text
请使用 daily-work skill，总结最近两天完成了什么。
帮我基于 Codex 对话日志整理今天的工作总结。
从 Codex 日志生成最近两天的工作总结。
```

### 时间窗口规则

- 未指定时，默认当天自然日
- “最近两天”表示最近 `2` 个自然日
- 不是最近 `48` 小时
- 时间边界按运行时本地时区计算

### 输出口径

- 单日窗口：`今日完成`
- 多日窗口：`周期内完成`

## 9. 安装方式

### 本地安装到当前 Codex

当前推荐的安装方式是将本仓库挂到：

```text
~/.codex/skills/daily-work
```

如果希望后续仓库修改能自动生效，优先使用链接而不是复制。

Windows 示例：

```powershell
New-Item -ItemType Junction `
  -Path $HOME\.codex\skills\daily-work `
  -Target C:\Users\zhangyifan\Desktop\Dailyreport\skill
```

Linux 示例：

```bash
mkdir -p ~/.codex/skills
ln -s /path/to/skill ~/.codex/skills/daily-work
```

安装后建议重启 `Codex CLI`，以确保新 Skill 被重新发现。

## 10. 测试方式

### 最小测试

用于验证 Skill 是否已被 Codex 发现：

```text
请使用 daily-work skill，总结今天已完成且有意义的工作结果。
```

如果当日日志里只有测试会话本身，出现空结果是正常现象。

### 有效测试

用于验证真实蒸馏效果：

```text
请使用 daily-work skill，总结最近两天已完成且有意义的工作结果。
```

这会让 Skill 读取最近两个自然日的 `rollout-*.jsonl`，并输出真实的工作结果总结。

### 当前已验证的事实

本仓库当前已经完成过以下验证：

- Skill 已安装到当前 `Codex` 技能目录
- 新起的 `codex exec` 进程能够发现并执行该 Skill
- Skill 能从 `~/.codex/sessions/.../rollout-*.jsonl` 读取会话日志
- Skill 能输出按结果组织的中文周期总结

## 11. Linux 迁移

当前结构非常适合迁移到 Linux，原因是：

- 当前版本没有平台专属脚本
- 蒸馏逻辑都在 `SKILL.md / prompts / references`
- 唯一平台相关点主要是日志发现路径

只要 Linux 上的 `Codex` 日志结构仍然满足：

```text
~/.codex/sessions/YYYY/MM/DD/rollout-*.jsonl
```

那么这套 Skill 基本可以直接迁移使用。

如果未来 Linux 侧日志结构与当前不同，通常只需要调整：

- `references/codex_log_workflow.md`

而不需要推倒整个蒸馏规则层。

## 12. 当前限制

当前版本有几个明确限制。

### 只支持 Codex

不支持 `Claude Code`、`OpenCode` 或其他 agent。

### 只看对话日志

不看 `Git`、`diff`、文件修改、`shell / test` 记录。

### 不输出未完成事项

当前版本只写“已完成且有意义”的结果，不写进行中事项、风险或计划。

### 对日志编码和读取环境有依赖

在某些 Windows PowerShell 默认编码场景下，直接读取中文 `SKILL.md` 可能出现乱码。  
当前实测中，使用 UTF-8 读取 `prompts/` 与 `references/` 能正常工作，因此不影响 Skill 主流程，但这是后续应继续收敛的兼容点。

## 13. 后续方向

当前仓库后续可继续沿这些方向迭代，但都不属于当前 `V1` 已交付范围：

- 优化 `SKILL.md` 的编码稳健性
- 继续前向测试不同时间窗口下的结果粒度
- 扩展到更多 code agent
- 引入更多辅助证据源
- 扩展周报等更长周期总结

## 14. 维护建议

维护这个 Skill 时，建议遵循以下分层，不要把职责混在一起：

- `SKILL.md`：入口与流程，不堆长规则
- `prompts/`：蒸馏阶段逻辑
- `references/`：宿主规则与判定规则
- `README.md`：给人看的仓库说明

如果后续确实出现“日志读取必须程序化”的需求，再单独引入 `tools/` 或脚本层，而不是提前过度设计。
