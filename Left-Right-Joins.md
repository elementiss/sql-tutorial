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

</details>
