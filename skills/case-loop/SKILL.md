---
name: case-loop
description: >-
  Generates and revises software test cases as JSON in the current project's
  .case-loop directory with minimal user input. Use when the
  user asks to 生成测试用例, 新增测试用例, 修改测试用例, 补测试用例, 写测试用例,
  or to create or update test cases.
---

# Case Loop

生成或修改测试用例。默认生成完整内容，用户只补必要缺口；不要让用户逐项填写用例字段。

页面只编辑已有 JSON，不在页面里生成用例。用例目录是 agent 当前工作区里的 `.case-loop`，随这次执行的项目变化。

可视化查看器随本 skill 发布，是单文件程序 `scripts/case-loop-viewer.mjs`（与本文件同目录）。用本机的 `node` 运行。它已经包含页面和接口，不要回到源码仓库，也不要执行 `npm install`。

## 先读再问

生成前先读；信息足够时第一轮即可写 JSON：

- 用户这次给出的需求、改动说明、相关代码
- 当前项目 `.case-loop/*.json` 的 `id`、`title` 和用例标题

然后判定：

- **修改**：已有文件的 id 或 title 对应该功能。点名这个文件。多个都像时，先问用哪一个。
- **新增**：没有对应用例文件。

用户口头说「新增」或「修改」时以用户为准。说修改但找不到文件时，问是新建还是另有文件，不要擅自新建。

## 追问

细节、停止条件和示例见 [questions.md](questions.md)。字段与落盘规则见 [schema.md](schema.md)。

- 优先从需求、已有用例和相关代码提取信息，不重复询问已知内容。
- 只追问会改变测试目标、主路径或可观察预期的关键缺口；不要求固定问题数。问题尽量集中在一轮，并给出容易回答的选项。
- 普通输入自动生成并写入步骤；优先级由 AI 初步判断，备注默认省略。不要为账号、环境、优先级等字段逐项追问。
- 公共条件与数据放在 root/group 中一次维护；同一缺口集中记录一次。具体规则见 schema.md。
- 信息足够或用户说「可以生成 / 够了 / 就这样」时直接写文件。无法确认的业务规则、真实数据标为待补充，保留 draft，不编造，也不阻塞其他用例生成。

## 写入

- 生成或修改用例正文前，先读 [用例设计与编写指引](references/test-design.md)，依据已确认规则选择界面、功能、接口检查点；不要机械凑齐分类或编造预期。
- 写入前按指引完成内容自检，检查依据、可执行性、可判定性、规则覆盖与重复场景。缺口按 questions.md 处理，不阻塞其余可确定用例。
- 路径：agent 当前工作区的 `.case-loop/<slug>.json`。用这次工作区的绝对路径，不要写死某台机器或某个仓库的路径。`slug` 为小写 kebab-case，与文档 `id` 一致。中文功能用简短英文或拼音 slug，中文放在 `title`。
- 一个功能一个文件。不要把无关功能塞进同一份 JSON。
- 新增：新建文件，`mode` 为 `create`。
- 修改：改原文件，保留未提及节点的 `id` 和正文；`mode` 为 `update`。用户没要求删除的过时用例标 `revision: "obsolete"`、`status: "skipped"`，不要物理删除。
- 写完后读回文件，确认是合法 JSON 且能通过 schema.md 的结构约束。文档的 `updatedAt` 写成这次写入的新时间，页面靠它判断要不要重新读取。

## 打开可视化

写完后必须先尝试用 agent 内置浏览器打开这次的页面；内置浏览器不可用或打开失败时，再用系统默认浏览器打开同一页面。最后告知 JSON 路径。

`http://127.0.0.1:3927/?dir=<项目根>&file=<slug>.json`

`dir` 是项目根绝对路径，不是 `.case-loop`。`dir` 和 `file` 都要做查询参数编码。

一台机器只开一个查看器，后台启动，不要传 `--dir`，也不要结束别人的进程。脚本用本 skill 目录下的 `scripts/case-loop-viewer.mjs`，和本文件放在一起。

先请求 `http://127.0.0.1:3927/api/health`，确认 `ok: true` 且 `multi: true` 后再打开 URL：

- 已满足：直接打开。
- 连不上：`node "<查看器脚本>"`，等到 health 满足后再打开。
- 没有 `multi: true`，或启动报端口占用后仍不满足：结束占用 3927 的进程，用同一命令重启，等到满足后再打开。
- 找不到脚本：告诉用户 skill 里没有 `scripts/case-loop-viewer.mjs`。不要 `npm install` 或 `npm run dev`。
