---
name: fhd-skill
description: "风火递(FHD)打单软件 - 查询打印记录和发货记录。FHD shipping software for querying print logs and send logs. Use when user mentions 风火递/FHD, print records/打印记录, shipment records/发货记录, waybill query/运单号查询, courier tracking, or logistics logs. Supports filtering by time range, platform, recipient, and order ID."
allowed-tools: [Read, Write, Bash]
---

# 风火递（FHD）打单软件 Skill

## 一、功能概述

本 skill 支持以下功能：

1. **查询打印记录** — 分页查询打印日志，支持时间范围、平台等筛选
2. **查询发货记录** — 分页查询发货记录，支持时间范围、平台等筛选

**Token 配置方式**：使用环境变量 `$FHD_TOKEN`（优先），持久化文件 `.claude/fhd-config.json` 仅用于跨会话恢复，API 调用时不从文件读取。

---

## 二、Token 管理流程

> **安全原则**：Claude 只负责检查 token 是否存在和使用它调用 API，绝不代为设置或存储 token 明文值。token 的设置和持久化由用户自行在终端完成。

### 首次使用 / Token 未配置时

当检测到 `$FHD_TOKEN` 环境变量未设置时，提示用户手动执行以下步骤：

```bash
# 1. 在终端设置环境变量（Claude 不会代为执行此命令）
export FHD_TOKEN="您的API Token"

# 2. （可选）持久化到文件，便于下次会话恢复
mkdir -p .claude && echo '{"token":"'"$FHD_TOKEN"'"}' > .claude/fhd-config.json
```

用户完成后告知 Claude 继续，Claude 不会看到 token 明文值。

### 每次 API 调用前

1. 使用 Bash 检查环境变量：
   `[ -n "$FHD_TOKEN" ] && echo "Token已设置 (${FHD_TOKEN:0:4}****)" || echo "未设置"`
2. 如未设置 → 检查 `.claude/fhd-config.json` 是否存在且可读，提示用户参考该文件设置环境变量
3. API 调用时使用 `$FHD_TOKEN` 引用环境变量，token 值由 shell 展开，不会出现在 Claude 的任何命令输出中

### Token 失效处理

如果 API 返回认证失败（`scode=6` 或 `code="AUTH_FAILED"` 或 HTTP 401）：
→ 提示用户："Token 已失效，请在终端重新设置：export FHD_TOKEN='新的Token'"
→ Claude 不执行 `unset`、`rm` 等操作，由用户自行处理

---

## 三、API 调用规范

### 基础配置

- **基础 URL**：`https://saasapi.fhd001.com/print/api/print/`
- **Content-Type**：`application/x-www-form-urlencoded`
- **请求方式**：POST
- **Token 传递**：所有请求都需要在参数中传递 `token` 字段

### curl 调用模板

```bash
curl -s -X POST "<BASE_URL><ENDPOINT>" \
  --data-urlencode "token=$FHD_TOKEN" \
  --data-urlencode "startTime=<UNIX_TIMESTAMP>" \
  --data-urlencode "endTime=<UNIX_TIMESTAMP>" \
  --data-urlencode "page=<PAGE>" \
  --data-urlencode "pageSize=<PAGESIZE>" \
  ...其他可选参数
```

**安全说明**：`$FHD_TOKEN` 为环境变量引用，curl 执行时由 shell 自动替换，token 值不会出现在 Claude 的任何输出中。

**重要**：每个参数必须使用独立的 `--data-urlencode` 传递。

### 响应格式

**正常响应**（分页结构）：
```json
{
  "list": [ /* 记录数组 */ ],
  "page": 1,
  "pageSize": 10,
  "total": 150
}
```

**错误响应**（可能为以下格式之一）：
```json
{"scode": 6, "errorMsg": "认证失败"}
{"code": "AUTH_FAILED", "message": "Token无效"}
{"code": "PARAM_ERROR", "message": "参数错误"}
```

---

## 四、打印记录查询

### API 端点

`POST queryPrintLogV2.do`

### 请求参数

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|:----:|--------|------|
| `token` | String | 是 | — | 认证 token |
| `startTime` | int | 是 | — | 开始时间（Unix 秒级时间戳） |
| `endTime` | int | 是 | — | 结束时间（Unix 秒级时间戳） |
| `page` | int | 否 | `1` | 页码（从1开始） |
| `pageSize` | int | 否 | `10` | 每页条数 |
| `type` | int | 否 | `0` | 打印类型 |
| `printType` | int | 否 | — | 打印方式（见枚举表） |
| `expressType` | int | 否 | — | 快递类型（见枚举表） |
| `platform` | String | 否 | — | 平台标识（source key），如 `fxg`、`ks`、`pdd` |
| `platformKey` | String | 否 | — | 店铺 ID |
| `wlbCode` | String | 否 | — | 运单号/物流单号 |
| `orderId` | String | 否 | — | 订单 ID |
| `consigneeName` | String | 否 | — | 收件人姓名 |
| `consigneeMobile` | String | 否 | — | 收件人手机号 |
| `operator` | String | 否 | — | 操作人 |
| `sort` | String | 否 | `createTime` | 排序字段 |
| `desc` | String | 否 | `true` | 是否倒序 |

### 用户交互流程

1. **检查 token**（按第二章流程）
2. **询问时间范围**：提示用户输入查询时间范围（如"最近7天"、"2026-06-01到2026-06-09"）
   - Claude 将自然语言时间转换为 Unix 秒级时间戳
   - 默认时间范围：最近 7 天
3. **可选筛选**：询问是否需要按 platform（平台）筛选
   - 如果用户指定了平台名称，查阅第六章枚举表找到对应 source key
4. **调用 API**：使用 Bash 执行 curl 命令
5. **格式化输出**：参考第七章模板展示结果

### 时间转换说明

将用户输入的时间转换为 Unix 秒级时间戳：
- 使用 Bash 命令 `date -d "<日期字符串>" +%s` 计算时间戳
- 如果当前系统不支持 GNU date，使用其他方式计算

---

## 五、发货记录查询

### API 端点

`POST querySendLogV2.do`

### 请求参数

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|:----:|--------|------|
| `token` | String | 是 | — | 认证 token |
| `startTime` | int | 是 | — | 开始时间（Unix 秒级时间戳） |
| `endTime` | int | 是 | — | 结束时间（Unix 秒级时间戳） |
| `page` | int | 否 | `1` | 页码（从1开始） |
| `pageSize` | int | 否 | `10` | 每页条数 |
| `platform` | String | 否 | — | 平台标识（source key），如 `fxg`、`ks`、`pdd` |
| `platformKey` | String | 否 | — | 店铺 ID |
| `outSid` | String | 否 | — | 运单号 |
| `orderId` | String | 否 | — | 订单 ID |
| `consigneeName` | String | 否 | — | 收件人姓名 |
| `province` | String | 否 | — | 收件人省份 |
| `companyCode` | String | 否 | — | 快递公司编码 |
| `companieName` | String | 否 | — | 快递公司名称 |
| `sort` | String | 否 | `sendTime` | 排序字段 |
| `desc` | String | 否 | `false` | 是否倒序 |

### 用户交互流程

同打印记录查询流程（第四章节），API 路径替换为 `querySendLogV2.do`，排序默认字段为 `sendTime`。

---

## 六、枚举查询表

### PrintType 打印方式枚举

| code | 说明 |
|:----:|------|
| 1 | 自主打印 |
| 2 | 代他人打印 |
| 3 | 他人代打 |
| 4 | 第三方代打 |
| 5 | 打印码打印 |
| 6 | 帮助分享打印 |
| 7 | 快递补打 |
| 8 | 快递公司打单 |
| 9 | 菜鸟云打印 |
| 10 | 自动打印 |
| 11 | 远程打印（小程序→桌面端） |

### ExpressType 快递类型枚举（常用）

| code | 说明 |
|:----:|------|
| 0 | 不打印 |
| 1 | 普通快递面单 |
| 2 | 网点电子面单 |
| 3 | 菜鸟电子面单 |
| 7 | 拼多多店铺电子面单 |
| 17 | 字节跳动电子面单 |
| 20 | 快手电子面单 |
| 27 | 小红书电子面单 |
| 30 | 视频号小店电子面单 |
| 39 | 京东快递 |
| 57 | 抖音即时零售 |

### FhdPlatform 常用电商平台枚举

> **注意**：两个 API 的 `platform` 参数都使用 **source key**（字符串），不是 int 编码。

| code | source | 名称 |
|:----:|--------|------|
| 1 | `pdd` | 拼多多 |
| 2 | `youzan` | 有赞 |
| 3 | `1688` | 阿里巴巴 |
| 4 | `jd` | 京东 |
| 6 | `wd` | 微店 |
| 8 | `tb` | 淘宝 |
| 9 | `tm` | 天猫 |
| 31 | `youzanyun` | 有赞云 |
| 33 | `ks` | 快手 |
| 34 | `fxg` | 抖店 |
| 35 | `ktt` | 快团团 |
| 36 | `c2m` | 淘工厂 |
| 43 | `xhs` | 小红书 |
| 45 | `ec` | 视频号小店 |
| 59 | `jdwl` | 京东物流 |
| 62 | `meituan` | 美团 |
| 68 | `goo_fish` | 闲鱼 |
| 123 | `fxgjsls` | 抖音即时零售 |

### FhdPlatform 自营平台枚举

| code | source | 名称 |
|:----:|--------|------|
| 0 | `fhd_normal` | 风火递 |
| 32 | `fhdant` | 风火蚁 |
| 39 | `fth` | 风天河 |
| 99 | `fhd` | 风火递 |
| 100 | `fhk` | 库小存 |
| 101 | `ant` | 商家版风火蚁 |
| 103 | `foo` | 开放平台自有商城 |
| 114 | `fhdwx_kd` | 风火递快递版 |
| 115 | `fhdwx_sy` | 风火递生意版 |
| 117 | `fhdxd` | 风火递小店 |
| 118 | `fhdmall` | 云打印商城 |

---

## 七、结果格式化模板

### 打印记录表格

展示字段（按优先级，根据返回数据可用性调整）：

| printId | 打印时间 | 收件人 | 运单号 | 打印方式 | 快递类型 | 快递公司 | 平台店铺 | 操作人 |
|---------|----------|--------|--------|----------|----------|----------|----------|--------|

- **打印时间**：将 `createTime`（Unix 时间戳）转换为可读格式
- **打印方式**：将 `printType` 编码转换为中文名称（查枚举表）
- **快递类型**：将 `expressType` 编码转换为中文名称（查枚举表）
- **快递公司**：显示 `cpCode`（快递公司编码）
- **平台店铺**：显示 `platformName`（平台店铺名称），如无则显示 `platform` source key

### 发货记录表格

展示字段：

| sendId | 发货时间 | 运单号 | 收件人 | 省份 | 快递公司 | 公司名称 | 平台店铺 |
|--------|----------|--------|--------|------|----------|----------|----------|

- **发货时间**：将 `sendTime`（Unix 时间戳）转换为可读格式
- **运单号**：`outSid`
- **收件人**：`receiverName`
- **省份**：`receiverProvince`
- **快递公司**：`companyName`
- **公司名称**：无直接字段，可省略
- **平台店铺**：`platformName`

### 分页摘要

每次查询结束后输出：
```
当前第 X 页 / 共 Y 页，共 Z 条记录
```
其中：Y = ceil(total / pageSize)

如果用户需要查看更多，提示可以指定页码继续查询。

### 时间戳转换

使用以下方式将 Unix 时间戳转为可读日期：
- Python（如可用）：`python -c "import datetime; print(datetime.datetime.fromtimestamp(<TS>).strftime('%Y-%m-%d %H:%M:%S'))"`
- 或在结果表格中直接展示 `YYYY-MM-DD HH:MM:SS` 格式

---

## 八、错误处理

| 错误场景 | 判断依据 | 处理方式 |
|----------|----------|----------|
| Token 未配置 | `$FHD_TOKEN` 环境变量未设置且 `.claude/fhd-config.json` 不存在 | 按第二章"首次使用"流程，提示用户在终端自行设置环境变量 |
| Token 失效 | API 返回 `scode=6` 或 `code="AUTH_FAILED"` 或 HTTP 401 | 提示用户"Token 已失效，请在终端更新环境变量" |
| 参数错误 | API 返回 `code="PARAM_ERROR"` | 提示用户检查输入参数（时间范围、平台等）是否正确 |
| 网络超时/连接失败 | curl 返回非零退出码或超时 | 提示"网络连接失败，请检查网络后重试" |
| 空结果 | `list` 为空数组且 `total=0` | 提示"查询时间范围内无记录，请调整查询条件" |
| JSON 解析失败 | curl 返回内容不是有效 JSON | 输出原始响应内容，提示"API 返回异常，请联系管理员" |

### 通用错误处理逻辑

curl 命令执行后：
1. 检查 curl 退出码（`$?`）— 非零则网络错误
2. 检查响应是否为有效 JSON
3. 检查是否包含 `scode` 或 `code` 错误字段
4. 按上表处理对应错误场景
5. 无错误则正常格式化输出

---

## 九、交互流程

### 显式调用（`/fhd-skill` 或直接提及 skill 名）

```
1. 检查 token
2. 展示菜单让用户选择：
   "请问您需要：
   1. 查询打印记录
   2. 查询发货记录
   请回复 1 或 2"
3. 根据用户选择进入对应流程
4. 询问时间范围（默认最近7天）
5. 可选：询问 platform 筛选
6. 调用 API → 格式化输出
```

### 隐式调用（关键词触发）

当用户消息中包含"打印记录"、"发货记录"、"风火递"等关键词时：
- 如果明确提到了"打印"→ 直接进入打印记录查询流程
- 如果明确提到了"发货"→ 直接进入发货记录查询流程
- 如果模糊不清 → 展示菜单让用户选择
- 跳过菜单，直接询问时间范围
- 执行 API 调用并输出结果
