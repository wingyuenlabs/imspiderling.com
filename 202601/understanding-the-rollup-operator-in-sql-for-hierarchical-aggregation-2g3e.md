---
Title: Understanding the ROLLUP Operator in SQL for Hierarchical Aggregation
Description: 
Author: Abdul Kdrav
Date: 2026-01-28T20:35:05.000Z
Robots: noindex,nofollow
Template: index
---
<p><small>What ROLLUP Does<br>
A regular GROUP BY returns one row per unique combination of the grouped columns.<br>
ROLLUP, on the other hand, creates a hierarchy of aggregations. For example:</small></p>

<p><small>detailed rows</small></p>

<p><small>subtotal rows</small></p>

<p><small>a final grand total row</small></p>

<p><small>This makes it ideal for reporting, dashboards, and any scenario where you need multi‑level summaries.</small></p>

<p>Working With Joins and Conditional Logic<br>
1.1 Displaying Department Managers and Locations<br>
<small>Goal: Show each department’s manager first name, department name, and city.<br>
<small>If a department has no manager, display "NO MANAGER".<br>
<small><br>
</small></small></small></p>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SELECT NVL(e.first_name, 'FARA MANAGER') AS manager_first_name,
       d.department_name,
       l.city
FROM hr.departments d LEFT JOIN hr.employees e
    ON d.manager_id = e.employee_id LEFT JOIN hr.locations l
    ON d.location_id = l.location_id;
</code></pre>

</div>


<p><br>
<small>Goal: Display the names of departments that have no employees.</small></p>

<p><small>Using NOT IN:<br>
</small></p>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SELECT 
    d.department_id,
    d.department_name
FROM hr.departments d
 where d.department_id not in(select e.department_id from hr.employees e where e.department_id is not null);
</code></pre>

</div>


<p><small>Using NOT EXISTS:<br>
</small></p>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SELECT d.department_name
FROM hr.departments d
WHERE NOT EXISTS (SELECT 1 FROM hr.employees e WHERE e.department_id = d.department_id);
</code></pre>

</div>


<p><small>Employee Seniority Classification</small></p>

<p><small>Goal: Display first name, last name, and a label indicating seniority:</small></p>

<p><small>Junior: &lt; 3 years<br>
<small>Mid‑Level: 3–7 years<br>
<small>Senior: &gt; 7 years<br>
</small></small></small></p>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SELECT 
    first_name,
    last_name,
    CASE
        WHEN TRUNC(MONTHS_BETWEEN(SYSDATE, hire_date) / 12) &lt; 3 
            THEN 'Junior'
        WHEN TRUNC(MONTHS_BETWEEN(SYSDATE, hire_date) / 12) BETWEEN 3 AND 7 
            THEN 'Mid-Level'
        WHEN TRUNC(MONTHS_BETWEEN(SYSDATE, hire_date) / 12) &gt; 7 
            THEN 'Senior' END AS vechime FROM hr.employees ORDER by last_name, first_name;
</code></pre>

</div>


<p><small>2. Outer Joins and Group Filtering<br>
Departments Without Employees (Outer Join Version)<br>
</small></p>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SELECT d.department_name,
       d.department_id FROM hr.departments d
LEFT JOIN hr.employees e
       ON d.department_id = e.department_id WHERE e.department_id IS NULL;
</code></pre>

</div>


<p><small>Employees Who Do NOT Belong to Small Departments<br>
</small></p>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SELECT e.employee_id,
       e.first_name,
       e.last_name,
       e.department_id
FROM hr.employees e JOIN hr.employees e2
     ON e.department_id = e2.department_id
GROUP BY e.employee_id, e.first_name, e.last_name, e.department_id
HAVING COUNT(e2.employee_id) &gt; 10;
</code></pre>

</div>


<p><small>ROLLUP and UNION for Multi‑Level Aggregation<br>
<small>ROLLUP: Departments, Managers, and Employee Counts<br>
</small></small></p>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SELECT d.department_name,
       m.first_name || ' ' || m.last_name AS manager_name,
       COUNT(e.employee_id) AS nr_angajati
FROM hr.employees e JOIN hr.employees m
     ON e.manager_id = m.employee_id JOIN hr.departments d
     ON e.department_id = d.department_id
GROUP BY ROLLUP (d.department_name, m.first_name, m.last_name);
</code></pre>

</div>


<p>UNION: Department Totals + Grand Total<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SELECT (SELECT department_name
        FROM hr.departments d
        WHERE d.department_id = e.department_id) AS department_name,
       COUNT(*) AS nr_angajati
FROM hr.employees e GROUP BY e.department_id UNION
SELECT 'TOTAL GENERAL' AS department_name,
       COUNT(*) AS nr_angajati
FROM hr.employees;
</code></pre>

</div>



<p><small>ROLLUP: Employees per Job, Subtotals per Department, Grand Total<br>
</small></p>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SELECT department_id,
       job_id,
       COUNT(employee_id) AS nr_angajati
FROM hr.employees
GROUP BY ROLLUP (department_id, job_id);
</code></pre>

</div>


<p><small>START WITH / CONNECT BY<br>
Subordinates of Employee 109 (All Levels)<br>
</small></p>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SELECT employee_id, first_name,last_name,manager_id
FROM hr.employees
START WITH manager_id = 109
CONNECT BY PRIOR employee_id = manager_id;
</code></pre>

</div>


<p><small>Managers With at Least 3 Direct Reports<br>
</small></p>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SELECT manager_id, COUNT(*) AS nr_subordonati
FROM hr.employees WHERE manager_id IS NOT NULL
GROUP BY manager_id
HAVING COUNT(*) &gt;= 3;
</code></pre>

</div>


<p>Subordinates of Employee 101 With Hierarchy Level<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SELECT first_name, last_name, LEVEL
FROM hr.employees
START WITH manager_id = 101 CONNECT BY PRIOR employee_id = manager_id;
</code></pre>

</div>



<p><small>Number of Subordinates Reporting to Bruce<br>
</small></p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SELECT COUNT(*) AS nr_subordonati
FROM hr.employees
START WITH manager_id = (SELECT employee_id
      FROM hr.employees WHERE first_name = 'Bruce'
)CONNECT BY PRIOR employee_id = manager_id
AND LEVEL = 2;
</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SELECT COUNT(*) AS nr_subordonati
FROM hr.employees e
WHERE e.manager_id IN (
      SELECT employee_id
      FROM hr.employees
      WHERE manager_id = (
            SELECT employee_id
            FROM hr.employees
            WHERE first_name = 'Bruce'
      )
);

</code></pre>

</div>





<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>SELECT COUNT(*) AS nr_subordonati FROM hr.employees lvl2
JOIN hr.employees lvl1 ON lvl2.manager_id = lvl1.employee_id
JOIN hr.employees bruce
     ON lvl1.manager_id = bruce.employee_id
WHERE bruce.first_name = 'Bruce';
</code></pre>

</div>



<p><small>Procedure: Sum of All Salaries<br>
</small></p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>CREATE OR REPLACE PROCEDURE SUMA IS
    total_salary NUMBER;
BEGIN
    SELECT SUM(salary) INTO total_salary FROM hr.employees;
DBMS_OUTPUT.PUT_LINE('Total salary sum: ' || total_salary);
END;
</code></pre>

</div>



<p><small>Procedure: Number of Employees in a Department (param)<br>
</small></p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>CREATE OR REPLACE PROCEDURE NUMAR_ANGAJATI (p_department_id NUMBER) AS
    v_count NUMBER;
BEGIN
    SELECT COUNT(*) INTO v_count FROM hr.employees
    WHERE department_id = p_department_id;
DBMS_OUTPUT.PUT_LINE('Employees in department ' ||
                         p_department_id || ': ' || v_count);
END;
</code></pre>

</div>



<p><small>Procedure: Display Employee Info by ID<br>
</small></p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>CREATE OR REPLACE PROCEDURE afiseaza_salariat (
    p_emp_id IN NUMBER
) IS v_first_name VARCHAR2(50); v_last_name  VARCHAR2(50); v_salary   NUMBER;
BEGIN
    SELECT first_name, last_name, salary INTO v_first_name, v_last_name, v_salary
    FROM hr.employees WHERE employee_id = p_emp_id;
DBMS_OUTPUT.PUT_LINE(
        'Employee ' || v_first_name || ' ' || v_last_name ||
        ' earns ' || v_salary
    );
END;
</code></pre>

</div>



