# Lab 1 · 团队组建与协作工具链 —— 提交说明

> 团队：六边形小队 ｜ 提交人：蔡锦莉（PO，@Vespersea）｜ 日期：2026-11-12
> 依据：《CampusOverflow 实验部分》第 48–49 页（Lab 1，第 2 周 / Sprint 0）

## 1. 官方四项提交物对照

| # | 官方要求 | 交付位置 | 当前状态 |
| --- | --- | --- | --- |
| 1 | 团队代码仓库 URL（分支保护已配置） | `https://github.com/hbuas-se-2026fall/cs2411-team02-campus-overflow` | ⚠️ URL ✅；`main` 已设为默认并带保护；仍需在 Settings 里核验三项：**PR 必须 1 人审批**、**不允许管理员绕过**、`develop` 是否也要保护 |
| 2 | 团队看板截图（含初始 Backlog 至少 5 条） | `docs/submissions/lab1-kanban-board.png` | ❌ 现图是 `docs/submissions/src/kanban-board.html` 渲染的**模拟图**，不是真实工具截图，答辩前必须替换 |
| 3 | 《团队章程 v1.0》Markdown | `docs/team/charter-v1.0.md` | ✅ 已修正为真实成员与角色（见第 2 节）；待 4 人各自 commit 签署 |
| 4 | 每位成员的首次 PR 合并记录截图 | — | ⚠️ 仓库现有 4 个 PR（#10、#12 关闭；#11、#13 已合并），作者只有 `Vespersea`、`ty666-lang`；`wuchaoyang`、`zyxd336` 尚无 PR 记录 |

## 2. 团队与角色（已写入章程）

| GitHub 账号 | 姓名 | 角色 | 主责 |
| --- | --- | --- | --- |
| Vespersea | 蔡锦莉 | PO | Backlog 优先级、验收标准、仓库与 PR 管理 |
| zyxd336 | 张玉馨 | SM | 站会/计划/回顾、DoD 检查、障碍清除 |
| wuchaoyang | 吴超阳 | Dev A | 投票评论、悬赏、搜索推荐、CI/CD |
| ty666-lang | （姓名待补） | Dev B | Q&A 上下文后端、单元测试 |

## 3. 初始 Backlog（可直接建卡，8 条）

| ID | 卡片标题（建议直接作为 Issue 标题） | 类型 | 优先级 | 点 |
| --- | --- | --- | --- | --- |
| US-01 | 作为访客，我要用学号邮箱注册，以便进入校园问答社区 | story | P0 | 3 |
| US-02 | 作为学生，我要登录并保持会话，以便连续使用 | story | P0 | 2 |
| US-03 | 作为学生，我要发帖提问（含标签），以便让懂的人看到 | story | P0 | 3 |
| US-04 | 作为学生，我要回答问题，以便帮助他人 | story | P0 | 3 |
| US-05 | 作为学生，我要对问题/回答投票（赞/踩/改投），以便表达评价 | story | P0 | 5 |
| US-06 | 作为提问者，我要采纳一个回答，以便给出结论 | story | P0 | 3 |
| US-07 | 作为学生，我要搜索问题（中文分词），以便找到已有答案 | story | P1 | 5 |
| US-08 | 作为学生，我要收到"我的回答被采纳"的通知，以便及时获知 | story | P1 | 3 |

> 说明：这 8 条取自 `docs/backlog/product-backlog.md` 的 P0/P1 条目，与 S2/S3 的 TODO 划分一致，可直接粘进 GitHub Projects 作为初始 Backlog。

## 4. 本次已完成的仓库改动

| 类别 | 改动 |
| --- | --- |
| 章程 | `docs/team/charter-v1.0.md`：成员表、签署表由占位的"张三/李四/王五/赵六"改为真实 4 名成员；生效日期由"第 5 教学周"改为"第 2 教学周（Lab1）" |
| 仓库卫生 | 修复 5 个乱码文件名（`002-事务性发件箱.md`、`004-声誉账簿.md`、`007-契约优先.md`、`008-领域与持久化分离.md`、`docs/学生任务清单.md`），ADR 目录里原本全是断链 |
| Issue 模板 | `.github/ISSUE_TEMPLATE/`：`story.md`（含 INVEST + Gherkin + DoD）、`task.md`、`bug.md` |
| PR 模板 | `.github/PULL_REQUEST_TEMPLATE.md`：变更目的 / 关联 Issue / 验证输出 / 评审自检清单 / LGTM 记录 |
| 评审人 | `.github/CODEOWNERS`：按限界上下文分派默认评审人 |
| Lab0 交付 | `docs/notes/architecture.md`、`docs/notes/environment.md`、`docs/submissions/lab0.md` |

## 5. 仍需人工完成的三件事（附操作路径）

### 5.1 分支保护补齐（提交物 1）

仓库 → **Settings** → **Rules → Rulesets**（新）或 **Branches**（旧）：

- Target branch：`main`，勾选 **Require a pull request before merging**、**Require approvals = 1**、**Dismiss stale approvals**、**Require review from Code Owners**；
- 勾选 **Block force pushes**；
- Bypass list **留空**（即不允许管理员绕过）；
- 对 `develop` 同样加一条规则（团队实际开发在 develop 上，不保护等于没有门禁）。

### 5.2 真实看板（提交物 2）

仓库 → **Projects** → **New project** → 选 **Board** 模板 → 命名 `CampusOverflow Sprint 0`：
建四列 **To Do / In Progress / In Review / Done**，把第 3 节的 8 张卡建在 To Do，然后截图替换 `docs/submissions/lab1-kanban-board.png`（截图里要能同时看到项目名、四列和卡片数量）。

### 5.3 每人首次 PR（提交物 4）

每位成员各自执行（以 ty666-lang 为例）：

```bash
git checkout main && git pull
git checkout -b feature/ty666lang-hello
# 编辑 docs/team/hello-ty666-lang.md
git commit -m "docs(team): 补充 ty666-lang 自我介绍"
git push -u origin feature/ty666lang-hello
```

然后开 PR → 指定另一名成员 review → 回复 **LGTM** → 由 SM 合并 → **截图 PR 页面的 "Merged" 状态**（截图里要能看到作者、评审人和合并人三个名字）。

> 为什么不能由一个人代做：Lab1 评分 30% 看的就是"分支、PR、评审、合并流程是否规范"，四个 PR 都出自同一账号会直接被判为无效；且 PR 作者身份是 GitHub 账号级的，无法伪造。

## 6. 评分对照

| 评分项 | 权重 | 当前自评 |
| --- | --- | --- |
| 工具链配置完整性（仓库、看板、IM、分支保护） | 30% | 仓库 ✅ / IM ✅（班级群）/ 看板 ❌ 待建 / 分支保护 ⚠️ 待核验 |
| 团队章程质量 | 30% | 章程结构完整（价值观、沟通、开发规约、DoD、签署区），成员已实名 ✅ |
| Git 协作熟练度 | 30% | 现有 2 个合并 PR（#11、#13）流程完整；缺 2 名成员的首次 PR ⚠️ |
| 团队氛围 | 10% | 教师巡观察评分，无法自评 |
