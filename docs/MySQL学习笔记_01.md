
## MySQL实现序列(Sequence)效果

一般使用序列(Sequence)来处理主键字段，在mysql中是没有序列的，但是MySQL有提供了自增长(increment)来实现类似的目的，但也只是自增，而不能设置步长、开始索引、是否循环等，最重要的是一张表只能由一个字段使用自增，但有的时候我们需要两个或两个以上的字段实现自增（单表多字段自增），MySQL本身是实现不了的，但我们可以用创建一个序列表，使用函数来获取序列的值

### 创建序列表
```sql
-- MySQL创建序列(没有sequence对象，需要借助序列表)
drop table if exists sequence;
CREATE TABLE sequence (
  seq_name VARCHAR(50) not null, -- 序列名称
  current_value INT not null, -- 当前值
  increment_val INT not null DEFAULT 1, -- 步长为1
  PRIMARY KEY(seq_name)
);
```
### 新增序列
```sql
-- 新增3个序列
INSERT INTO sequence VALUES ('seq_test1_num1', '0', '1');
INSERT INTO sequence VALUES ('seq_test1_num2', '0', '2');
INSERT INTO sequence VALUES ('seq_employee_num1', '0', '1');
```
### 创建函数 
```sql
-- 创建函数用于获取序列当前值(v_seq_name 参数名 代表序列名称)
CREATE FUNCTION currval(v_seq_name VARCHAR(50))
RETURNS INTEGER
BEGIN
  DECLARE val integer;
  SET val = 0;
  SELECT current_value INTO val FROM sequence where seq_name = v_seq_name;
  RETURN val;
END
```
### 查询当前值
```sql
SELECT currval('seq_test1_num1');
```
### 创建函数用于获取序列下一个值
```sql
-- 创建函数用于获取序列下一个值(v_seq_name 参数名 代表序列名称)
CREATE FUNCTION nextval(v_seq_name varchar(50))
RETURNS INTEGER
BEGIN
  UPDATE sequence SET current_value = current_value + increment_val 
    WHERE seq_name = v_seq_name;
  RETURN currval(v_seq_name);
END
```
### 查询下一个值
```sql
SELECT nextval('seq_employee_num1');
```
### 新建触发器
```sql
-- 新建触发器 插入新纪录前给自增字段赋值实现字段自增效果
CREATE TRIGGER TRI_employee_num_1 before INSERT ON employee for EACH ROW 
BEGIN
  SET NEW.e_no = nextval('seq_employee_num1');
END

-- 最后测试自增效果
INSERT INTO employee VALUES (currval('seq_employee_num1'), '李四', 24, 1200.00, '东三旗', 20);

```
