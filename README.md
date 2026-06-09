# fhd-skill

风火递(FHD)打单软件 Claude Code 自定义 Skill。

## 功能

- 查询打印记录（支持按时间范围、平台、运单号、收件人等筛选）
- 查询发货记录（支持按时间范围、平台、运单号、收件人等筛选）

## 安装

```bash
npx skills add cycallj/fhd-skill -g
```

## 使用

安装后，在 Claude Code 中直接说：

- "帮我查风火递的打印记录"
- "查一下上个月的发货记录"
- 或使用 `/fhd-skill` 命令

首次使用需要配置 API Token，Skill 会自动引导完成。
