# MySQL 刷题 - 牛客网

## TopN 问题

> 该类问题可以用 **窗口函数** 或者 **自连接** 的方法来解决。

### 窗口函数

除了如下的 **SQL196** 题，类似的题还有

[SQL217(对所有员工的薪水按照salary降序进行1-N的排名)](https://www.nowcoder.com/practice/b9068bfe5df74276bd015b9729eec4bf?tpId=82&tags=&title=SQL217&difficulty=&judgeStatus=&rp=1&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26tab%3DSQL%25E7%25AF%2587%26topicId%3D82&gioEnter=menu)

[SQL206(获取每个部门中当前员工薪水最高的相关信息)](https://www.nowcoder.com/practice/4a052e3e1df5435880d4353eb18a91c6?tpId=82&tags=&title=SQL206&difficulty=&judgeStatus=&rp=1&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26tab%3DSQL%25E7%25AF%2587%26topicId%3D82&gioEnter=menu)

[SQL261(牛客每个人最近的登录日期(二))](https://www.nowcoder.com/practice/7cc3c814329546e89e71bb45c805c9ad?tpId=82&tags=&title=&difficulty=0&judgeStatus=3&rp=1&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26tab%3DSQL%25E7%25AF%2587%26topicId%3D82)

都可以使用窗口函数来解决。

#### SQL196 查找入职员工时间排名倒数第三的员工所有信息

分析：入职时间排名倒数第三，第一种方法，按照入职时间倒序排序，并且去除重复的日期时间（使用 `group by` 或者 `distinct`），拿到倒数第三的日期时间，再根据这个日期时间查询当天入职的所有员工信息。第二种方法，使用窗口函数 `dense_rank()`（并列排序，不会跳过重复序号）。

##### 代码一：group by & limit

```mysql
SELECT *
FROM employees
where hire_date = (SELECT hire_date FROM employees group by hire_date order by hire_date desc limit 2, 1);
```

##### 代码二：窗口函数

```mysql
SELECT emp_no, birth_date, first_name, last_name, gender, hire_date
FROM (SELECT emp_no,
             birth_date,
             first_name,
             last_name,
             gender,
             hire_date,
             dense_rank() over (order by hire_date desc) as 'rank'
      FROM employees) t
where t.rank = 3;
```

### 自连接

#### SQL212 获取当前薪水第二多的员工信息

##### 解法一：窗口函数

由于这种方法使用到了 `order by` 不符合题目要求，所以没通过，但不是说这种方法不是一个好方法。

```mysql
SELECT
  emp_no,
  salary,
  last_name,
  first_name
FROM
(
    SELECT
      e.emp_no,
      s.salary,
      e.last_name,
      e.first_name,
      dense_rank() over (
        order by
          s.salary desc
      ) as salary_rank
    FROM
      employees e
      left join salaries s on e.emp_no = s.emp_no
    where
      s.to_date = '9999-01-01'
  ) t
where
  t.salary_rank = 2;
```

##### 解法二：自连接

这道题使用自连接解法非常有代表性，能让你了解到自连接也可以用于解决 **TopN** 问题。

```mysql
SELECT e.emp_no, s.salary, e.last_name, e.first_name
FROM employees e
         left join salaries s on e.emp_no = s.emp_no
WHERE salary = (select s1.salary
                from salaries s1
                         join salaries s2 -- 自连接查询
                              on s1.salary <= s2.salary
                where s1.to_date = '9999-01-01'
                  and s2.to_date = '9999-01-01'
                group by s1.salary -- 当s1<=s2链接并以s1.salary分组时一个s1会对应多个s2
                having count(distinct s2.salary) = 2 -- (去重之后的数量就是对应的名次)
);
```

## N 日留存率问题

> 该类题目需要用到 **自连接** 或 **窗口函数** 来解决。
>
> 该类问题一般只会给一张表，所以肯定需要用到自己和自己关联。

### SQL262 牛客新登录用户的次日成功的留存率

> 题目：[SQL262 牛客新登录用户的次日成功的留存率](https://www.nowcoder.com/practice/16d41af206cd4066a06a3a0aa585ad3d?tpId=82&tags=&title=&difficulty=0&judgeStatus=0&rp=1&sourceUrl=%2Fexam%2Foj%3Fpage%3D2%26tab%3DSQL%25E7%25AF%2587%26topicId%3D82)
>
> 分析：先要获取每个用户第一次登录的日期形成一张表 A，然后再是次日留存（意思就是第二天接着登录），那么需要将 A 表 `left join` 原表，关联条件为同一个用户，原表的日期=第一次登录的日期 +1 天，这样的话，就可以从结果直接看出今天登录，次日有没有登录，没登录的话次日登录的日期就为 `NULL`。

题目答案：

```mysql
select round(count(l2.date) / count(l1.user_id), 3) as p
from (select user_id,
             min(date) as min_date
      from login
      group by user_id) l1
         left join (select user_id,
                           date
                    from login) l2 on l1.user_id = l2.user_id
    and l2.date = l1.min_date + INTERVAL 1 DAY;
```

### SQL264 统计一下牛客每个日期新用户的次日留存率

> 题目：[SQL264 统计一下牛客每个日期新用户的次日留存率](https://www.nowcoder.com/practice/ea0c56cd700344b590182aad03cc61b8?tpId=82&tags=&title=&difficulty=&judgeStatus=3&rp=1&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26tab%3DSQL%25E7%25AF%2587%26topicId%3D82&gioEnter=menu)
>
> 分析：[题解 | #牛客每个人最近的登录日期(五)](https://blog.nowcoder.net/n/33a427ea69e448d89f877f626ce84e5f)
>
> 1. 其实与上面一道题不同的地方在于该题使用的是 **窗口函数** 来获取每个用户第一次登录的日期形成表 A
>
>    ```mysql
>    select user_id, date, row_number() over (partition by user_id order by date) t_rank
>    from login
>    ```
>
>    ![Snipaste_2022-10-28_06-29-46](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202311082150893.png)
>
> 2. 然后就与前面一道题一样了，A 表 `left join` 原表，关联条件为同一个用户，原表的日期=第一次登录的日期 +1 天
>
>    ```mysql
>    select t.*, l.date
>        from (select user_id, date, row_number() over (partition by user_id order by date) t_rank
>              from login
>             ) t left join login l on t.user_id = l.user_id and l.date = t.date + INTERVAL 1 DAY;
>    ```
>
>    ![Snipaste_2022-10-28_06-33-04](https://fastly.jsdelivr.net/gh/xihuanxiaorang/img/202311082150214.png)
>
> 至此，就已经拿到了想要的全部数据，比如说 **当天有几个新用户** 以及 **该新用户第二天是否登录**，最终只需要根据题目要求来进行统计就行。
>
> 其实前一个题目也可以用 **窗口函数** 这种方法来获取第一次登录的日期，这种方法是不是一种用来解决次日留存率问题的公式呢？

题目答案：

```mysql
select t.date,
       round(IFNULL(sum(IF(t_rank = 1, IF(l.date, 1, 0), 0)) / sum(IF(t_rank = 1, 1, 0)), 0), 3) as p
from (select user_id, date, row_number() over (partition by user_id order by date) t_rank
      from login) t
         left join login l on t.user_id = l.user_id and l.date = t.date + INTERVAL 1 DAY
group by t.date
order by t.date;
```

## 中位数问题

### 序列求中位数

> 题目：[SQL269 考试分数(五)](https://www.nowcoder.com/practice/b626ff9e2ad04789954c2132c74c0512)，查询各个岗位分数的中位数位置上的所有 grade 信息，并且按 id 升序排序
>

方法一：

1. 既然是各个岗位的中位数位置，那么肯定需要按照岗位进行分组，接着 **求各个岗位中位数所在位置**，**使用 `round(count(*) / 2)` 找到起始位置，`round((count(*) + 1) / 2)` 找到结束位置** => 表 A
1. 使用窗口函数获取各个岗位的分数排名 => 表 B
2. 两表关联，条件为岗位相同，并且表 B 中的排名要等于表 A 的起始位置或者结束位置，也就是中位数所在位置

```mysql
select id,
       job,
       score,
       t_rank
from (select g.id,
             g.job,
             g.score,
             row_number() over (
                 partition by g.job
                 order by
                     score desc
                 ) t_rank,
             start_id,
             end_id
      from grade g
               left join (select job,
                                 round(count(*) / 2)       as start_id,
                                 round((count(*) + 1) / 2) as end_id
                          from grade
                          group by job) t on g.job = t.job) g2
where g2.t_rank = start_id
   or g2.t_rank = end_id
order by id
```

方法二：用一条规则统一奇数个数时和偶数个数时的中位数位置。无论奇偶，中位数的位置距离 ((个数 +1)/2) < 1。

```mysql
select id, job, score, t_rank
from (select id,
             job,
             score,
             row_number() over (partition by job order by score desc) t_rank,
             count(*) over (partition by job)                         num
      from grade) t
where abs(t_rank - ((num + 1) / 2)) < 1
order by id;
```

### 累加和求中位数

> 题目：[SQL282 最差是第几名(二)](https://www.nowcoder.com/practice/165d88474d434597bcd2af8bf72b24f1?tpId=82&tags=&title=&difficulty=0&judgeStatus=0&rp=1&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26tab%3DSQL%25E7%25AF%2587%26topicId%3D82)
>
> 分析：当某一个数的【正序和】和【逆序和】均大于等于整个序列的数字个数的一半即为中位数。

```mysql
select grade
from (select grade,
             sum(number) over (order by grade)      a,
             sum(number) over (order by grade desc) b,
             (select sum(number) from class_grade)  total
      from class_grade) t
where a >= total / 2
  and b >= total / 2
order by grade;
```

## 数据重复问题

### 删除重复的记录，保留 id 最小的记录

> 题目：[SQL236 删除emp_no重复的记录，只保留最小的id对应的记录](https://www.nowcoder.com/practice/3d92551a6f6d4f1ebde272d20872cf05?tpId=82&tags=&title=&difficulty=0&judgeStatus=0&rp=1&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26tab%3DSQL%25E7%25AF%2587%26topicId%3D82)
>
> 思路：根据某个字段分组，使用聚合函数 `min(id)` 找出最小的记录，然后删除不在里面的记录。SQL 如下所示：
>
> ```mysql
> delete from 表名 where id not in(
> 	select * form(
>     	select min(id) from 表名 group by 分组字段
>     ) t
> );
> ```
>
> 需要注意的是：**MySQL 不允许在子查询的同时删除原表数据**，**所以需要把从原始表中查询出来的 id 作为一个临时表（test）**，**再把从临时表中取出来的 id 作为条件，之后在原始表里删数据**

```mysql
delete
from titles_test
where id not in (select *
                 from (select min(id)
                       from titles_test
                       group by emp_no) t);
```

## 字符串

### 字符串拆分

> 题目：[SQL246 获取employees中的first_name](https://www.nowcoder.com/practice/74d90728827e44e2864cce8b26882105?tpId=82&tags=&title=&difficulty=0&judgeStatus=3&rp=1&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26tab%3DSQL%25E7%25AF%2587%26topicId%3D82)
>
> 考察知识点：字符串拆分
>
> 1. left(str, n)：从左开始分割，截取 n 位
> 2. right(str, n)：从右开始分割，截取 n 位
> 3. substr(str, n [,length])：n > 0，从左边第 n 位开始截取 length 个字符；n < 0，从右边数第 n 位开始截取 length 个字符；

题目答案：

```mysql
select first_name from employees order by substr(first_name, -2);
```

## 窗口函数

窗口函数除了用于解决 **TopN** 问题之外，还可以用来解决如下其他问题。

语法：`<窗口函数> over (partition by <用于分组的列名> order by <用于排序的列名> [asc|desc])`

- **一些专用的窗口函数，如下**：
  - 序号函数：row_number() / rank() / dense_rank()
  - 分布函数：percent_rank() / cume_dist()
  - 前后函数：lag() / lead()
  - 头尾函数：first_val() / last_val()
  - 其他函数：nth_value()
- **原有的聚合函数也可用作窗口函数**：如 sum()，avg()，count()，max()，min() 等等

### 累计和 running_total

> 题目：[SQL254 统计salary的累计和running_total](https://www.nowcoder.com/practice/58824cd644ea47d7b2b670c506a159a6?tpId=82&tags=&title=&difficulty=0&judgeStatus=3&rp=1&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26tab%3DSQL%25E7%25AF%2587%26topicId%3D82)
>

题目答案：

```mysql
select emp_no, salary, sum(salary) over (order by emp_no) as running_total
from salaries
where to_date = '9999-01-01';
```

## 插入数据

### 插入数据时，忽略已经存在的数据

> 题目：[SQL229 批量插入数据，不使用replace操作](https://www.nowcoder.com/practice/153c8a8e7805400ba8e384e03acc6b3e?tpId=82&tqId=29801&rp=1&ru=%2Fexam%2Foj&qru=%2Fexam%2Foj&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26tab%3DSQL%E7%AF%87%26topicId%3D82&difficulty=undefined&judgeStatus=undefined&tags=&title=)
>
> 考察知识点：
>
> 1. insert ignore into：若没有则插入，若存在则忽略。
> 2. replace into：若没有则正常插入，若存在则先删除后插入。

#### 解法一：INSERT IGNORE INTO

```mysql
insert ignore into actor  
values ('3', 'ED', 'CHASE', '2006-02-15 12:34:33');
```

#### 解法二：INSERT INTO IF EXISTS

```mysql
insert into actor
select '3',
       'ED',
       'CHASE',
       '2006-02-15 12:34:33'
from dual
where not exists(
        select actor_id
        from actor
        where actor_id = '3'
    );
```

## 更新数据

### 更新某行记录的数据

> 题目：[SQL237 将所有to_date为9999-01-01的全部更新为NULL](https://www.nowcoder.com/practice/859f28f43496404886a77600ea68ef59?tpId=82&tags=&title=&difficulty=0&judgeStatus=0&rp=1&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26tab%3DSQL%25E7%25AF%2587%26topicId%3D82)
>
> 考察知识点：更新语句，语法：`update 表名 set 字段1 = 值1, 字段2 = 值2, ... where 过滤条件`

题目答案：

```mysql
update
  titles_test
set
  to_date = NULL,
  from_date = '2001-01-01'
where
  to_date = '9999-01-01';
```

### 连表更新某行记录的数据

> 题目：[SQL242 将所有获取奖金的员工当前的薪水增加10%](https://www.nowcoder.com/practice/d3b058dcc94147e09352eb76f93b3274?tpId=82&tags=&title=&difficulty=0&judgeStatus=0&rp=1&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26tab%3DSQL%25E7%25AF%2587%26topicId%3D82)
>
> 考察知识点：连表更新语句，语法：`update 表1 join 表2 on 表1.key = 表2.key set 表1.字段1 = 值1, 表1.字段2 = 值2, ... where 过滤条件 `

题目答案：

```mysql
update
  salaries s
  inner join emp_bonus eb on s.emp_no = eb.emp_no
set
  s.salary = s.salary * (1 + 0.1)
where
  s.to_date = '9999-01-01';
```

## 创建表

### 复制其他表结构和数据来创建表

> 题目：[SQL230 创建一个actor_name表](https://www.nowcoder.com/practice/881385f388cf4fe98b2ed9f8897846df?tpId=82&tags=&title=&difficulty=0&judgeStatus=0&rp=1&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26tab%3DSQL%25E7%25AF%2587%26topicId%3D82)
>
> 考察知识点：创建表的方式
>
> 1. `create table if not exists table_name(字段1, 字段2, ...);`
>
> 2. `create table table_name like other_table;` 复制表结构
>
> 3. `create teble if not exists table_name select 字段1, 字段2, ... from other_table;` 复制表结构和数据。
>
> 4. `create table if not exists table_name(字段1, 字段2, ...) select 字段1, 字段2, ...from other_table; ` 只复制数据

题目答案：

```mysql
CREATE TABLE if not exists actor_name select first_name, last_name from actor;
```

## 修改表

### 添加字段

> 题目：[SQL234 在last_update后面新增加一列名字为create_date](https://www.nowcoder.com/practice/119f04716d284cb7a19fba65dd876b03?tpId=82&tags=&title=&difficulty=0&judgeStatus=0&rp=1&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26tab%3DSQL%25E7%25AF%2587%26topicId%3D82)
>
> 考察知识点：修改表结构，语法：`ALTER TABLE <表名> ADD COLUMN <新字段名> <数据类型> [约束条件] [FIRST|AFTER 已存在的字段名];`

题目答案：

```mysql
alter table actor
    add column create_date datetime not null default '2020-10-01 00:00:00' after last_update;
```

### 修改表名

> 题目：[SQL239 将titles_test表名修改为titles_2017](https://www.nowcoder.com/practice/5277d7f92aa746ab8aa42886e5d570d4?tpId=82&tags=&title=&difficulty=0&judgeStatus=0&rp=1&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26tab%3DSQL%25E7%25AF%2587%26topicId%3D82)
>
> 考察知识点：修改表名，语法：`alter table 表名 rename to 新表名 `

题目答案：

```mysql
alter table titles_test rename to titles_2017;
```

### 添加索引

> 题目：[SQL231 对first_name创建唯一索引uniq_idx_firstname](https://www.nowcoder.com/practice/e1824daa0c49404aa602cf0cb34bdd75?tpId=82&tags=&title=&difficulty=0&judgeStatus=0&rp=1&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26tab%3DSQL%E7%AF%87%26topicId%3D82)
>
> 考察知识点：创建索引的方式
>
> 1. 使用 `alter` 创建索引
>    - 添加主键索引：`alter table 表名 add primary key(col)`
>    - 添加唯一索引：`alter table 表名 add unique index <索引名> (col1, col2, ...)`
>    - 添加普通索引：`alter table 表名 add index <索引名> (col1, col2, ...,);`
> 2. 使用 `create` 创建索引，不能用于添加主键索引
>    - 添加唯一索引：`create unique index 索引名 on 表名(col1, col2, ...)`
>    - 添加普通索引：`create index 索引名 on 表名(col1, col2, ...)`

题目答案：

```mysql
alter table
  actor
add
  unique index uniq_idx_firstname (first_name),
add
  index idx_lastname (last_name);
```

### 添加外键

> 题目：[SQL240 在audit表上创建外键约束，其emp_no对应employees_test表的主键id](https://www.nowcoder.com/practice/aeaa116185f24f209ca4fa40e226de48?tpId=82&tags=&title=&difficulty=0&judgeStatus=0&rp=1&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26tab%3DSQL%25E7%25AF%2587%26topicId%3D82)
>
> 考察知识点：创建外键，语法：`alter table 表名 add constraint foreign key (col) references 关联表 (关联列)  `

题目答案：

```mysql
alter table
  audit
add
  constraint foreign key (EMP_no) references employees_test(ID);
```

## 视图

### 创建视图

>题目：[SQL232 针对actor表创建视图actor_name_view](https://www.nowcoder.com/practice/b9db784b5e3d488cbd30bd78fdb2a862?tpId=82&tags=&title=&difficulty=0&judgeStatus=0&rp=1&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26tab%3DSQL%25E7%25AF%2587%26topicId%3D82)
>
>考察知识点：创建视图

方式一：**直接在视图名的后面用小括号创建视图中的字段名**

```mysql
create view actor_name_view (first_name_v, last_name_v) as
select first_name, last_name
from actor;
```

方式二：**在 select 后面对列重命名为视图的字段名**

```mysql
create view actor_name_view as
select first_name first_name_v, last_name last_name_v
from actor;
```

## 其他

### 使用强制索引查询

> [SQL233 针对上面的salaries表emp_no字段创建索引idx_emp_no](https://www.nowcoder.com/practice/f9fa9dc1a1fc4130b08e26c22c7a1e5f?tpId=82&tags=&title=&difficulty=0&judgeStatus=0&rp=1&sourceUrl=%2Fexam%2Foj%3Fpage%3D1%26tab%3DSQL%25E7%25AF%2587%26topicId%3D82)
>
> 考察知识点：强制索引 `FORCE INDEX` 强制查询优化器使用指定的命名索引。查询优化器是 MySQL 数据库服务器中的一个组件，它为 SQL 语句提供最佳的执行计划。查询优化器使用可用的统计信息来提出所有候选计划中成本最低的计划。语法：`select ... from ... force index(index_name) where ...`

题目答案：

```mysql
select
  s.*
from
  salaries s force index(idx_emp_no);
```
