---
# try also 'default' to start simple
theme: default
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides (markdown enabled)
title: Welcome to Slidev
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# apply UnoCSS classes to the current slide
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
# transition: slide-left
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
---

# Aggregation, Joins

Presentation slides for developers

<div @click="$slidev.nav.next" class="mt-12 py-1" hover:bg="white op-10">
  Press Space for next page <carbon:arrow-right />
</div>

<div class="abs-br m-6 text-xl">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="slidev-icon-btn">
    <carbon:edit />
  </button>
  <a href="https://github.com/slidevjs/slidev" target="_blank" class="slidev-icon-btn">
    <carbon:logo-github />
  </a>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---

## Topics

<div class="p-4">

<v-clicks>

- "Summary" Functions
- Aggregation Functions: `GROUP BY`, `HAVING` 
- Copy data from a table to another
- Update using Case expression
- Set operations: Union, Intersection, Except
- Joins: **Inner** and **Outer** joins, **Natural** join, **Self** joins

</v-clicks>

</div>


---

## "Summary" Functions

<div class="p-4">

<v-clicks>

- `COUNT(*)` → number of rows in result set
- `COUNT(col_name)` → number of rows with non-null value for col_name
- `COUNT(DISTINCT col_name)` → number of distinct values of a column
- `SUM(col_name)` → sum of column values
- `AVG(col_name)` → average of column values
- `MIN(col_name)` → minimum value in a column
- `MAX(col_name)` → maximum value in a column

</v-clicks>


</div>

---

## COUNT(*) vs. COUNT(col_name)

<br>

<div class="pl-4">

<CsvTable
  src="/data/advisor.csv"
  caption="users"
  :max-rows="12"
  max-height="45vh"
  cell-max-width="16ch"
/>

<br>

<v-click>

- `COUNT(*)` → 3 (all rows)
- `COUNT(advisor_id)` → 2 (ignores Ravira's `NULL`)

</v-click>

</div>

---


## Summary Functions — Examples

<div class="p-4">

<v-click>

- Example: find the min, max and average instructor salary 

</v-click>


<v-click>


```sql
SELECT MIN(salary), MAX(salary), AVG(salary)
FROM instructor;
```

</v-click>

<br>

<v-click>

- Example: how many Comp. Sci. instructors earn $70,000 or more?

</v-click>

<v-click>


```sql
SELECT COUNT(*)
FROM instructor
WHERE dept_name = 'Comp. Sci' AND salary >= 70000;
```

</v-click>

</div>

---

## Aside: Introducing Aliases

<div class="p-4">


<v-click>

**Column Aliases**

</v-click>

<v-clicks depth="3">

- You can rename a column in the result set using `AS`
  - `SELECT first_name AS fname, last_name AS lname FROM employees;`
    - Result set headers will show fname and lname instead of first_name and last_name

</v-clicks>


<v-click>


**Aliasing Expressions**

</v-click>

<v-clicks>

- Can alias computed values
- `SELECT COUNT(*) AS total_employees FROM employees;`


</v-clicks>

</div>

---

## Aggregate Functions – GROUP BY

<div class="p-4">

<v-clicks depth="2">

- `GROUP BY` is used to aggregate rows that have the same values in one or more columns into summary rows.
- Most often paired with aggregate functions like `COUNT()`, `SUM()`, `AVG()`, `MIN()`, and `MAX()`
- Rows are grouped based on specified column(s), then for each group aggregates are computed
- Every column in the `SELECT` list must either: 
  - be in the `GROUP BY` clause, **or** 
  - be used inside an aggregate function.

</v-clicks>

</div>

---

## Example: GROUP BY

<div class="p-4">


<v-click>

Find the average salary of instructors in each department:

</v-click>


<v-click>


```sql
SELECT dept_name, AVG(salary) AS avg_salary
FROM instructor
GROUP BY dept_name;
```

</v-click>


<v-click>


![](/images/img1.png){class="w-120"}

</v-click>

</div>


---

## Example: GROUP BY error

<div class="p-4">


<v-click>

- Attributes in `SELECT` clause outside of aggregate functions must appear in `GROUP BY` list

</v-click>

<v-click>


```sql
-- Erroneous Query
SELECT dept_name, instructor_id, AVG(salary)
FROM instructor
GROUP BY dept_name;
```

</v-click>


<v-click>

A group of instructor rows by `dept_name` has multiple `id` values. So which `id` value will be returned for the group? 

</v-click>

</div>


---

## Aggregate Functions – HAVING Clause

<div class="p-4">

<v-clicks>

- `HAVING` is a filter for groups which filters after grouping/aggregation.
- `WHERE` filters rows before grouping/aggregation.

</v-clicks>

<br>

<v-clicks>

1. Take all rows.
2. Apply `WHERE` (filter rows).
3. Combine rows into groups with `GROUP BY`.
4. Calculate aggregates (`SUM`, `COUNT`, etc.).
5. Apply `HAVING` (filter groups).
6. Return results.

</v-clicks>

</div>


---

## Example: HAVING Clause

<div class="p-4">


<v-click>

- Find the names and average salaries of all departments whose average salary is greater than 42000

</v-click>


</div>

<div class="p-6">

<v-click>

```sql
SELECT dept_name, AVG(salary) AS avg_salary
FROM instructor
GROUP BY dept_name
HAVING AVG(salary) > 42000;
```

</v-click>

</div>

<div class="p-4">


<v-click>

- Remember: predicates in the `HAVING` clause are applied after the formation of groups whereas predicates in the `WHERE` clause are applied before forming groups

</v-click>

</div>


---

## Copy data from a table to another

<div class="p-4">

<v-click>

- Make each student in the Biology department who has earned more than 100 credit hours an instructor in the Biology department with a salary of $30,000.

</v-click>

<v-click>

<div class="pl-4">

```sql
INSERT INTO instructor (instructor_id, instructor_name, dept_name, salary)
SELECT student_id, student_name, dept_name, 30_000
FROM student
WHERE dept_name = 'Biology' AND tot_cred > 100;
```

</div>

</v-click>


<v-click>

- The `SELECT ... FROM ... WHERE` statement is evaluated fully before any of its results are inserted into the relation.

</v-click>

</div>

---

## Updates


<div class="p-4">

<v-click>

- Increase salaries of instructors whose salary is over $100,000 by 3%, and all others by a 5% 

</v-click>

<v-click>

Write two update statements:


```sql
UPDATE instructor
SET salary = salary * 1.03
WHERE salary > 100000;

UPDATE instructor
SET salary = salary * 1.05
WHERE salary <= 100000;
```

</v-click>


<v-clicks>

- The order is important
- Can be done better using the `CASE` statement (next slide)

</v-clicks>


</div>


---

## CASE Statement for Conditional Updates

<div class="p-4">

<v-click>

- Can use a `CASE` expression inside an `UPDATE` to conditionally set column values.

</v-click>

<v-click>

```sql
UPDATE table_name
SET column_name = CASE
   WHEN condition1 THEN result1
   WHEN condition2 THEN result2
   ELSE default_result
END
WHERE <filter_condition>;
```

</v-click>

</div>

---

## Example: CASE Statement for Conditional Updates

<div class="p-4">

<v-click>

- Same query as before but with case statement

</v-click>

<br>

<v-click>

```sql
UPDATE instructor
SET salary = CASE
                WHEN salary <= 100000 THEN salary * 1.05
                ELSE salary * 1.03
            END;
```

</v-click>

</div>


---

## Set Operations

<div class="p-4">

<v-click>

- Find courses that ran in Fall 2009 or in Spring 2010

</v-click>

<v-click>

```sql
(
   SELECT course_id
   FROM section
   WHERE semester = 'Fall' AND section_year = 2009
)
UNION
(
   SELECT course_id
   FROM section
   WHERE semester = 'Spring' AND section_year = 2010
);
```

</v-click>

</div>

---

## Set Operations (cont.)

<div class="p-4">


<v-click>

Find courses that ran in Fall 2009 and in Spring 2010

</v-click>


<v-click>

```sql
(
   SELECT course_id
   FROM section
   WHERE semester = 'Fall' AND section_year = 2009
)
INTERSECT
(
   SELECT course_id
   FROM section
   WHERE semester = 'Spring' AND section_year = 2010
);
```


</v-click>

</div>

---

## Find courses that ran in Fall 2009 but not in Spring 2010


<div class="p-4">


```sql
(
   SELECT course_id
   FROM section
   WHERE semester = 'Fall' AND section_year = 2009
)
EXCEPT
(
   SELECT course_id
   FROM section
   WHERE semester = 'Spring' AND section_year = 2010
);
```

<v-click>

`SELECT` statements must return the same number of columns and compatible data types for set operations.

</v-click>

</div>



---

## Set Operations (cont.)

<div class="p-4">

<v-clicks depth="4">

- Set Operations: `UNION`, `INTERSECT`, and `EXCEPT`
  - Each of the above operations eliminates duplicates.
- To retain all duplicates, use the following:
  - `UNION ALL`
  - `INTERSECT ALL`
  - `EXCEPT ALL`

</v-clicks>


</div>


---

## Joined Relations


<div class="grid grid-cols-2 gap-4">


<div>

<br><br>

<v-clicks>

- **Join operations** take two relations and<br>return as a result another relation.
- A join operation is based on *Cartesian* product  which requires that tuples in the two relations  match (specified by predicates). 
- The join operations are used as expressions in the `FROM` clause

</v-clicks>

</div>

<div>

<v-click>

![](/images/cartesian.png){class="w-80"}

</v-click>


</div>

</div>

---

## Types of JOIN operations


<div class="p-4">


<v-click>

`<table name> INNER JOIN <table name> ON <predicates>`
- Standard explicit join.
- keyword `INNER` is optional

</v-click>

<br>

<v-click>

`<table name> NATURAL JOIN <table name>`
- Joins automatically on all columns with the same name in both tables.
- Can be convenient, but dangerous if new columns with the same name are added later. 

</v-click>

<br>

<v-click>

`<table name> INNER JOIN <table name> USING <column names>`
- Shorthand when joining on identically named columns.

</v-click>


</div>


---

## Basic Query Structure 

<div class="p-4">


- A query over multiple tables has the form

```sql
SELECT A1, A2, ..., An
FROM R1 
JOIN R2 ON P 
JOIN R3 ON P ...
WHERE P
ORDER BY A1, ..., Am
```

<v-clicks>

- $A_i$ represents an attribute
- $R$ represents a relation
- $P$ is a predicate

</v-clicks>


</div>


---

## Aside: Table Aliases

<div class="p-4">


- You can give a shorter name to a table, useful in joins:

<br>

```sql
SELECT e.first_name, d.dept_name
FROM employees e
JOIN departments d ON e.dept_id = d.dept_id;
```

</div>


---

## How joins work


<div class="grid grid-cols-2 gap-8">

<div>

<br>

<v-click>

![](/images/inst_course.png){class="w-100"}

</v-click>


<v-click>


```sql
SELECT i.name, c.course
FROM instructor i
INNER JOIN course c ON i.id = c.id;
```

</v-click>

<br>

<v-click>


<div class="pl-6">

1. Cartesian product
2. Selection (σ)
3. Projection (π)


</div>


</v-click>



</div>

<div>

<v-click>

<br><br><br>

![](/images/join1.png){class="w-100"}

</v-click>


</div>

</div>

---

## Inner Join Visualized

<div class="p-4">

- A standard inner join only includes rows where there is a match in both tables
  - If no match is found for a row in one table, that row is excluded from the result


![](/images/table_ab.png){class="w-100"}


</div>

---

## Example

<div class="p-4">

```sql
student JOIN takes on student.ID=takes.ID
```


![](/images/join2.png){class="w-130"}

</div>


---

## Example (cont.) 
 
<div class="p-4">

```sql
student JOIN takes on student.ID=takes.ID
```


![](/images/join3.png){class="w-110"}

</div>



---

## JOIN Condition

<div class="p-4">


- The `ON` condition allows a general predicate over the relations being  joined
- This predicate is written like a where clause predicate except for the use of the keyword `ON`

<br>

- Query example:

```sql
SELECT *
FROM student
JOIN takes ON student.student_id = takes.student_id;
```

<br>

- The `ON` condition above specifies that a tuple from student matches a tuple from takes when their `id` values are equal.
- Since the `id` column appears in both tables, it is necessary to qualify it with the table name.
- If there is no match,  (i.e. the student has not taken any courses yet), the student row does not show up in the result.


</div>


---

## Natural Join

<div class="p-4">



- Natural join matches tuples with the same values for all common attributes, and retains only one copy of each common column.
- List the names of students along with the course ID of the courses that they have taken

```sql
SELECT student_name, course_id
FROM student
JOIN takes on student.student_id = takes.student_id
```

<br>

- Same query in SQL with “natural join” construct

```sql
SELECT student_name, course_id
FROM student NATURAL JOIN takes;
```

</div>


---


## Natural Join (cont.)

<div class="p-4">

<br>

- The FROM clause can have multiple relations combined using `NATURAL JOIN`

<br>

```sql
SELECT A1, A2, ..., An
FROM r1
NATURAL JOIN r2 NATURAL JOIN ... NATURAL JOIN rn
WHERE P;
```

</div>


---

## Danger in Natural Join

<div class="p-4">



- Beware of unrelated attributes with same name which get equated incorrectly
- Example --- List the names of students along with the titles of courses that they have taken


```sql
--- Correct 
SELECT student_name, course.title
FROM student
NATURAL JOIN takes
JOIN course ON takes.course_id = course.course_id;
```


```sql
--- Incorrect
SELECT student_name, course.title AS course_title
FROM student
NATURAL JOIN takes
NATURAL JOIN course;
```

- The natural join will include the predicate `student.dept_name = course.dept_name`  
- This changes the meaning of the query to courses taken in the student's major.

</div>

---

## Join with `USING` Clause

<div class="p-4">


- Query example

```sql
SELECT student, course
FROM student
NATURAL JOIN takes
JOIN course USING (course_id);
```

<br>

- Equivalent to 

```sql
SELECT student, course.title
FROM student
JOIN takes ON student.student_id = takes.student_id
JOIN course ON course.course_id = takes.course_id;
```


</div>



---

## Outer Join

<div class="p-4">


- An extension of the join operation that avoids loss of information.
- Computes the join and then adds tuples form one relation that do not match tuples in the other relation to the result of the join. 
- Uses null values for missing columns.
- Three forms of outer join:
  - left outer join – adds tuples from the left relation
  - right outer join – adds tuples from the right relation
  - full outer join – adds tuples from both relations


</div>


---

## Outer Join Examples

<div class="p-4">


- Relation `course`
![](/images/course.png){class="w-70"} <br>
- Relation `prereq`
![](/images/prereq.png){class="w-40"} <br>
- Observe that 
  - course information is missing CS-437
  - prereq information is missing CS-315
</div>


---

## Left Outer Join

<div class="p-4">


```sql
SELECT *
FROM course
LEFT JOIN prereq ON course.course_id = prereq.course_id;
```

![](/images/course_outer.png){class="w-100"}

</div>


---

## Right Outer Join


<div class="p-4">


```sql
SELECT *
FROM course
RIGHT OUTER JOIN prereq
ON course.course_id = prereq.course_id;
```

![](/images/right_outer.png){class="w-100"}


</div>


---

## Full Outer Join

<div class="p-4">



```sql
SELECT *
FROM course
FULL OUTER JOIN prereq
ON course.course_id = prereq.course_id;
```

![](/images/full_outer.png){class="w-100"}


</div>

---

## Left Outer Join, Grouping with 0 counts

<div class="p-4">


Find the number of students each instructor advises.  If an instructor does advise any students, report a count of 0.

Advisor tables has pair of student id, instructor id values.

![](/images/advisor.png){class="w-20"}


**Problem:**  Instructor Wu 12121 does not advise any students.  An inner join will not find any match for 12121 and so Wu will not be in the result set.


**Solution:**  Use left outer join and count column `s_ID`.  Left join will return a `s_ID` value of null and count of a null value will give 0.


</div>


---

## Left Outer Join, Grouping with 0 counts

<div class="p-4">


```sql
SELECT instructor.instructor_id, instructor.instructor_name,
    COUNT(advisor.student_id) AS advised_students
FROM instructor
LEFT JOIN advisor ON instructor.instructor_id = advisor.instructor_id
GROUP BY instructor.instructor_id, instructor.instructor_name
ORDER BY instructor.instructor_id;
```

![](/images/grouping.png){class="w-60"}

</div>

---

## Self Join Example

<div class="p-4">


```sql
CREATE TABLE emp_super (
   person VARCHAR(10) PRIMARY KEY,
   supervisor VARCHAR(10)
);
```

Find Bob's supervisor.
Find the supervisor of Bob’s supervisor.
Can you find  ALL the supervisors (direct and indirect) of Bob?


</div>


---

## Self Join and Rename operation

<div class="p-4">


- SQL allows renaming relations and attributes using the AS clause:
  - old-name AS new-name

- Find the names of all instructors who have a higher salary than Wu. 


```sql
SELECT DISTINCT T.name
FROM instructor AS T
JOIN instructor AS S ON T.salary > S.salary
WHERE S.name = 'Wu';
```

Keyword `AS` is optional and may be omitted

```sql
instructor AS T ≡ instructor T
```


</div>


---

## Self Join Example


<div class="p-4">


Find the supervisor of Bob’='s supervisor.

```sql
SELECT t1.person, t2.supervisor
FROM emp_super AS t1
JOIN emp_super AS t2 ON t1.supervisor = t2.person
WHERE t1.person = 'Bob';
```

</div>
