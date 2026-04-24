# Codex 日志读取规则

本文件定义 `V1` 的唯一日志读取主链路。

## 唯一事实源

`V1` 只使用 `Codex` 本地会话日志目录作为事实源：

`~/.codex/sessions/YYYY/MM/DD/rollout-*.jsonl`

不要将以下文件混入 `V1` 主链路：

- `~/.codex/history.jsonl`
- `~/.codex/logs_2.sqlite`
- 任何 `state_*.sqlite`

原因很简单：`sessions` 目录天然按日期拆分，便于按自然日窗口读取，并能直接保留会话级上下文。

## 时间窗口读取

按当前运行时本地时区计算自然日窗口。

读取步骤如下：

1. 先确定目标自然日集合。
2. 再定位对应的目录：
   - `~/.codex/sessions/YYYY/MM/DD/`
3. 读取该目录下全部 `rollout-*.jsonl` 文件。
4. 按文件时间与记录时间组织会话顺序。

## 记录筛选

优先使用以下记录作为主事实材料：

- `type = response_item`
- `payload.type = message`
- `payload.role = user`
- `payload.role = assistant`

以下记录只作为上下文或去重辅助，不作为主事实材料：

- `payload.role = developer`
- `type = turn_context`
- `type = session_meta`
- `type = event_msg`
- `type = reasoning`

尤其不要把这些事件直接写进工作总结：

- `task_started`
- `token_count`
- `turn_aborted`
- 其他运行态事件

## 去重原则

同一轮消息可能同时以不同事件形式出现。

处理时优先保留真实消息内容，不要把以下内容重复计入：

- `response_item.message`
- `event_msg.user_message`
- `event_msg.agent_message`

主链路以 `response_item.message` 为准。

## 中断处理

如果日志中出现 `turn_aborted`，说明该轮可能未闭环。

处理原则如下：

- 若后续对话重新推进并完成同一个结果，则按完整闭环结果判断
- 若中断后没有形成闭环，则该段不计入已完成结果

## 读取结果要求

进入蒸馏前，至少保证已拿到以下内容：

- 目标时间窗口内的全部相关 `rollout-*.jsonl`
- 其中的用户消息
- 其中的 assistant 消息
- 每条消息对应的时间顺序

如果无法满足这些条件，应明确说明无法完成日志读取，而不是猜测结果。
