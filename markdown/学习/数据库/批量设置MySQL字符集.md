# 批量设置表及表内字段字符集

## 1. 修改数据库编码及字符集

```mysql
ALTER DATABASE db_name CHARACTER SET = utf8mb4 COLLATE = utf8mb4_bin;
```

## 2. 查询该数据库下的所有表

```mysql
SELECT 
CONCAT("ALTER TABLE `", TABLE_NAME,"` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;") 
AS target_tables
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_SCHEMA="db_name"
AND TABLE_TYPE="BASE TABLE"
```

### 结果

```mysql
ALTER TABLE `table1` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
ALTER TABLE `table2` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
ALTER TABLE `table3` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
ALTER TABLE `table4` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
ALTER TABLE `table5` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
ALTER TABLE `table6` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
```

**注意**	这里使用 `CONVERT TO` 而非 `DEFAULT`，是因为后者不会修改表中字段的编码和字符集。

## 3. 修改数据表与表中字段的编码及字符集

将上一步搜索到的结果转存为.sql文件进行执行

