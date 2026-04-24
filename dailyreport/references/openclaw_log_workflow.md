# OpenClaw 日志读取规则

本文件定义 `OpenClaw` 宿主下的日志读取主链路。

## 唯一事实源

`OpenClaw` 官方文档说明其会话持久化包含两层：

- `sessions.json`
- transcript JSONL

其中实际对话 transcript 位于：

- `~/.openclaw/agents/<agentId>/sessions/<sessionId>.jsonl`

因此本宿主只使用 transcript JSONL 作为事实源，不使用 `sessions.json` 作为总结事实来源。

## 日志范围

读取范围如下：

- `~/.openclaw/agents/*/sessions/*.jsonl`

如果存在 session reset 或归档产生的 transcript 文件，并且其中记录落在目标时间窗口内，也应纳入同一宿主的读取范围。

不要将以下内容混入主链路：

- `sessions.json`
- 其他运行日志
- Git、diff、shell、test 等非对话日志

## 时间窗口读取

`OpenClaw` 支持多 agent，因此必须跨 agent 枚举 transcript 文件。

读取步骤如下：

1. 枚举 `~/.openclaw/agents/*/sessions/` 下的 transcript JSONL。
2. 以 transcript 记录中的时间戳作为自然日窗口判断依据。
3. 文件修改时间只能作为粗筛，不可替代记录时间戳。
4. 将目标窗口内的相关 transcript 合并为统一候选日志集合。

## 记录筛选

`OpenClaw` 官方文档说明 transcript 中不仅包含对话，还包含工具调用与 compaction 摘要。

只保留：

- 用户消息
- assistant 消息

过滤掉：

- tool call
- tool result
- compaction summary
- session store 信息
- 其他非用户可见运行态记录

如果当前 OpenClaw 版本的 transcript 字段布局与预期不同，先抽样读取少量记录，识别真实用户/assistant 记录字段后再批量处理。

## 多 agent 处理

如果目标时间窗口内多个 agent 都有对话记录，允许跨 agent 汇总，但前提是它们都属于同一个 `OpenClaw` 宿主事实源。

不要把 OpenClaw transcript 与其他宿主日志混用。

## 中断与重置

OpenClaw 的 reset 或 daily reset 可能导致同一逻辑工作被拆分到多个 transcript 中。

处理原则如下：

- 若多个 transcript 共同导向同一个完成结果，应在后续阶段合并判断
- 若只是中断，没有形成闭环，则不计入已完成结果

## 读取结果要求

进入蒸馏前，至少保证已拿到以下内容：

- 目标时间窗口内的相关 transcript JSONL
- 其中的用户消息
- 其中的 assistant 消息
- 每条消息对应的时间顺序

如果无法满足这些条件，应明确说明无法完成日志读取，而不是猜测结果。
