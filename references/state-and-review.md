# 状态与复习规范

路径相对于用户批准的学习目录。记录采用 Markdown；本规范由代理执行，没有后台服务或自动评分。
所有更新遵守用户的文件审批规则。只读或聊天模式下仅输出建议状态和续学摘要，明确未落盘。

## 文件与兼容性

| 路径 | 内容 |
| --- | --- |
| MISSION.md | 目标、完成标准、先修条件、时间预算和教学偏好 |
| RESOURCES.md | 来源 ID、链接或本地路径、章节、版本、核验状态和实际核验时间 |
| STATE.md | 每个知识点的当前状态、证据引用和下一步，不存标准答案 |
| REVIEW.md | 复习次数、恢复标记、到期安排、延期及原因 |
| learning-records/0001-topic.md | 追加式证据记录，编号递增，不覆盖历史 |
| lessons/0001-topic.md | 单目标小课，可按需改用 HTML |
| reference/glossary.md | 术语定义；其他 reference/*.md 保存速查 |

新建 STATE.md 和 REVIEW.md 时注明 `format: teach-to-learn/1`。
旧文件继续可读；以下四态的旧中文名、英文显示名均是别名，不另建知识点或整份记录。
不自动改写旧文件。缺失 ID、时间、次数或证据标为未知，核对后才在获准范围内补齐；保留自定义内容。

## 固定状态与证据

| status ID（存储） | 中文显示 | English display |
| --- | --- | --- |
| unassessed | 未评估 | Unassessed |
| scaffolded | 需支架 | Scaffolded |
| provisional | 初步独立 | Provisional Independence |
| consolidated | 延迟巩固 | Delayed Consolidation |

显示语言可以变化；concept_id 和 status ID 不变，不按翻译后的能力名称创建重复记录。
旧文件无法确认两个条目是否属于同一知识点时先询问，不擅自合并。
“待核验”是证据的限定说明，不是第五种能力状态。

STATE.md 最小表格：

| concept_id | status | evidence | next_step |
| --- | --- | --- | --- |
| py-await | unassessed | 无 | 一个短诊断任务 |

每次作答事件至少记录：event_id、concept_id、task_id、session_id、带时区的 happened_at（或 unknown）、result、帮助程度和作答依据摘要。
result 使用 independent、assisted、incorrect、skipped 或 pending。
independent 必须是可评估的新任务、无提示且未看过该题答案的真实成功；自述、未提交结果或无法判断的任务用 pending。
assisted 表示实际借助提示或答案完成；单纯索要答案但尚未作答用 pending。incorrect 表示在清楚、适当的任务上明确答错。坏题或含糊回答先澄清，不据此降级。
session_id 表示一次学习会话；重复读取同一会话或更换窗口不伪造新的作答与延迟证据。
event_id 在学习目录内唯一；重复读取同一事件不得再次更新状态或次数。历史结果不因换语言而重记。
记录真实作答摘要和提示程度，引用对应记录；不把标准答案或不必要的个人信息写进续学状态。

## 状态转移（逐知识点，按事件顺序）

| 真实事件与原状态 | 当前状态与处理 |
| --- | --- |
| skipped、pending 或不能评估 | 状态不变；记录限定说明，不能用缺失结果晋级 |
| incorrect 或 assisted，原状态任意 | scaffolded；保留历史成功证据，设置 recovery_pending=true，复习次数归零 |
| independent，正在恢复或原状态为 unassessed/scaffolded | provisional；解除恢复标记，次数为 0，从本次成功重新安排首轮复习 |
| independent，原状态为 provisional | 只有另一会话且距最近一次独立成功满 24 小时才变为 consolidated；否则保持 provisional |
| independent，原状态为 consolidated 且不在恢复中 | 保持 consolidated；是否推进排期另行判断 |

恢复规则优先：失败后的第一次独立成功只能回到 provisional，不能借用失败前的旧成功直接跳到 consolidated。
同一题看过答案后再做对不能成为 independent；必须换一个未见过的任务。
状态下降描述当前仍需帮助，不删除历史已取得的成绩。互相矛盾的自述标为 pending 并核对。
当前状态只对该 concept_id 生效；一个知识点失败不能自动降低其他知识点。

## REVIEW.md 与排期

| concept_id | success_count | recovery_pending | due | defer | reason |
| --- | --- | --- | --- | --- | --- |
| py-await | 0 | false | next-session | none | 首次独立成功，时间未知 |

success_count 仅统计连续通过的有效到期复习；首次独立学习不算一次复习。
recovery_pending 标识失败后待恢复。due 可为带时区的时间戳或 next-session；defer 可为 none、after-session:S1 或用户指定时间。
最近独立成功的时间与会话由证据记录查得，不凭空补造。新记录的次数初值为 0；旧次数缺失时保持未知并核对。

- 一天统一按满 24 小时计算，3/7/14 天分别是 72/168/336 小时，从实际本次作答时刻起算。
- 有可信时间时保留时区或明确使用 UTC；只有日期、未知偏移或无法确认时刻时，用 unknown 和 next-session，不伪造午夜。
- 首次或恢复后的 independent：次数为 0；时间可靠则 due=本次时间+24 小时，否则 next-session。
- 有效到期复习须同时满足：不在恢复中、due 已到、另一会话、距最近独立成功至少 24 小时、一次新的 independent。
- 每个知识点每个会话最多推进一次次数。通过后次数加一：1 次安排 +72 小时，2 次 +168 小时，3 次及以后 +336 小时。
- 提前练习、同会话追加题、时间不足或无法确认到期：保留证据，不增加次数、不改原 due；不能伪造延迟巩固。
- incorrect/assisted：次数归零，进入恢复，due=next-session。恢复成功后重启 +24 小时，不直接进入 +72 小时。
- skipped：不改状态、次数和原 due；无额外约定时 defer=after-session:当前会话，本会话不再主动提出，下次会话重新进入候选。
- 用户明确延期时采用其安排并记录原因；尚未解除 defer 的项目先排除。用户主动重新选择该项可解除本次延期。到期和逾期都不自动产生失败事件。
- 旧时间或次数不明时不猜测延迟升级或间隔级别；先核对，无法核对则保留旧记录并提出从已观察的新证据建立基线。

候选顺序：用户本次明确选择 → 当前目标确实被阻塞的先修项 → 待恢复项 → 最早到期项；完全并列时按 concept_id 字典序。
next-session 项在新会话可参与候选；同优先级中，有明确到期时间的先于未知时间。默认最多两项，用户随时可调整或停止。
目标迁移任务达标后由用户决定继续实践、维护复习或结束；不强制新增课程。

## 边界时间线（人工构造，UTC）

| 事件 | 结果与状态 | 次数 / due |
| --- | --- | --- |
| 11-01 10:00，S1，E1，新题独立成功 | provisional | 0 / 11-02 10:00 |
| 11-02 10:00，S2，E2，到期新题独立成功 | consolidated | 1 / 11-05 10:00 |
| 11-05 10:00，S3，E3，跳过 | consolidated，不产生失败 | 1 / 保留 11-05 10:00，本会话延期 |
| 11-06 10:00，S4，E4，明确答错 | scaffolded，待恢复 | 0 / next-session |
| 11-06 10:05，S4，E5，换新题独立成功 | provisional，恢复优先 | 0 / 11-07 10:05 |
| 11-07 10:04，S5，E6，提前独立练习 | provisional，不增加次数 | 0 / 保留 11-07 10:05 |
| 11-08 10:04，S6，E7，新题独立成功 | consolidated，距最近独立成功满 24h | 1 / 11-11 10:04 |

例中所有月份日期均为 2026 年；每行均为不同事件，只能处理一次。完整学习样例见[人工案例](../examples/python-await/README.md)。
