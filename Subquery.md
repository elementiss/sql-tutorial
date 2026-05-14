# Вложенные подзапросы

Возможны три варианта:

- Подзапрос возвращает одно единственное значение (одна колонка и одна строка). Пример: задача 1
- Подзапрос возвращает список значений (таблица с одной колонкой). Пример: задача 2
- Подзапрос возвращает таблицу (много колонок, любое количество строк)

## Подготовка данных

[Online Editor](https://onecompiler.com/sqlite/44nqkuk8k)

```sql
CREATE TABLE departments (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    location VARCHAR(50)
);

CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    department_id INT,
    salary DECIMAL(10,2),
    hire_date DATE,
    manager_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(id),
    FOREIGN KEY (manager_id) REFERENCES employees(id)
);

INSERT INTO departments VALUES
(10, 'IT', 'Киев'),
(20, 'HR', 'Винница'),
(30, 'Sales', 'Киев'),
(40, 'Finance', 'Львов');

INSERT INTO employees VALUES
(1, 'Иванов Иван', 10, 85000.00, '2020-01-15', NULL),
(2, 'Петрова Анна', 10, 95000.00, '2019-03-10', 1),
(3, 'Сидоров Алексей', 10, 70000.00, '2021-06-20', 2),
(4, 'Кузнецова Мария', 20, 65000.00, '2018-11-05', NULL),
(5, 'Смирнов Дмитрий', 20, 72000.00, '2020-08-12', 4),
(6, 'Попова Елена', 30, 80000.00, '2017-04-25', NULL),
(7, 'Волков Сергей', 30, 88000.00, '2019-09-01', 6),
(8, 'Новикова Ольга', 30, 60000.00, '2022-02-14', 7),
(9, 'Морозов Павел', 40, 90000.00, '2016-12-01', NULL),
(10, 'Лебедева Татьяна', 40, 75000.00, '2021-10-10', 9),
(11, 'Соколов Артем', 10, 105000.00, '2015-07-22', 1),
(12, 'Васильева Дарья', 20, 68000.00, '2023-01-15', 4);
```

## Задача 1. `AVG`

Найти всех сотрудников, чья зарплата выше средней зарплаты по компании.

<details><summary> Решение </summary>

```sql
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```
Внутренний запрос вычисляет одно значение (среднее), которое используется для сравнения во внешнем запросе.
</details>

## Задача 2. `IN`

Вывести названия отделов, в которых работает хотя бы один сотрудник с зарплатой выше 88 000.

<details><summary> Решение </summary>

```sql
SELECT d.name
FROM departments as d
WHERE d.id IN (
    SELECT department_id 
    FROM employees 
    WHERE salary > 88000
);
```
Подзапрос возвращает список `department_id`, который проверяется оператором `IN`.
</details>

## Задача 3. IN

Найти сотрудников, которые работают в отделах, расположенных в Киеве.

<details><summary> Решение </summary>

```sql
SELECT e.name, e.hire_date
FROM employees as e
WHERE e.department_id IN (
    SELECT d.id 
    FROM departments as d
    WHERE location = 'Киев'
);
```
</details>



## Задача 4. NOT EXISTS

Найти отделы, в которых **нет** сотрудников, принятых на работу в 2022 году или позже.

<details><summary> Решение </summary>

```sql
SELECT d.name
FROM departments d
WHERE NOT EXISTS (
    SELECT 1 
    FROM employees e 
    WHERE e.department_id = d.id 
      AND e.hire_date >= '2022-01-01'
);
```

`NOT EXISTS` проверяет отсутствие строк в коррелированном подзапросе. Безопаснее, чем `NOT IN`, так как не ломается при наличии `NULL`.
</details>

## Задача 5. Коррелированный подзапрос

Найти сотрудников, чья зарплата выше средней зарплаты **в их собственном отделе**.

<details><summary> Решение </summary>

```sql
SELECT e.name, e.department_id, e.salary
FROM employees e
WHERE salary > (
    SELECT AVG(salary) 
    FROM employees e2 
    WHERE e2.department_id = e.department_id
);
```
Внутренний запрос выполняется для каждой строки внешнего запроса, используя текущее значение `department_id`.
</details>

## Задача 6. `EXISTS`

Найти всех менеджеров (тех, кто управляет хотя бы одним сотрудником).

<details><summary> Решение </summary>

```sql
SELECT m.name
FROM employees m
WHERE EXISTS (
    SELECT 1 
    FROM employees e 
    WHERE e.manager_id = m.id
);
```
</details>

## Задача 7. Подзапрос с агрегатной функцией

Найти сотрудников, чья зарплата больше максимальной зарплаты в отделе HR.

<details><summary> Решение </summary>

```sql
SELECT e.name, e.salary
FROM employees e
WHERE salary > (
    SELECT MAX(salary) 
    FROM employees 
    WHERE department_id = (
        SELECT id 
        FROM departments d
        WHERE d.name = 'HR'
    )
);
```
Двойная вложенность: сначала ищем id отдела HR, затем максимальную зарплату, затем сравниваем.
</details>

## Задача 8. `MAX`

Найти отдел, у которого суммарный фонд зарплат выше, чем у любого другого отдела (отдел с максимальной суммой зарплат).

<details><summary> Решение </summary>

```sql
SELECT d.id, d.name, SUM(e.salary) AS total_salary
FROM departments d
JOIN employees e ON d.id = e.department_id
GROUP BY d.id, d.name
HAVING SUM(e.salary) = (
    SELECT MAX(dept_sum)
    FROM (
        SELECT SUM(salary) AS dept_sum
        FROM employees
        GROUP BY department_id
    ) t
);
```
</details>

## Задача 9. JOIN с подзапросов (производная таблица)

Для каждого отдела вывести его название и среднюю зарплату, но только для тех отделов, где средняя зарплата превышает 70 000.

<details><summary> Решение </summary>

```sql
SELECT d.name, a.avg_salary
FROM departments d
JOIN (
    SELECT department_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
    HAVING AVG(salary) > 70000
) AS a ON d.id = a.department_id;
```
Подзапрос в `FROM` выполняется первым, его результат используется как временная таблица для `JOIN`.

Важно, что используется INNER JOIN, а не LEFT JOIN.
</details>

----

## Задача 10.

Вывести фамилии сотрудников с минимальной зарплатой

<details><summary> Решение </summary>

```sql
Select name from employees
where salary = (Select min(salary) from employees)
```
</details>
