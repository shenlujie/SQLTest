## 建表语句：

```sql
create table Student(sid varchar(10),sname varchar(10),sage datetime,ssex nvarchar(10));
insert into Student values('01' , '赵雷' , '1990-01-01' , '男');
insert into Student values('02' , '钱电' , '1990-12-21' , '男');
insert into Student values('03' , '孙风' , '1990-05-20' , '男');
insert into Student values('04' , '李云' , '1990-08-06' , '男');
insert into Student values('05' , '周梅' , '1991-12-01' , '女');
insert into Student values('06' , '吴兰' , '1992-03-01' , '女');
insert into Student values('07' , '郑竹' , '1989-07-01' , '女');
insert into Student values('08' , '王菊' , '1990-01-20' , '女');
create table Course(cid varchar(10),cname varchar(10),tid varchar(10));
insert into Course values('01' , '语文' , '02');
insert into Course values('02' , '数学' , '01');
insert into Course values('03' , '英语' , '03');
create table Teacher(tid varchar(10),tname varchar(10));
insert into Teacher values('01' , '张三');
insert into Teacher values('02' , '李四');
insert into Teacher values('03' , '王五');
create table SC(sid varchar(10),cid varchar(10),score decimal(18,1));
insert into SC values('01' , '01' , 80);
insert into SC values('01' , '02' , 90);
insert into SC values('01' , '03' , 99);
insert into SC values('02' , '01' , 70);
insert into SC values('02' , '02' , 60);
insert into SC values('02' , '03' , 80);
insert into SC values('03' , '01' , 80);
insert into SC values('03' , '02' , 80);
insert into SC values('03' , '03' , 80);
insert into SC values('04' , '01' , 50);
insert into SC values('04' , '02' , 30);
insert into SC values('04' , '03' , 20);
insert into SC values('05' , '01' , 76);
insert into SC values('05' , '02' , 87);
insert into SC values('06' , '01' , 31);
insert into SC values('06' , '03' , 34);
insert into SC values('07' , '02' , 89);
insert into SC values('07' , '03' , 98);
```

## 表结构：

- 学生表 Student

SId 学生编号		Sname 学生姓名		Sage 出生年月		Ssex 学生性别

- 课程表 Course

CId 课程编号		Cname 课程名称		TId 教师编号

- 教师表 Teacher

TId 教师编号		Tname 教师姓名

- 成绩表 SC

SId 学生编号		CId 课程编号		score 分数

## 题目：

1、查询“01”课程比“02”课程成绩高的所有学生的学号：

```sql
SELECT s1.sid 
FROM
	( SELECT * FROM SC WHERE cid = 01 ) s1
	LEFT JOIN ( SELECT * FROM SC WHERE cid = 02 ) s2 ON s1.sid = s2.sid 
WHERE
	s1.score > s2.score;
```

2、查询平均成绩大于60分的同学的学号和平均成绩；

```sql
SELECT sid,AVG( score ) 
FROM SC 
GROUP BY sid 
HAVING AVG( score ) > 60;
```

3、查询所有同学的学号、姓名、选课数、总成绩

```sql
SELECT Student.sid,Student.sname,COUNT(DISTINCT cid),SUM(DISTINCT score)
FROM Student LEFT JOIN SC
ON Student.sid = SC.sid
GROUP BY Student.sid,Student.sid
```

4、查询姓“李”的老师的个数；

```sql
SELECT COUNT(DISTINCT tid) AS teacher_cnt
FROM Teacher
WHERE tname LIKE "李%"
```

5、查询没学过“张三”老师课的同学的学号、姓名；

```sql
SELECT DISTINCT Student.sid,Student.sname 
FROM Student LEFT JOIN SC 
ON Student.sid = SC.sid
WHERE SC.cid NOT IN
	(SELECT cid 
	FROM 	Course LEFT JOIN Teacher ON Course.tid = Teacher.tid 
	WHERE tname = "张三")
```

6、查询学过编号“01”并且也学过编号“02”课程的同学的学号、姓名；

```sql
SELECT t.sid AS sid,Student.sname
FROM 
	(SELECT sid 
	FROM SC
	GROUP BY sid
	HAVING COUNT(IF(cid = "01",score,NULL)) > 0 
	AND
	COUNT(IF(cid = "02",score,NULL)) > 0) AS t
LEFT JOIN Student
ON t.sid = Student.sid;
```

7、查询学过“张三”老师所教的课的同学的学号、姓名；

```sql
SELECT s.sid,Student.sname 
FROM
	(SELECT SC.sid
	FROM
		(SELECT DISTINCT cid
		FROM Course LEFT JOIN Teacher
		ON Course.tid = Teacher.tid
		WHERE Teacher.tname = "张三") c
		LEFT JOIN SC ON c.cid = SC.cid) s
		LEFT JOIN Student ON s.sid = Student.sid
```

8、查询课程编号“01”的成绩比课程编号“02”课程低的所有同学的学号、姓名；

```sql
SELECT t.sid,Student.sname
FROM
	(SELECT s1.sid
	FROM
		(SELECT sid,score
		FROM SC
		WHERE cid = "01" AND score IS NOT NULL) s1
		LEFT JOIN
		(SELECT sid,score
		FROM SC
		WHERE cid = "02" AND score IS NOT NULL) s2
		ON s1.sid = s2.sid
	WHERE s1.score < s2.score) t
LEFT JOIN Student
ON t.sid = Student.sid
```

9、查询所有课程成绩小于60分的同学的学号、姓名；

```sql
SELECT t.sid,Student.sname
FROM
	(SELECT DISTINCT sid
	FROM SC
	GROUP BY sid
	HAVING MAX(score) < 60) t
LEFT JOIN Student
ON t.sid = Student.sid
```

10、查询没有学全所有课的同学的学号、姓名；

```sql
SELECT t.sid,Student.sname
FROM
	(SELECT sid
	FROM SC
	GROUP BY sid
	HAVING COUNT(DISTINCT cid) < 
	(SELECT COUNT(DISTINCT cid)
	FROM Course)) t
LEFT JOIN Student
ON t.sid = Student.sid
```

11、查询至少有一门课与学号为“01”的同学所学相同的同学的学号和姓名；

```sql
SELECT t.sid,Student.sname
FROM
	(SELECT DISTINCT co.sid
	FROM
		(SELECT cid
		FROM SC 
		WHERE sid = "01") c1
	LEFT JOIN 
		(SELECT sid,cid
		FROM SC
		WHERE sid != "01") co
	ON c1.cid = co.cid) t
LEFT JOIN Student
ON t.sid = Student.sid
```

12、查询和"01"号的同学学习的课程完全相同的其他同学的学号和姓名

```sql
SELECT t.sid,Student.sname
FROM
	(SELECT co.sid
		FROM
			(SELECT cid
			FROM SC 
			WHERE sid = "01") c1
		LEFT JOIN 
			(SELECT sid,cid
			FROM SC
			WHERE sid != "01") co
		ON c1.cid = co.cid 
		GROUP BY co.sid
		HAVING COUNT(co.cid) = 
		(SELECT COUNT(DISTINCT cid)
		FROM SC
		WHERE sid = "01")) t
LEFT JOIN Student
ON t.sid = Student.sid
```

13、把“SC”表中“张三”老师教的课的成绩都更改为此课程的平均成绩；

14、查询没学过"张三"老师讲授的任一门课程的学生姓名

15、查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩

16、检索"01"课程分数小于60，按分数降序排列的学生信息

17、按平均成绩从高到低显示所有学生的平均成绩

18、查询各科成绩最高分、最低分和平均分：以如下形式显示：课程ID，课程name，最高分，最低分，平均分，及格率

19、按各科平均成绩从低到高和及格率的百分数从高到低顺序

20、查询学生的总成绩并进行排名

21、查询不同老师所教不同课程平均分从高到低显示

22、查询所有课程的成绩第2名到第3名的学生信息及该课程成绩

23、统计各科成绩各分数段人数：课程编号,课程名称,[100-85],[85-70],[70-60],[0-60]及所占百分比

24、查询学生平均成绩及其名次

25、查询各科成绩前三名的记录

26、查询每门课程被选修的学生数

27、查询出只选修了一门课程的全部学生的学号和姓名

28、查询男生、女生人数

29、查询名字中含有"风"字的学生信息

30、查询同名同性学生名单，并统计同名人数

31、查询1990年出生的学生名单(注：Student表中Sage列的类型是datetime)

32、查询每门课程的平均成绩，结果按平均成绩升序排列，平均成绩相同时，按课程号降序排列

37、查询不及格的课程，并按课程号从大到小排列

38、查询课程编号为"01"且课程成绩在60分以上的学生的学号和姓名；

40、查询选修“张三”老师所授课程的学生中，成绩最高的学生姓名及其成绩

42、查询每门功课成绩最好的前两名

43、统计每门课程的学生选修人数（超过5人的课程才统计）。要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列

44、检索至少选修两门课程的学生学号

45、查询选修了全部课程的学生信息

46、查询各学生的年龄

47、查询本周过生日的学生

48、查询下周过生日的学生

49、查询本月过生日的学生

50、查询下月过生日的学生