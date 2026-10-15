# 每日更新流程（Daily Run）

所有时间用北京时间（Asia/Shanghai）。`TODAY=$(TZ=Asia/Shanghai date +%F)`。

## 0. 准备
1. `cd /workspace/amazon-policy-radar-data && git pull --ff-only origin main`（目录不存在则 `gh repo clone SPC-95899/amazon-policy-radar-data`）。
2. 读取 `policies.json`、`updates.json`、`meta.json`，记下已有 `id`、`sourceUrl`、标题，以及 updates.json 中最新日期 `LAST`。

## 1. 检索新政策（US / CA / MX / EU）
覆盖从 `LAST` 前 7 天到今天的公告。至少搜索：
- Seller Central 新闻/帮助：sellercentral.amazon.com（US）、.ca、.com.mx、sellercentral-europe.amazon.com 及各欧洲站；"Seller News"、"Selling Partner announcements"、费用变更页面（referral / FBA fees）。
- sellingpartners.aboutamazon.com、aboutamazon.com 新闻。
- 卖家论坛 sellercentral.amazon.*/seller-forums 的官方 Announcement。
- 政府/监管来源（如 FTC、CBSA、SAT、欧盟/各国税务机关）在涉及亚马逊卖家时。
- 可信行业新闻（Marketplace Pulse、Seller Magnet、sellerkit 等）仅用于发现线索，尽量追溯到官方原文。

## 2. 筛选与去重
仅当同时满足才入库：
- 有明确日期（至少 `announced`；`effective` 未知时如实说明，不编造）；
- 有真实可打开的来源链接（用 curl/WebFetch 验证，非 404）；
- 与库内已有条目不是同一件事：比对 `sourceUrl`、标题关键词、站点 + 生效日期。已有条目若只是日期/细节更新，修改原条目而不是新增。

## 3. 编写条目（字段与现有一致）
`id`（小写短横线，含年份/日期，唯一）、`title`、`regions`、`category`（优先复用已有分类）、`status`（已生效/即将生效/灰度推广/政策说明，按今天与 effective 比较）、`announced`、`effective`、`zh`、`orig`（原文引用）、`plain`、`outlook`、`sourceName`、`sourceUrl`。
- `plain` 必须正好三行，用 `\n` 分隔：
  `这是啥：……` / `跟你有没有关系：……` / `你要做的事：……`
- `outlook`：说明这项政策服务于谁（政府 / 消费者 / 亚马逊），推测内容以"推测"标明；并点名库内相关政策（按标题）。
- 新条目插入 `policies.json` 开头。同时检查已有条目的 status 是否因到期需改为"已生效"。

## 4. 写日志（无论成败，必须执行）
1. 若 `LAST` 与今天之间有缺失日期，每天补一条：`{"date":D,"status":"未完成","finishedAt":null,"summary":"当天任务未运行，无记录（补记）","addedCount":0,"items":[]}`。
2. 追加今天一条（若今天已存在则更新）：
   `{"date":TODAY,"status":"完成"或"未完成","finishedAt":"$(TZ=Asia/Shanghai date '+%F %H:%M')","summary":"检索范围/结果；未完成必须写原因","addedCount":N,"items":[{"id","title"}]}`。
   没找到新政策也算"完成"，addedCount 为 0。
3. `updates.json` 按 date 倒序，每天仅一条。
4. `meta.json`：`lastRunAt` = 当前北京时间 ISO（如 `2026-10-15T09:03:00+08:00`），`lastRunStatus` = 今天的状态，`schemaVersion` 不变。

## 5. 校验、提交、推送
1. `for f in policies.json updates.json meta.json; do python3 -m json.tool $f >/dev/null || exit 1; done`；检查 id 唯一、plain 三行。
2. （首次在新机器上先运行 `gh auth setup-git`，否则 HTTPS 推送会要用户名）`git add -A && git commit -m "daily: $TODAY (+N)" && git push origin main`。
3. 推送失败：先 `git pull --rebase` 再推；仍失败则日志状态为"未完成"并报告原因。

## 6. 验证线上
`curl -s "https://raw.githubusercontent.com/SPC-95899/amazon-policy-radar-data/main/updates.json?t=$(date +%s)" | python3 -c "import json,sys;print(json.load(sys.stdin)[0]['date'])"` 应输出 TODAY（raw 有约 5 分钟 CDN 缓存，不一致时等待重试）。同样检查 meta.json 的 lastRunAt。
