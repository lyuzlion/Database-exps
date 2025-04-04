# 实验一
## 1.1
```sql
create table test1_deptor(
  pid char(18) not null, 
  pname varchar2(10) not null,
  sex char(2),
  age numeric(3),
  birthday date,
  parentpid char(18)
  )
```
## 1.2
```sql
create table test1_bank(
  bid char(4) not null, 
  bname varchar2(20) not null,
  tel varchar2(20),
  city varchar2(10),
  stru varchar2(10),
  hbid char(4),
  dailyrate numeric(5,4)
  )
```
## 1.3
```sql
create table test1_deposit(
  cid char(8) not null, 
  bid char(4) not null,
  pid char(18) not null,
  amount numeric(10,2),
  dtime date
  )
```

## 1.4
```sql
insert into test1_deptor
    values('370111197302022021', '王欣','女',40,date '1973-2-2',null)
insert into test1_deptor
    values('370111199303032012', '李军','男',20,date '1993-3-3','370111197302022011')
```


## 1.5
``` sql
insert into test1_bank
    values('0151', '北京中国银行','010-81810505','北京','国有银行',null,'0.0001')
insert into test1_bank
    values('0262', '上海招商银行','021-61619595','上海','股份银行','0151','0.0002')
```

## 1.6
```sql
insert into test1_deposit
    values('51001000', '0151','370111197302022021',10000,to_date('2019-11-11 11:11:11', 'yyyy-mm-dd hh24:mi:ss'))
insert into test1_deposit
    values('62002000', '0262','370111197302022021',20000,to_date('2019-12-12 12:12:12', 'yyyy-mm-dd hh24:mi:ss'))
```

# 实验二
## 2.1
```sql
create or replace view test2_01 as
select *
from bk.deptor
where 
    pname not like '张%'
    and
    pname not like '王%'
    and
    (pname like '%建%' or pname like '%平%')
    and
    extract(year from birthday) = 1971 
order by 
    birthday desc,
    sex asc,
    pname asc; 
```


## 2.2
```sql
create or replace view test2_02 as
select distinct pid, pname, did, bid, bname, amount, dtime, city
from bk.deptor, bk.bank, bk.deposit
where 
    pname like '李龙'
    and
    amount > 1000
    and
    bname like '%北京%'
```

## 2.3
```sql
CREATE OR REPLACE VIEW test2_03 AS
SELECT 
  d.pid, 
  d.pname, 
  d.sex,
  d.age, 
  d.birthday, 
  d.parentpid 
FROM 
  bk.deptor d
WHERE NOT exists (
    SELECT 1
    FROM bk.deposit dep
    WHERE dep.pid = d.pid
)
```

## 2.4
```sql
CREATE OR REPLACE VIEW test2_04 AS
SELECT *
FROM bk.deptor d
WHERE 
  EXISTS (
    SELECT 1
    FROM bk.deposit other_dep
    WHERE 
      other_dep.pid = d.pid  -- 关联当前储户
      AND (TRUNC(other_dep.dtime, 'DD'), other_dep.bid) IN (
        SELECT TRUNC(dtime, 'DD'), bid
        FROM bk.deposit
        WHERE pid = '410101197304071326'  -- 目标用户存款记录
      )
  )
  AND d.pid != '410101197304071326'
```

## 2.5
```sql
CREATE OR REPLACE VIEW test2_05 AS
SELECT 
    child.pid AS childpid,
    child.pname AS childname,
    child.birthday AS childbirthday,
    parent.pid AS parentpid,
    parent.pname AS parentname,
    parent.birthday AS parentbirthday
FROM 
    bk.deptor child  -- 子表别名
JOIN 
    bk.deptor parent  -- 父表别名
ON 
    child.parentpid = parent.pid  -- 通过 parentpid 关联父母与子女
WHERE 
    (child.birthday - parent.birthday) >= 365 * 22  -- 父母比子女大至少 22 年（天数差）
```

## 2.6
```sql
CREATE OR REPLACE VIEW test2_06 AS
SELECT
    child.pid      AS sonpid,
    child.pname    AS sonname,
    child.birthday AS sonbirthday,
    parent.pid     AS parentpid,
    parent.pname   AS parentname,
    parent.birthday AS parentbirthday
FROM bk.deptor child
JOIN bk.deptor parent 
    ON child.parentpid = parent.pid
WHERE
    child.birthday >= parent.birthday + 8030
    AND EXISTS (
        SELECT 1
        FROM bk.deposit d
        WHERE d.pid = child.pid 
          AND d.amount > 6000
    )
    AND EXISTS (
        SELECT 1
        FROM bk.deposit d
        WHERE d.pid = parent.pid 
          AND d.amount > 9000
    )
```


## 2.7
```sql
CREATE OR REPLACE VIEW test2_07 AS
SELECT
    child.*
FROM bk.deptor child
JOIN bk.deptor parent 
    ON child.parentpid = parent.pid
JOIN bk.deptor grand 
    ON parent.parentpid = grand.pid
WHERE EXISTS (
    SELECT 1
    FROM bk.deposit d
    WHERE d.pid = grand.pid
      AND d.amount > 8900
)
```

## 2.8
```sql
CREATE OR REPLACE VIEW test2_08 AS
SELECT 
    d.pid, d.pname, d.sex, d.age, d.birthday, d.parentpid
FROM bk.deptor d
WHERE 
    -- 必须在成都有存款记录
    EXISTS (
        SELECT 1
        FROM bk.deposit dep
        JOIN bk.bank b ON dep.bid = b.bid
        WHERE dep.pid = d.pid
          AND b.city = '成都'
    )
    -- 并且在西宁没有存款记录
    AND NOT EXISTS (
        SELECT 1
        FROM bk.deposit dep
        JOIN bk.bank b ON dep.bid = b.bid
        WHERE dep.pid = d.pid
          AND b.city = '西宁'
    )
```


## 2.9
```sql
CREATE OR REPLACE VIEW test2_09 AS
SELECT 
    d.pid,
    d.pname,
    d.sex,
    d.age,
    d.birthday,
    d.parentpid
FROM bk.deptor d
WHERE 
    EXISTS (
        SELECT 1
        FROM bk.deposit dep
        JOIN bk.bank b ON dep.bid = b.bid
        WHERE dep.pid = d.pid 
          AND b.city = '成都'
          AND dep.amount >= 5000
    )
    AND EXISTS (
        SELECT 1
        FROM bk.deposit dep
        JOIN bk.bank b ON dep.bid = b.bid
        WHERE dep.pid = d.pid 
          AND b.city = '重庆'
          AND dep.amount >= 6000
    );
```
## 2.10
```sql
CREATE OR REPLACE VIEW test2_10 AS
SELECT
    d.pid,
    d.pname,
    d.sex,
    d.age,
    d.birthday,
    d.parentpid
FROM bk.deptor d
WHERE NOT EXISTS (
    SELECT 1
    FROM bk.bank b
    WHERE NOT EXISTS (
        SELECT 1
        FROM bk.deposit dep
        WHERE dep.pid = d.pid
          AND dep.bid = b.bid
    )
)
```

# 实验三
## 3.1
```sql
CREATE TABLE test3_01 AS
SELECT *
FROM bk.deptor3
WHERE REGEXP_LIKE(pid, '^[0-9]+$')
```
## 3.2
```sql
CREATE TABLE test3_02 AS
SELECT *
FROM bk.deptor3;

DELETE FROM test3_02
WHERE TO_CHAR(birthday, 'YYYYMMDD') <> SUBSTR(pid, 7, 8);
```
## 3.3
```sql
CREATE TABLE test3_03 AS
SELECT * FROM bk.deptor3

DELETE FROM test3_03
WHERE 
  sex NOT IN ('男', '女')  -- 直接过滤非标准值
  and sex is not null
```

## 3.4
```sql
CREATE TABLE test3_04 AS
SELECT * FROM bk.deptor3

DELETE FROM test3_04
WHERE pname like '% %'
    or regexp_like(pname,'[A-Za-z0-9+-]')
```



## 3.5
```sql
CREATE TABLE test3_05 AS
SELECT * FROM bk.deptor3
DELETE FROM test3_05
WHERE age != 2019 - EXTRACT(YEAR FROM birthday)
```



## 3.6
```sql
CREATE TABLE test3_06 AS
SELECT *
FROM bk.deptor3
WHERE 
  REGEXP_LIKE(pid, '^[0-9]{17}[0-9X]$')  
  AND TO_DATE(SUBSTR(pid, 7, 8), 'YYYYMMDD') = birthday  
  AND (sex IN ('男', '女')  or sex is null)
  AND NOT REGEXP_LIKE(pname, '[A-Za-z0-9+-]')  
  AND pname not like '% %'  
  AND age = 2019 - EXTRACT(YEAR FROM birthday)
```


## 3.7
```sql
CREATE TABLE test3_07 AS
SELECT d.*
FROM bk.deptor3 d
WHERE EXISTS (
    SELECT 1
    FROM bk.deposit dep
    WHERE dep.pid = d.pid  -- 关联存款表中的储户
)
```


## 3.8
```sql
CREATE TABLE test3_08 AS
SELECT * FROM bk.deptor3
DELETE FROM test3_08 child
WHERE 
    child.parentpid IS NOT NULL  -- 仅处理有父身份证编号的记录
    AND NOT EXISTS (
        SELECT 1
        FROM test3_08 parent
        WHERE parent.pid = child.parentpid  -- 关联父母身份证编号
    );
```


## 3.9
```sql
CREATE TABLE test3_09 AS
SELECT *
FROM bk.deptor3
WHERE sex IN ('男', '女')
-- 步骤2：删除没有异性相同出生日期的记录
DELETE FROM test3_09 t1
WHERE NOT EXISTS (
    SELECT 1
    FROM test3_09 t2
    WHERE 
        t1.birthday = t2.birthday  -- 出生日期相同
        AND t1.sex != t2.sex       -- 性别不同
);
```


## 3.10
```sql
CREATE TABLE test3_10 AS
SELECT * FROM bk.deptor3
-- 步骤2：删除在 bk.deptor 中不存在的身份证+姓名组合
DELETE FROM test3_10 t
WHERE NOT EXISTS (
    SELECT 1
    FROM bk.deptor d
    WHERE 
        d.pid = t.pid    -- 身份证编号一致
        AND d.pname = t.pname  -- 姓名一致
);
```

# 实验四
## 4.1
```sql
CREATE TABLE test4_01 AS
SELECT * FROM bk.deptor3;

UPDATE test4_01
SET pid = 
    SUBSTR(pid, 1, 6) ||              -- 保留前6位地区码
    TO_CHAR(birthday, 'YYYYMMDD') ||   -- 更新为规范出生日期
    SUBSTR(pid, 15)                    -- 保留后4位

UPDATE test4_01
SET sex = '男'
WHERE 
    sex like '%男%'

UPDATE test4_01
SET sex = '女'
WHERE 
    sex like '%女%'

UPDATE test4_01
SET sex = '% %'
WHERE 
    sex like ''

UPDATE test4_01
SET pname = REGEXP_REPLACE(pname, '[a-zA-Z0-9+-]', '')  -- 正则过滤非中文字符

```

## 4.2
```sql
CREATE TABLE test4_02 AS
SELECT * FROM bk.deptor3;

UPDATE test4_02
SET age = 2019 - EXTRACT(YEAR FROM birthday)
WHERE 
    2019 - EXTRACT(YEAR FROM birthday) < age;  -- 仅修正年龄小于计算值的记录
```


## 4.3
```sql
CREATE TABLE test4_03 AS SELECT * FROM bk.deptor3
ALTER TABLE test4_03 ADD (
    total_count NUMBER,
    total_amount NUMBER,
    avg_amount NUMBER(10,2)
)
UPDATE test4_03 t
SET (total_count, total_amount, avg_amount) = (
    SELECT 
        COUNT(d.did),
        SUM(d.amount),
        AVG(d.amount)
    FROM bk.deposit d
    WHERE d.pid = t.pid
)
```



## 4.4
```sql
-- 步骤1：复制表结构及数据
CREATE TABLE test4_04 AS
SELECT * FROM bk.deptor3;

-- 步骤2：添加新列
ALTER TABLE test4_04
ADD (
    total_count NUMBER,
    total_amount NUMBER(10,2),
    avg_amount NUMBER(10,2)
);

-- 步骤3：仅更新有存单记录的储户
UPDATE test4_04 t
SET 
    total_count = (
        SELECT COUNT(*)
        FROM bk.deposit d
        WHERE d.pid = t.pid
    ),
    total_amount = (
        SELECT SUM(amount)
        FROM bk.deposit d
        WHERE d.pid = t.pid
    ),
    avg_amount = (
        SELECT ROUND(AVG(amount), 2)
        FROM bk.deposit d
        WHERE d.pid = t.pid
    )
WHERE EXISTS (
    SELECT 1
    FROM bk.deposit d
    WHERE d.pid = t.pid  -- 仅处理有存单记录的储户
);
```


## 4.5
```sql
CREATE TABLE test4_05 AS SELECT * FROM bk.deptor3;

ALTER TABLE test4_05
ADD (
    total_count INT,
    total_amount NUMBER(15,2),
    avg_amount NUMBER(10,2)
);

UPDATE test4_05 t
SET (total_count, total_amount, avg_amount) = (
    SELECT 
        COUNT(*) AS total_count,
        SUM(amount) AS total_amount,
        CASE 
            WHEN COUNT(*) = 0 THEN null
            ELSE SUM(amount) / COUNT(*) 
        END AS avg_amount
    FROM bk.deposit d
    WHERE d.pid IN (t.pid, t.parentpid)
);
```


## 4.6
```sql
CREATE TABLE test4_06 AS SELECT * FROM bk.deptor3;
ALTER TABLE test4_06
ADD COLUMN total_count INT,
ADD COLUMN total_amount DECIMAL(12,2),
ADD COLUMN avg_amount DECIMAL(12,2);
UPDATE test4_06 t
SET 
    total_count = (
        SELECT COUNT(d.did)
        FROM bk.deposit d
        WHERE d.pid = t.pid OR d.pid = t.parentpid
    ),
    total_amount = (
        SELECT SUM(d.amount)
        FROM bk.deposit d
        WHERE d.pid = t.pid OR d.pid = t.parentpid
    ),
    avg_amount = (
        SELECT AVG(d.amount)
        FROM bk.deposit d
        WHERE d.pid = t.pid OR d.pid = t.parentpid
    )
WHERE EXISTS (
    SELECT 1
    FROM bk.deposit d
    WHERE d.pid = t.parentpid
)

UPDATE test4_06 t
SET 
    total_count = null,
    total_amount = null,
    avg_amount = null
WHERE t.pid like '500101196707010559'
```



## 4.7
```sql
-- 将 bk.deptor3 表复制到主用户的 test4_07 表中
CREATE TABLE test4_07 AS SELECT * FROM bk.deptor3;
-- 添加总利息列，保留两位小数
ALTER TABLE test4_07 
ADD COLUMN total_interest DECIMAL(12,2);
-- 创建临时表存储利息计算结果
CREATE TABLE tmp_interest AS
SELECT 
    d.pid,
    SUM(ROUND(d.amount * (TRUNC(SYSDATE) - TRUNC(d.dtime)) * b.dailyrate, 2)) AS total_interest
FROM bk.deposit d
JOIN bk.bank b ON d.bid = b.bid
GROUP BY d.pid;

-- 使用临时表更新 test4_07
UPDATE test4_07 t
SET total_interest = (
    SELECT COALESCE(total_interest, 0)
    FROM tmp_interest
    WHERE pid = t.pid
);
```

## 4.8
```sql
CREATE TABLE test4_08 AS
SELECT pid, pname, sex, age, birthday, parentpid
FROM bk.deptor3
WHERE sex IN ('男', '女')

ALTER TABLE test4_08
ADD birthday1 DATE

UPDATE test4_08 t1
SET birthday1 = (
    SELECT MAX(t2.birthday)
    FROM test4_08 t2
    WHERE TO_CHAR(t2.birthday, 'MMDD') = TO_CHAR(t1.birthday, 'MMDD')
)
```


## 4.9
```sql
CREATE TABLE test4_09 AS 
SELECT pid, pname, sex, age, birthday, parentpid 
FROM bk.deptor3 
WHERE sex IN ('男', '女');
ALTER TABLE test4_09 
ADD pid1 VARCHAR2(20);  -- 假设 pid 为字符串类型
UPDATE test4_09 t1
SET pid1 = COALESCE(
    (
        SELECT MAX(t3.pid)
        FROM test4_09 t3
        WHERE 
            -- 条件 1: 月日相同
            TO_CHAR(t3.birthday, 'MMDD') = TO_CHAR(t1.birthday, 'MMDD')
            -- 条件 2: 年龄等于该生日组的最小年龄
            AND TO_CHAR(t3.birthday, 'YYYY') = (
                -- 子查询 2: 计算同生日组的最小年龄
                SELECT MAX(TO_CHAR(t2.birthday, 'YYYY'))
                FROM test4_09 t2
                WHERE TO_CHAR(t2.birthday, 'MMDD') = TO_CHAR(t1.birthday, 'MMDD')
            )
    ),
    t1.pid
);
```


## 4.10
```sql
-- 复制 bk.deptor3 中性别为男或女的数据到主用户的 test4_10 表
CREATE TABLE test4_10 AS 
SELECT pid, pname, sex, age, birthday, parentpid 
FROM bk.deptor3 
WHERE sex IN ('男', '女');
-- 为 test4_10 表添加用于存储「异性同生日最晚出生日期的最大学号」的列
ALTER TABLE test4_10 
ADD COLUMN pid1 VARCHAR2(20);  -- 假设 pid 为字符串类型
-- 更新男同学的 pid1（指向同生日女同学中最晚出生日期的最大 pid）
UPDATE test4_10 t1
SET pid1 = COALESCE(
    (
        SELECT MAX(t3.pid)
        FROM test4_10 t3
        WHERE 
            -- 条件 1: 月日相同
            TO_CHAR(t3.birthday, 'MMDD') = TO_CHAR(t1.birthday, 'MMDD')
            -- 条件 2: 性别为女
            AND t3.sex = '女'
            -- 条件 3: 出生日期等于该生日组女同学的最大日期
            AND t3.birthday = (
                SELECT MAX(t2.birthday)
                FROM test4_10 t2
                WHERE 
                    TO_CHAR(t2.birthday, 'MMDD') = TO_CHAR(t1.birthday, 'MMDD')
                    AND t2.sex = '女'
            )
    ),
    null  -- 若没有符合条件的女同学，pid1 设为自身
)
WHERE t1.sex = '男';

-- 更新女同学的 pid1（指向同生日男同学中最晚出生日期的最大 pid）
UPDATE test4_10 t1
SET pid1 = COALESCE(
    (
        SELECT MAX(t3.pid)
        FROM test4_10 t3
        WHERE 
            TO_CHAR(t3.birthday, 'MMDD') = TO_CHAR(t1.birthday, 'MMDD')
            AND t3.sex = '男'
            AND t3.birthday = (
                SELECT MAX(t2.birthday)
                FROM test4_10 t2
                WHERE 
                    TO_CHAR(t2.birthday, 'MMDD') = TO_CHAR(t1.birthday, 'MMDD')
                    AND t2.sex = '男'
            )
    ),
    null
)
WHERE t1.sex = '女';
```
# 实验五
## 5.1
```sql
CREATE TABLE test5_01 AS
SELECT 
    b.bname AS Bname,
    b.bid AS bid,
    COUNT(d.did) AS dcount,
    COUNT(DISTINCT d.pid) AS pcount,
    SUM(d.amount) AS sum_amount,
    ROUND(AVG(d.amount), 2) AS avg_amount
FROM 
    bk.deposit d
    JOIN bk.bank b ON d.bid = b.bid
    JOIN bk.deptor de ON d.pid = de.pid
GROUP BY 
    b.bid, b.bname
```

## 5.2
```sql
CREATE TABLE test5_02 AS
SELECT 
    b.city AS city,
    -- 判断是否为城市合计行，设置bid为'0000'
    CASE 
        WHEN GROUPING(b.bid) = 1 THEN '0000' 
        ELSE b.bid 
    END AS bid,
    -- 判断是否为城市合计行，设置Bname为'合计'
    CASE 
        WHEN GROUPING(b.bname) = 1 THEN '合计' 
        ELSE b.bname 
    END AS Bname,
    COUNT(d.did) AS dcount,  -- 存款次数
    SUM(d.amount) AS sum_amount  -- 存款总额
FROM 
    bk.deposit d
    JOIN bk.bank b ON d.bid = b.bid
    JOIN bk.deptor de ON d.pid = de.pid
-- 按城市分组，同时生成城市合计行和分银行明细
GROUP BY GROUPING SETS (
    (b.city, b.bid, b.bname),  -- 分银行明细
    (b.city)                   -- 城市合计行
)
ORDER BY 
    b.city, 
    bid
```
## 5.3
```sql
CREATE TABLE test5_03 AS
WITH deposit_data AS (
  SELECT 
    TRUNC(EXTRACT(YEAR FROM de.birthday), -1) || '后' AS generation,
    d.did,
    d.amount
  FROM bk.deposit d
  JOIN bk.deptor de ON d.pid = de.pid
  JOIN bk.bank b ON d.bid = b.bid
)
SELECT 
  generation,
  COUNT(did) AS dcount,
  SUM(amount) AS sum_amount,
  total_count,
  ROUND(COUNT(did)/total_count * 100, 2) AS count_percent,
  total_amount,
  ROUND(SUM(amount)/total_amount * 100, 2) AS amount_percent
FROM deposit_data
CROSS JOIN (
  SELECT 
    COUNT(did) AS total_count,
    SUM(amount) AS total_amount 
  FROM deposit_data
) totals
GROUP BY generation, total_count, total_amount
ORDER BY generation;
```
## 5.4
```sql
CREATE TABLE test5_04 AS
SELECT 
    b.city AS "CITY",
    -- 国有银行统计
    COUNT(CASE WHEN b.stru = '国有银行' THEN d.did END) AS "DCOUNT1",
    ROUND(SUM(CASE WHEN b.stru = '国有银行' THEN d.amount ELSE 0 END), 2) AS "SUM_AMOUNT1",
    ROUND(SUM(CASE WHEN b.stru = '国有银行' THEN d.amount ELSE 0 END) / 
        NULLIF(SUM(d.amount), 0) * 100, 2) AS "AMOUNT_PERCENT1",
    -- 股份银行统计
    COUNT(CASE WHEN b.stru = '股份银行' THEN d.did END) AS "DCOUNT2",
    ROUND(SUM(CASE WHEN b.stru = '股份银行' THEN d.amount ELSE 0 END), 2) AS "SUM_AMOUNT2",
    ROUND(SUM(CASE WHEN b.stru = '股份银行' THEN d.amount ELSE 0 END) / 
        NULLIF(SUM(d.amount), 0) * 100, 2) AS "AMOUNT_PERCENT2",
    -- 合计统计
    COUNT(d.did) AS "DCOUNT",
    SUM(d.amount) AS "SUM_AMOUNT"
FROM 
    bk.deposit d
    JOIN bk.bank b ON d.bid = b.bid
    JOIN bk.deptor de ON d.pid = de.pid
GROUP BY 
    b.city
ORDER BY 
    b.city
```
## 5.5
```sql
CREATE OR REPLACE VIEW test5_05 AS
SELECT 
    SUBSTR(b.bname,3,4) AS Bankname,  -- 提取银行名称（后四位）
    -- 北京地区统计（前两位=北京）
    COUNT(CASE WHEN SUBSTR(b.bname,1,2) = '北京' THEN d.did END) AS dcount1,
    ROUND(SUM(CASE WHEN SUBSTR(b.bname,1,2) = '北京' THEN d.amount ELSE 0 END),2) AS sum_amount1,
    -- 上海地区统计（前两位=上海）
    COUNT(CASE WHEN SUBSTR(b.bname,1,2) = '上海' THEN d.did END) AS dcount2,
    ROUND(SUM(CASE WHEN SUBSTR(b.bname,1,2) = '上海' THEN d.amount ELSE 0 END),2) AS sum_amount2
FROM 
    bk.deposit d
    INNER JOIN bk.bank b ON d.bid = b.bid
    INNER JOIN bk.deptor de ON d.pid = de.pid
GROUP BY 
    SUBSTR(b.bname,3,4)  -- 按银行名称分组
ORDER BY 
    SUBSTR(b.bname,3,4);
```
## 5.6
```sql
CREATE OR REPLACE VIEW test5_06 AS
WITH deposit_sum AS (
  SELECT 
    d.bid,
    d.pid,
    ROUND(SUM(d.amount), 2) AS sum_amount
  FROM bk.deposit d
  JOIN bk.deptor de ON d.pid = de.pid
  GROUP BY d.bid, d.pid
)
SELECT 
  bid,
  pid,
  sum_amount
FROM (
  SELECT 
    deposit_sum.*,
    DENSE_RANK() OVER (
      PARTITION BY bid 
      ORDER BY sum_amount DESC
    ) AS amount_rank
  FROM deposit_sum
)
WHERE amount_rank = 1
ORDER BY bid;
```
## 5.7
```sql
CREATE OR REPLACE VIEW test5_07 AS
SELECT 
    TO_CHAR(TRUNC(amount, -3), 'FM0000') || '-' || TO_CHAR(TRUNC(amount, -3) + 999, 'FM0000') AS "AMOUNT",
    COUNT(did) AS "DCOUNT",
    SUM(amount) AS "SUM_AMOUNT"
FROM 
    bk.deposit
WHERE 
    amount > 1999 
    AND amount < 10000
GROUP BY 
    TRUNC(amount, -3)
ORDER BY 
    TRUNC(amount, -3);
```

## 5.8
```sql
CREATE OR REPLACE VIEW test5_08 AS
SELECT 
    b.city AS CITY,
    TRUNC(EXTRACT(YEAR FROM de.birthday), -1) || '后' AS GENERATION,
    EXTRACT(YEAR FROM d.dtime) AS DYEAR,
    COUNT(d.did) AS DCOUNT,
    SUM(d.amount) AS SUM_AMOUNT
FROM 
    bk.deposit d
    JOIN bk.bank b ON d.bid = b.bid
    JOIN bk.deptor de ON d.pid = de.pid
GROUP BY 
    b.city,
    TRUNC(EXTRACT(YEAR FROM de.birthday), -1),
    EXTRACT(YEAR FROM d.dtime)
ORDER BY 
    b.city, 
    GENERATION, 
    DYEAR;
```
## 5.9
```sql
CREATE OR REPLACE VIEW test5_09 AS
SELECT 
    de.pid AS "PID",
    de.pname AS "PNAME",
    de.sex AS "SEX",
    COUNT(d.did) AS "DCOUNT",
    SUM(d.amount) AS "SUM_AMOUNT"
FROM 
    bk.deposit d
    JOIN bk.deptor de ON d.pid = de.pid
WHERE 
    de.sex = '女'              -- 性别过滤条件
    AND d.amount > 2000       -- 金额过滤条件
GROUP BY 
    de.pid, de.pname, de.sex
HAVING 
    COUNT(d.did) > 130        -- 存单数量过滤
    AND SUM(d.amount) < 900000 -- 存款总额过滤
ORDER BY 
    de.pid;
```
## 5.10
```sql
CREATE OR REPLACE VIEW test5_10 AS
SELECT 
    generation AS "GENERATION",
    COUNT(pid) AS "PCOUNT",
    SUM(total_amount) AS "SUM_AMOUNT"
FROM (
    -- 步骤1：筛选存款总额超50万的客户
    SELECT 
        de.pid,
        TRUNC(EXTRACT(YEAR FROM de.birthday), -1) || '后' AS generation,
        SUM(d.amount) AS total_amount
    FROM 
        bk.deposit d
        JOIN bk.deptor de ON d.pid = de.pid
    GROUP BY 
        de.pid, 
        TRUNC(EXTRACT(YEAR FROM de.birthday), -1)
    HAVING 
        SUM(d.amount) > 500000
) qualified_customers
GROUP BY 
    generation
ORDER BY 
    generation
```
# 实验六
## 6.1
```sql
CREATE OR REPLACE VIEW test6_01 AS
SELECT 
    SUBSTR(pname, 2) AS First_name,
    COUNT(*) AS frequency
FROM 
    bk.deptor
WHERE 
    pname IS NOT NULL
GROUP BY 
    SUBSTR(pname, 2)
ORDER BY 
    frequency DESC
```

## 6.2
```sql
CREATE OR REPLACE VIEW test6_02 AS
SELECT 
    letter,
    COUNT(*) AS frequency
FROM (
    -- 提取第二个字
    SELECT SUBSTR(pname, 2, 1) AS letter
    FROM bk.deptor
    WHERE LENGTH(pname) >= 2  -- 确保有名字部分
    
    UNION ALL
    
    -- 提取第三个字（仅处理三字姓名）
    SELECT SUBSTR(pname, 3, 1)
    FROM bk.deptor
    WHERE LENGTH(pname) >= 3
)
WHERE letter IS NOT NULL       -- 过滤空字符
GROUP BY letter
ORDER BY frequency DESC;
```

## 6.3
```sql
CREATE VIEW test6_03 AS
SELECT *
FROM bk.deptor d
WHERE d.sex = '男'
  AND d.age = 40
  AND d.pid IN (
      SELECT dp.pid
      FROM bk.deposit dp
      GROUP BY dp.pid
      HAVING SUM(dp.amount) > 600000
  )
```

## 6.4
```sql
CREATE VIEW test6_04 AS
SELECT d.pid, d.pname, SUM(dp.amount) AS sum_amount
FROM bk.deptor d
JOIN bk.deposit dp ON d.pid = dp.pid
JOIN bk.bank b ON dp.bid = b.bid
WHERE b.city = '北京'
GROUP BY d.pid, d.pname
HAVING SUM(dp.amount) > 500000;
```

## 6.5
```sql
CREATE OR REPLACE VIEW test6_05 AS
WITH valid_deposits AS (
    SELECT 
        d.bid,
        d.amount,
        d.did
    FROM 
        bk.deposit d
        JOIN bk.deptor de ON d.pid = de.pid  -- 确保有效客户
),
bank_max AS (
    SELECT 
        bid, 
        MAX(amount) AS max_amount
    FROM 
        valid_deposits
    GROUP BY 
        bid
)
SELECT 
    b.bid,
    b.bname,
    bm.max_amount,
    COUNT(vd.did) AS max_amount_count
FROM 
    bk.bank b
    JOIN bank_max bm ON b.bid = bm.bid
    JOIN valid_deposits vd ON vd.bid = bm.bid 
                            AND vd.amount = bm.max_amount
GROUP BY 
    b.bid, 
    b.bname, 
    bm.max_amount
ORDER BY 
    b.bid;
```

## 6.6
```sql
CREATE OR REPLACE VIEW test6_06 AS
SELECT 
    de.* 
FROM 
    bk.deptor de
WHERE 
    de.sex = '女'
    AND de.age = 40
    AND EXISTS (
        SELECT 1
        FROM bk.deposit d
        WHERE d.pid = de.pid
        AND d.amount > 6000
        GROUP BY d.bid
        HAVING COUNT(d.did) > 3
    );
```

## 6.7
```sql
CREATE VIEW test6_07 AS
SELECT d.pid, d.pname, d.sex, d.age, d.birthday, d.parentpid
FROM bk.deptor d
JOIN bk.deposit dp ON d.pid = dp.pid
WHERE d.sex = 'F'
  AND d.age = 40
  AND dp.amount > 200
GROUP BY d.pid, d.pname, d.sex, d.age, d.birthday, d.parentpid
HAVING COUNT(DISTINCT dp.bid) = (SELECT COUNT(*) FROM bk.bank);
```

## 6.8
```sql
CREATE VIEW test6_08 AS
SELECT d.pid, d.pname, d.sex, d.age, d.birthday, d.parentpid
FROM bk.deptor d
WHERE d.parentpid IN (
    SELECT parentpid
    FROM bk.deptor
    GROUP BY parentpid
    HAVING COUNT(*) > 1
);
```
## 6.9
```sql
CREATE OR REPLACE VIEW test6_09 AS
SELECT 
    de.* 
FROM 
    bk.deptor de
WHERE 
    de.sex = '女'
    AND FLOOR(MONTHS_BETWEEN(SYSDATE, de.birthday)/12 = 40  -- 精确计算40周岁
    AND EXISTS (
        -- 在北京存款总额超过30万
        SELECT 1
        FROM bk.deposit d
        JOIN bk.bank b ON d.bid = b.bid
        WHERE d.pid = de.pid
          AND b.city = '北京'
        GROUP BY d.pid
        HAVING SUM(d.amount) > 300000
    )
    AND NOT EXISTS (
        -- 排除西宁有存款记录
        SELECT 1
        FROM bk.deposit d
        JOIN bk.bank b ON d.bid = b.bid
        WHERE d.pid = de.pid
          AND b.city = '西宁'
    );
```

## 6.10
```sql
CREATE VIEW test6_10 AS
SELECT pid, rank, sum_amount
FROM (
    SELECT 
        d.pid,
        ROW_NUMBER() OVER (ORDER BY SUM(dp.amount) DESC) AS rank,
        SUM(dp.amount) AS sum_amount
    FROM bk.deptor d
    JOIN bk.deposit dp ON d.pid = dp.pid
    GROUP BY d.pid
) subquery
WHERE rank <= 20;
```

