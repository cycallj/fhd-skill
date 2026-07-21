# 一站式发货用户基本信息查询

## 接口信息

- **产品**：一站式发货
- **方法**：POST
- **URL**：`https://osdapi.fhd001.com/mgr/mgr/base/getOsdUser`
- **Content-Type**：`application/x-www-form-urlencoded`
- **必带参数**：`referer=mgrapi`、`token`、`code`、`isOsd=true`

调用前先阅读 [共享 API 规则](../../shared-api.md) 和 [用户信息展示规则](../presentation.md)。

## 查询参数

`code` 支持手机号、风火号或用户 ID：

1. 11 位纯数字优先按手机号查询。
2. 未按手机号命中时，按风火号查询。
3. 纯数字风火号仍未命中时，按用户 ID 查询。

因此只需将用户提供的查询值原样传入 `code`，无需在客户端判断类型。

## 调用模板

```bash
curl -s -X POST "https://osdapi.fhd001.com/mgr/mgr/base/getOsdUser" \
  --data-urlencode "referer=mgrapi" \
  --data-urlencode "token=$FIRE_TOKEN" \
  --data-urlencode "code=<CODE>" \
  --data-urlencode "isOsd=true"
```

## 响应处理

成功时返回用户对象；无用户匹配时返回 `null`。按 [用户信息展示规则](../presentation.md) 输出：

- 基础字段：`userId`、`userName`、`fhdCode`、`phone`、`createTime`
- 状态与归属：`main`、`organizationId`、`cluster`、`dbNo`
- 补充信息：`loginTime`、`users` 组织关系

默认仅汇总组织关系。主账号的 `users` 最多可能包含 100 名成员，只有用户明确要求时才展示成员明细；成员手机号同样必须脱敏。
