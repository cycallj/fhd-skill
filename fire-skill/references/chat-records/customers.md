# 通过风火号获取客户列表

## 接口信息

- **方法**：GET
- **URL**：`https://chcapi.fhd001.com/mgr/css/getCustomerList`
- **必带参数**：`referer=mgrapi`、`token`、`fhdCode`、`isChc=true`
- **响应类型**：非分页数组

调用前先阅读 [共享 API 规则](../shared-api.md) 和 [聊天记录查询流程](overview.md)。该接口与其他火花派接口共用当前 Token。

## 调用模板

```bash
curl -s -G "https://chcapi.fhd001.com/mgr/css/getCustomerList" \
  --data-urlencode "referer=mgrapi" \
  --data-urlencode "token=$FIRE_TOKEN" \
  --data-urlencode "fhdCode=<FHD_CODE>" \
  --data-urlencode "isChc=true"
```

## 响应字段

| 字段 | 说明 |
|------|------|
| `customerId` | 客户 ID；用于查询消息记录 |
| `name` | 客户名称或昵称 |
| `platform` | CSS 客户来源平台，见 [聊天记录查询流程](overview.md) 的平台映射 |
| `fhdCode` | 客户关联的风火号 |
| `avatar` | 客户头像 URL；默认不展示 |

接口按风火号精确匹配，可能返回同一风火号下多个平台的客户。查询为空时返回 `[]`，提示当前风火号下未找到 CSS 客户。
