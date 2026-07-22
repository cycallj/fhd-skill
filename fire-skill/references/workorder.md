# 工单查询 API 文档

## 目录

- [接口信息](#接口信息)
- [请求参数](#请求参数)
- [枚举表](#枚举表)
- [响应结构](#响应结构)
- [调用示例](#curl-调用示例)

调用前先阅读 [共享 API 规则](shared-api.md)。

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
| `page` | Integer | 页码（从 1 开始），默认 `1` |
| `pageSize` | Integer | 每页条数，默认 `20` |

### 可选筛选参数

| 参数名 | 类型 | 说明 |
|--------|------|------|
| `beginTime` | Integer | 开始时间，Unix 秒级时间戳 |
| `endTime` | Integer | 结束时间，Unix 秒级时间戳 |
| `processStatusInt` | Integer | 处理状态（见枚举表） |
| `typeInt` | Integer | 工单类型（见枚举表） |
| `productIdInt` | Integer | 产品 ID（见枚举表） |
| `questionTagIdInt` | Integer | 问题标签 ID（二级菜单） |
| `isTopInt` | Integer | 是否加急，`1` 表示加急 |
| `satisfaction` | Integer | 满意度 |
| `detectResults` | String | 质检结果，多个值用逗号分隔 |
| `detectCreateUser` | String | 质检人 |
| `createUser` | String | 工单创建人 |
| `nick` | String | 用户昵称（模糊匹配） |
| `questionContent` | String | 问题内容（模糊匹配） |
| `processNote` | String | 处理备注（模糊匹配） |
| `groupLeader` | String | 组长 |
| `distinct` | Boolean | 是否按天、风火号和客服 ID 去重，取最早一条 |
| `commitDevelopUser` | String | 提交研发人 |
| `userInfoAssociatedDto` | Object | 客户关联信息，支持 `wangwang` 和 `fhdCode`；同时传入时为 OR 条件 |

---

## 枚举表

### processStatusInt — 处理状态

| 值 | 状态 | 查询含义 |
|:--:|------|----------|
| 0 | 已完成 | `processStatus = 0` |
| 1 | 待处理 | `processStatus = 1`，且排除产品待处理 |
| 2 | 已废除 | `processStatus = 2` |
| 3 | 转研发 | `processStatus = 3` |
| 4 | 待处理加急 | 加急且未完成 |
| 5 | 转研发已完成 | 已指定研发人员、研发完成且工单已完成 |
| 6 | 产品待处理 | 产品待处理且工单未完成 |
| 7 | 转组长 | `processStatus = 7` |
| 8 | 转组长已完成 | `processStatus = 8` |

未指定 `processStatusInt` 时，接口默认排除已废除工单。

### typeInt — 工单类型

| 值 | 说明 |
|:--:|------|
| 1 | 普通咨询 |
| 2 | 投诉 |
| 3 | 退款 |
| 4 | 中差评 |
| 5 | 建议 |
| 6 | 发票 |
| 7 | 返现 |
| 8 | 操作咨询 |
| 9 | 产品跟进 |
| 10 | 故障 |
| 11 | 跨平台 |

### productIdInt — 产品

当前接口响应包含 `product.name` 时，优先使用该字段展示产品名称；缺少名称时按下表映射。未收录的 ID 保留原始值。

| 值 | 产品名称 |
|:--:|----------|
| 1 | 喜洋洋 |
| 9 | 千牛风火递 |
| 10 | 通用版 |
| 12 | 风火码 |
| 13 | 阿里巴巴 |
| 15 | 手机千牛版 |
| 18 | 拼多多 |
| 19 | 快手 |
| 20 | 抖音 |
| 21 | 京东 |
| 23 | 小程序生意版 |
| 24 | 小程序快递版 |
| 27 | 微店、有赞、蘑菇街、微盟、微信小商店 |
| 29 | 视频号小店 |
| 30 | 小红书 |
| 31 | 军师联盟 |
| 32 | 抖音移动端 |
| 34 | 快手厂家代发 |
| 35 | 抖店厂家代发 |
| 38 | 库小存 |
| 50 | 风火递小店 |
| 56 | 客户推广 |
| 61 | 有赞云商城 |
| 62 | 客服转化 |
| 63 | 风天河 |
| 64 | 风火递助手-寄快递 |
| 65 | 风火递助手-发货业务 |
| 66 | 聚合打单（一站式） |
| 68 | 抖店即时零售 |
| 70 | 弹幕捕手 |
| 71 | 内容转化 |

---

## 响应结构

### 成功响应

以下为常用字段示例，时间字段均为 Unix 秒级时间戳：

```json
{
  "page": 1,
  "pageSize": 20,
  "total": 1,
  "list": [
    {
      "id": 12345,
      "createTime": 1715000000,
      "updateTime": 1715080000,
      "createUser": "客服张三",
      "nick": "用户昵称",
      "questionContent": "问题内容",
      "type": 1,
      "processStatus": 0,
      "processStatusInt": 0,
      "typeInt": 1,
      "productIdInt": 10,
      "questionTagIdInt": 12,
      "processNote": "已同步处理完毕",
      "processUser": "客服张三",
      "processTime": 1715080000,
      "isTopInt": 0,
      "satisfaction": 1,
      "tag": { "id": 12, "name": "订单问题" },
      "product": { "id": 10, "name": "通用版" },
      "thirdLevelMenu": [{ "id": 45, "name": "订单同步失败" }],
      "userInfoAssociatedDto": { "wangwang": "旺旺号123", "fhdCode": "FH123456" }
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

### 基础查询（指定时间范围，第1页，每页20条）

```bash
curl -s -G "https://xyymgrapi.fhd001.com/mgr/cs/workOrder/workOrderList" \
  --data-urlencode "referer=mgrapi" \
  --data-urlencode "token=$FIRE_TOKEN" \
  --data-urlencode "page=1" \
  --data-urlencode "pageSize=20" \
  --data-urlencode "beginTime=<START_UNIX_SECONDS>" \
  --data-urlencode "endTime=<END_UNIX_SECONDS>"
```

### 按状态筛选（待处理工单）

```bash
curl -s -G "https://xyymgrapi.fhd001.com/mgr/cs/workOrder/workOrderList" \
  --data-urlencode "referer=mgrapi" \
  --data-urlencode "token=$FIRE_TOKEN" \
  --data-urlencode "page=1" \
  --data-urlencode "pageSize=10" \
  --data-urlencode "processStatusInt=0"
```

### 按用户昵称搜索

```bash
curl -s -G "https://xyymgrapi.fhd001.com/mgr/cs/workOrder/workOrderList" \
  --data-urlencode "referer=mgrapi" \
  --data-urlencode "token=$FIRE_TOKEN" \
  --data-urlencode "page=1" \
  --data-urlencode "pageSize=10" \
  --data-urlencode "nick=张三"
```

### 组合筛选

```bash
curl -s -G "https://xyymgrapi.fhd001.com/mgr/cs/workOrder/workOrderList" \
  --data-urlencode "referer=mgrapi" \
  --data-urlencode "token=$FIRE_TOKEN" \
  --data-urlencode "page=1" \
  --data-urlencode "pageSize=20" \
  --data-urlencode "beginTime=<START_UNIX_SECONDS>" \
  --data-urlencode "endTime=<END_UNIX_SECONDS>" \
  --data-urlencode "processStatusInt=0" \
  --data-urlencode "typeInt=1" \
  --data-urlencode "productIdInt=1"
```
