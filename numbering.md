# Нумерация и ранг строк

## ROW_NUMBER - нумерация строк 

[ROW_NUMBER](https://infocenter.sybase.com/help/index.jsp?topic=%2Fcom.sybase.help.sqlanywhere.12.0.1%2Fdbusage%2Fug-olap-s-51258147.html)

Присваивает каждой строке в результирующем наборе уникальный последовательный номер, начиная с 1. Нумерация зависит от порядка строк и может сбрасываться для каждой группы.

Номера всегда уникальны — даже если значения в `ORDER BY` одинаковые, функция всё равно присвоит разные номера

```sql
SELECT Column1, ROW_NUMBER() OVER (ORDER BY Column1) FROM Table1
```
Пример. [Online Editor](https://onecompiler.com/mariadb/44ngsbdcg)

Після оператора `OVER` в дужках перераховуються додаткові
параметри, які потрібно застосувати при нумерації:

- `PARTITION BY` – необязателен. Делит данные на логические группы (как `GROUP BY`, но не сворачивает строки). В каждой группе нумерация начинается заново с 1.
- `ORDER BY` – сортування (також в кінці можемо вказати як саме сортувати: ASC/DESC)


```sql
CREATE TABLE Customers (
    CustomerId   INT PRIMARY KEY,
    CustomerName  VARCHAR(100), 
    City          VARCHAR(100)
);

INSERT INTO Customers VALUES
(1, 'Иванов', 'Львов'),
(2, 'Петрова', 'Киев'),
(3, 'Сидоров', 'Житомир'),
(4, 'Кузнецова', 'Львов'),
(5, 'Смирнов', 'Львов'),
(6, 'Васильева', 'Киев');

SELECT 
   CustomerId, 
   ROW_NUMBER() OVER (ORDER BY CustomerName) as N,
   CustomerName FROM Customers 
ORDER BY CustomerName DESC;

-- первый ORDER BY используется для нумерации, а второй - для вывода


SELECT 
   CustomerId, City,
   ROW_NUMBER() OVER (PARTITION BY City ORDER BY CustomerName) as N,
   CustomerName FROM Customers 
ORDER BY City, CustomerName;

-- в каждом городе нумерация начинается с 1
```

Нумерація рядків часто використовується для пошуку першого
значення в діапазоні дат, коли в довідниках немає дати початку і кінця, а лише дата зміни.

Можно использовать `ROW_NUMBER` в производной таблице, чтобы можно было применять дополнительные ограничения, даже соединения, над значениями `ROW_NUMBER`:

см [Online Editor](https://onecompiler.com/mariadb/44ngu82va)

```sql
SELECT *
FROM ( SELECT Description, Quantity,
       ROW_NUMBER( ) OVER ( ORDER BY ID ASC ) AS RowNum
       FROM Products ) AS DT
WHERE RowNum <= 3
ORDER BY RowNum;
```

Нельзя использовать в `WHERE` напрямую:

> [!Note]
> Оконные функции вычисляются после `WHERE`, но до `ORDER BY` основного запроса.

Кроме того, `ROW_NUMBER` может возвращать недетерминированные результаты, когда `ORDER BY` в окне задан по неуникальным выражениям; порядок строк непредсказуем.

`ROW_NUMBER` предназначена для работы по всей секции, поэтому с функцией `ROW_NUMBER` нельзя указывать `ROWS` или `RANGE`.

Производительность. Требует сортировки. Для больших таблиц убедитесь, что столбцы в `PARTITION BY` и `ORDER BY` проиндексированы.

Когда использовать `ROW_NUMBER()`

- Присвоение стабильных идентификаторов в отчётах
- Сложная пагинация или разбиение на страницы с дополнительными условиями
- Выбор "первого" или "последнего" элемента в группе (например, последний заказ клиента)
- Выбор топ-N в каждой группе


## RANK, DENSE_RANK - ранг

Тоже для нумерации строк.

```sql
SELECT Column1,
  RANK() OVER (PARTITION BY Column0 ORDER BY Column1 ASC)
  FROM Table1

SELECT Column1,
  DENSE_RANK() OVER (PARTITION BY Column0 ORDER BY Column1 DESC)
  FROM Table1
```

Різниця між цими трьома варіантами у роботі з дублікатами.

- `ROW_NUMBER` здійснить послідовну нумерацію незалежно від наявності дублікатів (1,2,3,4)
- `DENSE_RANK` для дублікатів поверне однакові номери, при цьому нумерація буде
послідовною, без ‘прогалини’ після дублікатів (1,1,2,3)
- `RANK` для дублікатів поверне однакові номери, після дублікатів буде ‘прогалина’ в
нумерації (1,1,3,4)

### Задачи

#### Подготовка данных:

[Online Editor](https://onecompiler.com/mariadb/44np44vpm)

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    department VARCHAR(50),
    salary NUMERIC(10, 2)
);

INSERT INTO employees (id, name, department, salary) VALUES
(1, 'Иванов',    'IT',    95000.00),
(2, 'Петрова',   'IT',    95000.00),
(3, 'Сидоров',   'IT',    85000.00),
(4, 'Кузнецова', 'IT',    70000.00),
(5, 'Смирнов',   'Sales', 110000.00),
(6, 'Попова',    'Sales', 110000.00),
(7, 'Волков',    'Sales', 90000.00),
(8, 'Новикова',  'Sales', 85000.00),
(9, 'Морозов',   'HR',    80000.00),
(10, 'Васильева','HR',    80000.00),
(11, 'Лебедев',  'HR',    75000.00),
(12, 'Соколова', 'HR',    65000.00);
```
#### Задача 1. Базовый ROW_NUMBER()

Пронумеровать всех сотрудников по убыванию зарплаты. Номера должны быть уникальными (1, 2, 3...). Если зарплаты совпадают, нумеровать в алфавитном порядке.

<details>
<summary>Решение</summary>

```sql
SELECT name, salary,
       ROW_NUMBER() OVER (ORDER BY salary DESC, name ASC) as row_num
FROM employees;
```
Без вторичной сортировки `name ASC` порядок при совпадающих зарплатах не гарантирован.
</details>

#### Задача 2. Базовый DENSE_RANK()

Построить плотный рейтинг по зарплате. При совпадающих значениях ранг совпадает, но следующий номер не пропускается.

<details>
<summary>Решение</summary>

```sql
SELECT name, salary,
       DENSE_RANK() OVER (ORDER BY salary DESC) as dense_rnk
FROM employees;
```
</details>

#### Задача 3. Сравнение всех трёх функций

Вывести имя, зарплату и результаты всех трёх функций в одном запросе, чтобы наглядно увидеть разницу.

<details>
<summary>Решение</summary>

```sql
SELECT name, salary,
       ROW_NUMBER() OVER (ORDER BY salary DESC) as rn,
       RANK()       OVER (ORDER BY salary DESC) as rk,
       DENSE_RANK() OVER (ORDER BY salary DESC) as drk
FROM employees;
```
</details>

#### Задача 4. ROW_NUMBER() внутри отделов (PARTITION BY)

Пронумеровать сотрудников внутри каждого отдела отдельно, сортируя по зарплате от высокой к низкой.

<details>
<summary>Решение</summary>

```sql
SELECT department, name, salary,
       ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC, name ASC) as dept_row
FROM employees
ORDER BY department, dept_row;
```
</details>

#### Задача 5. Топ-2 сотрудника в каждом отделе

Вывести сотрудников, чья зарплата входит в топ-2 по отделу. Если на 2-м месте несколько человек с одинаковой зарплатой, показать их всех.

<details>
<summary>Решение</summary>

```sql
WITH Ranked AS (
    SELECT department, name, salary,
           DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) as drnk
    FROM employees
)
SELECT department, name, salary, drnk
FROM Ranked
WHERE drnk <= 2
ORDER BY department, drnk, salary DESC;
```

Если ваша СУБД не поддерживает CTE (`WITH ... AS`), классическая альтернатива — вложенный подзапрос в блоке `FROM`. 

Решение через подзапрос

```sql
SELECT department, name, salary, drnk
FROM (
    SELECT department, name, salary,
           DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) as drnk
    FROM employees
) AS ranked_employees
WHERE drnk <= 2
ORDER BY department, drnk, salary DESC;
```
</details>

#### Задача 6. Выбор одного лучшего сотрудника на отдел

Для каждого отдела оставить только одного сотрудника с максимальной зарплатой. Если таких несколько, выбрать того, чьё имя идёт первым по алфавиту.

<details>
<summary>Решение</summary>

```sql
WITH Ranked AS (
    SELECT department, name, salary,
           ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC, name ASC) as rn
    FROM employees
)
SELECT department, name, salary
FROM Ranked
WHERE rn = 1;
```

Это классический паттерн "удаления дубликатов" или выбора "одной записи на группу".
</details>

 
#### Задача 7. Комбинация ранга и агрегата

Вывести ранг сотрудника в отделе (RANK) и общее количество сотрудников в этом же отделе (COUNT), используя оконные функции в одном запросе.

<details>
<summary>Решение</summary>

```sql
SELECT department, name, salary,
       RANK() OVER (PARTITION BY department ORDER BY salary DESC) as rnk,
       COUNT(*) OVER (PARTITION BY department) as dept_size
FROM employees;
```

считает строки в каждом разделе без сворачивания результата.
</details>


## LAG

## LEAD

## FIRST_VALUE, LAST_VALUE

