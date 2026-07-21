# FHD Skills

风火递团队的 Agent Skills 集合。仓库中的每个一级目录都是可独立安装的 Skill。

## 安装

```bash
npx skills add cycallj/fhd-skill
```

命令会列出仓库内可用的 Skills，请按交互提示选择需要安装的项目。

## Skills

| Skill | 用途 | 常见触发语句 |
| --- | --- | --- |
| [`fhd-skill`](fhd-skill/) | 查询风火递的打印记录和发货记录，支持时间范围、平台、运单号、收件人等筛选。 | "查风火递打印记录"、"查上个月的发货记录" |
| [`fire-skill`](fire-skill/) | 查询火花派后台的客服工单、弹幕捕手企微客户及多产品用户基本信息。 | "查客服工单"、"查企微客户"、"查通用版用户" |

## 目录结构

```text
.
├── fhd-skill/                 # 风火递打印与发货记录查询
│   └── SKILL.md
└── fire-skill/                # 火花派工单、企微客户与用户信息查询
    ├── SKILL.md
    └── references/            # 工单、企微客户和多产品用户信息文档
```

各 Skill 的 API 配置和使用流程请参阅其对应的 `SKILL.md`。
