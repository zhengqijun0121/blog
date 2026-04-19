# SQLite3 语法详细总结


# SQLite3 数据库语法详细总结
SQLite3 是一款轻量级、文件型、零配置、支持ACID的关系型数据库，兼容大部分SQL-92标准，同时有自身特有的语法特性与约束。本文从基础到进阶，全面梳理其核心语法、特性与最佳实践。

## 一、核心数据类型（SQLite特有动态类型系统）
SQLite 采用**基于存储类的动态类型系统**，而非传统数据库的列强制类型。列声明的类型为「亲和类型」，仅做类型推荐，不强制约束数据存储。

### 1. 5种核心存储类
| 存储类 | 说明 |
|--------|------|
| `NULL` | 空值 |
| `INTEGER` | 带符号整数，根据数值大小自动适配1/2/3/4/6/8字节存储 |
| `REAL` | 浮点型，8字节IEEE双精度浮点数 |
| `TEXT` | 字符串，默认使用UTF-8/UTF-16编码存储 |
| `BLOB` | 二进制数据，完全按输入原样存储 |

### 2. 列亲和类型与常用别名
SQLite 会根据列声明的类型，自动映射为5种亲和类型，兼容主流数据库的类型别名，无需修改语法即可适配。
| 亲和类型 | 兼容的类型别名 |
|----------|----------------|
| `INTEGER` | `INT`、`BIGINT`、`SMALLINT`、`TINYINT`、`BOOLEAN`、`SERIAL` |
| `REAL` | `FLOAT`、`DOUBLE`、`DECIMAL`、`NUMERIC` |
| `TEXT` | `VARCHAR(n)`、`CHAR(n)`、`TEXT`、`CLOB` |
| `BLOB` | `BLOB`、`BINARY` |
| `NUMERIC` | `DATE`、`DATETIME`、`TIMESTAMP`、`NUMERIC` |

### 3. 关键注意事项
1.  **无原生布尔类型**：`BOOLEAN` 本质是 `INTEGER`，`0` 代表false，`1` 代表true。
2.  **无原生日期时间类型**：推荐3种存储方式：
    - `TEXT`：ISO8601格式字符串（`YYYY-MM-DD HH:MM:SS.SSS`，最通用）
    - `INTEGER`：Unix时间戳（秒/毫秒级）
    - `REAL`：儒略日数，用于日期计算
3.  **类型无强制约束**：`INT` 列可以存储 `TEXT` 数据，需业务层自行校验，避免隐式转换导致索引失效。

---

## 二、DDL 数据定义语言
SQLite 无 `CREATE DATABASE` 语法，数据库对应单个磁盘文件，直接打开/创建文件即可完成库的初始化。核心DDL围绕表、索引、视图、触发器等对象。

### 1. 表操作
#### 1.1 创建表 `CREATE TABLE`
**核心语法**
```sql
-- 基础语法
CREATE [TEMP|TEMPORARY] TABLE [IF NOT EXISTS] 表名 (
    列名1 亲和类型 [列级约束],
    列名2 亲和类型 [列级约束],
    ...
    [表级约束]
) [WITHOUT ROWID];
```

**关键参数与示例**
- `TEMP/TEMPORARY`：创建临时表，仅当前数据库连接可见，连接关闭自动删除
- `IF NOT EXISTS`：避免表已存在时报错
- `WITHOUT ROWID`：关闭默认的ROWID自增列，必须指定`PRIMARY KEY`，适合主键查询场景，优化存储空间与性能
- **约束支持**：`NOT NULL`、`UNIQUE`、`PRIMARY KEY`、`FOREIGN KEY`、`CHECK`、`DEFAULT`

**完整示例**
```sql
-- 创建用户表
CREATE TABLE IF NOT EXISTS user_info (
    user_id INTEGER PRIMARY KEY AUTOINCREMENT, -- 自增主键
    username TEXT NOT NULL UNIQUE, -- 用户名非空唯一
    age INTEGER CHECK (age >= 0 AND age <= 150), -- 年龄校验
    gender TEXT DEFAULT '未知', -- 默认值
    create_time TEXT NOT NULL DEFAULT (datetime('now', 'localtime')), -- 创建时间
    dept_id INTEGER,
    -- 表级外键约束（需手动开启外键开关）
    FOREIGN KEY (dept_id) REFERENCES dept_info (dept_id) ON DELETE SET NULL
);

-- 无ROWID表示例
CREATE TABLE IF NOT EXISTS product (
    product_code TEXT PRIMARY KEY,
    product_name TEXT NOT NULL,
    price REAL NOT NULL CHECK (price >= 0)
) WITHOUT ROWID;
```

**主键与自增关键说明**
1.  `INTEGER PRIMARY KEY` 列会自动关联SQLite内置的`ROWID`，插入时不指定值会自动生成唯一自增整数，无额外开销。
2.  `AUTOINCREMENT`：严格递增不重复的自增主键，保证ID永不复用（即使删除数据），但有额外性能开销，仅需严格唯一ID时使用。
3.  主键默认自带`NOT NULL`和`UNIQUE`约束。

#### 1.2 修改表 `ALTER TABLE`
SQLite 的 `ALTER TABLE` 功能有限，仅支持以下操作，不支持直接删除列、修改列类型/约束：
```sql
-- 1. 重命名表名
ALTER TABLE 旧表名 RENAME TO 新表名;

-- 2. 新增列（3.25.0+版本支持）
ALTER TABLE 表名 ADD COLUMN 列名 亲和类型 [约束];

-- 3. 重命名列（3.25.0+版本支持）
ALTER TABLE 表名 RENAME COLUMN 旧列名 TO 新列名;
```

**不支持操作的替代方案**：新建符合结构的表 → 迁移原表数据 → 删除原表 → 重命名新表。
示例（删除列）：
```sql
BEGIN TRANSACTION;
-- 新建目标结构表
CREATE TABLE user_info_new (
    user_id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT NOT NULL UNIQUE,
    age INTEGER CHECK (age >= 0 AND age <= 150),
    create_time TEXT NOT NULL DEFAULT (datetime('now', 'localtime'))
);
-- 迁移数据
INSERT INTO user_info_new SELECT user_id, username, age, create_time FROM user_info;
-- 替换表
DROP TABLE user_info;
ALTER TABLE user_info_new RENAME TO user_info;
COMMIT;
```

#### 1.3 删除表 `DROP TABLE`
```sql
-- 基础删除
DROP TABLE [IF EXISTS] 表名;
-- 删除临时表
DROP TABLE IF EXISTS temp.临时表名;
```
- `IF EXISTS`：避免表不存在时报错
- 删除表会同时删除表关联的索引、触发器、约束
- SQLite 不支持 `CASCADE` 级联删除，需通过外键的`ON DELETE`子句实现级联操作

### 2. 索引操作
索引是优化查询性能的核心，SQLite 支持B树索引、唯一索引、复合索引、覆盖索引。

#### 2.1 创建索引
```sql
-- 普通索引
CREATE [UNIQUE] INDEX [IF NOT EXISTS] 索引名 ON 表名 (列1 [ASC/DESC], 列2 [ASC/DESC], ...);

-- 条件索引（部分索引，仅索引符合条件的数据）
CREATE INDEX [IF NOT EXISTS] 索引名 ON 表名 (列名) WHERE 条件;
```
**示例**
```sql
-- 普通单值索引
CREATE INDEX IF NOT EXISTS idx_user_username ON user_info (username);

-- 唯一复合索引
CREATE UNIQUE INDEX IF NOT EXISTS idx_user_dept_age ON user_info (dept_id, age DESC);

-- 部分索引（仅索引成年用户）
CREATE INDEX IF NOT EXISTS idx_user_adult ON user_info (age) WHERE age >= 18;
```

#### 2.2 删除索引
```sql
DROP INDEX [IF EXISTS] 索引名;
```

#### 索引关键注意事项
1.  索引会加速`WHERE`、`JOIN`、`ORDER BY`、`GROUP BY`，但会降低`INSERT`/`UPDATE`/`DELETE`的性能，避免过度建索引。
2.  遵循最左匹配原则：复合索引仅在查询条件包含索引最左前列时生效。
3.  `LIKE '%xxx'`、`!=`、`NOT IN`、函数包裹列 会导致索引失效。
4.  视图上无法创建索引，仅虚拟表支持特殊索引。

### 3. 视图操作
视图是虚拟表，基于查询结果封装，不存储实际数据，简化复杂查询、控制数据访问权限。

#### 3.1 创建视图
```sql
CREATE [TEMP|TEMPORARY] VIEW [IF NOT EXISTS] 视图名 [列名1, 列名2, ...]
AS
查询语句
[WITH [CASCADED|LOCAL] CHECK OPTION];
```
**示例**
```sql
-- 创建成年用户视图
CREATE VIEW IF NOT EXISTS v_user_adult
AS
SELECT user_id, username, age, dept_id
FROM user_info
WHERE age >= 18
WITH CHECK OPTION; -- 限制插入/更新的数据必须符合视图WHERE条件
```

#### 3.2 删除视图
```sql
DROP VIEW [IF EXISTS] 视图名;
```

#### 视图关键说明
1.  视图默认只读，无法直接执行`INSERT`/`UPDATE`/`DELETE`，需通过`INSTEAD OF`触发器实现可写视图。
2.  `WITH CHECK OPTION`：确保通过视图修改的数据，始终符合视图的过滤条件。
3.  视图每次查询都会执行内部的SELECT语句，复杂视图可通过临时表优化性能。

### 4. 触发器操作
触发器是绑定到表的事件回调，当表执行`INSERT`/`UPDATE`/`DELETE`时，自动触发预设的SQL逻辑，用于数据校验、审计日志、级联操作等。

#### 4.1 创建触发器
```sql
CREATE [TEMP|TEMPORARY] TRIGGER [IF NOT EXISTS] 触发器名
[BEFORE|AFTER|INSTEAD OF] [INSERT|UPDATE|UPDATE OF 列名|DELETE]
ON 表名
[FOR EACH ROW] -- SQLite仅支持行级触发器
[WHEN 触发条件]
BEGIN
    -- 触发执行的SQL语句
    语句1;
    语句2;
    ...
END;
```

**核心关键字**
- `BEFORE`：事件执行前触发，常用于数据校验、修改输入值
- `AFTER`：事件执行后触发，常用于审计日志、数据同步
- `INSTEAD OF`：替代原事件执行，仅用于视图，实现可写视图
- `NEW`：指代插入/更新后的新行数据
- `OLD`：指代更新/删除前的旧行数据

**示例（审计日志触发器）**
```sql
-- 创建用户操作日志表
CREATE TABLE IF NOT EXISTS user_operate_log (
    log_id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL,
    operate_type TEXT NOT NULL,
    operate_time TEXT NOT NULL DEFAULT (datetime('now', 'localtime')),
    old_value TEXT,
    new_value TEXT
);

-- 创建用户更新后触发器
CREATE TRIGGER IF NOT EXISTS trig_user_update_log
AFTER UPDATE ON user_info
FOR EACH ROW
BEGIN
    INSERT INTO user_operate_log (user_id, operate_type, old_value, new_value)
    VALUES (OLD.user_id, 'UPDATE', OLD.username, NEW.username);
END;
```

#### 4.2 删除触发器
```sql
DROP TRIGGER [IF EXISTS] 触发器名;
```

---

## 三、DML 数据操纵语言
用于数据的增、删、改操作，SQLite 扩展了冲突解决、UPSERT等特有语法。

### 1. 插入数据 `INSERT`
#### 1.1 基础插入
```sql
-- 单行插入
INSERT INTO 表名 (列1, 列2, 列3, ...)
VALUES (值1, 值2, 值3, ...);

-- 多行批量插入（3.7.11+版本支持）
INSERT INTO 表名 (列1, 列2, 列3, ...)
VALUES (值1, 值2, 值3, ...),
       (值1, 值2, 值3, ...),
       (值1, 值2, 值3, ...);

-- 从其他表查询插入
INSERT INTO 表名 (列1, 列2, 列3, ...)
SELECT 列1, 列2, 列3, ... FROM 其他表 WHERE 条件;
```

#### 1.2 冲突解决与UPSERT（核心特有语法）
当插入数据违反`UNIQUE`/`PRIMARY KEY`约束时，可通过冲突子句指定处理策略，SQLite 支持两种冲突处理方式：

**方式1：OR 冲突策略（兼容旧版本）**
```sql
INSERT OR [ROLLBACK|ABORT|FAIL|IGNORE|REPLACE]
INTO 表名 (列1, 列2, ...) VALUES (值1, 值2, ...);
```
- `IGNORE`：忽略冲突行，不插入也不报错，继续执行后续语句
- `REPLACE`：冲突时删除原有冲突行，插入新行（UPSERT核心）
- `ABORT`：默认策略，终止执行，回滚当前语句的修改

**方式2：标准UPSERT语法（3.24.0+版本推荐）**
兼容PostgreSQL风格的UPSERT，支持冲突时更新或忽略，功能更灵活。
```sql
INSERT INTO 表名 (列1, 列2, ...)
VALUES (值1, 值2, ...)
ON CONFLICT (冲突列1, 冲突列2, ...) -- 冲突的唯一键/主键列
DO UPDATE SET 列1 = 新值1, 列2 = 新值2, ... [WHERE 条件] -- 冲突时更新
| DO NOTHING; -- 冲突时忽略
```

**UPSERT示例**
```sql
-- 插入用户，用户名冲突时更新年龄和更新时间
INSERT INTO user_info (username, age, update_time)
VALUES ('zhangsan', 20, datetime('now', 'localtime'))
ON CONFLICT (username)
DO UPDATE SET age = excluded.age, update_time = excluded.update_time;
-- excluded 指代INSERT语句中待插入的新值
```

### 2. 更新数据 `UPDATE`
```sql
-- 基础更新
UPDATE 表名
SET 列1 = 值1, 列2 = 值2, ...
[WHERE 条件]
[ORDER BY 排序规则]
[LIMIT 行数];

-- 关联更新（3.33.0+版本支持 UPDATE FROM）
UPDATE 表名1
SET 列1 = 表名2.列值
FROM 表名2
WHERE 表名1.关联列 = 表名2.关联列 [AND 其他条件];

-- 冲突策略更新
UPDATE OR [ROLLBACK|ABORT|FAIL|IGNORE|REPLACE]
表名 SET 列1 = 值1, ... [WHERE 条件];
```

**示例**
```sql
-- 基础更新
UPDATE user_info SET age = 25, gender = '男' WHERE user_id = 1;

-- 关联更新：同步部门名称到用户表
UPDATE user_info u
SET dept_name = d.dept_name
FROM dept_info d
WHERE u.dept_id = d.dept_id;
```

### 3. 删除数据 `DELETE`
```sql
-- 基础删除
DELETE FROM 表名
[WHERE 条件]
[ORDER BY 排序规则]
[LIMIT 行数];

-- 清空全表数据（无TRUNCATE语法，用DELETE替代）
DELETE FROM 表名;
```
- 无`WHERE`条件时，会删除表中所有数据，自增主键的计数不会重置（需`VACUUM`重置）
- 大表清空推荐：`DROP TABLE` → 重建表，性能远高于`DELETE`全表

---

## 四、DQL 数据查询语言
SQLite 兼容标准SQL的查询语法，支持复杂查询、联表、子查询、CTE、窗口函数等高级特性，是SQLite最核心的语法模块。

### 1. 基础查询语法
```sql
SELECT [DISTINCT] 列1 [AS 别名1], 列2 [AS 别名2], 聚合函数(), 表达式...
FROM 表名 [AS 表别名]
[WHERE 行过滤条件]
[GROUP BY 分组列1, 分组列2...]
[HAVING 分组后过滤条件]
[ORDER BY 排序列1 [ASC/DESC], 排序列2 [ASC/DESC]...]
[LIMIT 行数 [OFFSET 偏移量]];
```

#### 核心子句说明
1.  **SELECT 子句**：指定查询返回的列，支持常量、表达式、函数、别名
    - `*`：匹配所有列，生产环境不推荐，易导致性能问题和字段变更风险
    - `DISTINCT`：对结果集去重，去除重复行
    示例：
    ```sql
    SELECT user_id, username AS 用户名, age + 1 AS 明年年龄, 'SQLite' AS 来源 FROM user_info;
    SELECT DISTINCT gender FROM user_info;
    ```

2.  **WHERE 子句**：行级过滤，在分组、聚合前执行，不支持聚合函数
    支持的运算符：
    - 比较运算符：`=`、`!=`/`<>`、`>`、`<`、`>=`、`<=`
    - 逻辑运算符：`AND`、`OR`、`NOT`
    - 范围运算符：`BETWEEN 最小值 AND 最大值`、`IN (值1,值2...)`、`NOT IN (...)`
    - 空值判断：`IS NULL`、`IS NOT NULL`（禁止用`= NULL`，永远返回false）
    - 模糊匹配：`LIKE`（不区分大小写，通配符`%`匹配任意字符，`_`匹配单个字符）、`GLOB`（区分大小写，通配符`*`匹配任意字符，`?`匹配单个字符）
    - 存在判断：`EXISTS (子查询)`

    示例：
    ```sql
    -- 18-30岁的男性用户
    SELECT * FROM user_info WHERE age BETWEEN 18 AND 30 AND gender = '男';
    -- 用户名以zhang开头的用户
    SELECT * FROM user_info WHERE username LIKE 'zhang%';
    -- 部门ID在1/2/3的用户
    SELECT * FROM user_info WHERE dept_id IN (1,2,3);
    -- 存在部门的用户
    SELECT * FROM user_info u WHERE EXISTS (SELECT 1 FROM dept_info d WHERE d.dept_id = u.dept_id);
    ```

3.  **GROUP BY + HAVING 子句**：分组聚合
    - `GROUP BY`：按指定列对结果集分组，相同值分为一组，常与聚合函数搭配
    - `HAVING`：分组后的过滤，支持聚合函数，在分组聚合后执行（与WHERE核心区别）

    示例：
    ```sql
    -- 按部门统计用户数、平均年龄，仅展示用户数大于5的部门
    SELECT dept_id, COUNT(*) AS user_count, AVG(age) AS avg_age
    FROM user_info
    WHERE dept_id IS NOT NULL
    GROUP BY dept_id
    HAVING user_count > 5;
    ```

4.  **ORDER BY 子句**：结果集排序
    - `ASC`：升序（默认），`DESC`：降序
    - 支持多字段排序，按顺序依次生效

    示例：
    ```sql
    SELECT * FROM user_info ORDER BY age DESC, create_time ASC;
    ```

5.  **LIMIT + OFFSET 子句**：分页查询
    两种写法：
    ```sql
    -- 写法1：LIMIT 偏移量, 行数（兼容MySQL）
    SELECT * FROM user_info LIMIT 10, 20; -- 跳过前10条，取20条

    -- 写法2：LIMIT 行数 OFFSET 偏移量（SQLite推荐，语义更清晰）
    SELECT * FROM user_info LIMIT 20 OFFSET 10;
    ```
    性能优化：大偏移量分页性能差，推荐主键分页法：
    ```sql
    -- 优化后：上一页最后一条user_id为100
    SELECT * FROM user_info WHERE user_id > 100 LIMIT 20;
    ```

### 2. 联表查询
SQLite 支持主流联表类型，**不支持 RIGHT JOIN、FULL OUTER JOIN**，可通过其他方式替代。

| 联表类型 | 语法 | 说明 |
|----------|------|------|
| 内连接 INNER JOIN | `A INNER JOIN B ON A.关联列 = B.关联列` | 仅返回两张表中匹配关联条件的行，默认JOIN等价于INNER JOIN |
| 左连接 LEFT JOIN | `A LEFT JOIN B ON A.关联列 = B.关联列` | 返回左表A的所有行，右表B无匹配的行填充NULL |
| 交叉连接 CROSS JOIN | `A CROSS JOIN B` | 返回两张表的笛卡尔积，A的每一行与B的每一行组合，无ON子句 |
| 自然连接 NATURAL JOIN | `A NATURAL JOIN B` | 自动按两张表中同名的列进行内连接，无需写ON子句 |

**示例**
```sql
-- 内连接：查询用户及其所属部门信息
SELECT u.user_id, u.username, d.dept_name
FROM user_info u
INNER JOIN dept_info d ON u.dept_id = d.dept_id;

-- 左连接：查询所有用户，无部门的用户也展示
SELECT u.user_id, u.username, d.dept_name
FROM user_info u
LEFT JOIN dept_info d ON u.dept_id = d.dept_id;
```

### 3. 子查询
子查询是嵌套在主查询中的SELECT语句，用于复杂条件过滤、数据来源，分为3类：
1.  **标量子查询**：返回单个值（1行1列），可用于`=`、`>`等比较运算符
    ```sql
    -- 查询年龄大于平均年龄的用户
    SELECT * FROM user_info WHERE age > (SELECT AVG(age) FROM user_info);
    ```

2.  **列子查询**：返回单列多行，用于`IN`、`EXISTS`、`ANY`、`ALL`
    ```sql
    -- 查询有用户的部门
    SELECT * FROM dept_info WHERE dept_id IN (SELECT DISTINCT dept_id FROM user_info);
    ```

3.  **表子查询**：返回多行多列，用于FROM子句作为临时表
    ```sql
    -- 查询部门平均年龄大于20的部门详情
    SELECT d.*, t.avg_age
    FROM dept_info d
    INNER JOIN (SELECT dept_id, AVG(age) AS avg_age FROM user_info GROUP BY dept_id) t
    ON d.dept_id = t.dept_id
    WHERE t.avg_age > 20;
    ```

### 4. CTE 公用表表达式（3.8.3+版本支持）
CTE 用于封装子查询，提升SQL可读性，支持递归，分为普通CTE和递归CTE。

#### 4.1 普通CTE
```sql
WITH cte名称 [列名1, 列名2...] AS (
    查询语句
)
SELECT * FROM cte名称;
```
示例：
```sql
WITH dept_user_stat AS (
    SELECT dept_id, COUNT(*) AS user_count, AVG(age) AS avg_age
    FROM user_info
    GROUP BY dept_id
)
SELECT d.dept_name, s.user_count, s.avg_age
FROM dept_info d
LEFT JOIN dept_user_stat s ON d.dept_id = s.dept_id;
```

#### 4.2 递归CTE
用于处理树形结构、层级数据（如部门树、分类树），语法固定分为锚点成员和递归成员：
```sql
WITH RECURSIVE cte名称 AS (
    -- 锚点成员：递归起始点
    SELECT * FROM 表名 WHERE 起始条件
    UNION ALL
    -- 递归成员：引用自身，实现递归
    SELECT t.* FROM 表名 t
    INNER JOIN cte名称 c ON t.父级列 = c.主键列
)
SELECT * FROM cte名称;
```
示例（查询部门树形结构）：
```sql
-- 部门表
CREATE TABLE IF NOT EXISTS dept_info (
    dept_id INTEGER PRIMARY KEY,
    dept_name TEXT NOT NULL,
    parent_dept_id INTEGER,
    FOREIGN KEY (parent_dept_id) REFERENCES dept_info (dept_id)
);

-- 递归查询所有子部门
WITH RECURSIVE dept_tree AS (
    -- 锚点：顶级部门
    SELECT dept_id, dept_name, parent_dept_id, 1 AS dept_level
    FROM dept_info WHERE parent_dept_id IS NULL
    UNION ALL
    -- 递归：查询子部门
    SELECT d.dept_id, d.dept_name, d.parent_dept_id, dt.dept_level + 1 AS dept_level
    FROM dept_info d
    INNER JOIN dept_tree dt ON d.parent_dept_id = dt.dept_id
)
SELECT * FROM dept_tree;
```

### 5. 集合操作
用于合并多个SELECT语句的结果集，要求多个查询的列数、列类型一致。
| 操作符 | 说明 |
|--------|------|
| `UNION` | 合并两个结果集，自动去重 |
| `UNION ALL` | 合并两个结果集，不去重，性能远高于UNION |
| `INTERSECT` | 取两个结果集的交集（同时存在的行），自动去重 |
| `EXCEPT` | 取两个结果集的差集（第一个有、第二个没有的行），自动去重 |

示例：
```sql
-- 合并两个部门的用户，不去重
SELECT username FROM user_info WHERE dept_id = 1
UNION ALL
SELECT username FROM user_info WHERE dept_id = 2;
```

### 6. 窗口函数（3.25.0+版本支持）
窗口函数基于指定的窗口范围计算，不改变原结果集的行数，实现排名、累计、同比环比等复杂分析，核心语法：
```sql
窗口函数名([参数]) OVER (
    [PARTITION BY 分组列] -- 按列分区，类似GROUP BY，分区内独立计算
    [ORDER BY 排序列 [ASC/DESC]] -- 分区内排序
    [ROWS/RANGE BETWEEN 边界1 AND 边界2] -- 窗口帧范围
)
```

#### 常用窗口函数
| 分类 | 函数名 | 说明 |
|------|--------|------|
| 排名函数 | `ROW_NUMBER()` | 分区内连续排名，相同值排名不同，无并列 |
| 排名函数 | `RANK()` | 分区内跳跃排名，相同值排名相同，后续排名跳过 |
| 排名函数 | `DENSE_RANK()` | 分区内连续排名，相同值排名相同，后续排名不跳过 |
| 偏移函数 | `LAG(列名, 偏移量, 默认值)` | 取当前行前面第N行的值 |
| 偏移函数 | `LEAD(列名, 偏移量, 默认值)` | 取当前行后面第N行的值 |
| 聚合函数 | `SUM()/AVG()/COUNT()/MAX()/MIN()` | 窗口内聚合计算 |

**示例**
```sql
-- 按部门分区，对用户年龄排名
SELECT
    user_id, username, dept_id, age,
    ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY age DESC) AS row_num,
    RANK() OVER (PARTITION BY dept_id ORDER BY age DESC) AS rank_num,
    DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY age DESC) AS dense_rank_num
FROM user_info;

-- 累计求和：按创建时间排序，累计用户数
SELECT
    create_time,
    COUNT(*) AS daily_add,
    SUM(COUNT(*)) OVER (ORDER BY create_time ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS total_count
FROM user_info
GROUP BY create_time;
```

---

## 五、核心内置函数
### 1. 字符串函数
| 函数 | 语法 | 说明 |
|------|------|------|
| `length` | `length(str)` | 返回字符串的字节长度（UTF-8中文占3字节） |
| `lower/upper` | `lower(str)`/`upper(str)` | 字符串转小写/大写 |
| `trim/ltrim/rtrim` | `trim(str [, 字符集])` | 去除字符串两端/左侧/右侧的指定字符（默认空格） |
| `substr` | `substr(str, 起始位置, 长度)` | 截取子串，起始位置从1开始，负数从末尾倒数 |
| `instr` | `instr(str, 子串)` | 返回子串在字符串中第一次出现的位置，无匹配返回0 |
| `replace` | `replace(str, 旧子串, 新子串)` | 替换字符串中的指定子串 |
| `concat` | `concat(str1, str2, ...)` | 拼接多个字符串（3.38.0+版本支持） |
| `concat_ws` | `concat_ws(分隔符, str1, str2, ...)` | 带分隔符拼接字符串（3.38.0+版本支持） |
| `group_concat` | `group_concat(列名 [, 分隔符])` | 聚合函数，将分组内的字符串拼接，默认分隔符为逗号 |

### 2. 数值函数
| 函数 | 语法 | 说明 |
|------|------|------|
| `abs` | `abs(num)` | 返回绝对值 |
| `round` | `round(num, 小数位数)` | 四舍五入到指定小数位数 |
| `ceil/floor` | `ceil(num)`/`floor(num)` | 向上取整/向下取整 |
| `mod` | `mod(num, 除数)` | 取模运算，返回余数 |
| `random` | `random()` | 返回-9223372036854775808到9223372036854775807之间的随机整数 |
| `sqrt` | `sqrt(num)` | 平方根 |
| `pow` | `pow(num, 指数)` | 幂运算 |

### 3. 日期时间函数
SQLite 日期函数核心基于ISO8601格式，支持时间戳、日期计算，核心函数如下：
| 函数 | 语法 | 说明 |
|------|------|------|
| `date` | `date(时间源, 修饰符1, 修饰符2...)` | 返回日期字符串，格式`YYYY-MM-DD` |
| `time` | `time(时间源, 修饰符1, 修饰符2...)` | 返回时间字符串，格式`HH:MM:SS` |
| `datetime` | `datetime(时间源, 修饰符1, 修饰符2...)` | 返回日期时间字符串，格式`YYYY-MM-DD HH:MM:SS` |
| `julianday` | `julianday(时间源, 修饰符...)` | 返回儒略日数，用于日期差值计算 |
| `strftime` | `strftime(格式符, 时间源, 修饰符...)` | 按指定格式格式化日期，是所有日期函数的底层函数 |

**核心参数说明**
1.  **时间源**：
    - `now`：当前UTC时间
    - `localtime`：本地时间
    - ISO8601格式字符串：`2026-04-16`、`2026-04-16 12:00:00`
    - Unix时间戳：`unixepoch`修饰符配合，如`strftime('%Y-%m-%d', 1713254400, 'unixepoch', 'localtime')`

2.  **常用修饰符**：
    - `+N years/months/days/hours/minutes/seconds`：时间加N个单位
    - `-N years/months/days/hours/minutes/seconds`：时间减N个单位
    - `start of year/month/day/hour`：时间取到年/月/日/小时的开始
    - `localtime`：UTC时间转本地时间
    - `utc`：本地时间转UTC时间
    - `unixepoch`：将Unix时间戳转为日期时间

3.  **常用格式符**：
    - `%Y`：4位年份，`%m`：2位月份，`%d`：2位日期
    - `%H`：24小时制小时，`%M`：分钟，`%S`：秒
    - `%s`：Unix时间戳（秒），`%w`：星期几（0=周日，6=周六）

**常用示例**
```sql
-- 当前本地日期时间
SELECT datetime('now', 'localtime');

-- 当前日期
SELECT date('now');

-- 当前Unix时间戳
SELECT strftime('%s', 'now');

-- 日期加减：明天、上个月、一年后
SELECT date('now', '+1 day');
SELECT date('now', '-1 month');
SELECT date('now', '+1 year');

-- 当月第一天、当月最后一天
SELECT date('now', 'start of month');
SELECT date('now', 'start of month', '+1 month', '-1 day');

-- 两个日期的差值（天数）
SELECT julianday('now') - julianday('2026-01-01') AS day_diff;

-- 时间戳转日期时间
SELECT datetime(1713254400, 'unixepoch', 'localtime');
```

### 4. 条件与空值函数
| 函数 | 语法 | 说明 |
|------|------|------|
| `ifnull` | `ifnull(表达式, 替代值)` | 表达式为NULL时返回替代值，否则返回表达式本身（SQLite核心空值处理函数） |
| `nullif` | `nullif(表达式1, 表达式2)` | 两个表达式相等时返回NULL，否则返回表达式1 |
| `iif` | `iif(条件, 真值, 假值)` | 三元运算符，条件为true返回真值，否则返回假值（3.32.0+版本支持） |
| `CASE WHEN` | 简单CASE：`CASE 表达式 WHEN 值1 THEN 结果1 ... ELSE 默认值 END` <br> 搜索CASE：`CASE WHEN 条件1 THEN 结果1 ... ELSE 默认值 END` | 多条件分支判断，兼容所有SQLite版本 |

**示例**
```sql
-- 空值处理：部门ID为NULL时显示'无部门'
SELECT username, ifnull(dept_id, '无部门') AS dept FROM user_info;

-- 三元运算：年龄大于等于18标记为成年，否则未成年
SELECT username, iif(age >= 18, '成年', '未成年') AS age_tag FROM user_info;

-- CASE WHEN 多条件判断
SELECT username, age,
    CASE
        WHEN age < 18 THEN '未成年'
        WHEN age < 30 THEN '青年'
        WHEN age < 60 THEN '中年'
        ELSE '老年'
    END AS age_group
FROM user_info;
```

---

## 六、事务与ACID控制
SQLite 支持完整的ACID事务特性，默认开启自动提交模式（每条单独的SQL语句都是一个独立事务，执行完成自动提交），手动事务可批量操作、保证数据一致性、提升批量操作性能。

### 1. 核心事务语法
```sql
-- 1. 开启事务
BEGIN [TRANSACTION] [DEFERRED|IMMEDIATE|EXCLUSIVE];

-- 2. 提交事务
COMMIT [TRANSACTION];
-- 等价于 END TRANSACTION;

-- 3. 回滚事务
ROLLBACK [TRANSACTION];
```

### 2. 三种事务类型（核心区别：锁的获取时机）
| 事务类型 | 说明 | 适用场景 |
|----------|------|----------|
| `DEFERRED` | 默认模式，开启事务时不获取任何锁，第一次读操作获取共享锁，第一次写操作获取排他锁 | 只读事务、低并发场景，默认首选 |
| `IMMEDIATE` | 开启事务时立即获取数据库级共享读锁，其他连接可继续读，无法写 | 读写事务，避免并发写冲突，高并发写推荐 |
| `EXCLUSIVE` | 开启事务时立即获取数据库级排他锁，其他连接无法读也无法写 | 严格的批量写操作，保证数据绝对一致性 |

### 3. 保存点（SAVEPOINT）
SQLite 不支持真正的嵌套事务，通过保存点实现事务内的部分回滚，语法如下：
```sql
-- 创建保存点
SAVEPOINT 保存点名;

-- 回滚到指定保存点
ROLLBACK TRANSACTION TO SAVEPOINT 保存点名;

-- 释放保存点
RELEASE SAVEPOINT 保存点名;
```

**示例**
```sql
BEGIN TRANSACTION;
INSERT INTO user_info (username, age) VALUES ('lisi', 22);
SAVEPOINT sp1; -- 创建保存点
INSERT INTO user_info (username, age) VALUES ('wangwu', 25);
ROLLBACK TO sp1; -- 回滚到sp1，wangwu的插入被撤销，lisi的插入保留
COMMIT; -- 最终仅lisi的数据被提交
```

### 事务关键注意事项
1.  批量插入/更新/删除必须用事务包裹，可提升百倍以上性能（避免每条语句都触发磁盘IO和事务提交）。
2.  SQLite 写锁是排他锁，同一时间仅允许一个写事务执行，长事务会阻塞其他所有写操作，需避免长事务。
3.  事务执行中出现异常，必须执行`ROLLBACK`回滚，否则会一直持有锁，导致数据库锁死。

---

## 七、PRAGMA 指令（SQLite特有核心配置）
PRAGMA 是SQLite特有的指令，用于配置数据库参数、查询元数据、控制数据库行为，是SQLite优化和运维的核心。

### 1. 核心配置类PRAGMA
| 指令 | 用法 | 说明 |
|------|------|------|
| `foreign_keys` | `PRAGMA foreign_keys = ON/OFF;` | 开启/关闭外键约束，**默认OFF**，使用外键必须手动开启 |
| `journal_mode` | `PRAGMA journal_mode = WAL;` | 设置日志模式，可选值：`DELETE`(默认)、`TRUNCATE`、`PERSIST`、`MEMORY`、`WAL`、`OFF`；**WAL模式推荐高并发场景使用**，支持读写并发，大幅提升读性能 |
| `synchronous` | `PRAGMA synchronous = FULL/NORMAL/OFF;` | 同步模式，控制数据持久化安全与性能：`FULL`(最安全，断电不丢数据)、`NORMAL`(WAL模式推荐，平衡安全与性能)、`OFF`(性能最高，断电可能丢数据，仅批量操作临时使用) |
| `busy_timeout` | `PRAGMA busy_timeout = 5000;` | 锁等待超时时间，单位毫秒，锁冲突时等待指定时间再报错，避免频繁锁异常 |
| `cache_size` | `PRAGMA cache_size = -20000;` | 设置页面缓存大小，负数代表KB数（-20000=20000KB=20MB），增大缓存可提升查询性能 |
| `temp_store` | `PRAGMA temp_store = MEMORY;` | 临时表/索引存储位置，`MEMORY`(内存，最快)、`FILE`(磁盘)、`DEFAULT`(默认) |
| `auto_vacuum` | `PRAGMA auto_vacuum = 0/1/2;` | 自动真空，`0`(关闭，默认)、`1`(全量自动)、`2`(增量自动)，回收删除数据的磁盘空间 |
| `encoding` | `PRAGMA encoding = 'UTF-8';` | 设置数据库字符编码，仅新建数据库时可修改，默认UTF-8 |

### 2. 元数据查询类PRAGMA
| 指令 | 用法 | 说明 |
|------|------|------|
| `table_info` | `PRAGMA table_info(表名);` | 查询表的完整结构：列ID、列名、类型、是否非空、默认值、是否主键 |
| `index_list` | `PRAGMA index_list(表名);` | 查询表的所有索引列表：索引名、是否唯一、索引类型 |
| `index_info` | `PRAGMA index_info(索引名);` | 查询索引的列信息：列ID、列名 |
| `foreign_key_list` | `PRAGMA foreign_key_list(表名);` | 查询表的所有外键约束信息 |
| `database_list` | `PRAGMA database_list;` | 查询当前连接的所有数据库列表 |
| `version` | `PRAGMA user_version;` | 查询/设置用户自定义版本号，常用于数据库版本升级，`PRAGMA user_version = 1;` |

---

## 八、常见坑与最佳实践
### 1. 常见避坑指南
1.  **外键默认不生效**：必须执行`PRAGMA foreign_keys = ON;`开启，且仅当前连接有效，每次新建连接都需重新执行。
2.  **动态类型隐式转换坑**：`WHERE id = '123'` 中id是INTEGER类型，字符串值会导致索引失效，必须保证查询值与列类型一致。
3.  **AUTOINCREMENT滥用**：`INTEGER PRIMARY KEY` 已实现自增，`AUTOINCREMENT` 会增加额外的CPU、磁盘、IO开销，仅需ID永不复用时使用。
4.  **LIKE '%xxx' 索引失效**：前缀模糊匹配`LIKE 'xxx%'`可使用索引，后缀模糊匹配`LIKE '%xxx'`无法使用索引，需用全文搜索FTS5优化。
5.  **无WAL模式并发性能差**：默认DELETE模式下，写操作会阻塞所有读操作，高并发场景必须开启WAL模式`PRAGMA journal_mode=WAL;`。
6.  **大offset分页性能差**：`LIMIT 100000, 20` 会扫描前10万行数据，推荐用主键分页`WHERE id > 100000 LIMIT 20`。
7.  **SQLite 不支持存储过程、自定义函数（需扩展）、RIGHT JOIN、FULL OUTER JOIN**，避免直接复用MySQL/PostgreSQL的复杂语法。

### 2. 性能最佳实践
1.  **批量操作必须用事务**：批量插入/更新/删除时，用BEGIN+COMMIT包裹，避免单条语句自动提交，性能可提升100倍以上。
2.  **合理创建索引**：为`WHERE`、`JOIN`、`ORDER BY`、`GROUP BY`的高频字段建索引，避免冗余索引，联合索引遵循最左匹配原则。
3.  **开启WAL模式**：高并发场景必开，配合`synchronous = NORMAL`，平衡性能与数据安全。
4.  **避免SELECT ***：只查询需要的列，减少数据传输，更容易命中覆盖索引。
5.  **定期执行VACUUM**：频繁删除/更新数据后，执行`VACUUM;`整理数据库碎片，回收磁盘空间，优化查询性能。
6.  **使用参数化查询**：避免SQL注入，同时复用SQL语句的执行计划，提升查询性能。
7.  **避免长事务**：SQLite的写锁是排他的，长事务会阻塞所有写操作，事务尽量短小精悍，快速提交/回滚。


