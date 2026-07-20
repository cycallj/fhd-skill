# 工单查询 API 文档

## 接口信息

- **方法**：GET
- **URL**：`https://xyymgrapi.fhd001.com/mgr/cs/workOrder/workOrderList`
- **必带参数**：`referer=mgrapi`、`token`

---

## 请求参数

### 必填参数

| 参数名 | 类型 | 说明 |
|--------|------|------|
| `referer` | String | 固定值 `mgrapi` |
| `token` | String | 认证 token |
| `page` | int | 页码（从 1 开始） |
| `pageSize` | int | 每页条数 |

### 可选筛选参数

| 参数名 | 类型 | 说明 |
|--------|------|------|
| `beginTime` | String | 开始时间，格式 `YYYY-MM-DD HH:mm:ss` |
| `endTime` | String | 结束时间，格式 `YYYY-MM-DD HH:mm:ss` |
| `processStatusInt` | int | 处理状态（见枚举表） |
| `typeInt` | int | 工单类型（见枚举表） |
| `productIdInt` | int | 产品 ID（见枚举表） |
| `questionTagIdInt` | int | 问题标签 ID（见枚举表） |
| `nick` | String | 用户昵称（模糊匹配） |
| `questionContent` | String | 问题内容（模糊匹配） |

---

## 枚举表

### processStatusInt — 处理状态

| 值 | 说明 |
|:--:|------|
| 0 | 待处理 |
| 1 | 处理中 |
| 2 | 已处理 |
| 3 | 已关闭 |

### typeInt — 工单类型

| 值 | 说明 |
|:--:|------|
| 1 | 普通工单 |
| 2 | 紧急工单 |
| 3 | 投诉工单 |
| 4 | 建议工单 |

> 注：以上枚举值为常用值，完整枚举以 API 实际返回为准。

### productIdInt — 产品

| 值 | 说明 |
|:--:|------|
| 1 | 风火递 |
| 2 | 风火蚁 |
| 3 | 风天河 |

> 注：以上枚举值为常用值，完整枚举以 API 实际返回为准。

---

## 响应结构

### 成功响应

```json
{
  "page": 1,
  "pageSize": 10,
  "total": 150,
  "list": [
    {
      "id": 12345,
      "nick": "用户昵称",
      "questionContent": "问题内容",
      "processStatusInt": 0,
      "typeInt": 1,
      "productIdInt": 1,
      "questionTagIdInt": 5,
      "createTime": "2026-07-17 10:30:00",
      "updateTime": "2026-07-17 12:00:00"
    }
  ]
}
```

### 错误响应

```json
{"scode": 6, "errorMsg": "认证失败"}
{"code": "AUTH_FAILED", "message": "Token无效"}
{"code": "PARAM_ERROR", "message": "参数错误"}
```

---

## curl 调用示例

### 基础查询（最近7天工单，第1页，每页10条）

```bash
TOKEN=$(cat fire-skill/.token)
curl -s -G "https://xyymgrapi.fhd001.com/mgr/cs/workOrder/workOrderList" \
  --data-urlencode "referer=mgrapi" \
  --data-urlencode "token=$TOKEN" \
  --data-urlencode "page=1" \
  --data-urlencode "pageSize=10" \
  --data-urlencode "beginTime=2026-07-10 00:00:00" \
  --data-urlencode "endTime=2026-07-17 23:59:59"
```

### 按状态筛选（待处理工单）

```bash
TOKEN=$(cat fire-skill/.token)
curl -s -G "https://xyymgrapi.fhd001.com/mgr/cs/workOrder/workOrderList" \
  --data-urlencode "referer=mgrapi" \
  --data-urlencode "token=$TOKEN" \
  --data-urlencode "page=1" \
  --data-urlencode "pageSize=10" \
  --data-urlencode "processStatusInt=0"
```

### 按用户昵称搜索

```bash
TOKEN=$(cat fire-skill/.token)
curl -s -G "https://xyymgrapi.fhd001.com/mgr/cs/workOrder/workOrderList" \
  --data-urlencode "referer=mgrapi" \
  --data-urlencode "token=$TOKEN" \
  --data-urlencode "page=1" \
  --data-urlencode "pageSize=10" \
  --data-urlencode "nick=张三"
```

### 组合筛选

```bash
TOKEN=$(cat fire-skill/.token)
curl -s -G "https://xyymgrapi.fhd001.com/mgr/cs/workOrder/workOrderList" \
  --data-urlencode "referer=mgrapi" \
  --data-urlencode "token=$TOKEN" \
  --data-urlencode "page=1" \
  --data-urlencode "pageSize=20" \
  --data-urlencode "beginTime=2026-07-01 00:00:00" \
  --data-urlencode "endTime=2026-07-17 23:59:59" \
  --data-urlencode "processStatusInt=0" \
  --data-urlencode "typeInt=1" \
  --data-urlencode "productIdInt=1"
```
