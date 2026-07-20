# 弹幕捕手企微标签列表 API 文档

## 接口信息

- **方法**：GET
- **URL**：`https://fireapi.fhd001.com/mgr/pd/xhsdm/queryQwTags`
- **必带参数**：`referer=mgrapi`、`token`
- **响应类型**：非分页数组

调用前先阅读 [共享 API 规则](shared-api.md)。该接口与企微客户查询共用同一个 Token。

## 请求参数

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|:----:|--------|------|
| `status` | Integer | 否 | `0` | `0` 查询正常标签，`1` 查询已删除标签，`-1` 查询全部标签 |

## 使用方式

### 作为客户查询的辅助接口

当用户按标签名称筛选客户，或需要将客户的 `tag` ID 显示为真实名称时：

1. 使用 `status=0` 查询正常标签，并在当前请求内构建 `id -> groupName/tagName` 映射。
2. 将用户指定的标签名称解析为本地 `id`，传给客户接口的 `tagIds` 参数。
3. 将客户结果的 `tag` 字段按同一映射展示为标签名称。

本地 `id` 用于客户的 `tag` 和 `tagIds`；企微侧 `tagId` 不可用于客户列表筛选。

### 作为直接查询

用户明确要求“查看企微标签列表”“查看标签字典”或“查已删除标签”时，直接调用本接口并按 [企微标签结果模板](qw-tags-format-template.md) 展示。默认只查询 `status=0` 的正常标签；用户要求全部或已删除标签时，分别使用 `-1` 或 `1`。

## curl 调用模板

```bash
curl -s -G "https://fireapi.fhd001.com/mgr/pd/xhsdm/queryQwTags" \
  --data-urlencode "referer=mgrapi" \
  --data-urlencode "token=$FIRE_TOKEN" \
  --data-urlencode "status=<STATUS>"
```

## 响应字段

| 字段 | 说明 |
|------|------|
| `id` | 本地标签主键；用于客户查询的 `tagIds` 筛选与客户 `tag` 字段映射 |
| `groupId` / `groupName` | 企微标签组 ID 与名称 |
| `tagId` | 企微侧标签唯一 ID；不用于客户列表筛选 |
| `tagName` | 标签真实名称 |
| `status` | `0` 正常，`1` 已删除 |
| `useAi` | `0` 启用 AI 打标，`1` 不启用 AI 打标 |
| `createTime` | Unix 秒级标签创建时间 |

标签接口按状态、标签组名称和标签名称排序。客户结果含有无法在 `status=0` 字典中找到的 ID 时，可能对应已删除或未同步标签；按原始 ID 标示，不擅自匹配。
