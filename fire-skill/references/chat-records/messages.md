# 获取客户消息记录

## 接口信息

- **方法**：GET
- **URL**：`https://chcapi.fhd001.com/mgr/css/getMessageRecordList`
- **必带参数**：`referer=mgrapi`、`token`、`customerId`、`startTime`、`endTime`、`isChc=true`
- **响应类型**：非分页数组

调用前先阅读 [共享 API 规则](../shared-api.md) 和 [聊天记录结果模板](format-template.md)。

## 查询参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `customerId` | Long | 从 [客户列表接口](customers.md) 获取的客户 ID |
| `startTime` | Integer | 查询开始时间，Unix 秒级时间戳 |
| `endTime` | Integer | 查询结束时间，Unix 秒级时间戳 |

三个参数均为必填项。时间过滤基于会话 `createTime`，不是单条消息的实际发送时间。

## 调用模板

```bash
curl -s -G "https://chcapi.fhd001.com/mgr/css/getMessageRecordList" \
  --data-urlencode "referer=mgrapi" \
  --data-urlencode "token=$FIRE_TOKEN" \
  --data-urlencode "customerId=<CUSTOMER_ID>" \
  --data-urlencode "startTime=<START_TIME>" \
  --data-urlencode "endTime=<END_TIME>" \
  --data-urlencode "isChc=true"
```

## 关键字段与枚举

| 字段 | 说明 |
|------|------|
| `content` | 消息内容 |
| `agentName` | 客服名称 |
| `createTime` | 消息创建时间，Unix 秒级时间戳 |
| `replyType` | `1` 客户提问、`2` 自动回复、`3` AI 回复、`4` 人工回复、`5` 结束语 |
| `type` | `1` 文本、`2` 图片、`3` 文件、`4` 视频、`5` 特殊消息 |
| `state` | `1` 未读、`2` 已读、`3` 已撤回、`4` AI |
| `score` | `0` 未评价、`1` 有用、`2` 相关、`3` 无用 |
| `specialType` | `1` 转人工、`2` 评价组件、`3` 客服插件 |
| `plugContent` | AI 插件详情；默认不输出原始参数和结果 |

接口返回空数组时，提示指定时间范围内无聊天记录。
