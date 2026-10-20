# 用例文件

路径：`<当前项目>/.case-loop/<slug>.json`

`<slug>` 匹配 `^[a-z0-9][a-z0-9-]{0,63}$`，与 `id` 相同。

```json
{
  "id": "login",
  "title": "登录",
  "mode": "create",
  "updatedAt": "2026-10-15T00:00:00.000Z",
  "nodes": [
    {
      "id": "n_root",
      "parentId": null,
      "kind": "root",
      "title": "登录",
      "createdAt": "2026-10-15T00:00:00.000Z",
      "updatedAt": "2026-10-15T00:00:00.000Z"
    },
    {
      "id": "n_group_fail",
      "parentId": "n_root",
      "kind": "group",
      "title": "失败提示",
      "createdAt": "2026-10-15T00:00:00.000Z",
      "updatedAt": "2026-10-15T00:00:00.000Z"
    },
    {
      "id": "n_wrong_password",
      "parentId": "n_group_fail",
      "kind": "case",
      "title": "密码错误时停留在登录页并提示",
      "priority": "blocker",
      "status": "draft",
      "source": "ai",
      "revision": "added",
      "preconditions": "已有账号 ada，密码不是 secret",
      "steps": ["打开登录页", "输入 ada / wrong", "提交"],
      "expectedResult": "仍在登录页，出现密码错误提示，不进入首页",
      "note": "未确认提示文案是否与账号不存在不同",
      "createdAt": "2026-10-15T00:00:00.000Z",
      "updatedAt": "2026-10-15T00:00:00.000Z"
    }
  ]
}
```

## 结构

- 有且仅有一个 `kind: "root"`，其 `parentId` 为 `null`。
- 其余节点的 `parentId` 必须指向存在的节点，不能成环。
- `kind`：`root` | `group` | `case`。分组表示模块或场景，用例只放在 `case`。
- 节点 `id` 在文件内唯一。用 `n_` 加短随机串。修改时不要重写未改节点的 id。

## 用例字段

`case` 的字段：

| 字段 | 取值 |
| --- | --- |
| priority | `blocker` `major` `minor` |
| status | `draft` `ready` `implemented` `passed` `failed` `skipped` |
| source | `ai` `user` `imported` |
| revision | `added` `updated` `unchanged` `obsolete` |
| preconditions, inputData, expectedResult, note | 字符串 |
| steps | 字符串数组，一步一条 |

`mode`：新增 `create`，修改 `update`。

技能生成的新用例：`source` 为 `ai`，`status` 为 `draft`，`revision` 为 `added`。不要写测试类型（单元、集成、E2E、手动、回归）。仍不确定的点写入 `note`，不要编造成已确认的预期。

## 修改

- 未提及的节点：原样保留，`revision` 保持 `unchanged`（原先没有 revision 的不要强行补）。
- 改了步骤或预期的旧用例：就地更新，`revision` 设为 `updated`。
- 新用例：追加到合适的 group 下；没有合适分组再新建 group。
- 作废：`revision` 设为 `obsolete`，`status` 设为 `skipped`，`note` 写明原因。不要删除节点。

## 少填写的数据约定

- 用例正文优先生成 title、steps、expectedResult。普通输入直接写进步骤，不重复写 inputData；仅复杂数据块或独立数据有必要时使用 inputData。
- root/group 也允许可选的 preconditions、inputData、note，表示该节点后代共用的条件、数据与待补充信息。修改公共节点即应用到其后代，无需复制到每条用例。
- 条件从 root 到当前节点累加。子节点仅写特殊要求；存在冲突时明确说明局部例外，不能静默覆盖。
- 公共缺口在最小公共祖先的 note 中记录一次，以「待补充：」开头。单条缺口写在 case.note；仍受缺口影响的用例保持 draft。
- 真实账号、订单、环境和未知业务规则不得伪造；可写数据要求，明确待补充。示例值不得表述成真实可用数据。
- priority 自动初判；非必要 note 和其他空字段省略。历史独立 inputData 与备注仍保留并可编辑。
- 编辑器默认展示标题、步骤、预期；已有条件和输入按需展示，空的可选字段及元信息放在折叠详情中。公共条件从祖先读取，用户在所属公共节点修改一次。
