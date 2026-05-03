# Функции Sql Anywhere

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

## Строки и текст
