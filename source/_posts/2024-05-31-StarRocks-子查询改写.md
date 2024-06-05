---
title: 子查询改写
date: 2024-05-31 17:34:10
updated: 2024-05-31 17:34:10
categories: DataBase
tags: [SQL]
description: 子查询改写
---

```sql
-- sub query 
SELECT DISTINCT user_no
FROM cos.op_h5_landing_page_request_record t1
WHERE date_updated > '2024-05-30' 
  AND date_updated < '2024-05-31'
  AND user_no IN (SELECT cast(user_id AS VARCHAR)
                    FROM users_u
                  WHERE created > '2024-05-28');
                  
-- left join
SELECT DISTINCT t1.user_no,
                t2.id
FROM cos.op_h5_landing_page_request_record t1
LEFT JOIN users_u t2
       ON t2.created > '2024-05-28'
      AND t1.user_no=cast(t2.user_id AS VARCHAR)
WHERE t1.date_updated > '2024-05-30'
  AND t1.date_updated < '2024-05-31'
  AND t2.id IS NOT NULL;


-- 相关子查询
SELECT DISTINCT t1.user_no
FROM cos.op_h5_landing_page_request_record t1
WHERE exists (SELECT cast(user_id AS VARCHAR)
             FROM users_u t2
             WHERE t2.created > '2024-05-28'
             AND t1.user_no=cast(t2.user_id AS VARCHAR))
   AND  t1.date_updated > '2024-05-30'
   AND t1.date_updated < '2024-05-31';
```

StarRocks 中子查询和相关子查询都被改写成 `LEFT SEMI JOIN`，而 `LEFT JOIN` 被改写成 `INNER JOIN`。
