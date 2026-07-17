---
name: fire-skill
description: "火花派员工后台系统 - 查询工单等工作信息。Fire company employee backend system."
allowed-tools: [Read, Write, Bash]
---

# 火花派（Fire）员工后台 Skill

## 一、技能概述

火花派是火花公司的员工后台系统。本 skill 支持以下功能：

1. **工单查询** — 分页查询客服工单，支持时间范围、处理状态、工单类型、产品、问题标签、用户昵称、问题内容等筛选

后续将扩展更多功能。

**Token 配置方式**：Token 存储在 `fire-skill/.token` 文件中（纯文件方式），每次 API 调用时从文件读取。

---

## 二、Token 管理

### Token 存储位置

```
fire-skill/.token
```

文件内容仅包含 token 字符串本身，无换行、无引号、无其他字符。

### 首次使用 / Token 文件不存在时

当检测到 `fire-skill/.token` 文件不存在时，提示用户在聊天界面输入 API Token。用户提供后，将其写入 `.token` 文件。

### 每次 API 调用前

1. 检查 `fire-skill/.token` 文件是否存在
2. 如文件不存在 → 按"首次使用"流程提示用户
3. API 调用时使用 `$(cat fire-skill/.token)` 读取文件内容并拼入 curl 命令

### Token 失效处理

如果 API 返回认证失败（`scode=6` 或 `code="AUTH_FAILED"` 或 HTTP 401）：
→ 提示用户："Token 已失效，请提供新的 Token"
→ 用户提供新 token 后，覆盖写入 `.token` 文件
→ `.gitignore` 已配置忽略该文件，确保不会被提交到 git

---

## 三、API 调用规范

### 基础配置

- **基础 URL**：`https://xyymgrapi.fhd001.com/mgr/cs/workOrder/`
- **请求方式**：GET（所有参数通过 URL query string 传递）
- **必带参数**：`referer=mgrapi`、`token=...`
- **分页参数**：`page`、`pageSize`

### curl 调用模板

```bash
TOKEN=$(cat fire-skill/.token)
curl -s -G "<BASE_URL><ENDPOINT>" \
  --data-urlencode "referer=mgrapi" \
  --data-urlencode "token=$TOKEN" \
  --data-urlencode "page=<PAGE>" \
  --data-urlencode "pageSize=<PAGESIZE>" \
  ...其他可选参数
```

**重要**：每个参数必须使用独立的 `--data-urlencode` 传递。Token 从文件读取，由 shell 变量替换，token 值不会出现在输出中。

### 响应格式

**正常响应**（分页结构）：
```json
{
  "page": 1,
  "pageSize": 10,
  "total": 150,
  "list": [ /* 记录数组 */ ]
}
```

**错误响应**（可能为以下格式之一）：
```json
{"scode": 6, "errorMsg": "认证失败"}
{"code": "AUTH_FAILED", "message": "Token无效"}
{"code": "PARAM_ERROR", "message": "参数错误"}
```

---

## 四、通用错误处理

| 错误场景 | 判断依据 | 处理方式 |
|----------|----------|----------|
| Token 文件不存在 | `fire-skill/.token` 文件不存在 | 按第二章"首次使用"流程，提示用户提供 Token 并写入文件 |
| Token 失效 | API 返回 `scode=6` 或 `code="AUTH_FAILED"` 或 HTTP 401 | 提示用户"Token 已失效，请提供新的 Token"，接收后覆盖写入 `.token` 文件 |
| 参数错误 | API 返回 `code="PARAM_ERROR"` | 提示用户检查输入参数（时间范围、筛选条件等）是否正确 |
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

## 五、交互流程

### 显式调用

```
1. 检查 token 文件是否存在
2. 展示菜单让用户选择：
   "火花派员工后台，当前支持以下功能：
   1. 查询客服工单
   请回复 1"
3. 根据用户选择进入对应子功能流程
4. 按子功能文档执行查询 → 格式化输出
```

### 隐式调用（关键词触发）

当用户消息中包含"工单"、"火花派"、"客服工单"、"工单查询"、"fire"等关键词时：
- 直接进入工单查询流程
- 按 `workorder/` 子功能文档执行

---

## 六、功能索引

| 功能 | 文档路径 | 触发关键词 |
|------|----------|------------|
| 工单查询 | [workorder/](workorder/) | 工单、客服工单、工单查询、火花派工单 |
