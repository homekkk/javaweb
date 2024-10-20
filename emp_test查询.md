#1. 查询所有员工的姓名、邮箱和工作岗位。--
SELECT first_name, last_name,email, job_title FROM employees;

#2. 查询所有部门的名称和位置。
SELECT dept_name, location FROM departments;

#3查询工资超过70000的员工姓名和工资。
SELECT CONCAT(first_name, ' ', last_name) AS full_name, salary FROM employees WHERE salary > 70000;
#4查询IT部门的所有员工。
SELECT * FROM employees WHERE dept_id = (SELECT dept_id FROM departments WHERE dept_name = 'IT');

#5查询入职日期在2020年之后的员工信息。
SELECT * FROM employees WHERE hire_date > '2020-01-01';

#6计算每个部门的平均工资。
SELECT dept_id, AVG(salary) AS average_salary FROM employees GROUP BY dept_id;

#7查询工资最高的前3名员工信息。
SELECT * FROM employees ORDER BY salary DESC LIMIT 3;

#8查询每个部门员工数量。
SELECT dept_id, COUNT(*) AS employee_count FROM employees GROUP BY dept_id;

#9查询没有分配部门的员工。
SELECT * FROM employees WHERE dept_id IS NULL;

#10查询参与项目数量最多的员工。
SELECT emp_id, COUNT(project_id) AS project_count 
FROM employee_projects 
GROUP BY emp_id 
ORDER BY project_count DESC 
LIMIT 1;

#11计算所有员工的工资总和。
SELECT SUM(salary) AS total_salary FROM employees;

#12查询姓"Smith"的员工信息。
SELECT * FROM employees WHERE last_name = 'Smith';

#13查询即将在半年内到期的项目。
SELECT * FROM projects WHERE end_date BETWEEN CURDATE() AND DATE_ADD(CURDATE(), INTERVAL 6 MONTH);

#14查询至少参与了两个项目的员工。
SELECT emp_id, COUNT(project_id) AS project_count 
FROM employee_projects 
GROUP BY emp_id 
HAVING project_count >= 2;

#15查询没有参与任何项目的员工。
SELECT * FROM employees 
WHERE emp_id NOT IN (SELECT DISTINCT emp_id FROM employee_projects);

#16计算每个项目参与的员工数量。
SELECT project_id, COUNT(emp_id) AS employee_count 
FROM employee_projects 
GROUP BY project_id;

#17查询工资第二高的员工信息。
SELECT * FROM employees 
WHERE salary = (SELECT DISTINCT salary FROM employees ORDER BY salary DESC LIMIT 1 OFFSET 1);

#18查询每个部门工资最高的员工。
SELECT * FROM employees e1 
WHERE salary = (SELECT MAX(salary) FROM employees e2 WHERE e1.dept_id = e2.dept_id);

#19计算每个部门的工资总和，并按照工资总和降序排列。
SELECT dept_id, SUM(salary) AS total_salary 
FROM employees 
GROUP BY dept_id 
ORDER BY total_salary DESC;

#20查询员工姓名、部门名称和工资。
SELECT CONCAT(e.first_name, ' ', e.last_name) AS full_name, d.dept_name, e.salary 
FROM employees e 
JOIN departments d ON e.dept_id = d.dept_id;

#21查询每个员工的上级主管（假设emp_id小的是上级）。
SELECT e1.first_name AS employee_name, e2.first_name AS supervisor_name 
FROM employees e1 
LEFT JOIN employees e2 ON e1.emp_id = e2.emp_id + 1; 

#22查询所有员工的工作岗位，不要重复。
SELECT DISTINCT job_title FROM employees;

#23查询平均工资最高的部门。
SELECT dept_id, AVG(salary) AS average_salary 
FROM employees 
GROUP BY dept_id 
ORDER BY average_salary DESC 
LIMIT 1;

#24查询工资高于其所在部门平均工资的员工。
SELECT e.* 
FROM employees e 
WHERE e.salary > (SELECT AVG(salary) FROM employees WHERE dept_id = e.dept_id);

#25查询每个部门工资前两名的员工。
SELECT * FROM (
    SELECT e.*, 
           ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rn 
    FROM employees e
) AS ranked 
WHERE rn <= 2;

#第二部分
#1查询所有学生的信息。
SELECT * FROM student;

#2查询所有课程的信息。
SELECT * FROM course;

#3查询所有学生的姓名、学号和班级。
SELECT name, student_id, my_class FROM student;

#4查询所有教师的姓名和职称。
SELECT name, title FROM teacher;

#5查询不同课程的平均分数。
SELECT c.course_id, c.course_name, AVG(s.score) AS average_score
FROM course c
LEFT JOIN score s ON c.course_id = s.course_id
GROUP BY c.course_id;

#6查询每个学生的平均分数。
SELECT student_id, AVG(score) AS average_score 
FROM score 
GROUP BY student_id;

#7查询分数大于85分的学生学号和课程号。
SELECT student_id, course_id 
FROM score 
WHERE score > 85;

#8查询每门课程的选课人数。
SELECT course_id, COUNT(student_id) AS enrolled_count 
FROM score 
GROUP BY course_id;

#9查询选修了"高等数学"课程的学生姓名和分数。
SELECT s.name, sc.score 
FROM student s 
JOIN score sc ON s.student_id = sc.student_id 
WHERE sc.course_id = 'C001';

#10查询没有选修"大学物理"课程的学生姓名。
SELECT s.name 
FROM student s 
WHERE s.student_id NOT IN (SELECT student_id FROM score WHERE course_id = 'C002');

#11查询C001比C002课程成绩高的学生信息及课程分数。
SELECT s.name, sc1.course_id, sc1.score AS score_c001, sc2.score AS score_c002
FROM student s 
JOIN score sc1 ON s.student_id = sc1.student_id AND sc1.course_id = 'C001'
JOIN score sc2 ON s.student_id = sc2.student_id AND sc2.course_id = 'C002'
WHERE sc1.score > sc2.score;

#12统计各科成绩各分数段人数。
SELECT 
    c.course_id,
    c.course_name,
    SUM(CASE WHEN s.score BETWEEN 85 AND 100 THEN 1 ELSE 0 END) AS '100-85',
    SUM(CASE WHEN s.score BETWEEN 70 AND 84 THEN 1 ELSE 0 END) AS '85-70',
    SUM(CASE WHEN s.score BETWEEN 60 AND 69 THEN 1 ELSE 0 END) AS '70-60',
    SUM(CASE WHEN s.score < 60 THEN 1 ELSE 0 END) AS '60-0',
    COUNT(s.student_id) AS total_students,
    (SUM(CASE WHEN s.score >= 60 THEN 1 ELSE 0 END) / COUNT(s.student_id)) * 100 AS pass_rate,
    (SUM(CASE WHEN s.score BETWEEN 70 AND 80 THEN 1 ELSE 0 END) / COUNT(s.student_id)) * 100 AS middle_rate,
    (SUM(CASE WHEN s.score BETWEEN 80 AND 90 THEN 1 ELSE 0 END) / COUNT(s.student_id)) * 100 AS good_rate,
    (SUM(CASE WHEN s.score >= 90 THEN 1 ELSE 0 END) / COUNT(s.student_id)) * 100 AS excellent_rate
FROM course c
LEFT JOIN score s ON c.course_id = s.course_id
GROUP BY c.course_id;

#13查询选择C002课程但没选择C004课程的成绩情况。
SELECT s.student_id, s.name, sc2.score AS score_c002, sc4.score AS score_c004
FROM student s
LEFT JOIN score sc2 ON s.student_id = sc2.student_id AND sc2.course_id = 'C002'
LEFT JOIN score sc4 ON s.student_id = sc4.student_id AND sc4.course_id = 'C004'
WHERE sc2.score IS NOT NULL AND sc4.score IS NULL;

#14查询平均分数最高的学生姓名和平均分数。
SELECT s.name, AVG(sc.score) AS average_score
FROM student s
JOIN score sc ON s.student_id = sc.student_id
GROUP BY s.student_id
ORDER BY average_score DESC 
LIMIT 1;

#15查询总分最高的前三名学生的姓名和总分。
SELECT s.name, SUM(sc.score) AS total_score
FROM student s
JOIN score sc ON s.student_id = sc.student_id
GROUP BY s.student_id
ORDER BY total_score DESC 
LIMIT 3;

#16查询各科成绩最高分、最低分和平均分。
SELECT c.course_id, c.course_name, 
       MAX(s.score) AS highest_score, 
       MIN(s.score) AS lowest_score, 
       AVG(s.score) AS average_score,
       SUM(CASE WHEN s.score >= 60 THEN 1 ELSE 0 END) / COUNT(s.student_id) * 100 AS pass_rate,
       SUM(CASE WHEN s.score BETWEEN 70 AND 80 THEN 1 ELSE 0 END) / COUNT(s.student_id) * 100 AS middle_rate,
       SUM(CASE WHEN s.score BETWEEN 80 AND 90 THEN 1 ELSE 0 END) / COUNT(s.student_id) * 100 AS good_rate,
       SUM(CASE WHEN s.score >= 90 THEN 1 ELSE 0 END) / COUNT(s.student_id) * 100 AS excellent_rate
FROM course c
LEFT JOIN score s ON c.course_id = s.course_id
GROUP BY c.course_id
ORDER BY COUNT(s.student_id) DESC, c.course_id ASC;

#17查询男生和女生的人数。
SELECT gender, COUNT(*) AS count 
FROM student 
GROUP BY gender;

#18查询年龄最大的学生姓名。
SELECT name 
FROM student 
ORDER BY birth_date ASC 
LIMIT 1;

#19查询年龄最小的教师姓名。
SELECT name 
FROM teacher 
ORDER BY birth_date DESC 
LIMIT 1;

#20查询学过「张教授」授课的同学的信息。
SELECT s.* 
FROM student s 
JOIN score sc ON s.student_id = sc.student_id 
WHERE sc.course_id IN (SELECT course_id FROM course WHERE teacher_id = 'T001');

#21查询至少有一门课与学号为"2021001"的同学所学相同的同学的信息。
SELECT DISTINCT s2.* 
FROM student s1 
JOIN score sc1 ON s1.student_id = sc1.student_id 
JOIN score sc2 ON sc1.course_id = sc2.course_id 
JOIN student s2 ON sc2.student_id = s2.student_id 
WHERE s1.student_id = '2021001' AND s2.student_id != '2021001';

#22查询每门课程的平均分数，并按平均分数降序排列。
SELECT c.course_id, c.course_name, AVG(s.score) AS average_score 
FROM course c 
LEFT JOIN score s ON c.course_id = s.course_id 
GROUP BY c.course_id 
ORDER BY average_score DESC;

#23查询学号为"2021001"的学生所有课程的分数。
SELECT c.course_name, sc.score 
FROM score sc 
JOIN course c ON sc.course_id = c.course_id 
WHERE sc.student_id = '2021001';

#24查询所有学生的姓名、选修的课程名称和分数。
SELECT s.name, c.course_name, sc.score 
FROM student s 
LEFT JOIN score sc ON s.student_id = sc.student_id 
LEFT JOIN course c ON sc.course_id = c.course_id;

#25查询每个教师所教授课程的平均分数。
SELECT t.name, AVG(s.score) AS average_score 
FROM teacher t 
JOIN course c ON t.teacher_id = c.teacher_id 
LEFT JOIN score s ON c.course_id = s.course_id 
GROUP BY t.teacher_id;

#26查询分数在80到90之间的学生姓名和课程名称。
SELECT s.name, c.course_name 
FROM student s 
JOIN score sc ON s.student_id = sc.student_id 
JOIN course c ON sc.course_id = c.course_id 
WHERE sc.score BETWEEN 80 AND 90;

#27查询每个班级的平均分数。
SELECT s.my_class, AVG(sc.score) AS avg_score
FROM student s
JOIN score sc ON s.student_id = sc.student_id
GROUP BY s.my_class;

#28查询没学过"王讲师"老师讲授的任一门课程的学生姓名。
SELECT s.name 
FROM student s 
WHERE s.student_id NOT IN (SELECT sc.student_id 
                            FROM score sc 
                            JOIN course c ON sc.course_id = c.course_id 
                            WHERE c.teacher_id = 'T003');

#29查询两门及其以上小于85分的同学的学号，姓名及其平均成绩。
SELECT s.student_id, s.name, AVG(sc.score) AS average_score 
FROM student s 
JOIN score sc ON s.student_id = sc.student_id 
WHERE sc.score < 85 
GROUP BY s.student_id 
HAVING COUNT(sc.course_id) >= 2;

#30查询所有学生的总分并按降序排列。
SELECT s.student_id, s.name, SUM(sc.score) AS total_score 
FROM student s 
JOIN score sc ON s.student_id = sc.student_id 
GROUP BY s.student_id 
ORDER BY total_score DESC;

#31查询平均分数超过85分的课程名称。
SELECT c.course_name 
FROM course c 
JOIN score s ON c.course_id = s.course_id 
GROUP BY c.course_id 
HAVING AVG(s.score) > 85;

#32查询每个学生的平均成绩排名。
SELECT s.student_id, s.name, AVG(sc.score) AS avg_score,
       DENSE_RANK() OVER (ORDER BY AVG(sc.score) DESC) AS ranking
FROM student s
JOIN score sc ON s.student_id = sc.student_id
GROUP BY s.student_id, s.name;

#33查询每门课程分数最高的学生姓名和分数。
SELECT s.name, sc.score, c.course_name 
FROM score sc 
JOIN student s ON sc.student_id = s.student_id 
JOIN course c ON sc.course_id = c.course_id 
WHERE (sc.course_id, sc.score) IN 
    (SELECT course_id, MAX(score) 
     FROM score 
     GROUP BY course_id);

#34查询选修了"高等数学"和"大学物理"的学生姓名。
SELECT s.name 
FROM student s 
JOIN score sc1 ON s.student_id = sc1.student_id AND sc1.course_id = 'C001' 
JOIN score sc2 ON s.student_id = sc2.student_id AND sc2.course_id = 'C002';

#35按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩（没有选课则为空）。
SELECT s.name, c.course_name, sc.score, 
       (SELECT AVG(score) FROM score WHERE student_id = s.student_id) AS average_score 
FROM student s 
LEFT JOIN score sc ON s.student_id = sc.student_id 
LEFT JOIN course c ON sc.course_id = c.course_id 
ORDER BY average_score DESC;

#36查询分数最高和最低的学生姓名及其分数。
SELECT s.name, sc.score 
FROM score sc 
JOIN student s ON sc.student_id = s.student_id 
WHERE sc.score = (SELECT MAX(score) FROM score) 
   OR sc.score = (SELECT MIN(score) FROM score);

#37查询每个班级的最高分和最低分。
SELECT s.my_class, MAX(sc.score) AS highest_score, MIN(sc.score) AS lowest_score 
FROM student s 
JOIN score sc ON s.student_id = sc.student_id 
GROUP BY s.my_class;

#38查询每门课程的优秀率（优秀为90分）。
SELECT c.course_id, c.course_name, 
       SUM(CASE WHEN s.score >= 90 THEN 1 ELSE 0 END) / COUNT(s.student_id) * 100 AS excellent_rate 
FROM course c 
LEFT JOIN score s ON c.course_id = s.course_id 
GROUP BY c.course_id;

#39查询平均分数超过班级平均分数的学生。
SELECT s.name 
FROM student s 
JOIN score sc ON s.student_id = sc.student_id 
GROUP BY s.student_id 
HAVING AVG(sc.score) > (SELECT AVG(score) FROM score WHERE student_id IN 
                         (SELECT student_id FROM student WHERE my_class = s.my_class));

#40查询每个学生的分数及其与课程平均分的差值。
SELECT s.name, sc.score, (sc.score - AVG(sc2.score)) AS score_difference 
FROM student s 
JOIN score sc ON s.student_id = sc.student_id 
JOIN score sc2 ON sc.course_id = sc2.course_id 
GROUP BY s.student_id, sc.course_id;
