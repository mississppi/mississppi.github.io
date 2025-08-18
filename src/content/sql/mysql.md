---
title: "mysql"
slug: "mysql"
order: 1
source: "sql"
---

わざわざググらないよう基本構文

```
INSERT INTO テーブル名 (列1, 列2, ...)
VALUES (値1, 値2, ...);

UPDATE テーブル名
SET 列1 = 値1, 列2 = 値2, ...
WHERE 条件;

DELETE FROM テーブル名
WHERE 条件;


CREATE USER 'new_user'@'localhost' IDENTIFIED BY 'my_password';


```

サンプル - 販売実績ダッシュボード

```
SELECT
    category,
    COUNT(DISTINCT product_name) as unique_products,
    COUNT(*) as total_transactions,
    SUM(quantity) as total_quantity,
    ROUND(AVG(quantity), 1) as avg_quantity_per_sale,
    ROUND(MIN(unit_price), 2) as min_price,
    ROUND(MAX(unit_price), 2) as max_price,
    ROUND(SUM(quantity * unit_price), 2) as total_revenue
FROM sales
GROUP BY category
ORDER BY total_revenue DESC;
```

・各カテゴリないのユニークな商品をカウントする。DISTINCT
・包括的な売上
・MAX、MIN、平均

```
SELECT
    region,
    COUNT(DISTINCT category) as categories_sold,
    COUNT(DISTINCT product_name) as unique_products,
    SUM(quantity) as total_quantity,
    ROUND(SUM(quantity * unit_price), 2) as total_revenue,
    ROUND(SUM(quantity * unit_price) / SUM(quantity), 2) as avg_price_per_unit
FROM sales
GROUP BY region
HAVING total_revenue > 1000
ORDER BY total_revenue DESC;
```

・販売地域でグループ化
・合計収益、数量
・収益を数量で割った
・HAVING は少ない地域をフィルタイング

```
SELECT
    sale_date,
    COUNT(DISTINCT category) as categories_sold,
    COUNT(DISTINCT product_name) as unique_products,
    COUNT(*) as transactions,
    SUM(quantity) as total_quantity,
    ROUND(SUM(quantity * unit_price), 2) as daily_revenue,
    ROUND(AVG(quantity * unit_price), 2) as avg_transaction_value
FROM sales
GROUP BY sale_date
ORDER BY sale_date;
```

・日次のトレンド分析

## インデックス再確認

```
インデックスが貼られている

MariaDB [lab_db]> explain select * from users where username='john_doe';
+------+-------------+-------+------+---------------+--------------+---------+-------+------+-----------------------+
| id   | select_type | table | type | possible_keys | key          | key_len | ref   | rows | Extra                 |
+------+-------------+-------+------+---------------+--------------+---------+-------+------+-----------------------+
|    1 | SIMPLE      | users | ref  | idx_username  | idx_username | 767     | const | 1    | Using index condition |
+------+-------------+-------+------+---------------+--------------+---------+-------+------+-----------------------+

インデックス貼られてない！
MariaDB [lab_db]> explain select * from users where city = 'New York';
+------+-------------+-------+------+---------------+------+---------+------+------+-------------+
| id   | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra       |
+------+-------------+-------+------+---------------+------+---------+------+------+-------------+
|    1 | SIMPLE      | users | ALL  | NULL          | NULL | NULL    | NULL | 3    | Using where |
+------+-------------+-------+------+---------------+------+---------+------+------+-------------+
1 row in set (0.000 sec)
```

mysqldump

```
sudo mysqldump -u root mydatabase > /home/labex/backup/mydatabase_backup.sql
```

特定のテーブル

```

```
