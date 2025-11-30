# JOIN, UNION и WITH в PostgreSQL

## JOIN vs INNER JOIN

В PostgreSQL (и во многих других СУБД) `INNER JOIN` и просто `JOIN` — это одно и то же.

```sql
-- Эти запросы равнозначны:
SELECT * FROM users u
JOIN orders o ON u.id = o.user_id;

SELECT * FROM users u
INNER JOIN orders o ON u.id = o.user_id;
```

**Почему тогда пишут INNER JOIN?**

- `JOIN` — это сокращение, синтаксический сахар
- `INNER JOIN` — явное указание типа соединения, повышает читаемость
- **Рекомендация:** в больших проектах используйте `INNER JOIN`, чтобы явно показать тип соединения

---

## Типы JOIN

`JOIN` используется для объединения строк из двух (или более) таблиц на основе логического условия (обычно — по общим столбцам).

### 1. INNER JOIN (внутреннее соединение)

Возвращает только те строки, у которых есть совпадения в обеих таблицах.

```sql
SELECT *
FROM users u
INNER JOIN orders o ON u.id = o.user_id;
```

**Результат:** только те пользователи, у которых есть заказы.

### 2. LEFT JOIN (левое внешнее соединение)

Возвращает все строки из левой таблицы и совпадающие строки из правой. Если совпадений нет — справа будет `NULL`.

```sql
SELECT *
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;
```

**Результат:** все пользователи, даже если у них нет заказов.

### 3. RIGHT JOIN (правое внешнее соединение)

Аналогично `LEFT JOIN`, но с правой таблицей: все строки из правой таблицы и совпадения из левой.

```sql
SELECT *
FROM orders o
RIGHT JOIN users u ON u.id = o.user_id;
```

### 4. FULL JOIN (полное внешнее соединение)

Объединяет `LEFT` и `RIGHT JOIN` — возвращает все строки из обеих таблиц. Где нет совпадений, подставляется `NULL`.

```sql
SELECT *
FROM users u
FULL JOIN orders o ON u.id = o.user_id;
```

### 5. CROSS JOIN (декартово произведение)

Каждая строка из одной таблицы объединяется со всеми строками из другой.

```sql
SELECT *
FROM users u
CROSS JOIN products p;
```

**Результат:** если 10 пользователей и 5 продуктов — получится 50 строк.

### 6. SELF JOIN (соединение таблицы с самой собой)

Полезно для иерархий, например, сотрудники и их начальники.

```sql
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

---

## UNION

`UNION` объединяет результаты двух или более `SELECT`-запросов. Столбцы должны совпадать по количеству и типам.

### UNION (без дубликатов)

```sql
SELECT city FROM customers
UNION
SELECT city FROM suppliers;
```

**Результат:** список городов без повторений.

### UNION ALL (с дубликатами)

```sql
SELECT city FROM customers
UNION ALL
SELECT city FROM suppliers;
```

**Результат:** список городов с повторениями.

---

## WITH (Common Table Expressions)

`WITH` позволяет создавать временные подзапросы с именем, которые можно использовать в основном запросе.

```sql
WITH recent_orders AS (
  SELECT * FROM orders 
  WHERE order_date > CURRENT_DATE - INTERVAL '30 days'
)
SELECT o.id, o.order_date, u.name
FROM recent_orders o
JOIN users u ON o.user_id = u.id;
```

### Преимущества CTE

- Повышает читаемость кода
- Можно использовать рекурсивно
- Хорошо подходит для разбивки сложных запросов на логические блоки