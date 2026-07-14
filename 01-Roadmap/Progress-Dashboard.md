# 📊 Senior Roadmap 进度看板

## 🎯 总览

```dataview
TABLE WITHOUT ID
  length(rows) AS "数量",
  status AS "状态"
FROM "02-Tech-Notes"
GROUP BY status
```

## 🟢 Tier 1 进度

```dataview
TABLE tech, topic, difficulty, status
FROM "02-Tech-Notes"
WHERE tier = "Tier-1"
SORT tech ASC
```

## 🔥 今日该复习

```dataview
LIST
FROM "02-Tech-Notes"
WHERE contains(string(file.frontmatter.review-1), string(date(today)))
   OR contains(string(file.frontmatter.review-2), string(date(today)))
   OR contains(string(file.frontmatter.review-3), string(date(today)))
```

## 📚 LeetCode 累计

```dataview
TABLE WITHOUT ID
  length(rows) AS "已刷"
FROM "03-LeetCode"
WHERE status = "solved"
```

## 🏷️ 按主题分布

```dataview
TABLE WITHOUT ID
  topic AS "主题",
  length(rows) AS "题数"
FROM "03-LeetCode"
GROUP BY topic
SORT length(rows) DESC
```