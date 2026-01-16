---
dg-publish: true
dg-home: true
title: "@sandmrk 的 AI Art Gallery"
description: 精选 AI 生成艺术、Midjourney / Flux / SD 提示词、工具合集与灵感瞬间。深色调卡片画廊，探索无限创意。
cssclasses:
  - gallery-home
  - dark-mode
---
# @sandmrk 的 AI Art Gallery

欢迎来到我的个人 AI 艺术与提示词收藏库  
这里汇集了从 X 上精选的生成艺术瞬间、提示词实验、工具推荐与设计灵感  
所有作品以深色调卡片呈现，支持标签浏览与搜索。

（英雄横幅或简介图片可放这里，例如：）

![[site-hero.jpg|banner]]  <!-- 如果有大图就放，宽屏展示 -->

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
```

## 按主题快速浏览

- [[工具|🛠️ 工具 & Skills]]  
- [[提示词|✨ 提示词合集]]  
- [[AI艺术|🎨 AI 生成艺术]]  
- [[灵感|💡 设计灵感 & 金句]]  

## 所有标签云

```dataview
LIST WITHOUT ID
  tag + " (" + length(rows) + ")"
FLATTEN file.etags AS tag
WHERE dg-publish = true
GROUP BY tag
SORT length(rows) DESC
```


### 搜索提示
右上角搜索框支持关键词查询，例如：  
**cyberpunk Midjourney v6 人像 写实 赛博朋克 工具 Claude Skills**

**Created with ❤️ by @sandmrk**  
Powered by Obsidian + Digital Garden
