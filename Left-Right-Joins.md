# Левые и правые джойны

## Задача 1

Посчитать остаток денежных средств на каждом пункте приема (точке) с отчетностью не чаще одного раза в день. 
Вывести: пункт, остаток.

- Таблица `income_o` с полями `point` - точка, `inc` - доход
- Таблица `outcome_o` с полями `point` - точка, `out` - расход

<details><summary> Решение </summary>

[Online Editor](https://onecompiler.com/sqlite/44p8f3dck)

```sql
SELECT c1, c2-
  (CASE
    WHEN o2 is null THEN 0
    ELSE o2
   END)
FROM
  (SELECT point c1, sum(inc) c2 FROM income_o
     group by point) as t1
  LEFT JOIN
  (SELECT point o1, sum(out) o2 FROM outcome_o
     group by point) as t2
  on c1=o1
```

Общий остаток денег (одно число):

```sql
SELECT
  (SELECT sum(inc) FROM Income_o)
  -
  (SELECT sum(out) FROM Outcome_o)
AS remain
```
</details>

## Задача 2

Вывести все операции прихода и расхода из таблиц Income и Outcome в следующем виде: 

дата, порядковый номер записи за эту дату, пункт прихода, сумма прихода, пункт расхода, сумма расхода. 
При этом все операции прихода по всем пунктам, совершённые в течение одного дня, упорядочены по полю code, и так же все операции расхода упорядочены по полю code.

В случае, если операций прихода/расхода за один день было не равное количество, выводить NULL в соответствующих колонках на месте недостающих операций. 

<details><summary> Решение </summary>

```sql
Select distinct A.date, A.R, B.point, B.inc, C.point, C.out
From
 (Select distinct date, ROW_Number() OVER(PARTITION BY date ORDER BY code asc) as R From Income
Union
 Select distinct date, ROW_Number() OVER(PARTITION BY date ORDER BY code asc) From Outcome) A
LEFT Join (Select date, point, inc
                , ROW_Number() OVER(PARTITION BY date ORDER BY code asc) as RI FROM Income
           ) B on B.date=A.date and B.RI=A.R
LEFT Join (Select date, point, out
                , ROW_Number() OVER(PARTITION BY date ORDER BY code asc) as RO FROM Outcome
           ) C on C.date=A.date and C.RO=A.R
```
</details>
