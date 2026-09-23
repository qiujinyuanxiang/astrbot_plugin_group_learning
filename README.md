# 群聊主题学习插件

插件先保存白名单群的文字原话，再在设定的每日或每周时间按指定主题批量筛选和提炼为群知识。它不会训练模型；“学习”是建立一份可检索、可清空的本地知识库。

## 工作流程

```text
白名单群消息
  → 保存候选原话到本地
  → 到达每日/每周学习时间
  → 主题词排序（例如 vrchat 或 vrc 的消息优先）
  → 代码过滤垃圾信息
  → 小模型逐条回复 KEEP / DROP
  → 将 KEEP 消息按条数和字符数合并为批次
  → 大模型一次提炼多条知识
  → 回答时从当前群检索相关知识
```

## 后台配置

重载插件后，在 AstrBot WebUI 的“插件 → 群聊学习”中配置：

1. 在目标 QQ 群发送 `/sid`，取得群 ID；填入 `allowed_group_ids`。
2. `capture_all_messages` 默认开启，会先保存白名单群的所有文字原话；如只想保存命中主题的原话可关闭它。
3. 在 `listen_topics` 填需要重点学习的词。例如 `vrchat` 与 `vrc`；命中消息会优先交给模型，但不命中的候选原话仍会由小模型判断是否值得学习。
4. 选择 `schedule_mode`：`daily`（每天）或 `weekly`（每周）。
5. 在 `schedule_time` 填时间，例如 `03:00` 或 `21:30`；每周模式再填 `weekly_weekdays`，可用 `mon` 到 `sun`。
6. 选择 `filter_provider_id` 小模型，以及 `learning_provider_id` 大模型。
7. 按群聊情况调整垃圾关键词、正则表达式、每次最多处理的消息数，以及大模型批次大小。

默认每 10 条通过小模型筛选的原话才调用一次大模型，且单批原话最多 3000 个字符。可以通过 `batch_size`、`max_batch_characters` 和 `max_memories_per_batch` 控制成本与提炼质量。

候选原话保存在 `pending_messages.json`；小模型或大模型请求失败时，会保留到下一次定时任务重试。已提炼知识保存在 `group_memories.json`。两者都位于 `data/plugin_data/astrbot_plugin_group_learning/`。

## 群内命令

- `/群学习状态`：显示候选原话和已提炼知识数量。
- `/清空群学习`：清空当前群的候选原话和知识；仅 AstrBot 管理员可执行。

原话会发送给你选定的小模型和大模型，因此启用前请取得群成员同意，并选择你信任的模型服务商。
