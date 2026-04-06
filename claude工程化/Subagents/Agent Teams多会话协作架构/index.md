
Sub-Agents vs Agent Teams：从“任务委托”到“团队协作”

在前几讲中介绍的子代理工作模式虽然也各有差异，但是基本模式一致。都是主对话像老板一样，把任务委派给各个专职员工（子代理）。这种模式解决了上下文污染、权限边界、任务并行等问题。但它有一个根本性的限制：子代理只能向主对话汇报，不能互相交流。
![alt text](image.png)

这在很多场景下足够好用。但我们在实际生产环境中可能会遇到更复杂、更诡异的场景：比如多假设并行验证。你的系统出现了一个奇怪 bug——用户登录后偶尔会话丢失，没有明确的规律。你怀疑可能是：假设 A：JWT token 过期时间计算有问题假设 B：Redis session 存储的竞态条件假设 C：负载均衡器的 sticky session 配置


如果子代理 B 能看到子代理 C 的发现，它可能会说：等等，Redis 连接数上限问题可能是因为 sticky session 5 分钟后切换了服务器，导致新的 Redis 连接被创建。

这正是 Agent Teams 要解决的问题——让代理之间能够直接交流、互相挑战、协作推进

Agent Teams 让你可以协调多个 Claude Code 实例作为一个团队工作。一个会话作为  Team Lead（团队领导），协调工作、分配任务、综合结果。Teammates（队友）各自独立工作，每个都有自己的上下文窗口，并且可以直接互相通信。

![alt text](image-1.png)

因此我们说，“Agent Teams 模式”与“主会话 - 子代理模式”最本质的区别是，Teammates 可以互相发消息、共享发现、挑战彼此的结论。

# 创建和使用 Agent Teams
刚才说了，Agent Teams 默认是关闭的，启用方式是在 settings.json 中设置环境变量：
```
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```
或者在 shell 环境中设置：
```
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```
在 Claude Code 中可以通过下面两种方式创建团队。
![alt text](image-2.png)

启用后，用自然语言告诉 Claude 创建团队并描述任务：

```
我在设计一个 CLI 工具来追踪代码库中的 TODO 注释。
创建一个 agent team 从不同角度探索这个问题：
一个 teammate 负责 UX，一个负责技术架构（用最好的模型），一个扮演审评质疑者（用普通模型）。
```
Claude 会建团队，生成指定的 Teammates 让它们探索问题，然后综合各方发现，完成后清理团队。
团队成功创建后，一个 Agent Team 由以下组件构成。
![alt text](image-3.png)

团队和任务相关的数据会存储在本地。
团队配置：~/.claude/teams/{team-name}/config.json
任务列表：~/.claude/tasks/{team-name}/
团队配置包含  members  数组，记录每个 Teammate 的名称、agent ID 和类型。Teammates 可以读取这个文件来发现其他团队成员。这些都是 Claude Code 自行搞定的，不需要我们操心去存放。Agent Teams 支持两种显示模式：
![alt text](image-3.png)
完成工作后，Lead 会向 Teammate 发送关闭请求，Teammate 可以批准（优雅退出）或拒绝（并解释原因）。
最后，全部工作收尾，Lead 会清理团队，移除共享的团队资源。需要注意，如果出现需要用户人工指示清理的化，清理前先关闭所有 Teammates，而且应该只让 Lead 执行清理（Teammates 的 team 上下文可能不正确）。
# 实战项目：全栈 Bug 猎人
项目场景是一个 Express.js 电商应用（ShopStream），其中刻意植入了多个相互关联 bug。用户报告了三个看似独立的症状：会话丢失、API 变慢、数据泄漏。真相是这些症状由相互 bug 的级联故障造成：
```
Bug 1: DB 连接池太小（db.js）
    ↓ 连接耗尽
Bug 2: Redis Session 不处理重连（middleware/session.js）
    ↓ Session 写入静默失败
Bug 3: 订单查询 N+1 问题（routes/orders.js）
    ↓ 大量连接被占用 → 加剧 Bug 1
Bug 4: 缓存竞态条件（middleware/cache.js）
    ↓ 缓存 key 缺少用户标识 → 数据泄漏
    ... ...
```
那么为什么这里需要 Agent Teams 呢？因为单独调查者容易“锚定”，找到一个 bug 后就满意了，不继续挖掘。而 bug 之间的关联需要跨视角发现，只有 Session 侦探和数据库侦探各自的发现互相对照，才能看到级联效应。此外，架构侦探可以把其他侦探的发现串联起来，通过辩论机制产出更完整的根因链。

项目中具体的文件内容和说明如下。

![alt text](image-4.png)

把 team-prompt.md 中介绍的某一种启动模式（我设计了多个 Teams 协同模式，这里只展示竞争假设模式）拷贝到 Claude Code 命令行中，启动 Teams 即可。

```
阅读 bug-report.md 中描述的三个症状。然后创建一个 agent team 来调查这些问题。

生成 4 个 investigator teammates：
- "Session 侦探"：假设根因在 Session/Redis 层。重点审查 middleware/session.js 和 server.js 中的 session 配置。
- "数据库侦探"：假设根因在数据库连接和查询层。重点审查 db.js 和 routes/ 下所有路由的数据库操作。
- "缓存侦探"：假设根因在缓存机制。重点审查 middleware/cache.js 以及缓存与用户隔离相关的逻辑。
- "架构侦探"：不预设假设，从整体架构角度分析各组件的交互。重点关注错误处理、资源管理和并发安全。

每个 teammate 的 prompt 中包含：
1. buggy-app/ 目录包含完整的应用代码
2. 他们需要用 Read/Grep/Glob 工具审查代码
3. 找到可疑问题后，要发消息告诉其他 teammates
4. 如果其他 teammate 的发现与自己的发现有关联，要主动指出
5. 特别注意：三个症状可能不是独立的，要寻找它们之间的因果关系

要求所有 teammates 在完成初步调查后互相分享发现，并尝试挑战彼此的结论。

最终综合所有发现，生成一份按照 findings-template.md 格式的调查报告。
```
我们用自然语言启动 Teams
![alt text](image-5.png)

Claude Code 根据指示，阅读项目中相关的配置文档，开始创建 Teams。

四大协作设计模式Agent Teams 最强大的地方在于它支持的协作模式。我从官方文档和工程实践中提炼出四种核心设计模式。

# 模式一：竞争假设（Competing Hypotheses）

这种模式适用于根因不明确，需要从多个方向同时验证的场景。单个 Agent 调查时容易“锚定”——找到一个合理解释后就停止探索，可能错过真正的根因。此时，让多个 Teammates 各自持有不同假设，并要求它们互相挑战、试图推翻对方的理论。

```
用户报告应用在发送一条消息后就退出了，而不是保持连接。
生成 5 个 agent teammates 调查不同的假设。
让它们互相对话，试图推翻对方的理论，像科学辩论一样。
将最终共识更新到 findings 文档。
```
该模式架构如下：
![alt text](image-6.png)
想一想，这种模式的工程价值在哪里？避免锚定效应——多个独立调查者，各自寻找证据支持自己的假设。辩论机制迫使每个假设必须经受挑战才能“存活”。最终“存活”的假设更可能是真正的根因。

# 模式二：分层评审（Parallel Review）
这种模式适用于代码审查、PR Review 等需要从多个维度同时评估的任务
因为单个 reviewer 容易聚焦于某一类问题（比如只看代码风格），而忽略其他维度。让多个 Teammates 各自负责不同的审查维度，同时工作。
创建一个 agent team 来审查 PR #142。生成三个 reviewer：
- 一个专注安全隐患
- 一个检查性能影响
- 一个验证测试覆盖
让它们各自审查并汇报发现。
该模式架构如下：
![alt text](image-7.png)
这种模式的工程价值有哪些呢？每个维度都得到充分关注，不会因为注意力分散而遗漏。并行执行，时间成本不增加不同专家可能发现关联问题（Security Reviewer 发现的输入验证问题可能影响 Performance）

# 模式三：模块化开发（Module Ownership）
这种模式适用于新功能开发，涉及多个独立模块（前端、后端、数据库、测试）的场景。由于让一个 Agent 依次开发容易混淆上下文，多个子代理并行又无法协调接口。因此，让每个 Teammate 拥有一个模块，通过共享任务列表协调工作。该模式架构如下：
![alt text](image-8.png)

这种模式的关键机制是：任务依赖：任务可以声明依赖其他任务，被阻塞的任务不能被认领。自动解锁：当依赖任务完成后，被阻塞的任务自动变为可认领。文件所有权：每个 Teammate 负责不同的文件，避免冲突。

# 模式四：规划 - 审批（Plan Approval）
这种模式适用于复杂或高风险任务，需要在实施前确认方案。

如果直接让 Teammate 开干，可能因方向错误导致返工。因此要求 Teammate 在实施前先提交计划，Lead 审批通过后才能执行。

```
生成一个 architect teammate 来重构认证模块。
要求在修改任何代码前先提交计划等待审批。
```
该模式工作流如下。
![alt text](image-9.png)
控制 Lead 的审批标准：
```
生成一个 architect teammate 来重构认证模块。
要求在修改任何代码前先提交计划等待审批。
只批准包含测试计划的方案。
拒绝任何修改数据库 schema 的方案。
```
该模式工作流如下。![alt text](image-10.png)
控制 Lead 的审批标准：
```
生成一个 architect teammate 来重构认证模块。
要求在修改任何代码前先提交计划等待审批。
只批准包含测试计划的方案。
拒绝任何修改数据库 schema 的方案。
```
Lead 会根据你提供的标准自主做出批准 / 拒绝决策。你影响 Lead 判断的方式是在 prompt 中给出明确的标准，比如“只批准包含测试覆盖的方案”或“拒绝修改数据库 schema 的方案”。
# Sub-Agents vs Agent Teams：选型决策树\
你可能还有这样的疑惑：到底 Sub-Agents 和 Agent Teams 这两种架构如何选择？这里给出一个选型决策树。
```
你的任务需要多个 workers 吗？
├── 否 → 直接在主对话或用单个子代理
└── 是 → Workers 需要互相通信吗？
    ├── 否 → Sub-Agents
    │   └── 特点：
    │       • 更低 token 成本
    │       • 更简单的协调
    │       • 适合：并行探索、流水线编排
    │
    └── 是 → Agent Teams
        └── 特点：
            • 支持讨论和挑战
            • 共享任务列表
            • 适合：竞争假设、协作开发、多角度审查
```
我们用一张表格厘清两种模式的差异。
![alt text](image-11.png)
一句话总结，核心判断标准是你的 workers 是否需要互相通信？Sub-Agents 是快速、廉价的“派出去 - 带回来”模式，如果只需汇报结果 ，就选 Sub-Agents；Agent Teams 是复杂、协作的“组队讨论”模式，如果需要互相讨论、挑战、协调 ，就选 Agent Teams。

# Token 成本考量
最后说说成本。Claude 指出，Agent Teams 使用的 token 显著高于单会话或子代理，因为每个 Teammate 是独立的 Claude 实例，有独立的上下文窗口。消息通信消耗额外 token，而且团队越大，成本越高。因此，适合 Agent Teams 的任务特征包括：并行探索能带来真正的价值；讨论和挑战能提高结果质量；任务复杂度值得额外成本。不适合 Agent Teams 的任务特征包括：常规任务，单会话就够；任务之间没有协作需求以及预算有限的场景。