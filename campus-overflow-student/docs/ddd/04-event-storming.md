# 事件风暴工作坊记录（Sprint 0 · T0-06）

- 时间：90 分钟，全员参与
- 方式：橙（领域事件）→ 蓝（命令）→ 黄（聚合）→ 紫（策略 / 规则）→ 画边界
- 产出：本目录 4 份文档；争论未决的点记入第 4 节，S3 定稿前必须关闭

## 1. 按时间轴铺事件（橙色便利贴）

按用户旅程铺出已发生的事实，一句话一个事件，动词用过去式：

1. 学生注册 → `UserRegisteredEvent`
2. 教师建课、学生加入课程 → 课程成员关系变更（Identity 内部事件，不跨上下文）
3. 学生提问 → `QuestionPostedEvent`；改题 → `QuestionEditedEvent`
4. 学生回答 → `AnswerSubmittedEvent`
5. 学生投票 / 改投 / 撤销 → `VoteCastEvent`
6. 学生评论、@某人 → `CommentPostedEvent`
7. 提问者采纳 → `AnswerAcceptedEvent`
8. 教师认证回答 → `AnswerEndorsedEvent`
9. 提问者或课程管理者删除 → `ContentDeletedEvent`
10. 声誉记账 → `ReputationChangedEvent`；授徽 → `BadgeAwardedEvent`
11. 发起悬赏 / 结算 / 到期 → `BountyOpenedEvent` / `BountyAwardedEvent` / `BountyClosedEvent`

## 2. 事件反推命令与聚合

| 领域事件 | 命令（用例） | 聚合 / 聚合根 | 关键规则（策略） |
| --- | --- | --- | --- |
| `QuestionPostedEvent` | 发布问题 | `Question` | 标题 10–150 字符、1–5 个标签、正文 Markdown（`Title`、`Tags`、`MarkdownBody` 值对象自校验） |
| `AnswerSubmittedEvent` | 提交回答 | `Answer` | 问题必须未删除（`ensureOpenForAnswers`）；不能自答自采纳之外的越权 |
| `VoteCastEvent` | 投票 / 改投 / 撤销 | `Vote` | 不能给自己投票（QA-1015）；每人每内容一票，改投按差值计分 |
| `AnswerAcceptedEvent` | 采纳 | `Question` | 每题最多一次且不可更改（QA-2002）；仅提问者或课程管理者（QA-3001）；被采纳回答须属于本题且未删除 |
| `AnswerEndorsedEvent` | 教师认证 | `Answer` | 仅课程教师 / 助教；每个回答只能认证一次 |
| `ContentDeletedEvent` | 删除问题 / 回答 | `Question` / `Answer` | 软删除；提问者只能删无回答的问题；被采纳的回答作者不可删 |
| `ReputationChangedEvent` | 记账 | `ReputationAccount` + `LedgerEntry` | 只能由流水改变；幂等键 `(eventId, userId, reason)`；每日上限 200 |
| `BountyAwardedEvent` | 结算悬赏 | `Bounty` | `OPEN → AWARDED`，冻结积分转出；终态不可变 |

## 3. 划边界：为什么是 4 个上下文

| 边界 | 判定依据 |
| --- | --- |
| Q&A 独立 | 语言与规则最复杂（采纳、软删除、认证），是核心域；其他上下文只消费它的事件 |
| Reputation 独立 | 有独立的账本、货币（悬赏积分）、反作弊规则；改规则不影响问答规则 |
| Identity 独立 | 校园身份、课程成员关系是授权依据，被三个上下文共同依赖，适合做上游 |
| Discovery 独立 | 搜索 / 推荐 / 通知是通用能力，模型是 `SearchDocument` 投影而非 `Question`，符合“将来可拆分”的演进方向 |

## 4. 本组补充与待定

| 编号 | 事项 | 结论 / 状态 |
| --- | --- | --- |
| D-01 | 通知是否独立成上下文 | 暂留在 Discovery 内（`Notification` 聚合），S3 若通知量增大再拆 |
| D-02 | 课程成员关系变化是否发跨上下文事件 | 暂不需要：Q&A 采纳时同步查 `IdentityApi.canManageCourse` |
| D-03 | 搜索投影重建方式 | 先支持事件增量折叠，S3 补一个全量重建脚本 |
| D-04 | 排行榜与声誉是否同库 | 同库（Redis ZSET + MySQL 账本），对账由 `ReconciliationService` 负责 |

## 5. 与指导书第 3 章的差异

1. 指导书 3.4 的领域模型、3.6 的编码约定作为“参考实现”直接沿用，未作改动。
2. 事件清单在指导书 3.5 基础上补充了“关键字段”一列，供跨组联调对齐。
3. 术语表补充了错误码前缀与“正确说法 / 反例”对照，便于评审时直接比对。
