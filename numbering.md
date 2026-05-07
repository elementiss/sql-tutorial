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

Нельзя использовать в `WHERE` напрямую

Кроме того, `ROW_NUMBER` может возвращать недетерминированные результаты, когда `ORDER BY` в окне задан по неуникальным выражениям; порядок строк непредсказуем.

`ROW_NUMBER` предназначена для работы по всей секции, поэтому с функцией `ROW_NUMBER` нельзя указывать `ROWS` или `RANGE`.

Производительность. Требует сортировки. Для больших таблиц убедитесь, что столбцы в `PARTITION BY` и `ORDER BY` проиндексированы.

Когда использовать `ROW_NUMBER()`

- Присвоение стабильных идентификаторов в отчётах
- Сложная пагинация или разбиение на страницы с дополнительными условиями
- Выбор "первого" или "последнего" элемента в группе (например, последний заказ клиента)
- Выбор топ-N в каждой группе


## RANK, DENSE_RANK

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


## LAG

## LEAD

## FIRST_VALUE, LAST_VALUE

