# 通用版用户基本信息查询

## 接口信息

- **产品**：通用版
- **方法**：GET
- **URL**：`https://mgrapi.fhd001.com/mgr/cs/info/getUserBaseInfo`
- **必带参数**：`referer=mgrapi`、`token`、`searchId`、`searchType`
- **附加参数**：`isFhd=true`

调用前先阅读 [共享 API 规则](../../shared-api.md) 和 [用户信息展示规则](../presentation.md)。

## 查询参数

| 用户提供的查询值 | `searchType` | `searchId` |
|------------------|:------------:|------------|
| 用户 ID | `1` | 用户 ID |
| 手机号 | `2` | 手机号 |
| 风火号 | `3` | 风火号 |

风火号会先转换为用户 ID；转换失败或用户不存在时，接口返回空响应。

## 调用模板

```bash
curl -s -G "https://mgrapi.fhd001.com/mgr/cs/info/getUserBaseInfo" \
  --data-urlencode "referer=mgrapi" \
  --data-urlencode "token=$FIRE_TOKEN" \
  --data-urlencode "searchId=<SEARCH_ID>" \
  --data-urlencode "searchType=<SEARCH_TYPE>" \
  --data-urlencode "isFhd=true"
```

## 响应处理

成功响应是用户对象；无用户匹配时 HTTP 200 但 body 为空。按 [用户信息展示规则](../presentation.md) 输出以下基础字段：

- `userBase.userId`、`userBase.nickName`、`userBase.cellphone`、`userBase.createTime`
- `fhdCode`、`status`

按需补充 `accountInfo`、`sms`、`consume`、`introducerNick`、`clientInfo`、`clientVersion`、`order` 与 `netpointCustomerMap` 摘要。不要输出 `idNum`、`identityInfo` 或 `realName`。
