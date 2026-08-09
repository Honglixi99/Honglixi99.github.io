---
title: "Claude Code 高效使用指南"
date: 2026-08-09
tags: ["Claude Code", "AI 编程", "效率"]
description: "从聊天框到会自己干活的团队：CLAUDE.md、Skills、Subagents、MCP、Hooks、Plugins 六种能力一网打尽"
showToc: true
---

> 本章整合自卡码笔记三篇文章：《Claude Code 高效使用指南》《Claude Code 完整使用指南：六种扩展能力》《CLAUDE.md 到底怎么写》
>
> 一句话主线：**Claude Code 从来不是靠一个"万能 Prompt"变强，而是靠一套分工清楚的工作系统。把"每次都要重复的东西"一层层固化——固化得越多，你要操的心越少。**

---

## 1. 为什么不能把 Claude Code 当聊天框用

绝大多数人装好 Claude Code 后，只是把它当成"会写代码的聊天框"——问一句、答一句、复制粘贴、关掉。这么用没错，但没发挥它的价值。

问题在于：**新会话里的 Claude Code 是"失忆"的。** 它不知道你的技术栈、不知道你的代码规范、不知道哪些坑不能踩。你每次口头交代一遍，就是重复劳动；而且任务一长，临时说过的话会被文件内容、命令输出、错误日志淹没，一旦上下文被压缩，规则就丢了。

团队协作更麻烦：你告诉过 Claude 的规则，换个同事、换台电脑、换个项目，又要从头讲一遍。

> **核心结论：聊天适合表达当前任务，不能承载整套工程制度。**

真正的价值在于，把它从一个"每次都得从头解释一遍的临时工"，调教成一个**懂你项目、能自己干活、还能并行开多线程的团队**。

---

## 2. 总览：六种能力，各管一件事

Claude Code 的扩展能力不是"装得越多越强"，而是六套分工明确的工作系统：

| 问题 | 应该用什么 |
|---|---|
| 每次进入项目都应该知道什么？ | **CLAUDE.md** |
| 某类任务应该按什么方法做？ | **Skills** |
| 哪些工作需要独立上下文或独立角色？ | **Subagents** |
| 怎么访问仓库外的系统和数据？ | **MCP** |
| 哪些动作命中事件就必须执行？ | **Hooks** |
| 怎么把整套能力发给其他项目和队友？ | **Plugins** |

> **一句话记住核心分工：CLAUDE.md 管常驻规则，Skill 管按需方法，Subagent 管独立干活，MCP 管外部连接，Hook 管确定性动作，Plugin 管打包分发。**

从"能用"到"用得高效"是一条阶梯，每一级都把一类重复劳动固化下来：

```text
聊天框（问一句答一句）         ← 大多数人停留的地方
  ↓ 项目规则固化
CLAUDE.md（常驻项目说明）
  ↓ 专项流程固化
Skills（按需加载的方法工具箱）
  ↓ 必做动作固化
Hooks（每次确定触发的动作）
  ↓ 大任务固化
Subagents / 动态工作流（并行编排）
  ↓ 无人值守固化
CLI 管道 + 定时任务（Routines）
```

---

## 3. CLAUDE.md：先让 Claude 知道"这个项目怎么干"

### 3.1 它到底是什么

`CLAUDE.md` 是 Claude Code **会自动读取**的项目说明文件，本质是：

- **普通 Markdown**——不是配置文件，不需要复杂语法
- **给 Claude Code 看的**——不是给人看的 README
- **会进入上下文的**——规则参与后续推理，但也占上下文窗口

> **README 面向人，重点是项目是什么、怎么启动；CLAUDE.md 面向 Agent，重点是接到任务以后应该怎么行动。**

### 3.2 和 README / Prompt / Memory 的区别

| 载体 | 回答的问题 | 典型内容 |
|---|---|---|
| README | 人怎么理解项目 | 项目介绍、安装方式、使用文档 |
| Prompt | 这次要做什么 | "修登录失败，但别改后端接口"（临时） |
| CLAUDE.md | 在这个项目里怎么干活 | 架构、命令、规范、禁区（跨任务稳定） |
| Memory | 个人/团队经验 | 用户级放个人习惯，项目级放团队规则 |

### 3.3 放在哪里

| 位置 | 作用范围 | 适合放什么 |
|---|---|---|
| `~/.claude/CLAUDE.md` | 本机所有项目 | 个人偏好、通用工作习惯（不提交 Git） |
| `项目根目录/CLAUDE.md` | 当前项目，团队共享 | 技术栈、命令、全局规则（提交 Git） |
| `项目根目录/CLAUDE.local.md` | 当前项目，仅自己 | 本地地址、个人测试习惯（加 .gitignore） |
| `某个子目录/CLAUDE.md` | 进入该模块时按需加载 | 模块命令、局部架构和禁区 |

> 新版更推荐在项目记忆里用 `@path` import 引入个人文件（`@~/.claude/my-project-preferences.md`），个人规则不用进仓库。

### 3.4 什么值得写（判断标准）

> **它是否会反复影响 Claude Code 的行动？** 会 → 写进 CLAUDE.md；只对当前任务有效 → 放 prompt；只是给人看的背景 → 放 README。

值得写的六类内容：

```md
## 常用命令          ← 最应该写，命令写完整，尤其 monorepo
- 安装依赖：pnpm install
- 本地开发：pnpm dev

## 目录结构          ← 写目录职责，不要贴完整文件树
- `src/api/`：接口封装，组件不要直接调用 fetch

## 编码规范          ← 写具体规则，不写空话
- 新组件使用函数组件，样式优先使用已有 token

## 禁止事项          ← AI 编程最怕乱动不该动的地方
- 不要修改数据库 schema，除非用户明确要求
- 不要提交 .env、token、密钥

## 验证要求
- 修改业务逻辑后，必须运行相关测试；如果无法运行，在最终回复说明原因

## 常见坑
- 支付模块的金额单位是分，不是元
```

### 3.5 什么不要写

- ❌ 完整接口文档（几十个接口全贴进去 → 放专门文档，最多写"API 文档见 docs/api.md"）
- ❌ 历史流水账（"2025-01-03 修了登录问题" → 要写成规则结论）
- ❌ 空泛口号（"写高质量代码""注意性能" → 改成可执行规则）
- ❌ 过期规则（**过期规则比没有规则更危险**，npm 换成 pnpm 后旧规则会带着 Claude 继续错）
- ❌ 太细的临时任务（"今天先修用户 A 的导出 bug"）

### 3.6 怎么写才有效：短、准、硬

| 太软 ❌ | 改成 ✅ |
|---|---|
| 尽量注意测试 | 修改业务逻辑后必须运行相关测试，无法运行则在最终回复说明原因 |
| 代码要符合项目风格 | 新增 API 请求必须放在 `src/api/`，页面组件只能调用封装后的方法 |

### 3.7 大项目怎么拆

小项目一个根 `CLAUDE.md` 够用；大项目（尤其 monorepo）**一定要拆**：

```text
repo/
  CLAUDE.md            ← 全局：包管理器、Git 流程、安全要求、全局禁区、通用验证
  apps/web/CLAUDE.md   ← 模块：目录职责、启动/测试命令、常见坑
  packages/ui/CLAUDE.md
```

> **不是让模型看见更多，而是让它看见更有价值的信息。**

### 3.8 与上下文窗口的关系

- CLAUDE.md 不是免费空间，它占上下文；官方会用 **prompt caching** 降低重复读取成本——**缓存降的是计费压力，不代表不占空间**
- 根 CLAUDE.md 控制在几屏内、每条尽量一行、长文档用链接或 import 引用、定期删过期内容
- **它不是安全边界**：真正必须阻止的动作要交给权限设置或 PreToolUse Hook；规则没生效先运行 `/memory` 确认加载

### 3.9 团队维护五项实践

1. 项目根 CLAUDE.md **进仓库**，团队共享
2. 个人偏好放用户级 memory，不污染团队文件
3. 规则变更**像代码一样审查**（写错就是把错误流程自动化）
4. 踩坑之后**及时沉淀**成规则
5. **定期删**：过期命令、已不存在的目录、重复规则、临时任务残留

---

## 4. Skills：把重复方法做成"按需工具箱"

**Skill 解决什么问题？** 一套发布流程（检查工作区 → 跑测试 → 生成变更说明 → 回滚方案）很重要，但写普通业务代码时用不到。塞进 CLAUDE.md 会让每个会话都背着它。**Skill 的价值：只在相关任务出现时加载专项知识和流程。**

**一个 Skill 长什么样**（项目级放 `.claude/skills/`，个人通用放 `~/.claude/skills/`）：

```text
.claude/skills/release-check/
├── SKILL.md          ← 入口和导航
├── checklist.md      ← 参考资料（大段内容放单独文件）
└── scripts/
    └── verify.sh     ← 脚本负责确定性检查
```

```md
---
name: release-check
description: Check whether the current changes are ready for release. Use for release preparation, pre-deployment review, or rollback planning.
disable-model-invocation: true
---

## Release check
1. Read `checklist.md`.
2. Inspect the current git diff.
3. Run `scripts/verify.sh`.
4. Report blockers, risks, and rollback steps.
```

关键点：`description` 是告诉 Claude **什么时候该加载它**（写清任务、触发语境、不适用范围，别写"帮助开发"这种废话）；`disable-model-invocation: true` 表示只能用户主动调用；脚本负责确定性检查，Claude 负责结合现场解释结果。

**什么时候该做 Skill？** 同一段说明复制过三次、CLAUDE.md 里某一节越来越像操作手册、任务有固定输入/步骤/输出格式、需要模板示例脚本、多个角色复用同一套知识。

> **一句话：重复三次以上的流程，就该封成 Skill。** 注意不是越多越好——描述过宽会在不相关任务乱触发，内容太大会挤占上下文。

---

## 5. Subagents：把复杂工作交给独立角色

**为什么有了 Skill 还需要 Subagent？** Skill 给当前 Agent 加方法，但工作仍发生在**当前上下文**。安全审查、全仓探索这类任务会产生大量中间输出，把主会话上下文搞脏；而且让写代码的 Agent 审自己的代码，容易"我已经改好了，所以应该没问题"的自我认可。

> **Subagent 的价值：另开独立上下文，让专门角色完成任务，只把结论和证据带回来。**

**怎么创建**（项目级放 `.claude/agents/`）：

```md
---
name: security-reviewer
description: Review code changes for authentication, authorization, injection, secret exposure, and unsafe data handling.
tools: Read, Grep, Glob, Bash
model: inherit
skills:
  - secure-coding
---

You are an independent security reviewer.
Review changes without modifying business code.
For every finding, report: file and location; trigger condition; impact; evidence; recommended fix.
```

**适合拆的任务**：过程很长但主会话只需要结果——只读探索大代码库、安全/性能/测试独立复核、多个互不依赖模块的并行调查、需要不同角色从相反角度挑错、希望限制角色的工具/模型/权限。

**不适合拆**：一个文件的小修改、强依赖主会话隐含信息的任务、子任务互相等待、拆分成本比任务本身还高。

> **Skill 是方法，Subagent 是拿着方法独立干活的角色。** 两者可以组合：在 Subagent 的 `skills` 字段里预加载规范。

---

## 6. MCP：让 Claude Code 接上仓库外的世界

Claude Code 自带文件和终端工具，但真实开发还要访问 GitHub、Jira、Sentry、数据库、内部文档、设计工具等外部系统。**MCP（Model Context Protocol）解决 AI 应用如何用统一方式发现和调用外部工具与数据**——Claude Code 作为 MCP Client 连接 Server，不需要给每个 Agent 手写一套集成。

**添加 MCP Server：**

```bash
# 远程 HTTP Server
claude mcp add --transport http issue-tracker https://mcp.example.com/mcp

# 本地 stdio Server
claude mcp add --transport stdio my-tools -- node ./tools/mcp-server.js

claude mcp list          # 查看已添加
claude mcp get issue-tracker
```

进入 Claude Code 后用 `/mcp` 查看连接和认证状态。团队共享的项目级配置可以写进 `.mcp.json`（从仓库拉下来的配置需要信任和审批，别因为进了 Git 就默认安全）。

**三个最常踩的坑：**

1. **权限给太大**——查询数据库用只读账号，评审 PR 就别顺手给删仓库权限
2. **工具太多**——接几十个 Server、暴露几百个工具，增加选择成本、污染上下文；先接最常用、返回结果最干净的
3. **把 MCP 当成工作方法**——MCP 只负责"能连接、能调用"，查询什么、怎么判断、输出什么格式，仍交给 CLAUDE.md、Skill 或 Subagent

> **最常见的组合：MCP 提供手和眼，Skill 提供做事方法。**

---

## 7. Hooks：让关键动作不再依赖"记得"

在 CLAUDE.md 里写"每次改完文件都要格式化"是**提醒**；配置 Hook 让编辑成功后自动运行格式化命令，这是**确定性动作**。Hooks 在生命周期特定事件上触发：会话开始注入环境信息、工具执行前检查拦截、文件修改后自动格式化、工具失败记录诊断、等待确认发通知、上下文压缩前保存状态。

**配置示例**（写在 `.claude/settings.json` 或 `~/.claude/settings.json`）：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write"
          }
        ]
      }
    ]
  }
}
```

真实项目建议把复杂逻辑放进独立脚本（`.claude/hooks/check-edited-file.sh`），能进 Git、能单独测试，不用把一长串 shell 塞进 JSON。配置后运行 `/hooks` 确认加载。

**Hook 怎么用才不会变成新坑：**

- 匹配范围尽量窄，不要什么事件都用 `.*`
- 默认快速执行，重任务不要卡住每次编辑
- 脚本要有明确退出码和错误信息
- **不要在 Hook 里偷偷做发布、删除、推送等高风险动作**
- 先手动运行脚本，再接入 Hook
- 需要复杂判断时用 Skill 或独立审查 Agent，不要堆一坨 shell

> **Skill 是"需要时才调的能力"，Hooks 是"每次都确定触发的动作"。最适合 Hook 的是"发生到这里就必须做"的动作；最不适合的是需要阅读大量上下文、权衡多方方案的开放问题。**

---

## 8. Plugins：把一套能力变成团队可安装的产品

Plugin 不是第七种能力，而是**包装和分发格式**。一个 Plugin 可以同时带上 Skills、Subagents、Hooks、MCP Server 配置、LSP 配置等所有可复用组件。在项目的 `.claude/` 目录做实验适合快速迭代；配置稳定、需要跨项目/跨团队安装升级时，再做成 Plugin。

**最小结构：**

```text
team-toolkit/
├── .claude-plugin/
│   └── plugin.json
├── skills/release-check/SKILL.md
├── agents/security-reviewer.md
├── hooks/hooks.json
└── .mcp.json
```

```json
{
  "name": "team-toolkit",
  "description": "Shared release and review workflows for our engineering team",
  "version": "1.0.0"
}
```

安装：Claude Code 里输入 `/plugin` 打开面板，或 `/plugin install github@claude-plugins-official`；本地开发用 `claude --plugin-dir ./team-toolkit` 加载。**注意：先看清插件提供什么组件、需要什么权限、会连接哪些外部系统——安装插件不是给它无限授权。**

> **Plugin 解决的是分发，不会自动修复里面写得很差的 Skill、过宽的 Hook 或权限过大的 MCP。**

---

## 9. 六种能力怎么组合：一个真实案例

**场景**：给电商系统增加"退款审批"功能，涉及业务规则、数据库、权限、测试、工单和发布。正确做法不是把所有要求塞进一个超级 Prompt，而是按问题分层：

1. **CLAUDE.md** —— 放每次都要知道的稳定事实：金额以分为整数存储、退款 API 必须幂等、所有审批动作需审计日志、改支付模块后跑 `pnpm test:payments`、禁止未审批的生产迁移。
2. **Skill** —— 创建 `refund-review` Skill：幂等检查清单、金额精度规则、审批状态机、审计字段要求、测试模板、报告格式。
3. **Subagent** —— `payment-security-reviewer` 只读检查：能否越权审批、重放请求会不会重复退款、日志是否泄露敏感信息、状态转换能否绕过，主 Agent 只接收带证据的结论。
4. **MCP** —— 读取退款工单、查询错误平台历史异常、查看 PR、用只读账号查测试环境数据。
5. **Hooks** —— 修改支付代码后自动格式化+静态扫描、执行数据库命令前检查环境、准备提交时确认关键测试通过。
6. **Plugin** —— 以上都跑稳后打包成 `payment-engineering`，新同事安装即获得同一套能力。

**六层关系总结：**

```text
CLAUDE.md：告诉 Claude 这个项目长期怎么做
Skill：    告诉 Claude 这类任务具体怎么做
Subagent： 安排一个独立角色去做
MCP：      给这个角色接上外部工具和数据
Hook：     在关键节点自动检查和拦截
Plugin：   把前面几层打包给团队
```

---

## 10. 从零开始，按什么顺序配置？

不要第一天就装几十个 Plugin、接十几个 MCP、开一队 Subagents——配置越多，冲突、权限和上下文成本越高。**按问题出现的顺序来：**

1. **先写根 CLAUDE.md 到能用**：只写命令、架构、禁区、最常见的坑；运行 `/memory` 确认加载
2. **把重复出现的流程做成一个 Skill**：从最常复制的发布/评审/排障流程开始；运行 `/skills` 检查描述和作用域
3. **给"必须发生"的动作加 Hook**：先接格式化、lint、危险命令拦截；运行 `/hooks` 验证成功和失败路径
4. **只为明确场景创建 Subagent**：优先做安全审查或大范围只读探索；运行 `/agents` 检查工具/模型/Skills
5. **按真实需求接 MCP**：先接一个高频系统，给最小权限；运行 `/mcp` 查看认证状态
6. **稳定以后再做 Plugin**：先证明在项目里真的有用，再考虑分发——插件化太早只会把没想清楚的配置更快复制出去

**故障排查**：配置不生效时先运行 `/doctor`、`/status`、`/permissions`，确认"有没有加载、从哪里加载、最终权限是什么"，再怀疑模型。

---

## 11. 最容易配错的六个地方

1. **把 CLAUDE.md 写成百科全书** → 每轮占上下文，重要规则反而不突出；稳定事实留下，专项流程移到 Skill
2. **Skill 的 description 写得太空**（"帮助开发"什么都能匹配）→ 写清任务、触发语境和不适用范围
3. **为了"并行"滥用 Subagents** → 拆任务、传上下文、汇总都有成本；只有任务能独立完成或过程噪音需要隔离时才拆
4. **MCP 一上来就连生产写权限** → 只读优先、最小权限、敏感动作保留人工确认
5. **Hook 做得又重又宽** → 缩小 matcher，复杂逻辑放可测试脚本，重任务改成按需 Skill
6. **把 Plugin 当"装得越多越强"** → 组件可能冲突、工具列表膨胀、权限来源难审计；只装能解决明确问题的

---

## 12. 高频问题速答

- **CLAUDE.md 和自动 Memory 的区别？** CLAUDE.md 是团队明确维护、可进 Git 的项目规则；自动 Memory 是 Claude 积累的本机经验（调试发现、个人习惯）。团队制度写进 CLAUDE.md，不要等自动 Memory 猜。
- **Skill 和 Hook 怎么选？** 需要结合上下文判断、执行一套方法 → Skill；命中事件必须执行、结果稳定可验证 → Hook。"按团队标准审查 PR"是 Skill，"编辑后自动格式化"是 Hook。
- **Skill 和 Subagent 怎么选？** 给当前上下文补知识 → Skill；把长过程隔离出去只拿结果 → Subagent。安全角色需要固定方法就让 Subagent 预加载 Skill。
- **MCP 和 Plugin 的关系？** MCP 是连接协议（怎么调用外部系统），Plugin 是分发包（怎么把 MCP 配置连同 Skills、Agents、Hooks 一起发给别人）。
- **配齐六件套就一定更好吗？** 不一定。小项目可能只需要 30 行 CLAUDE.md 和一个格式化 Hook。**最好的配置不是组件最多，而是每一层都在解决真实问题。**

---

## 13. 跳出聊天框：CLI 管道与定时任务

Claude Code 遵循 Unix 哲学，**可以管道化、能塞进脚本和 CI**。用 `claude -p`（print 模式）把它变成命令行的一环：

```bash
# 盯日志，发现异常就通知
tail -200 app.log | claude -p "如果发现任何异常就通知我"

# 批量审查改动文件的安全问题
git diff main --name-only | claude -p "审查这些改动文件有没有安全问题"
```

再往上是**定时任务**，让重复的活儿自动发生：早晨 PR 审查、夜里 CI 失败分析、每周依赖审计：

- 终端里临时轮询，用 `/loop`（背后是"写 Loop 不写 Prompt"的思路）
- 想要**电脑关机也照跑**，用 **Routines**——跑在 Anthropic 托管的基础设施上，还能被 API 调用或 GitHub 事件触发

另外，Claude Code 不绑死在一个界面：终端、VS Code/JetBrains、桌面 App、网页背后是**同一个引擎**（CLAUDE.md、设置、MCP 在哪儿都通用）。离开工位可用手机/浏览器远程接管正在跑的会话；网页或 iOS 上起长任务，回头用 `claude --teleport` 拽回终端。**工作跟着你走，不跟着设备走。**

> **能用一句话提问，也能塞进管道和定时任务无人值守地跑——后者才是效率的天花板。**

---

## 总结

高效用 Claude Code 就一条主线：**把"每次都要重复的东西"一层层固化下来**——项目规则固化进 CLAUDE.md，专项流程固化成 Skill，必做动作固化成 Hooks，大任务固化成并行编排，再到无人值守的管道与定时任务。固化得越多，你要操的心越少。

而贯穿所有能力的唯一指导思想是：

> **CLAUDE.md 不是魔法，它不会让 Claude Code 瞬间变成懂你公司所有业务的老员工。但它能解决一个非常实际的问题：别让 AI 每次进项目都从零猜。短一点，准一点，硬一点——这就够了。**

## 参考链接

- Claude Code 官方文档（概述）：https://code.claude.com/docs/zh-CN/overview
- Claude Code 官方文档（扩展能力总览）：https://code.claude.com/docs/en/features-overview
- Claude Code 官方文档（CLAUDE.md 与 Memory）：https://code.claude.com/docs/en/memory
- Claude Code 官方文档（Skills）：https://code.claude.com/docs/en/slash-commands
- Claude Code 官方文档（Subagents）：https://code.claude.com/docs/en/sub-agents
- Claude Code 官方文档（MCP）：https://code.claude.com/docs/en/mcp
- Claude Code 官方文档（Hooks）：https://code.claude.com/docs/en/hooks-guide
- Claude Code 官方文档（Plugins）：https://code.claude.com/docs/en/plugins
