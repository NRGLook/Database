# 1.Создать пул запросов для сложной выборки из базы данных.

### Запросы с несколькими условиями:

1. Получить список пользователей (users), у которых есть роль "Admin" и купленные продукты (products) дороже 150 единиц:

```sql
SELECT username, email, title AS product_title, price
FROM users, roles, l_products_users, products
WHERE users.id = roles.user_id AND roles.role_name = 'Admin'
AND users.id = l_products_users.users_id AND l_products_users.products_id = products.id
AND products.price > 150;
```

2. Получить список пользователей (users), у которых количество задач (tasks) больше 2 и они просрочены:

```sql
SELECT users.username, COUNT(tasks.id) AS task_count
FROM users, tasks 
WHERE  users.id = tasks.user_id AND  tasks.due_date < NOW()
GROUP BY users.username
HAVING COUNT(tasks.id) > 2;
```

3. Запрос, который выводит статистику по количеству задач для каждого пользователя и категории.
```sql
CREATE VIEW user_task_stats_category AS 
WITH user_task_stats AS (
    SELECT u.username, l.category, COUNT(*) AS number_of_tasks
    FROM tasks t
    INNER JOIN users u ON u.id = t.user_id
    INNER JOIN labels l ON l.id = t.category_id
    GROUP BY u.id, l.id
)
SELECT uts.username, uts.category, uts.number_of_tasks,
       DENSE_RANK() OVER (PARTITION BY uts.username ORDER BY uts.number_of_tasks DESC) AS place
FROM user_task_stats uts;
SELECT username, category, number_of_tasks
FROM temp
WHERE place = 1;
```

### Запросы с вложенными конструкциями:

1. Получить список пользователей (users) и их задач (tasks), где каждая задача связана с меткой (label), у которой категория задачи равна "Category A":

```sql
SELECT users.username, tasks.title AS task_title, labels.category AS task_category
FROM users, tasks, labels 
WHERE users.id = tasks.user_id AND tasks.category_id = labels.id
AND labels.category = 'Category A';
```

2. Получить список пользователей (users), у которых есть хотя бы один ачивмент (achievments), и для каждого ачивмента указать количество пользователей, которые его получили:

```sql
SELECT users.username, achievments.achievment_name,
	(SELECT COUNT(*) FROM l_achievments_users WHERE achievment_id = achievments.id) AS user_count
FROM users, l_achievments_users, achievments
WHERE users.id = l_achievments_users.users_id  AND l_achievments_users.achievment_id = achievments.id;
```

### Прочие сложные выборки:

1. Получить список пользователей, у которых есть роль "Admin":

```sql
SELECT username, email
FROM users
WHERE id IN (SELECT user_id FROM roles WHERE role_name = 'Admin');
```

2. Найти все посты, у которых есть хотя бы один комментарий и количество комментариев более 1:

```sql
SELECT title, description 
FROM posts
WHERE id IN (SELECT post_id FROM comments GROUP BY post_id HAVING COUNT(id) > 1);
```

3. Найти сумму всех покупок для каждого пользователя:

```sql
SELECT username, (
	SELECT SUM(total_amount)
	FROM purchase_history
	WHERE purchase_history.id = users.id
) AS total_purchase_amount_user
FROM users;
```

4. Получить список задач (tasks) пользователя "user1", которые просрочены на данный момент:

```sql
SELECT title, description, due_date
FROM tasks
WHERE user_id = 1
AND due_date < NOW();
```

# 2.Создать пул запросов для получения представлений в базе данных.

### 1. INNER JOIN: Получить список постов (posts) и соответствующих им пользователей (users):

```sql
CREATE OR REPLACE VIEW post_user_view AS 
SELECT posts.id as post_id, posts.title AS post_title, posts.description AS post_description,
      users.id AS user_id, users.username AS user_username
FROM posts
INNER JOIN users ON posts.user_id = users.id;
```

### 2. LEFT OUTER JOIN: Создать представление, которое отображает всех пользователей (users) и их роли (roles), даже если у пользователя нет ролей:

```sql
CREATE OR REPLACE VIEW user_role_view AS
SELECT users.id AS user_id, users.username AS user_username,
    roles.role_name AS user_role
FROM users
LEFT JOIN roles ON users.id = roles.user_id;
```

### 3. CROSS JOIN: Создать представление, которое пересекает все пользователи (users) с продуктами (products), чтобы получить все возможные комбинации:

```sql
CREATE OR REPLACE VIEW user_product_cross_view AS
SELECT users.id AS user_id, users.username AS user_username,
    products.id AS product_id, products.title AS product_title
FROM users
CROSS JOIN products;
```

### 4. SELF JOIN: Создать представление, которое показывает связь между пользователями (users) в виде пар (пользователь 1, пользователь 2), где пользователь 1 следует за пользователем 2:

```sql
CREATE OR REPLACE VIEW user_following_view AS
SELECT u1.id AS user1_id, u1.username AS user1_username,
	u2.id AS user2_id, u2.username AS user2_username
FROM users u1
INNER JOIN users u2 ON u1.id <> u2.id;
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
SELECT users.username, AVG(tasks.progress) AS average_progress
FROM users
JOIN tasks ON users.id = tasks.user_id
GROUP BY users.username
HAVING AVG(tasks.progress) > 50;
```

2. Получить общее количество ачивментов (achievments) для каждого пользователя (users) и упорядочить результаты по убыванию количества ачивментов:

```sql
SELECT users.username, COUNT(l_achievments_users.achievment_id) AS total_achievments
FROM users
LEFT JOIN l_achievments_users ON users.id = l_achievments_users.users_id
GROUP BY users.username
ORDER BY total_achievments DESC;
```

### Использование оконных функций с PARTITION и PARTITION OVER:

3.Набор данных с каждой строкой, представленной как комбинация значений из столбцов:
* users.username
* labels.category
* products.price
Помимо этих основных столбцов, запрос также включает следующие вычисляемые столбцы:
* total_category_price: Сумма цен продуктов в каждой категории.
* total_category_count: Общее количество записей (строк) в каждой категории.
* average_category_price: Средняя цена продуктов в каждой категории.
* max_category_price: Максимальная цена продукта в каждой категории.
* row_num_in_category: Номер строки в пределах каждой категории, упорядоченной по цене продукта.
* global_rank: Глобальный ранг каждой строки в результате, упорядоченном по цене продукта:
```sql
SELECT 
    users.username,
    labels.category,
    products.price,
    SUM(products.price) OVER (PARTITION BY labels.category) AS total_category_price,
    COUNT(*) OVER (PARTITION BY labels.category) AS total_category_count,
    AVG(products.price) OVER (PARTITION BY labels.category) AS average_category_price,
    MAX(products.price) OVER (PARTITION BY labels.category) AS max_category_price,
    ROW_NUMBER() OVER (PARTITION BY labels.category ORDER BY products.price) AS row_num_in_category,
    RANK() OVER (ORDER BY products.price) AS global_rank
FROM 
    users
LEFT JOIN 
    l_products_users ON users.id = l_products_users.users_id
LEFT JOIN 
    products ON l_products_users.products_id = products.id
LEFT JOIN 
    labels ON products.title = labels.priority;

```

### HAVING:

4. Найти категории (category) из таблицы меток (labels), у которых суммарный прогресс (progress) всех задач (tasks) в этой категории более 100 и только для категорий, в которых есть хотя бы 2 задачи:

```sql
SELECT labels.category
FROM labels
LEFT JOIN tasks ON labels.id = tasks.category_id
GROUP BY labels.category
HAVING COUNT(tasks.id) > 1 AND SUM(tasks.progress) > 100;
```

### UNION:

5. Объединить результаты из двух запросов: первый запрос получает список пользователей (users), у которых есть роль "Admin", а второй запрос получает список пользователей, у которых есть роль "Super user":

```sql
SELECT users.username, users.email, 'Admin' AS role
FROM users
JOIN roles ON users.id = roles.user_id
WHERE roles.role_name = 'Admin'
UNION
SELECT users.username, users.email, 'Super user' AS role
FROM users
JOIN roles ON users.id = roles.user_id
WHERE roles.role_name = 'Super user';
```

# 4. Создать пул запросов, необходимых для сложных операций над данными в БД.

### 1. Использование EXISTS:

Получить список пользователей (users), у которых есть хотя бы одна задача (tasks):

```sql
SELECT username
FROM users
WHERE EXISTS (SELECT 1 FROM tasks WHERE tasks.user_id = users.id);
```

### 2. Использование INSERT INTO SELECT:

Создать новую таблицу "tasks_completed" и скопировать туда задачи (tasks), которые имеют прогресс (progress) более 50:

```sql
INSERT INTO tasks_completed
SELECT *
FROM tasks
WHERE progress > 50;
```

### 3. Использование CASE:

Создать запрос, который выводит "Complete" для задач (tasks) с прогрессом (progress) более 50 и "In progress" для задач с прогрессом до 50:

```sql
SELECT title,
CASE
  WHEN progress > 50 THEN 'Complete'
  ELSE 'In progress'
  END AS status
FROM tasks;
```

### 4. Использование EXPLAIN:

Использовать EXPLAIN для анализа выполнения запроса. Например, чтобы узнать, как будет выполняться запрос на выборку пользователей (users) с ролью "Admin":

```sql
EXPLAIN SELECT username
FROM users
WHERE EXISTS (SELECT 1 FROM roles WHERE roles.user_id = users.id AND roles.role_name = 'Admin');
```
