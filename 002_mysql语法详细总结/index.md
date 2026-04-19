# MySQL 语法详细总结


# MySQL 数据库语法详细总结
本文基于 MySQL 主流版本（5.7/8.0），覆盖从基础到进阶的全量核心语法，按功能模块结构化梳理，标注关键避坑点与生产最佳实践。

## 一、数据库操作（DDL-库管理）
核心用于数据库的创建、查看、切换、修改、删除，是操作表的前置基础。
```sql
-- 1. 创建数据库（推荐指定字符集，utf8mb4支持emoji和全量中文）
CREATE DATABASE [IF NOT EXISTS] 数据库名
DEFAULT CHARSET = utf8mb4
DEFAULT COLLATE = utf8mb4_general_ci;

-- 2. 查看所有数据库
SHOW DATABASES;
-- 查看数据库的创建语句
SHOW CREATE DATABASE 数据库名;

-- 3. 切换/使用数据库
USE 数据库名;

-- 4. 修改数据库字符集
ALTER DATABASE 数据库名 DEFAULT CHARSET = utf8mb4 COLLATE = utf8mb4_general_ci;

-- 5. 删除数据库
DROP DATABASE [IF EXISTS] 数据库名;
```

## 二、表结构操作（DDL-表管理）
核心用于数据表的创建、修改、查看、删除，是数据存储的基础，需配合数据类型与约束使用。

### 2.1 创建表
完整语法，包含字段、约束、引擎、字符集等核心配置：
```sql
CREATE TABLE [IF NOT EXISTS] 表名 (
    字段名1 数据类型 [列级约束] [COMMENT '字段注释'],
    字段名2 数据类型 [列级约束] [COMMENT '字段注释'],
    -- 表级约束
    [PRIMARY KEY (主键字段1, 主键字段2)],
    [UNIQUE KEY 唯一索引名 (唯一字段)],
    [FOREIGN KEY (外键字段) REFERENCES 主表名(主键字段)]
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4 COMMENT = '表注释';
```
- 核心说明：**InnoDB** 是MySQL默认引擎，支持事务、行锁、外键、崩溃恢复，生产环境优先使用；MyISAM不支持事务和行锁，已基本淘汰。
- 字段约束见本文「五、约束与索引」章节。

### 2.2 查看表结构
```sql
-- 查看表字段与类型
DESC 表名;
-- 查看表的完整创建语句（含索引、约束、注释）
SHOW CREATE TABLE 表名;
-- 查看当前库所有表
SHOW TABLES;
```

### 2.3 修改表结构（ALTER TABLE）
```sql
-- 1. 添加字段
ALTER TABLE 表名 ADD [COLUMN] 字段名 数据类型 [约束] [FIRST|AFTER 已有字段];

-- 2. 修改字段类型/约束
ALTER TABLE 表名 MODIFY [COLUMN] 字段名 新数据类型 [新约束];

-- 3. 修改字段名+类型
ALTER TABLE 表名 CHANGE [COLUMN] 旧字段名 新字段名 数据类型 [约束];

-- 4. 删除字段
ALTER TABLE 表名 DROP [COLUMN] 字段名;

-- 5. 修改表名
ALTER TABLE 旧表名 RENAME [TO] 新表名;

-- 6. 修改表字符集
ALTER TABLE 表名 CONVERT TO CHARSET utf8mb4;

-- 7. 添加/删除主键
ALTER TABLE 表名 ADD PRIMARY KEY (字段名);
ALTER TABLE 表名 DROP PRIMARY KEY;

-- 8. 添加/删除外键
ALTER TABLE 表名 ADD CONSTRAINT 外键名 FOREIGN KEY (外键字段) REFERENCES 主表(主键字段);
ALTER TABLE 表名 DROP FOREIGN KEY 外键名;
```

### 2.4 删除与清空表
```sql
-- 1. 删除表（表结构+数据全删除）
DROP TABLE [IF EXISTS] 表名1, 表名2;

-- 2. 清空表（保留表结构，全表数据删除，自增主键重置）
TRUNCATE TABLE 表名;
```
⚠️ 关键区别：`TRUNCATE` 是DDL语句，执行速度快，不可回滚，自增ID重置；`DELETE FROM 表名` 是DML语句，可回滚，自增ID不重置，可加`WHERE`条件筛选删除。

## 三、数据操作（DML-增删改查）
核心用于表中数据的写入、修改、删除、查询，是业务开发中最高频的语法。

### 3.1 插入数据（INSERT）
```sql
-- 1. 指定字段插入（推荐，兼容性强，不受表字段顺序变更影响）
INSERT INTO 表名 (字段1, 字段2, 字段3)
VALUES (值1, 值2, 值3);

-- 2. 全字段插入（必须和表字段顺序完全一致，不推荐生产使用）
INSERT INTO 表名 VALUES (值1, 值2, 值3, ...);

-- 3. 批量插入（性能远高于单条循环插入，减少IO与事务开销）
INSERT INTO 表名 (字段1, 字段2)
VALUES (v1, v2), (v3, v4), (v5, v6);

-- 4. 插入或更新（唯一键冲突时执行更新，避免重复插入报错）
INSERT INTO 表名 (唯一键字段, 字段2)
VALUES (v1, v2)
ON DUPLICATE KEY UPDATE 字段2 = v2;

-- 5. 插入查询结果（将一张表的数据写入另一张表）
INSERT INTO 目标表 (字段1, 字段2)
SELECT 字段a, 字段b FROM 源表 WHERE 筛选条件;
```

### 3.2 更新数据（UPDATE）
```sql
-- 基础语法
UPDATE 表名
SET 字段1 = 新值1, 字段2 = 新值2
[WHERE 筛选条件]
[ORDER BY 排序字段]
[LIMIT 行数限制];

-- 多表关联更新
UPDATE 表1 t1
JOIN 表2 t2 ON t1.id = t2.t1_id
SET t1.字段 = 新值
WHERE 筛选条件;
```
⚠️ 生产红线：`UPDATE` 必须加 `WHERE` 条件，否则会全表更新，造成不可逆的数据灾难；建议开启 `sql_safe_updates=1` 防护。

### 3.3 删除数据（DELETE）
```sql
-- 基础语法
DELETE FROM 表名
[WHERE 筛选条件]
[ORDER BY 排序字段]
[LIMIT 行数限制];

-- 多表关联删除
DELETE t1 FROM 表1 t1
JOIN 表2 t2 ON t1.id = t2.t1_id
WHERE 筛选条件;
```
⚠️ 生产红线：`DELETE` 必须加 `WHERE` 条件，否则会全表删除，建议同UPDATE开启安全模式防护。

### 3.4 查询数据（SELECT，核心高频）
#### 3.4.1 完整语法与执行顺序
**书写顺序**：
```sql
SELECT [DISTINCT] 字段列表|表达式|*
FROM 表名
[JOIN 联表操作]
[WHERE 行级筛选条件]
[GROUP BY 分组字段]
[HAVING 分组后筛选条件]
[ORDER BY 排序字段 ASC/DESC]
[LIMIT 分页限制];
```

**执行顺序（核心！决定语法正确性）**：
`FROM/JOIN` → `WHERE` → `GROUP BY` → 聚合函数 → `HAVING` → `SELECT` → `DISTINCT` → `ORDER BY` → `LIMIT`

#### 3.4.2 基础查询
```sql
-- 1. 指定字段查询（生产推荐，禁止用SELECT *）
SELECT 字段1, 字段2 FROM 表名;

-- 2. 字段别名（简化SQL，解决重名问题）
SELECT 字段1 AS 别名1, 字段2 别名2 FROM 表名;

-- 3. 去重查询（多字段为组合去重）
SELECT DISTINCT 字段1, 字段2 FROM 表名;

-- 4. 常量与表达式查询
SELECT 100, '测试文本', 字段1 * 10 AS 计算结果 FROM 表名;
```

#### 3.4.3 WHERE 条件筛选
用于行级数据过滤，支持以下运算符：
| 运算符类型 | 常用语法 | 说明 |
| :--- | :--- | :--- |
| 比较运算符 | `=`、`!=/<>`、`>`、`<`、`>=`、`<=` | 基础值比较 |
| 范围查询 | `BETWEEN 最小值 AND 最大值` | 闭区间范围匹配 |
| 集合查询 | `IN (值1,值2,...)`、`NOT IN (...)` | 集合内匹配 |
| 空值判断 | `IS NULL`、`IS NOT NULL` | 唯一正确的NULL判断方式，禁止用`=NULL` |
| 模糊查询 | `LIKE '通配符'` | `%`匹配任意字符，`_`匹配单个字符；`%开头`会导致索引失效 |
| 逻辑运算符 | `AND`、`OR`、`NOT` | 多条件组合，AND优先级高于OR，可用括号调整优先级 |
| 正则匹配 | `REGEXP '正则表达式'` | 正则规则匹配，如`REGEXP '^[a-z]'`匹配小写字母开头 |

示例：
```sql
SELECT * FROM emp
WHERE dept_id IN (1,2,3)
AND salary BETWEEN 5000 AND 20000
AND emp_name LIKE '张%'
AND manager_id IS NOT NULL;
```

#### 3.4.4 排序与分页
```sql
-- 1. 排序：ASC升序（默认），DESC降序，支持多字段排序
SELECT * FROM emp
ORDER BY salary DESC, create_time ASC;

-- 2. 分页：LIMIT 偏移量, 每页行数
SELECT * FROM emp LIMIT 0,10; -- 第1页，每页10条
SELECT * FROM emp LIMIT 10,10; -- 第2页，每页10条

-- MySQL8.0+ 支持更易读的分页语法
SELECT * FROM emp LIMIT 10 OFFSET 10;
```

#### 3.4.5 聚合函数（分组统计）
聚合函数会忽略NULL值，必须配合`GROUP BY`使用，或用于全表统计，**禁止在WHERE中使用聚合函数**。
| 函数 | 功能说明 | 最佳实践 |
| :--- | :--- | :--- |
| `COUNT(*)` | 统计总行数（包含NULL值行） | InnoDB中统计行数优先使用，性能最优 |
| `COUNT(字段)` | 统计该字段非NULL的行数 |  |
| `SUM(字段)` | 求和 |  |
| `AVG(字段)` | 求平均值 |  |
| `MAX(字段)` | 求最大值 |  |
| `MIN(字段)` | 求最小值 |  |
| `GROUP_CONCAT(字段)` | 分组内字段值拼接，默认逗号分隔 | 可指定分隔符：`GROUP_CONCAT(字段 SEPARATOR ';')` |

#### 3.4.6 分组查询（GROUP BY + HAVING）
```sql
-- 基础分组：统计每个部门的员工人数、平均薪资
SELECT dept_id,
       COUNT(*) AS emp_num,
       AVG(salary) AS avg_salary
FROM emp
GROUP BY dept_id;

-- 分组后筛选：只保留员工人数大于5的部门
SELECT dept_id, COUNT(*) AS emp_num
FROM emp
GROUP BY dept_id
HAVING emp_num > 5;
```
⚠️ 核心区别：`WHERE` 是分组前筛选，不能使用聚合函数；`HAVING` 是分组后筛选，只能配合`GROUP BY`使用，可使用聚合函数。
⚠️ 规范：MySQL5.7+ 默认开启`ONLY_FULL_GROUP_BY`，`SELECT` 中的非聚合字段必须全部出现在`GROUP BY`中，避免数据错误。

#### 3.4.7 联表查询（JOIN）
用于多张表的关联数据查询，核心分为以下类型：
```sql
-- 1. 内连接（INNER JOIN，默认JOIN）：只返回两张表匹配条件的记录
SELECT t1.*, t2.dept_name
FROM emp t1
INNER JOIN dept t2 ON t1.dept_id = t2.id;

-- 2. 左连接（LEFT JOIN）：返回左表全部记录，右表匹配不到的字段为NULL
SELECT t1.*, t2.dept_name
FROM emp t1
LEFT JOIN dept t2 ON t1.dept_id = t2.id;

-- 3. 右连接（RIGHT JOIN）：返回右表全部记录，左表匹配不到的字段为NULL
SELECT t1.*, t2.dept_name
FROM emp t1
RIGHT JOIN dept t2 ON t1.dept_id = t2.id;

-- 4. 自连接：表自身与自身关联，常用于树形结构、上下级关系
SELECT a.emp_name AS 员工名, b.emp_name AS 上级名
FROM emp a
LEFT JOIN emp b ON a.manager_id = b.id;

-- 5. 交叉连接（CROSS JOIN）：返回两张表的笛卡尔积，慎用！
SELECT * FROM emp CROSS JOIN dept;
```
> 最佳实践：小表驱动大表；避免超过3张表的JOIN关联；关联字段必须建立索引，且类型完全一致（避免隐式转换导致索引失效）。

#### 3.4.8 子查询
嵌套在SQL语句中的查询，按返回结果分为4类：
```sql
-- 1. 标量子查询：返回单个值，配合比较运算符使用
SELECT * FROM emp
WHERE salary > (SELECT AVG(salary) FROM emp);

-- 2. 列子查询：返回一列值，配合IN/ANY/ALL使用
SELECT * FROM emp
WHERE dept_id IN (SELECT id FROM dept WHERE dept_name LIKE '销售%');

-- 3. 行子查询：返回一行多列
SELECT * FROM emp
WHERE (dept_id, salary) = (SELECT dept_id, MAX(salary) FROM emp WHERE dept_id=1);

-- 4. 表子查询：返回多行多列，放在FROM中必须加别名
SELECT t.dept_id, AVG(t.salary)
FROM (SELECT dept_id, salary FROM emp WHERE salary > 5000) t
GROUP BY t.dept_id;

-- 5. EXISTS子查询：判断子查询是否有结果，大数据量下性能优于IN
SELECT * FROM dept d
WHERE EXISTS (SELECT 1 FROM emp e WHERE e.dept_id = d.id);
```

#### 3.4.9 结果合并（UNION/UNION ALL）
```sql
-- 合并两个查询结果，不去重，性能更高，优先使用
SELECT 字段1,字段2 FROM 表1
UNION ALL
SELECT 字段1,字段2 FROM 表2;

-- 合并两个查询结果，去重（会触发排序，性能低）
SELECT 字段1,字段2 FROM 表1
UNION
SELECT 字段1,字段2 FROM 表2;
```
⚠️ 要求：两个查询的字段数量、数据类型必须完全一致，结果字段名以第一个查询为准。

## 四、常用数据类型
建表时字段类型的核心选择依据，直接影响存储效率与数据准确性，以下为生产高频类型。

### 4.1 数值类型
| 类型 | 存储空间 | 取值范围 | 适用场景 |
| :--- | :--- | :--- | :--- |
| TINYINT | 1字节 | 有符号：-128~127；无符号：0~255 | 性别、状态、类型等枚举值 |
| INT | 4字节 | 有符号：-2147483648~2147483647；无符号：0~4294967295 | 主键ID、普通数字字段，最常用 |
| BIGINT | 8字节 | 超大范围 | 分布式ID、雪花ID、超大数值 |
| DECIMAL(M,D) | 自定义 | M为总位数，D为小数位，精确存储 | 金额、价格等对精度要求极高的场景，如`DECIMAL(10,2)` |
| BOOL/BOOLEAN | 1字节 | 本质是TINYINT(1)，0=false，非0=true | 布尔值判断 |

> 避坑：`INT(10)` 仅为显示宽度，不影响存储范围，MySQL8.0已废弃该语法；FLOAT/DOUBLE存在精度丢失，禁止用于存储金额。

### 4.2 字符串类型
| 类型 | 特性 | 最大长度 | 适用场景 |
| :--- | :--- | :--- | :--- |
| CHAR(M) | 定长字符串，查询效率高 | M最大255字符 | 手机号、身份证号、固定长度编码 |
| VARCHAR(M) | 变长字符串，节省存储空间 | 最大65535字节 | 用户名、地址、备注等非固定长度文本，最常用 |
| TEXT | 长文本，无默认值 | 最大64KB | 文章详情、富文本内容等长文本 |
| ENUM | 枚举类型，只能选指定值 | 最多65535个枚举项 | 性别、状态等固定选项的字段 |

### 4.3 日期时间类型
| 类型 | 格式 | 取值范围 | 特性与适用场景 |
| :--- | :--- | :--- | :--- |
| DATE | YYYY-MM-DD | 1000-01-01 ~ 9999-12-31 | 仅存储日期，如生日、业务日期 |
| DATETIME | YYYY-MM-DD HH:MM:SS | 1000-01-01 ~ 9999-12-31 | 与时区无关，无2038问题，生产推荐，如创建时间、更新时间 |
| TIMESTAMP | YYYY-MM-DD HH:MM:SS | 1970-01-01 ~ 2038-01-19 | 受时区影响，支持自动更新，如`ON UPDATE CURRENT_TIMESTAMP` |
| TIME | HH:MM:SS | -838:59:59 ~ 838:59:59 | 仅存储时间 |

## 五、约束与索引
### 5.1 字段约束（数据完整性保障）
| 约束 | 功能说明 | 核心特性 |
| :--- | :--- | :--- |
| PRIMARY KEY（主键约束） | 唯一标识一行数据，非空+唯一 | 一张表只能有一个主键，InnoDB主键为聚簇索引，建议用自增ID/雪花ID |
| NOT NULL（非空约束） | 字段不允许为NULL | 生产建议所有字段都加，配合DEFAULT默认值，避免NULL带来的查询异常 |
| UNIQUE（唯一约束） | 字段值唯一，不可重复 | 一张表可多个，允许NULL值，自动创建唯一索引，如手机号、邮箱 |
| DEFAULT（默认约束） | 字段未赋值时使用默认值 | 如`DEFAULT 0`、`DEFAULT CURRENT_TIMESTAMP` |
| FOREIGN KEY（外键约束） | 保证两张表的参照完整性 | 生产环境慎用，会影响写入性能，级联操作有风险，建议用业务代码保证 |
| CHECK（检查约束） | 字段值必须满足指定条件 | MySQL8.0原生支持，如`CHECK(age > 0 AND age < 150)` |

### 5.2 索引（查询性能优化核心）
索引用于加速查询，本质是排好序的数据结构，InnoDB默认使用B+树索引。
```sql
-- 1. 创建索引
-- 普通索引
CREATE INDEX 索引名 ON 表名 (字段1, 字段2);
-- 唯一索引
CREATE UNIQUE INDEX 索引名 ON 表名 (字段);
-- 建表时创建索引
CREATE TABLE 表名 (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50),
    INDEX idx_name (name),
    UNIQUE INDEX idx_phone (phone)
) ENGINE=InnoDB;

-- 2. 查看索引
SHOW INDEX FROM 表名;

-- 3. 删除索引
DROP INDEX 索引名 ON 表名;
```
> 核心规范：
> 1. 联合索引遵循**最左前缀原则**，查询条件必须包含索引的最左字段，否则索引失效；
> 2. 索引不是越多越好，会降低INSERT/UPDATE/DELETE的写入性能，仅给高频查询、筛选、排序、关联字段建索引；
> 3. 避免索引失效场景：函数操作索引字段、隐式类型转换、模糊查询%开头、OR连接非索引字段、NOT IN/!=。

## 六、高级特性（MySQL8.0+ 窗口函数）
窗口函数是MySQL8.0的核心进阶特性，解决分组内排名、累计计算、跨行引用等复杂查询场景，不压缩结果行数，每行都返回计算结果。

### 6.1 基础语法
```sql
函数名(参数) OVER (
    [PARTITION BY 分组字段]  -- 分组，类似GROUP BY
    [ORDER BY 排序字段 ASC/DESC]  -- 分组内排序
    [ROWS/RANGE 窗口范围]  -- 计算的行范围
)
```

### 6.2 常用窗口函数
```sql
-- 1. 排名函数
-- ROW_NUMBER()：连续排名，相同值排名不同，1,2,3,4
-- RANK()：跳跃排名，相同值排名相同，1,1,3,4
-- DENSE_RANK()：连续排名，相同值排名相同，1,1,2,3
SELECT emp_name, dept_id, salary,
       ROW_NUMBER() OVER(PARTITION BY dept_id ORDER BY salary DESC) AS rank_num
FROM emp;

-- 2. 聚合窗口函数：累计求和、移动平均
SELECT date, amount,
       SUM(amount) OVER(ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cumulative_sum
FROM `order`;

-- 3. 偏移函数：跨行引用
-- LAG(字段,偏移量,默认值)：取当前行之前的行数据
-- LEAD(字段,偏移量,默认值)：取当前行之后的行数据
SELECT date, amount,
       LAG(amount,1,0) OVER(ORDER BY date) AS last_month_amount,
       LEAD(amount,1,0) OVER(ORDER BY date) AS next_month_amount
FROM `order`;
```

## 七、事务控制（TCL）
事务是InnoDB引擎的核心特性，保证一组SQL操作的原子性，要么全部执行成功，要么全部回滚。

### 7.1 事务ACID特性
- **原子性（Atomicity）**：事务是不可分割的最小单元，操作要么全成功，要么全失败；
- **一致性（Consistency）**：事务执行前后，数据的完整性约束不被破坏；
- **隔离性（Isolation）**：多个事务之间相互隔离，互不干扰；
- **持久性（Durability）**：事务提交后，数据永久写入磁盘，不可回滚。

### 7.2 事务语法
```sql
-- 1. 开启事务
START TRANSACTION;
-- 或简写
BEGIN;

-- 2. 业务SQL操作
INSERT INTO 表名 VALUES (...);
UPDATE 表名 SET 字段=值 WHERE ...;

-- 3. 提交事务（成功后执行，永久生效）
COMMIT;

-- 4. 回滚事务（出错时执行，撤销所有操作）
ROLLBACK;

-- 5. 保存点：实现部分回滚
SAVEPOINT sp1;
-- 执行SQL
ROLLBACK TO SAVEPOINT sp1;

-- 6. 关闭自动提交（MySQL默认autocommit=1，每条SQL自动提交）
SET autocommit = 0;
```

### 7.3 事务隔离级别
| 隔离级别 | 脏读 | 不可重复读 | 幻读 | 说明 |
| :--- | :---: | :---: | :---: | :--- |
| READ UNCOMMITTED（读未提交） | ✅ | ✅ | ✅ | 最低级别，基本不用 |
| READ COMMITTED（读已提交） | ❌ | ✅ | ✅ | Oracle默认级别，解决脏读 |
| REPEATABLE READ（可重复读） | ❌ | ❌ | ✅ | MySQL InnoDB默认级别，通过MVCC解决幻读 |
| SERIALIZABLE（串行化） | ❌ | ❌ | ❌ | 最高级别，强制事务串行，性能极差，基本不用 |

```sql
-- 查看当前隔离级别
SELECT @@transaction_isolation;

-- 设置隔离级别
SET [SESSION|GLOBAL] TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

## 八、数据库对象（视图、存储过程、函数、触发器）
### 8.1 视图（VIEW）
虚拟表，基于查询结果封装，简化复杂SQL，实现权限控制，数据随原表实时更新。
```sql
-- 1. 创建视图
CREATE VIEW 视图名 AS
SELECT t1.emp_name, t1.salary, t2.dept_name
FROM emp t1
JOIN dept t2 ON t1.dept_id = t2.id
WITH CHECK OPTION; -- 保证插入/更新的数据符合视图的WHERE条件

-- 2. 查询视图（和查询表语法一致）
SELECT * FROM 视图名;

-- 3. 修改视图
ALTER VIEW 视图名 AS SELECT 语句;

-- 4. 删除视图
DROP VIEW [IF EXISTS] 视图名;
```

### 8.2 存储过程（PROCEDURE）
预编译的SQL集合，支持入参、出参、流程控制，用于批量处理与复杂业务逻辑封装。
```sql
-- 1. 创建存储过程（临时修改结束符，避免与SQL内的;冲突）
DELIMITER //
CREATE PROCEDURE 存储过程名(
    IN 入参名 数据类型,  -- 入参
    OUT 出参名 数据类型  -- 出参
)
BEGIN
    -- 业务逻辑
    SELECT * FROM emp WHERE dept_id = 入参名;
    SET 出参名 = (SELECT COUNT(*) FROM emp WHERE dept_id = 入参名);
END //
DELIMITER ;

-- 2. 调用存储过程
CALL 存储过程名(1, @total_num);
-- 查看出参结果
SELECT @total_num;

-- 3. 删除存储过程
DROP PROCEDURE [IF EXISTS] 存储过程名;
```

### 8.3 自定义函数（FUNCTION）
可带参数，必须有返回值，可直接在SQL语句中调用，用于通用逻辑封装。
```sql
-- 1. 创建函数
DELIMITER //
CREATE FUNCTION 函数名(参数名 数据类型)
RETURNS 返回数据类型
DETERMINISTIC -- 确定性：相同输入返回相同输出
BEGIN
    DECLARE 变量名 数据类型 DEFAULT 0;
    -- 业务逻辑
    SET 变量名 = 参数名 * 1.1;
    RETURN 变量名;
END //
DELIMITER ;

-- 2. 调用函数
SELECT 函数名(100);
SELECT emp_name, 函数名(salary) AS after_tax FROM emp;

-- 3. 删除函数
DROP FUNCTION [IF EXISTS] 函数名;
```

### 8.4 触发器（TRIGGER）
在表发生INSERT/UPDATE/DELETE事件时，自动执行的SQL逻辑，用于数据审计、自动更新、数据校验。
```sql
-- 1. 创建触发器
DELIMITER //
CREATE TRIGGER 触发器名
AFTER INSERT ON emp -- 插入后触发，可选BEFORE/AFTER，INSERT/UPDATE/DELETE
FOR EACH ROW -- 行级触发器，每一行数据变更都触发
BEGIN
    -- 逻辑：NEW代表新行数据，OLD代表修改前的行数据
    INSERT INTO emp_log(operate_type, emp_id, operate_time)
    VALUES ('INSERT', NEW.id, NOW());
END //
DELIMITER ;

-- 2. 查看触发器
SHOW TRIGGERS;

-- 3. 删除触发器
DROP TRIGGER [IF EXISTS] 触发器名;
```
> 生产建议：触发器慎用，会隐藏业务逻辑，增加排查难度，影响写入性能，多数场景可通过业务代码替代。

## 九、用户与权限管理（DCL）
用于数据库的账号创建、权限分配、回收，是数据库安全的核心，遵循最小权限原则。
```sql
-- 1. 创建用户
CREATE USER '用户名'@'访问主机' IDENTIFIED BY '复杂密码';
-- 示例：创建允许所有地址访问的用户
CREATE USER 'test_user'@'%' IDENTIFIED BY 'Test@123456';
-- 访问主机：localhost仅本地访问，192.168.1.%允许指定网段访问，%允许所有地址访问

-- 2. 授权
GRANT 权限1,权限2 ON 库名.表名 TO '用户名'@'访问主机';
-- 示例：授予test库所有表的查询、写入权限
GRANT SELECT,INSERT,UPDATE,DELETE ON test_db.* TO 'test_user'@'%';
-- 授予所有权限（慎用，仅管理员账号）
GRANT ALL PRIVILEGES ON *.* TO 'admin_user'@'%' WITH GRANT OPTION;

-- 3. 刷新权限（授权后必须执行，使权限生效）
FLUSH PRIVILEGES;

-- 4. 查看用户权限
SHOW GRANTS FOR 'test_user'@'%';

-- 5. 回收权限
REVOKE DELETE ON test_db.* FROM 'test_user'@'%';

-- 6. 修改用户密码
ALTER USER 'test_user'@'%' IDENTIFIED BY 'NewPass@123456';

-- 7. 删除用户
DROP USER 'test_user'@'%';
```
> 生产规范：禁止root账号远程访问；普通账号仅授予业务所需最小权限；密码必须满足复杂度要求，定期更换。

## 十、常用内置函数
### 10.1 字符串函数
```sql
-- 字符串拼接，CONCAT_WS带分隔符，忽略NULL
SELECT CONCAT('a','b','c'), CONCAT_WS('-', 'a', 'b', NULL);
-- 字符串长度：LENGTH字节长度，CHAR_LENGTH字符长度
SELECT LENGTH('测试'), CHAR_LENGTH('测试');
-- 截取字符串
SELECT SUBSTRING('abcdef', 1, 3);
-- 替换字符串
SELECT REPLACE('abcabc', 'a', 'x');
-- 去除首尾空格
SELECT TRIM('  abc  ');
```

### 10.2 日期时间函数
```sql
-- 当前日期时间
SELECT NOW(), CURDATE(), CURTIME();
-- 日期格式化
SELECT DATE_FORMAT(NOW(), '%Y-%m-%d %H:%i:%s');
-- 字符串转日期
SELECT STR_TO_DATE('2024-01-01', '%Y-%m-%d');
-- 日期加减
SELECT DATE_ADD(NOW(), INTERVAL 7 DAY);
-- 日期差值
SELECT DATEDIFF('2024-01-10', '2024-01-01');
SELECT TIMESTAMPDIFF(YEAR, '2000-01-01', NOW()); -- 计算年龄
```

### 10.3 流程控制函数
```sql
-- IF三元表达式
SELECT IF(salary>10000, '高薪', '普通') FROM emp;
-- NULL值替换
SELECT IFNULL(manager_id, 0) FROM emp;
-- CASE多条件分支
SELECT emp_name, salary,
       CASE WHEN salary>20000 THEN 'S级'
            WHEN salary>10000 THEN 'A级'
            ELSE 'B级' END AS salary_level
FROM emp;
```

## 十一、生产避坑核心规范
1.  禁止无`WHERE`条件的`UPDATE/DELETE`，开启`sql_safe_updates`安全模式；
2.  生产环境禁止使用`SELECT *`，只查询业务所需字段，减少IO开销；
3.  统一使用`utf8mb4`字符集，避免emoji、生僻字存储异常；
4.  避免长事务，长事务会导致锁阻塞、undo log膨胀，影响数据库性能；
5.  关联查询必须保证关联字段类型一致，且建立索引，避免隐式转换导致索引失效；
6.  大数据量分页优化，避免`LIMIT 100000,10`大偏移量分页，改用主键过滤`WHERE id>100000 LIMIT 10`；
7.  批量操作优先使用批量插入、批量更新，减少循环单条操作的网络IO与事务开销；
8.  禁止在循环中执行SQL，杜绝N+1查询问题。


