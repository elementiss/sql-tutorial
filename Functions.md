# Функции и переменные в Sql Anywhere. Оператор CASE


> [!NOTE]
> В презентации ссылается на документацию [SyBooks Online](https://infocenter.sybase.com/help/index.jsp?topic=%2Fcom.sybase.help.sqlanywhere.12.0.1%2Fdbusage%2Fug-olap-s-51258147.html)

## Дата-время

`GETDATE()` - текущее время и дата

`DATEPART` – повертає частину дати у вигляді числа. [DATEPART](https://help.sap.com/docs/SAP_SQL_Anywhere/93079d4ba8e44920ae63ffb4def91f5b/81f6395d6ce2101493a1988dc0ef0ecd.html?locale=en-US) не входит в стандарт.

- Month - MM
- Year - YY
- Quarter - QQ
- Day - DD (1-31)
- Dayofyear - DY
- Weekday - DW (Monday is 1 and Sunday is 7)
- Hour - HH - 0-23
- Minute - MI - 0-59
- Second - SS - 0-59
- Millisecond - MS и др

`DATEADD` – дозволяє додати до дати певну кількість днів, місяців або ж років.
Працює також і з від’ємними значеннями. Дни можно добавлять и вычитать гораздо проще: +1 или -1 к дате.

`DATEDIFF` – знаходить різницю між датами по вказаному компоненту дати. [DATEDIFF](https://help.sap.com/docs/SAP_SQL_Anywhere/93079d4ba8e44920ae63ffb4def91f5b/81f61e7f6ce21014babfb284bdc19007.html?locale=en-US)

`DATEFORMAT` – перетворює дату в рядок, використовуючи вказаний формат. [DATEFORMAT](https://help.sap.com/docs/SAP_SQL_Anywhere/93079d4ba8e44920ae63ffb4def91f5b/81f627596ce21014888b92bc7ad3e3f9.html?locale=en-US)

- YY, YYYY
- MM, MMM, MMMM и т.д. MMM produces JAN. mmm produces jan. yyyy/Mm/Dd could produce 2002/1/1 - смешанный регистр подавляет ведущие нули
- D, DDD[d...] - 1 = Sunday, 7 = Saturday, день недели
- DD - день месяца 1-31
- HH - часы
- NN - минуты 
- SS - секунды

`DATEFLOOR` - округлює дату до вказаної частини

`YEAR` – год в виде числа

`MONTH` – повертає номер місяця

`DAY` – повертає номер дня (1-31)

`HOUR` – повертає частину дати (годину) у вигляді числа

`DAYNAME` – повертає назву дня. Sunday, Monday, Tuesday, Wednesday, Thursday, Friday, Saturday

`MONTHNAME` – повертає назву місяця. September

`QUARTER` – повертає квартал (число) 1 2 3 4

```sql
SELECT DATEPART(month, GETDATE()) FROM dbo.iq_dummy
SELECT DATEPART( month , '1987/05/02' );

SELECT DATEADD(dd, 43829, '1900-01-01') FROM dbo.iq_dummy
SELECT DATEADD(month, 12, '2015/05/02');
SELECT DATEADD(day, -31, NOW());

SELECT DATEDIFF(day, '2020-07-01', '2020-02-15') FROM dbo.iq_dummy
SELECT DATEDIFF(hour, '4:00AM', '5:50AM');
SELECT DATEDIFF(day, '1999/07/19 00:00', '1999/07/23 23:59'); -- 4
SELECT DATEDIFF(month, '1999/07/19', '1999/07/23'); -- 0

SELECT DATEFORMAT(GETDATE(), 'YYYY-MM-DD') FROM dbo.iq_dummy
SELECT DATEFORMAT('1989-01-01', 'Mmm dd, yyyy');

SELECT DATEFLOOR(month, GETDATE()) FROM dbo.iq_dummy
SELECT DATEFLOOR(quarter, GETDATE()) FROM dbo.iq_dummy

SELECT YEAR(GETDATE()) FROM dbo.iq_dummy
SELECT MONTH(GETDATE()) FROM dbo.iq_dummy
SELECT DAY(GETDATE()) FROM dbo.iq_dummy
SELECT HOUR(GETDATE()) FROM dbo.iq_dummy
SELECT DAYNAME(GETDATE()) FROM dbo.iq_dummy
SELECT MONTHNAME(GETDATE()) FROM dbo.iq_dummy
SELECT QUARTER(GETDATE()) FROM dbo.iq_dummy
```

Еще есть в доке

- DATE(выражение) - возвращает дату, отбрасывает время
- TODAY() - текущая дата
- NOW(*) - время и дата

## Строки и текст

LEN – повертає довжину рядка. синоним LENGTH = BYTE_LENGTH, есть еще CHAR_LENGTH

TRIM, LTRIM, RTRIM – видаляє відступи зліва і справа або лише з однієї сторони.

LEFT – виводить вказану кількість символів з лівого краю рядка. 

RIGHT діє аналогічно для правого краю.

[SUBSTRING(s, start, length)](https://help.sap.com/docs/SAP_SQL_Anywhere/93079d4ba8e44920ae63ffb4def91f5b/81fdb87b6ce21014aee8bcac40855664.html?locale=en-US) – вирізає із рядка частину певної довжини, починаючи з вказаного символу. Синоним SUBSTR

REPLACE(s, что, на-что-заменять) – замінює частину рядка на вказаний текст. заменяет все вхождения

PATINDEX('%pattern%', s) – знаходить позицію (номер по порядку) для першого входження певного шаблону в рядку. Працює по типу LIKE. Если не найдено, возвращает 0. Поддерживает:

- `%` - любая строка, в том числе пустая
- `_` - один символ
- `[]` - Any single character in the specified range or set. Например `LIKE 'sm[iy]th'`, `LIKE '[a-r]ough'`
- `[^]` - Any single character not in the specified range or set
- экранирование `[[]` - просто символ `[`, `[_]`,`[%]`,`[^]`

CHARINDEX(что, s) – знаходить позицію (номер по порядку) для першого входження шуканого тексту в рядку.

LOCATE – знаходить позицію (номер по порядку) шуканого тексту в рядку. Для цієї функції можна задати позицію, з якої потрібно почати шукати.

UPPER – переводить всі символи в рядку в верхній регістр.

LOWER - переводить всі символи в рядку в нижній регістр.

CHAR – на вхід приймає число від 0 до 255, повертає відповідний символ https://www.ascii-code.com/CP1251

ASCII – повертає числове значення відповідно ASCII первого символа строки или 0 для пустой строки

```sql
SELECT LENGTH( 'chocolate' ); -- 9

SELECT LEFT('Example', 3) FROM dbo.iq_dummy  -- Exa
SELECT SUBSTRING('Cut example', 5, 7) FROM dbo.iq_dummy

SELECT REPLACE('Example', 'ple', 'EN') FROM dbo.iq_dummy
SELECT REPLACE(REPLACE('K1,; K2', '; ', ''), ' ', '') FROM dbo.iq_dummy

SELECT PATINDEX('_xam%', 'Example') FROM dbo.iq_dummy
SELECT PATINDEX( '%hoco%', 'chocolate' ); -- 2
SELECT PATINDEX( '%4_5_', '0a1A 2a3A 4a5A' ); -- 11

SELECT CHARINDEX('SQL','Learn SQL') FROM dbo.iq_dummy
WHERE CHARINDEX( 'K', Surname ) = 1 -- начинается на K

SELECT LOCATE('example exam', 'ex', 5) FROM dbo.iq_dummy
SELECT LOCATE(
   'office party this week ...', 'party', 2);  -- 8

SELECT UPPER('Test string') FROM dbo.iq_dummy
SELECT LOWER('Learn SQL') FROM dbo.iq_dummy

SELECT CHAR(70) FROM dbo.iq_dummy
SELECT ASCII('K'), ASCII('К') FROM dbo.iq_dummy
SELECT ASCII('Київ') FROM dbo.iq_dummy

```
Следующее выражение можно использовать для извлечения всего до и включая первый неалфавитно-цифровой символ в строке:

```sql
SELECT LEFT( @string, PATINDEX( '%[^a-zA-Z0-9]%', @string ) );
```

## Числа 

ROUND – заокруглює число до вказаної кількості після коми.

ABS – повертає модуль числа.

POWER – підносить число у вказану степінь.

ISNUMERIC – визначає чи є значення числом.

```sql
SELECT ROUND(1234.5678, 2) FROM dbo.iq_dummy
SELECT ABS(-123) FROM dbo.iq_dummy
SELECT POWER(3, 2) FROM dbo.iq_dummy
SELECT ISNUMERIC(12.34) FROM dbo.iq_dummy
SELECT ISNUMERIC('24') FROM dbo.iq_dummy
SELECT ISNUMERIC('Learn SQL') FROM dbo.iq_dummy
```

При математичних операціях Sybase округляє значення. Щоб цього уникнути є
невеличкий «лайфхак» – множення на 1.0

```sql
SELECT TOP 1 1.0*ininum/iniden FROM dbo.tb_sc_vRates
```

Також, через округлення, можна використовувати такі умови:

```sql
WHERE TopicID/100 = 26
```

## Приведение типов

Для перетворення типів використовують функції `CAST` і `CONVERT`.

[CAST](https://help.sap.com/docs/SAP_SQL_Anywhere/93079d4ba8e44920ae63ffb4def91f5b/81f441b06ce210149ce9bffbc27402fc.html?locale=en-US) перетворює вказане значення у вибраний тип.

[CONVERT](https://help.sap.com/docs/SAP_SQL_Anywhere/93079d4ba8e44920ae63ffb4def91f5b/81f4f0cf6ce21014aa7eddf86712f7b1.html?locale=en-US) робить те ж саме, але має можливість задати точний формат.

```sql
SELECT CAST(123 AS varchar(3)) FROM dbo.iq_dummy
SELECT CAST( '2000-10-31' AS DATE );
SELECT CAST( 1 + 2 AS CHAR );
SELECT CAST ( 'Surname' AS CHAR(5) ); -- обрезка строки

SELECT CONVERT(VARCHAR(10), GETDATE(), 104) FROM dbo.iq_dummy
SELECT CONVERT( integer, 5.2 );  -- 5
```

- 104 это dd.mm.yy[yy]
- 103 это dd/mm/yy[yy]
- 102 это [yy]yy.mm.dd
- 108 это hh:nn:ss и др

В більшості випадків достатньо CAST, але для дат частіше всього лише
CONVERT може правильно виконати перетворення.

Окремо для чисел в шістнадцятирічній системі є наступні команди перетворення:

```sql
SELECT HEXTOINT('0x00000003')
SELECT INTTOHEX(3)
```

##  COALESCE

[COALESCE](https://help.sap.com/docs/SAP_SQL_Anywhere/93079d4ba8e44920ae63ffb4def91f5b/81f4a26b6ce21014959b96490ac5f8fe.html?locale=en-US)

В функцию необходимо передать не менее двух выражений, и все выражения должны быть сравнимыми.

возвращает первое не-NULL выражение из списка.

Результат равен NULL только в том случае, если все аргументы равны NULL.

```sql
SELECT COALESCE( NULL, 34, 13, 0 ); -- 34
```

## Переменные

Змінна це об’єкт, який зберігає деяке значення.

Назва локальної змінної (на DWH ми використовуємо локальні) повинна розпочинатись з символу `@`.

Для змінної обов’язково потрібно вказувати тип даних.

Для створення змінних використовуються команди `DECLARE` або `CREATE VARIABLE`.

[DECLARE](https://help.sap.com/docs/SAP_SQL_Anywhere/93079d4ba8e44920ae63ffb4def91f5b/816dc0576ce210149edaa1e0e7ff651c.html?locale=en-US) створить змінну в рамках поточного запиту, яка буде недоступною після
його завершення, [CREATE VARIABLE](https://help.sap.com/docs/SAP_SQL_Anywhere/93079d4ba8e44920ae63ffb4def91f5b/816d2ee66ce2101495f2da956754df03.html?locale=en-US) створить змінну доступну протягом існуючого підключення.

Для присвоєння значення змінній використовується оператор `SET`.

```sql
DECLARE @Example DATE;
SET @Example = '2020-01-01';
```

Можна присвоїти значення відразу після оголошення змінної
```sql
DECLARE @StartDate DATE = '2020-01-01';
```

Сохранение результата запроса в переменной:
```
declare @DateEnd date = (select getdate()+1 from dbo.iq_dummy).
```

Примеры - не проверяла
```
DECLARE @Counter INT = 0;
DECLARE @Name NVARCHAR(100) = 'Test';
SET @Counter = @Counter + 1;
SELECT @Name;
--
declare @veryhigh money 
select @veryhigh = max (price) 
    from titles
--
SELECT COUNT(*) INTO @counter 
    FROM Customers 
    WHERE City = 'Berlin';
```

В SAP Sybase ASE и SQL Server переменные надо начинать с @, в SQl Anywhere - не надо. ASE (Adaptive Server Enterprise)

@ позволяет чётко отличать переменные от имён колонок, таблиц и литералов.

## Оператор CASE 

Оператор [CASE](https://help.sap.com/docs/SAP_SQL_Anywhere/93079d4ba8e44920ae63ffb4def91f5b/3be45efb6c5f1014bd8df5da41b23288.html?locale=en-US) в залежності від вказаних умов повертає одне з множини можливих значень.

```sql
-- 1 форма
CASE expression
    WHEN value1 THEN result1
    WHEN value2 THEN result2
    ...
    ELSE default_result
END

-- 2 форма
CASE
  WHEN [ условие | NULL] THEN statement-list ...
  WHEN [ условие | NULL] THEN statement-list ...
  ELSE statement-list 
END [ CASE ]


SELECT 
    customer_id,
    order_amount,
    CASE 
        WHEN order_amount >= 1000 THEN 'VIP'
        WHEN order_amount >= 500  THEN 'Premium'
        WHEN order_amount >= 100  THEN 'Standard'
        ELSE 'New'
    END AS customer_segment
FROM orders;
```

Значення в CASE повинні бути одного типу. Неможна перетворювати в
число, текст і дату, лише в щось одне в рамках одного CASE.

Конструкцію CASE можна використовувати не лише тільки для
створення нових стовпців, але і в умовах WHERE і в об’єднанні таблиць

Пример: CASE в агрегатных функциях

```sql
SELECT 
    department,
    COUNT(*) AS total_employees,
    SUM(CASE WHEN gender = 'M' THEN 1 ELSE 0 END) AS male_count,
    SUM(CASE WHEN gender = 'F' THEN 1 ELSE 0 END) AS female_count,
    AVG(CASE WHEN salary > 80000 THEN salary END) AS high_salary_avg
FROM employees
GROUP BY department;
```

Пример в UPDATE
```sql
UPDATE employees
SET bonus = 
    CASE 
        WHEN performance_rating = 'A' THEN salary * 0.20
        WHEN performance_rating = 'B' THEN salary * 0.15
        WHEN performance_rating = 'C' THEN salary * 0.05
        ELSE 0
    END
WHERE hire_date >= '2024-01-01';
```

Пример. CASE в ORDER BY (приоритетная сортировка)

```sql
SELECT * FROM tasks
ORDER BY 
    CASE status
        WHEN 'Critical' THEN 1
        WHEN 'High'     THEN 2
        WHEN 'Medium'   THEN 3
        ELSE 4
    END,
    due_date ASC;
```

Если ELSE не указан и ни одно условие не выполнилось → возвращается `NULL`.

Проверка на IS NULL

```
SELECT 
    name,
    CASE 
        WHEN middle_name IS NULL THEN 'Нет отчества'
        WHEN middle_name = '' THEN 'Пустое отчество'
        ELSE middle_name
    END AS middle_name_status
FROM customers;
```

CASE вычисляется последовательно сверху вниз. Ставьте самые вероятные/дешёвые условия первыми.

Еще есть в доке [IF Statement](https://help.sap.com/docs/SAP_SQL_Anywhere/93079d4ba8e44920ae63ffb4def91f5b/8171069a6ce210148e5b8c02ad6d4fdb.html?locale=en-US)

```sql
IF условие THEN statement-list
[ ELSEIF { условие | operation-type } THEN statement-list ] ...
[ ELSE statement-list ]
{ END IF | ENDIF }
```
