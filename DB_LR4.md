# 1.Создать пул запросов для сложной выборки из базы данных.

### Запросы с несколькими условиями:

1. Получить список пользователей (users), у которых есть роль "Role 1" и купленные продукты (products) дороже 150 единиц:

```sql
SELECT u.username, u.email, p.title AS product_title, p.price
FROM public."users" u, public."roles" r, public."l_products_users" pu, public."products" p
WHERE u.id = r.user_id AND r.role_name = 'Role 1'
  AND u.id = pu.users_id AND pu.products_id = p.id
  AND p.price > 150;
```

2. Получить список пользователей (users), у которых количество задач (tasks) больше 2 и они просрочены:

```sql
SELECT u.username, COUNT(t.id) AS task_count
FROM public."users" u, public."tasks" t
WHERE u.id = t.user_id AND t.due_date < NOW()
GROUP BY u.username
HAVING COUNT(t.id) > 2;
```

### Запросы с вложенными конструкциями:

1. Получить список пользователей (users) и их задач (tasks), где каждая задача связана с меткой (label), у которой приоритет (priority) равен "High":

```sql
SELECT u.username, t.title AS task_title, l.category AS task_category
FROM public."users" u, public."tasks" t, public."labels" l
WHERE u.id = t.user_id AND t.category_id = l.id
  AND l.priority = 'High';
```

2. Получить список пользователей (users), у которых есть хотя бы один ачивмент (achievments), и для каждого ачивмента указать количество пользователей, которые его получили:

```sql
SELECT u.username, a.achievment_name,
  (SELECT COUNT(*) FROM public."l_achievments_users" au WHERE au.achievment_id = a.id) AS user_count
FROM public."users" u, public."l_achievments_users" au, public."achievments" a
WHERE u.id = au.users_id AND au.achievment_id = a.id;
```

### Прочие сложные выборки:

1. Получить список пользователей, у которых есть роль "Role 1":

```sql
SELECT username, email
FROM public."users"
WHERE id IN (SELECT user_id FROM public."roles" WHERE role_name = 'Role 1');
```

2. Найти все посты, у которых есть хотя бы один комментарий и количество комментариев более 1:

```sql
SELECT title, description
FROM public."posts"
WHERE id IN (SELECT post_id FROM public."comments" GROUP BY post_id HAVING COUNT(id) > 1);
```

3. Получить список пользователей, у которых есть хотя бы один ачивмент, а также список ачивментов, которые они получили:

```sql
SELECT u.username, a.achievment_name
FROM public."users" u
WHERE u.id IN (SELECT users_id FROM public."l_achievments_users")
AND u.id = a.id;
```

4. Найти сумму всех покупок для каждого пользователя:

```sql
SELECT username, (
    SELECT SUM(total_amount)
    FROM public."purchase_history" ph
    WHERE ph.user_id = u.id
) AS total_purchase_amount
FROM public."users" u;
```

5. Получить список задач (tasks) пользователя "user1", которые просрочены на данный момент:

```sql
SELECT title, description, due_date
FROM public."tasks"
WHERE user_id = 1
AND due_date < NOW();
```

# 2.Создать пул запросов для получения представлений в базе данных.

### 1. INNER JOIN: Получить список постов (posts) и соответствующих им пользователей (users):

```sql
CREATE OR REPLACE VIEW post_user_view AS
SELECT p.id AS post_id, p.title AS post_title, p.description AS post_description, u.id AS user_id, u.username AS user_username
FROM public."posts" p
INNER JOIN public."users" u ON p.user_id = u.id;
```

### 2. LEFT OUTER JOIN: Создать представление, которое отображает всех пользователей (users) и их роли (roles), даже если у пользователя нет ролей:

```sql
CREATE OR REPLACE VIEW user_role_view AS
SELECT u.id AS user_id, u.username AS user_username, r.role_name AS user_role
FROM public."users" u
LEFT JOIN public."roles" r ON u.id = r.user_id;
```

### 3. CROSS JOIN: Создать представление, которое пересекает все пользователи (users) с продуктами (products), чтобы получить все возможные комбинации:

```sql
CREATE OR REPLACE VIEW user_product_cross_view AS
SELECT u.id AS user_id, u.username AS user_username, p.id AS product_id, p.title AS product_title
FROM public."users" u
CROSS JOIN public."products" p;
```

### 4. SELF JOIN: Создать представление, которое показывает связь между пользователями (users) в виде пар (пользователь 1, пользователь 2), где пользователь 1 следует за пользователем 2:

```sql
CREATE OR REPLACE VIEW user_following_view AS
SELECT u1.id AS user1_id, u1.username AS user1_username, u2.id AS user2_id, u2.username AS user2_username
FROM public."users" u1
INNER JOIN public."users" u2 ON u1.id <> u2.id;
```

### 5. FULL OUTER JOIN: Создать представление, которое объединяет все ачивменты (achievments) и пользователей (users), чтобы увидеть, какие пользователи имеют ачивменты, и какие ачивменты доступны для пользователей:

```sql
CREATE OR REPLACE VIEW user_achievment_full_view AS
SELECT u.id AS user_id, u.username AS user_username, a.id AS achievment_id, a.achievment_name AS achievment_name
FROM public."users" u
FULL JOIN public."l_achievments_users" au ON u.id = au.users_id
FULL JOIN public."achievments" a ON au.achievment_id = a.id;
```

# 3.Создать пул запросов для получения сгруппированных данных.

### Группировка с использованием GROUP BY и агрегирующих функций:

1. Найти средний прогресс (progress) задач (tasks) для каждого пользователя (users) и отобразить только тех пользователей, у которых средний прогресс больше 50:

```sql
SELECT u.username, AVG(t.progress) AS average_progress
FROM public."users" u
JOIN public."tasks" t ON u.id = t.user_id
GROUP BY u.username
HAVING AVG(t.progress) > 50;
```

2. Получить общее количество ачивментов (achievments) для каждого пользователя (users) и упорядочить результаты по убыванию количества ачивментов:

```sql
SELECT u.username, COUNT(au.achievment_id) AS total_achievments
FROM public."users" u
LEFT JOIN public."l_achievments_users" au ON u.id = au.users_id
GROUP BY u.username
ORDER BY total_achievments DESC;
```

### Использование оконных функций с PARTITION и PARTITION OVER:

3. Рассчитать сумму общей стоимости продуктов (price) для каждого пользователя (users) и показать эту сумму для каждого пользователя в контексте его категории (category) из таблицы меток (labels):

```sql
SELECT u.username, l.category, p.price,
       SUM(p.price) OVER (PARTITION BY l.category) AS total_category_price
FROM public."users" u
LEFT JOIN public."l_products_users" pu ON u.id = pu.users_id
LEFT JOIN public."products" p ON pu.products_id = p.id
LEFT JOIN public."labels" l ON p.title = l.priority;
```

### HAVING:

4. Найти категории (category) из таблицы меток (labels), у которых суммарный прогресс (progress) всех задач (tasks) в этой категории более 100 и только для категорий, в которых есть хотя бы 2 задачи:

```sql
SELECT l.category
FROM public."labels" l
LEFT JOIN public."tasks" t ON l.id = t.category_id
GROUP BY l.category
HAVING COUNT(t.id) > 1 AND SUM(t.progress) > 100;
```

### UNION:

5. Объединить результаты из двух запросов: первый запрос получает список пользователей (users), у которых есть роль "Role 1", а второй запрос получает список пользователей, у которых есть роль "Role 2":

```sql
SELECT u.username, u.email, 'Role 1' AS role
FROM public."users" u
JOIN public."roles" r ON u.id = r.user_id
WHERE r.role_name = 'Role 1'
UNION
SELECT u.username, u.email, 'Role 2' AS role
FROM public."users" u
JOIN public."roles" r ON u.id = r.user_id
WHERE r.role_name = 'Role 2';
```

# 4. Создать пул запросов, необходимых для сложных операций над данными в БД.

### 1. Использование EXISTS:

Получить список пользователей (users), у которых есть хотя бы одна задача (tasks):

```sql
SELECT username
FROM public."users" u
WHERE EXISTS (SELECT 1 FROM public."tasks" t WHERE t.user_id = u.id);
```

### 2. Использование INSERT INTO SELECT:

Создать новую таблицу "tasks_completed" и скопировать туда задачи (tasks), которые имеют прогресс (progress) более 50:

```sql
CREATE TABLE public."tasks_completed" AS
SELECT *
FROM public."tasks"
WHERE progress > 50;
```

### 3. Использование CASE:

Создать запрос, который выводит "Выполнено" для задач (tasks) с прогрессом (progress) более 50 и "В процессе" для задач с прогрессом до 50:

```sql
SELECT title, CASE WHEN progress > 50 THEN 'Выполнено' ELSE 'В процессе' END AS status
FROM public."tasks";
```

### 4. Использование EXPLAIN:

Использовать EXPLAIN для анализа выполнения запроса. Например, чтобы узнать, как будет выполняться запрос на выборку пользователей (users) с ролью "Role 1":

```sql
EXPLAIN SELECT username
FROM public."users" u
WHERE EXISTS (SELECT 1 FROM public."roles" r WHERE r.user_id = u.id AND r.role_name = 'Role 1');
```
