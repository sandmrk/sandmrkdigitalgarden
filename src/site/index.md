---
dg-publish: true
dg-home: true                     # ★设为 true，成为网站根路径首页
title: "@sandmrk 的 AI Art Gallery"
description: 精选 AI 生成艺术、Midjourney/Flux/SD 提示词、工具合集与灵感瞬间。深色调卡片画廊，探索无限创意。
banner: "./attachments/site-banner.jpg"   # 可选：自定义站点顶部大横幅（放一张代表性 AI 大图）
cssclasses:
  - gallery-home
  - dark-mode
  - masonry-grid
---

# @sandmrk 的 AI Art Gallery

欢迎来到我的个人 AI 艺术与提示词收藏库。  
这里汇集了从 X 上精选的生成艺术瞬间、提示词实验、工具推荐和设计灵感。  
所有作品均以深色调卡片呈现，支持标签浏览与搜索。

![[site-hero.jpg|banner]]  <!-- 如果有站点英雄图/横幅，可放这里，宽屏大图 -->

## 快速探索方式

- **最新收藏**：下方实时更新 12 篇
- **热门标签**：点击直接过滤
- **搜索**：右上角搜索框（DG 自带全文搜索）

## 最新入库作品（Masonry 画廊风格）

```dataview
TABLE WITHOUT ID
  ("![" + cover + "]( " + cover + " )") AS "预览",
  file.link AS "标题",
  author AS "作者/来源",
  dateformat(created, "yyyy-MM-dd") AS "日期",
  tags AS "标签"
FROM "X Posts" OR "AI-Art" OR "提示词" OR "工具"
WHERE dg-publish = true AND cover
SORT created DESC
LIMIT 24


---
dg-publish: true
dg-home: true
title: "@sandmrk 的 AI Art Gallery"
description: 精选 AI 生成艺术、Midjourney / Flux / SD 提示词、工具合集与灵感瞬间。深色调卡片画廊，探索无限创意。
banner: "./attachments/site-hero.jpg"   # 可选：替换成你自己的站点横幅图路径或 URL
cssclasses:
  - gallery-home
  - dark-mode
  - masonry-like
---

# @sandmrk 的 AI Art Gallery

欢迎来到我的个人 AI 艺术与提示词收藏库  
这里汇集了从 X 上精选的生成艺术瞬间、提示词实验、工具推荐与设计灵感  
所有作品以深色调卡片呈现，支持标签浏览与搜索

<div class="hero-banner">
  ![[site-hero.jpg|banner]]   <!-- 如果有大横幅图就放这里，宽屏展示 -->
</div>

## 最新入库作品

```dataview
TABLE WITHOUT ID
  ("![" + cover + "](" + cover + ")") AS "预览",
  file.link AS "标题",
  author AS "作者",
  dateformat(created, "yyyy-MM-dd") AS "日期",
  join(tags, " • ") AS "标签"
FROM "X Posts" OR "提示词" OR "AI艺术" OR "工具"
FLATTEN cover
WHERE dg-publish = true AND cover
SORT created DESC
LIMIT 24

---

### 搜索提示
右上角搜索框支持关键词查询，例如输入：  
**cyberpunk Midjourney v6 人像 写实 赛博朋克 工具 Claude Skills**

快速发现更多灵感！

**Created with ❤️ by @sandmrk**  
Powered by Obsidian + Digital Garden

