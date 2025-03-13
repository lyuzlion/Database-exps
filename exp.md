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

## 实验四
### 4.1
```sql
CREATE TABLE test4_01 AS
SELECT * FROM bk.deptor3;

UPDATE test4_01
SET pid = 
    SUBSTR(pid, 1, 6) ||              -- 保留前6位地区码
    TO_CHAR(birthday, 'YYYYMMDD') ||   -- 更新为规范出生日期
    SUBSTR(pid, 15)                    -- 保留后4位

UPDATE test4_01
SET sex = 
    REPLACE(REPLACE(sex,
        '男性', '男'),   -- 替换字母标识
        '女性', '女')

UPDATE test4_01
SET pname = REGEXP_REPLACE(pname, '[ a-zA-Z0-9+-]', '');  -- 正则过滤非中文字符
```

### 4.2
```sql
CREATE TABLE test4_02 AS
SELECT * FROM bk.deptor3;

UPDATE test4_02
SET age = 2019 - EXTRACT(YEAR FROM birthday)
WHERE 
    2019 - EXTRACT(YEAR FROM birthday) < age;  -- 仅修正年龄小于计算值的记录
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
