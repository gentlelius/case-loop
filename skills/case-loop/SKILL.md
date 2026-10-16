---
name: case-loop
description: >-
  Generates and revises software test cases as JSON in the current project's
  .case-loop directory after a Socratic detail-gathering round. Use when the
  user asks to 生成测试用例, 新增测试用例, 修改测试用例, 补测试用例, 写测试用例,
  or to create or update test cases.
---

# Case Loop

生成或修改测试用例。追问是为了补齐细节，不是教学：问清楚后必须写出 JSON。

页面只编辑已有 JSON，不在页面里生成用例。用例目录是 agent 当前工作区里的 `.case-loop`，随这次执行的项目变化。

可视化查看器随本 skill 发布，是单文件程序 `scripts/case-loop-viewer.mjs`（与本文件同目录）。用本机的 `node` 运行。它已经包含页面和接口，不要回到源码仓库，也不要执行 `npm install`。

## 先读再问

第一轮禁止写 JSON。先读：

- 用户这次给出的需求、改动说明、相关代码
- 当前项目 `.case-loop/*.json` 的 `id`、`title` 和用例标题

然后判定：

- **修改**：已有文件的 id 或 title 对应该功能。点名这个文件。多个都像时，先问用哪一个。
- **新增**：没有对应用例文件。

用户口头说「新增」或「修改」时以用户为准。说修改但找不到文件时，问是新建还是另有文件，不要擅自新建。

## 追问

细节、停止条件和示例见 [questions.md](questions.md)。字段与落盘规则见 [schema.md](schema.md)。

- 用用户的语言。每轮 3 到 5 个具体问题，只问上下文里还没有的缺口。
- 开放问题写在对话里。只有封闭选项时才用 AskQuestion。
- 用户说「可以生成 / 够了 / 就这样」后停止追问并写文件。
- 用户坚持马上生成：只再问一轮仍空缺的阻塞项（主路径、可观察结果；修改时再加变更点）。仍拒绝就生成，并把缺口写进对应用例的 `note`，`status` 保持 `draft`。

## 写入

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
