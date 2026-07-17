# 火花派（Fire）Skill

火花派员工后台系统 skill，提供工单查询等工作信息查询能力。

## 安装

1. 将 `fire-skill/` 目录放置在项目目录下
2. 确保 skill 已在所在 agent 平台中注册
3. 首次使用时，agent 会提示输入 API Token，或手动创建 `.token` 文件：

```bash
echo "您的API Token" > fire-skill/.token
```

> `.token` 文件已在 `.gitignore` 中配置忽略，不会被提交到版本控制系统。

## 使用

触发关键词：

- **工单查询**：说"工单"、"客服工单"、"工单查询"、"火花派工单"等

## 功能

| 功能 | 说明 |
|------|------|
| 工单查询 | 分页查询客服工单，支持时间范围、状态、类型、产品等多维筛选 |

## 文件结构

```
fire-skill/
├── SKILL.md              # 核心 skill 文件
├── README.md             # 本文件
├── .gitignore            # 忽略 .token 文件
├── .token                # API Token（不入 git，需自行创建）
└── workorder/
    ├── README.md         # 工单查询接口文档
    └── format_template.md # 结果排版模板
```
