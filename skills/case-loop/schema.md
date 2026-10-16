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

仅 `case` 使用：

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
