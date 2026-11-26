# 领域事件清单

## 1. 事件约定

1. 命名用“业务事实 + 过去式 + `Event`”，如 `AnswerAcceptedEvent`；一个事件只表达一件已经发生的事，不用“请求 / 建议”式命名（不写 `AcceptAnswerEvent`）。
2. 事件是跨上下文传播的不可变 `record`（`IntegrationEvent`），带全局唯一 `eventId`，发布后不可修改、不可复用。
3. 事件只携带下游需要的字段，不透传整个实体；需要更多信息时由下游回调上游的 `*Api` 查询。
4. 发布一律经发件箱（outbox）：业务事务内写 outbox 表，事务提交后由投递器发出，保证“改库与发事件同成败”。
5. 订阅方必须幂等：用 `ProcessedEventStore` 按 `eventId` 去重；重复投递不产生重复记账、重复通知。
6. 同一上下文内部用领域事件解耦聚合，跨上下文只用上表登记的事件；新增事件要在本文件登记后才可发布。

## 2. 事件清单

| 事件 | 发布者（触发用例） | 订阅者及处理 |
| --- | --- | --- |
| `UserRegisteredEvent` | Identity：注册 / 创建教职工 | Reputation：开立声誉账户并按角色发放初始悬赏积分 |
| `QuestionPostedEvent` | Q&A：发布问题 | Discovery：插入搜索投影 |
| `QuestionEditedEvent` | Q&A：编辑问题 | Discovery：更新投影文本与标签 |
| `AnswerSubmittedEvent` | Q&A：提交回答 | Reputation：授予“初来乍到”；Discovery：投影回答数 +1、通知提问者 |
| `AnswerAcceptedEvent` | Q&A：采纳 | Reputation：+15、徽章、悬赏结算；Discovery：标记已采纳、通知回答者 |
| `AnswerEndorsedEvent` | Q&A：教师认证 | Reputation：授予“名师认可”；Discovery：通知回答者 |
| `VoteCastEvent` | Q&A：投票 / 改投 / 撤销 | Reputation：按差值计分（受每日上限约束）；Discovery：问题得分同步 |
| `CommentPostedEvent` | Q&A：评论 | Discovery：解析 @提及并通知 |
| `ContentDeletedEvent` | Q&A：软删除问题 / 回答 | Discovery：标记投影删除 / 回答数 −1 |
| `ReputationChangedEvent` | Reputation：记账 | Reputation 排行榜：更新周 / 月 / 学期 ZSET（经 `ProcessedEventStore` 幂等） |
| `BadgeAwardedEvent` | Reputation：授予徽章 | Discovery：通知获得者 |
| `BountyOpenedEvent` / `BountyAwardedEvent` / `BountyClosedEvent` | Reputation：悬赏发起 / 结算 / 关闭 | Discovery：维护“悬赏中”标记；`BountyAwarded` 通知获奖者 |

## 3. 事件与关键字段（供联调对齐）

| 事件 | 下游必需字段 | 备注 |
| --- | --- | --- |
| `AnswerAcceptedEvent` | `eventId`、`questionId`、`questionAuthorId`、`answerId`、`answerAuthorId`、`bountyId`（可空）、`occurredAt` | Reputation 需要 `questionAuthorId` 才能给提问者加分，字段由 Q&A 在事件中提供；有悬赏时同时结算 |
| `VoteCastEvent` | `eventId`、`targetType`、`targetId`、`voterId`、`value`、`previousValue`、`targetAuthorId` | 按差值计分，因此必须带改投前的值 |
| `ContentDeletedEvent` | `eventId`、`targetType`、`targetId`、`deleteReason` | Discovery 据此标记投影删除，不做物理删除 |
| `UserRegisteredEvent` | `eventId`、`userId`、`displayName`、`role`、`verified` | 不含学号、邮箱等个人信息 |

> 字段以 `co-shared-kernel` 中的 `IntegrationEvent` 定义与各 `*-api` 事件 record 为准，本表仅用于评审时对齐口径。
