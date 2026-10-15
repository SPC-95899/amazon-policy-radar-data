# 亚马逊政策雷达 数据源

为 https://us-amz-xingui.grok.me/ 提供公开 JSON 数据（美国 / 加拿大 / 墨西哥 / 欧洲 亚马逊卖家政策）。浏览器可直接 fetch：

- `https://raw.githubusercontent.com/SPC-95899/amazon-policy-radar-data/main/policies.json`
- `https://raw.githubusercontent.com/SPC-95899/amazon-policy-radar-data/main/updates.json`
- `https://raw.githubusercontent.com/SPC-95899/amazon-policy-radar-data/main/meta.json`

## policies.json（数组）
| 字段 | 说明 |
|---|---|
| id | 唯一标识 |
| title | 中文标题 |
| regions | 站点数组：US / CA / MX / EU / GLOBAL |
| category | 分类（费用、保险、合规……） |
| status | 已生效 / 即将生效 / 灰度推广 / 政策说明 |
| announced | 发布日期 YYYY-MM-DD |
| effective | 生效日期 YYYY-MM-DD |
| zh | 中文翻译 |
| orig | 原文摘录 |
| plain | 说人话，三行：这是啥 / 跟你有没有关系 / 你要做的事 |
| outlook | 展望（推测）：服务于谁（政府/消费者/亚马逊）+ 库内相关政策 |
| sourceName / sourceUrl | 来源名称与链接 |

## updates.json（数组，每天一条，最新在前）
`date`、`status`（完成 / 未完成）、`finishedAt`（北京时间 `YYYY-MM-DD HH:mm`，未完成可为 null）、`summary`（说明或未完成原因）、`addedCount`、`items`（`[{id,title}]`）。

## meta.json
`lastRunAt`、`lastRunStatus`、`schemaVersion`。

每日更新流程见 [update.md](update.md)。
