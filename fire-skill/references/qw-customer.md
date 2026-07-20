# 弹幕捕手企微客户查询 API 文档

## 接口信息

- **方法**：GET
- **URL**：`https://fireapi.fhd001.com/mgr/pd/xhsdm/queryQwCustomerPage`
- **必带参数**：`referer=mgrapi`、`token`

调用前先阅读 [共享 API 规则](shared-api.md)。该接口与工单查询共用同一个 Token。

## 请求参数

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|:----:|--------|------|
| `page` | Integer | 否 | `1` | 页码，从 1 开始 |
| `pageSize` | Integer | 否 | `20` | 每页条数，必须为正整数 |
| `name` | String | 否 | - | 模糊匹配微信昵称和客户名称 |
| `createTimeBegin` | Integer | 否 | - | 创建开始时间，Unix 秒级时间戳，含边界 |
| `createTimeEnd` | Integer | 否 | - | 创建结束时间，Unix 秒级时间戳，含边界 |
| `tagIds` | String | 否 | - | 本地标签 ID，多个 ID 以逗号分隔，按 AND 逻辑筛选 |
| `sortField` | String | 否 | `createTime` | `createTime` 或 `updateTime` |
| `sortOrder` | String | 否 | `desc` | `asc` 或 `desc` |

## 查询流程

1. 按 [共享 API 规则](shared-api.md) 检查 Token。
2. 从用户请求提取筛选条件。未指定页码时使用 `page=1`、`pageSize=20`；未指定排序时按创建时间倒序。
3. 用户提到时间范围时，将其转换为 `createTimeBegin` 和 `createTimeEnd` 的 Unix 秒级时间戳。
4. 用户指定多个标签时，将标签 ID 以逗号拼接为 `tagIds`；说明接口会返回同时包含全部标签的客户。
5. 调用接口，并按 [企微客户结果模板](qw-customer-format-template.md) 展示结果。

## curl 调用模板

```bash
curl -s -G "https://fireapi.fhd001.com/mgr/pd/xhsdm/queryQwCustomerPage" \
  --data-urlencode "referer=mgrapi" \
  --data-urlencode "token=$FIRE_TOKEN" \
  --data-urlencode "page=<PAGE>" \
  --data-urlencode "pageSize=<PAGE_SIZE>" \
  --data-urlencode "sortField=<SORT_FIELD>" \
  --data-urlencode "sortOrder=<SORT_ORDER>" \
  ...其他可选筛选参数
```

每个参数必须使用独立的 `--data-urlencode` 传递。不得将 Token 明文写入命令、文档或输出。

## 响应结构

```json
{
  "page": 1,
  "pageSize": 20,
  "total": 150,
  "list": []
}
```

| 字段 | 说明 |
|------|------|
| `id` | 客户记录主键 |
| `customerName` | 企微侧客户备注名或企业名称 |
| `name` | 微信昵称 |
| `gender` | `0` 未知、`1` 男、`2` 女 |
| `userId` | 企微内部跟进人 ID |
| `followRecord` | 客户跟进总结 |
| `tag` | 已关联的本地标签 ID，逗号分隔 |
| `externalUserid` | 企微外部用户唯一标识 |
| `createTime` / `updateTime` | Unix 秒级创建/更新时间 |

## 错误处理

复用 [共享 API 规则](shared-api.md) 的错误处理。额外处理以下参数错误：

- `page` 或 `pageSize` 小于等于 0：提示用户提供正整数页码和每页条数。
- 非 `createTime` / `updateTime` 的排序字段：改用 `createTime`。
- 非 `asc` 的排序方向：改用 `desc`。
