# 共享 API 规则

在调用任一火花派接口前，先遵循本文件；接口路径、筛选参数和结果展示以各功能文档为准。

## Token 生命周期

Token 的有效期约为一天。将 Token 保存为当前聊天项目中 `fire-skill/.token` 的唯一内容，并以该文件的最后修改时间作为获取时间。

每次调用前执行以下流程：

1. 检查 `fire-skill/.token` 是否存在，以及最后修改时间是否在 23 小时内。
2. 文件缺失或已超过 23 小时时，在聊天中提示用户：“请提供火花派 API Token（有效期约一天）”。
3. 收到 Token 后，去除首尾空白并覆盖写入 `fire-skill/.token`；文件修改时间随即成为新的获取时间。
4. 将文件内容赋给当前请求的 `FIRE_TOKEN` 变量，并调用接口。

不要复述、确认、展示或记录用户输入的 Token。Token 只能用于当前火花派接口请求，不得写入结果、文档、命令输出或其他项目文件。

```bash
FIRE_TOKEN=$(cat "fire-skill/.token")
```

接口返回 HTTP 401、`scode=6` 或 `code="AUTH_FAILED"` 时，视为 Token 提前失效：重新在聊天中索取 Token，覆盖文件后仅重试原请求一次。第二次仍失败时，报告鉴权失败，不要继续重试。

## 请求规范

- 请求方法、`Content-Type` 和参数传递方式以各功能接口文档为准；不要将当前功能使用的 `GET` 视为全局约束。
- 每次请求必须按接口文档携带 `referer=mgrapi`、`token` 及所需的分页参数。
- 使用 `GET` 时，将参数放在 URL query string，并以独立的 `--data-urlencode` 编码。

现有多个查询接口使用 `GET`，调用形式如下：

```bash
curl -s -G "<ENDPOINT>" \
  --data-urlencode "referer=mgrapi" \
  --data-urlencode "token=$FIRE_TOKEN" \
  ...接口定义的其他参数
```

## 响应与错误处理

分页接口的成功响应通常包含 `page`、`pageSize`、`total` 和 `list`；非分页接口以其功能文档定义的响应结构为准。先检查以下情况，再格式化结果：

| 场景 | 判断 | 处理 |
|------|------|------|
| Token 缺失或过期 | `.token` 缺失或修改时间超过 23 小时 | 在聊天中索取新的 Token 并覆盖保存 |
| Token 失效 | HTTP 401、`scode=6` 或 `code="AUTH_FAILED"` | 在聊天中索取新的 Token，覆盖保存后仅重试一次 |
| 参数错误 | `code="PARAM_ERROR"` | 说明无效参数并请用户确认筛选条件 |
| 网络失败 | curl 非零退出或超时 | 告知连接失败，不编造结果 |
| 非 JSON 响应 | 无法解析为 JSON | 仅说明接口返回异常，不输出可能含敏感信息的原始响应 |
| 空结果 | `list` 为空且 `total=0` | 使用对应结果模板的空结果提示 |

## 分页

仅对分页接口使用本节规则：默认使用功能文档给出的 `page` 与 `pageSize`，每次响应后计算 `ceil(total / pageSize)`，并在结果末尾说明当前页、总页数和总记录数。用户要求查看更多时，沿用原筛选条件，仅修改 `page`。
