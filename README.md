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
| [`fire-skill`](fire-skill/) | 查询火花派员工后台的客服工单，支持时间范围、状态、类型、产品等筛选。 | "查客服工单"、"火花派工单查询" |

## 目录结构

```text
.
├── fhd-skill/                 # 风火递打印与发货记录查询
│   └── SKILL.md
└── fire-skill/                # 火花派客服工单查询
    ├── SKILL.md
    ├── references/            # 工单 API 文档和结果模板
    └── .gitignore             # 忽略本地 Token 文件
```

各 Skill 的 API 配置和使用流程请参阅其对应的 `SKILL.md`。
