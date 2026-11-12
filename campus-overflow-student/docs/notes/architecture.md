# CampusOverflow 架构理解笔记（S0）

> 撰写：蔡锦莉（PO，@Vespersea）｜ 团队：六边形小队 ｜ 版本：v1.0
> 依据：仓库 README、`backend/pom.xml`、`docs/adr/`、`ArchitectureTest.java`、`docs/storming/`
> 用途：S0 交付物之一（环境跑通 + 读懂 Identity 参考实现与架构文档后的书面产出），也是 Lab1 章程的技术附件

## 1. 一句话理解

CampusOverflow 是一个**模块化单体**：物理上是一个 Spring Boot 应用、一个数据库、一次部署；逻辑上被切成 4 个限界上下文 + 1 个共享内核，上下文之间只能经 `api` 包（发布语言）或集成事件说话，谁越界谁的构建就红。

这样选的理由是课程要考察"领域建模与架构约束"，而不是"分布式部署"。代价是上下文边界必须靠自动化约束守住，否则半年后就退化成大泥球——所以 `ArchitectureTest` 里的 6 条适应度函数不是装饰，是这条路的必要成本。

## 2. 四个限界上下文

| 上下文 | 包前缀 | 职责 | 本项目中的状态 |
| --- | --- | --- | --- |
| Identity 身份与认证 | `com.campusoverflow.identity` | 注册、登录、课程、角色（教师/学生）、会话 | **完整参考实现**，我们只读不重写，S1 在其上做增量 |
| Q&A 问答核心 | `com.campusoverflow.qa` | 问题、回答、投票、评论、标签、采纳 | S2 主要开发战场（36 个 TODO） |
| Reputation 声誉与激励 | `com.campusoverflow.reputation` | 声誉账本、徽章、悬赏、排行榜、对账 | S3（与 Discovery 合计 71 个 TODO） |
| Discovery 内容发现 | `com.campusoverflow.discovery` | 搜索投影、相关推荐、通知中心、@提及 | S3 |
| （共享内核） | `com.campusoverflow.shared` | 领域异常、Actor、事件契约、分页等公共契约 | 已提供，不许随便加东西 |

上下文边界由业务能力划定，不按技术层（没有"数据库上下文""前端上下文"这种东西）。

## 3. 27 个 Maven 模块是怎么长出来的

`backend/pom.xml` 下 6 个父模块，每个上下文再拆成 5 个六边形层模块，加 1 个装配模块：

```
co-shared-kernel                 公共契约（被所有上下文依赖，自己不依赖任何上下文）
co-identity      ─┬ api / domain / application / infrastructure / interfaces
co-qa            ─┼ 同上 5 层
co-reputation    ─┤
co-discovery     ─┘
co-bootstrap                     唯一可执行模块：Spring Boot 启动、安全链、发件箱中继、全局异常、Flyway
```

6 + 4×5 + 1 = **27**。拆成模块的目的不是"显得专业"，是让 Maven 在编译期就拦住越界依赖（模块间不声明依赖，代码就引不到）。

## 4. 分层与依赖方向

每个上下文内部固定五层，依赖只能由外向内：

```
interfaces（REST 控制器、DTO）  ──▶ application（用例编排、事务边界）
                                        │
infrastructure（JPA/Redis 适配器） ──▶ domain（聚合、值对象、业务规则，纯 Java）
                                        ▲
                                   bootstrap（唯一装配者）
```

三条硬规则（由 FF-4 / FF-1 / ADR-008 保证）：

1. **domain 不认识框架**：不能出现 Spring、JPA、Jackson、Swagger 的 import；领域对象能脱离容器单独 new 出来跑单元测试。
2. **JPA 实体不进 domain**：`@Entity` 必须落在 `infrastructure`，它只是持久化模型，和领域聚合是两套对象（ADR-008）。
3. **interfaces 与 infrastructure 只允许被 bootstrap 访问**，application 只允许被 interfaces / infrastructure / bootstrap 访问——所以控制器不可能直接摸到仓储实现。

## 5. 上下文之间怎么说话

两条通道，别的都不许：

1. **同步查询**：经对方 `*-api` 包的接口（发布语言）。`api` 包只允许依赖 JDK 与 shared kernel（FF-5），保证引用它不会捎带一坨框架进来。
2. **异步通知**：集成事件 + **事务性发件箱**（ADR-002）。业务事务里把事件写进 `outbox_event` 表，提交后由中继投递，消费端必须幂等。

为什么要发件箱而不是 `ApplicationEventPublisher`：事务回滚后事件已经发出去（脏事件），或者进程崩了事件彻底丢（漏事件）。发件箱把"事件"和"业务状态"放进同一个事务，至少一次投递 + 消费端幂等，就能同时挡住这两种事故。

对应到本项目最典型的一条链路：

```
采纳回答 → [Q&A 事务] AnswerAccepted 事件入 outbox
        → 中继投递 → Reputation（+15、徽章、悬赏结算）
                   → Discovery（搜索投影刷新、通知作者/回答者）
```

幂等的抓手：账本唯一键 `(event_id, user_id, reason)`、通知 `(source_event_id, recipient_id)`、`processed_event` 表。

## 6. 六条适应度函数（`ArchitectureTest` 真实内容）

| 编号 | 约束 | 违反的典型症状 |
| --- | --- | --- |
| FF-1 | 领域层零框架依赖 | 为了省事在聚合上打 `@Entity` |
| FF-2 | 跨上下文只经 `api` 或事件 | Q&A 直接 import Reputation 的 domain 去加积分 |
| FF-3 | 上下文之间无环 | A 调 B 的 api、B 又调 A 的 api |
| FF-4 | 六边形分层方向由外向内 | 控制器直接调仓储、domain 反向依赖 application |
| FF-5 | `api` 包只依赖 JDK + shared | 在 api 的 DTO 上挂 `@JsonIgnore` 之类框架注解 |
| FF-6 | 表名按上下文前缀（`id_` / `qa_` / `rep_` / `dis_`） | 两个上下文共用一张 `user` 表 |

对我们开发的实际含义：**先把 api 契约想清楚再写实现**。契约一改，跨模块编译立刻红，这种"红"是好事。

## 7. ADR 速览（`docs/adr/`）

| 编号 | 决策 | 我抓到的要点 |
| --- | --- | --- |
| 001 | 模块化单体 + 六边形分层 | 见本文第 1、4 节（**文件缺失，需补**） |
| 002 | 跨上下文集成用事务性发件箱 | 见第 5 节 |
| 003 | 认证用 Redis 服务端会话而非 JWT | 会话可即时失效，登出/封禁不用等 token 过期（**文件缺失，需补**） |
| 004 | 声誉用只追加账本 + 定期对账 | 每一分都要能回答"从哪来"；不变量：余额 = 流水之和 |
| 005 | 中文搜索用 MySQL ngram 全文索引 | 自建库必须配 `ngram_token_size=2`，否则中文搜不到（**文件缺失，需补**） |
| 006 | 用户内容一律软删除 | 删除保留审计痕迹（**文件缺失，需补**） |
| 007 | 前后端以 OpenAPI 契约协作 | 契约先改、再改实现，避免联调期集中返工 |
| 008 | 领域对象与 JPA 实体分离 | 见第 4 节 |
| 009 | 投票声誉每日上限 200 + 悬赏冻结积分 | 正向增量按 UTC 自然日封顶；悬赏只动积分不动声誉 |

> 现状说明：`docs/adr/` 里 001/003/005/006 四个文件尚不存在（README 表格里的链接是断的），另有四个文件名因编码问题显示为乱码。这属于 Lab2/Lab3 的文档债，已登记在第 9 节疑问清单，不作为 Lab0/Lab1 交付内容。

## 8. Identity 参考实现的读法（我给全队的导读）

按"从外到内"读三遍，比从 domain 往上读快得多：

1. **第一遍看接口**：`co-identity-interfaces` 的控制器只做三件事——参数校验、调 application、拼响应。看它能确认"对外承诺了什么"（注册、登录、课程、角色、会话）。
2. **第二遍看用例**：`co-identity-application` 的 Service 是事务边界，注册流程里能看到"建账号 → 发事件 → 会话下发"的顺序，以及异常在哪儿抛。
3. **第三遍看领域**：`co-identity-domain` 的聚合里是真正的业务规则（唯一性、角色约束、状态流转），这里没有一行 Spring。

然后去 `infrastructure` 看适配器怎么把领域对象映射成 `id_` 前缀的表，最后在 `co-bootstrap` 看它怎么被装配进容器。**照这个骨架去写 Q&A**，就不会写出"控制器里写 if 业务"的东西。

## 9. 我的疑问清单（下次站会/评审会上讨论）

1. `campus-overflow-student/` 是起始代码的整体目录，但 `.github/workflows` 在里面 → 仓库根的 Actions 页一片空白，**CI 从来没跑过**。这个目录要不要整体提到根？影响所有人，需要一次集体决策。
2. ADR 001/003/005/006 文件缺失、四个 ADR 文件名乱码，链接全断——谁来补、补在哪周？
3. `docs/adr/ADR-002-package-structure.md` 和 `002-事务性发件箱.md` 撞编号，是否重编为 010？
4. 覆盖率门禁 `domain.coverage.minimum` 已从 0 改成 0.70，但本机跑不了 `mvn verify`（缺 JDK 21 / Maven / Docker），这条门禁目前只能靠 CI 验证——CI 又不跑，等于双重失效。
5. 前端 `npm run dev` 与后端联调的最低可运行路径，需要有人给出一份"新人 10 分钟跑起来"的确认版步骤（现有 README 是起始代码作者的版本，未经验证）。
