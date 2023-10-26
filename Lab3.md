## 1.Разработать физическую модель базы данных (создать БД на вашем устройстве) + 2.Наложить на базу данных ограничения:

~~~
CREATE TABLE public."users"
(
    id bigint NOT NULL,
    username character varying(30) NOT NULL,
    email character varying(30) NOT NULL,
    password character varying(30) NOT NULL,
    CONSTRAINT users_pkey PRIMARY KEY (id)
)

CREATE TABLE public."posts"
(
    id bigint NOT NULL,
    title character varying(50) NOT NULL,
    description character varying NOT NULL,
    user_id bigint NOT NULL,
    CONSTRAINT posts_pkey PRIMARY KEY (id),
    CONSTRAINT user_id_fk FOREIGN KEY (user_id) REFERENCES public.users (id) MATCH SIMPLE
        ON UPDATE CASCADE
        ON DELETE CASCADE
)

CREATE TABLE public."comments"
(
    id bigint NOT NULL,
    content character varying(1000) NOT NULL,
    user_id bigint NOT NULL,
    post_id bigint NOT NULL,
    CONSTRAINT comments_pkey PRIMARY KEY (id),
    CONSTRAINT post_id_fk FOREIGN KEY (post_id)
        REFERENCES public.posts (id) MATCH SIMPLE
        ON UPDATE CASCADE
        ON DELETE CASCADE,
    CONSTRAINT user_id_fk FOREIGN KEY (user_id)
        REFERENCES public.users (id) MATCH SIMPLE
        ON UPDATE CASCADE
        ON DELETE CASCADE
)

CREATE TABLE public."achievments"
(
    id bigint NOT NULL,
    achievment_name character varying(50) NOT NULL,
    description character varying(1000) NOT NULL,
    styles character varying NOT NULL,
    CONSTRAINT achievments_pkey PRIMARY KEY (id)
)

CREATE TABLE public."reviews"
(
    id bigint NOT NULL,
    description character varying(1000) NOT NULL,
    create_time date NOT NULL,
    user_id bigint NOT NULL,
    CONSTRAINT reviews_pkey PRIMARY KEY (id),
    CONSTRAINT user_id_fk FOREIGN KEY (user_id)
        REFERENCES public.users (id) MATCH SIMPLE
        ON UPDATE NO ACTION
        ON DELETE NO ACTION
)

CREATE TABLE public."l_achievments_users"
(
    id bigint NOT NULL,
    achievment_id bigint NOT NULL,
    users_id bigint NOT NULL,
    CONSTRAINT l_achievments_users_pkey PRIMARY KEY (id),
    CONSTRAINT achievment_id_fk FOREIGN KEY (achievment_id)
        REFERENCES public.products (id) MATCH SIMPLE
        ON UPDATE CASCADE
        ON DELETE CASCADE,
    CONSTRAINT user_id_fk FOREIGN KEY (users_id)
        REFERENCES public.users (id) MATCH SIMPLE
        ON UPDATE CASCADE
        ON DELETE CASCADE
)

CREATE TABLE public."products"
(
    id bigint NOT NULL,
    title character varying(50) NOT NULL,
    description character varying(1000) NOT NULL,
    price bigint NOT NULL,
    CONSTRAINT products_pkey PRIMARY KEY (id)
)

CREATE TABLE public."l_products_users"
(
    id bigint NOT NULL,
    products_id bigint NOT NULL,
    users_id bigint NOT NULL,
    CONSTRAINT l_products_users_pkey PRIMARY KEY (id),
    CONSTRAINT products_id_fk FOREIGN KEY (products_id)
        REFERENCES public.products (id) MATCH SIMPLE
        ON UPDATE CASCADE
        ON DELETE CASCADE,
    CONSTRAINT user_id_fk FOREIGN KEY (users_id)
        REFERENCES public.users (id) MATCH SIMPLE
        ON UPDATE CASCADE
        ON DELETE CASCADE
)

CREATE TABLE public."labels"
(
    id bigint NOT NULL,
    category character varying(15) NOT NULL,
    priority character varying(15) NOT NULL,
    style character varying(15) NOT NULL,
    CONSTRAINT labels_pkey PRIMARY KEY (id)
)

CREATE TABLE public."purchase_history"
(
    id bigint NOT NULL,
    product_list character varying[] NOT NULL,
    total_amount bigint NOT NULL,
    user_id bigint NOT NULL,
    CONSTRAINT purchase_history_pkey PRIMARY KEY (id),
    CONSTRAINT user_id_fk FOREIGN KEY (user_id)
        REFERENCES public.users (id) MATCH SIMPLE
        ON UPDATE NO ACTION
        ON DELETE NO ACTION
)
CREATE TABLE public."roles"
(
    id bigint NOT NULL,
    role_name character varying(15) NOT NULL,
    user_id integer NOT NULL,
    CONSTRAINT roles_pkey PRIMARY KEY (id),
    CONSTRAINT user_id_fk FOREIGN KEY (user_id)
        REFERENCES public.users (id) MATCH SIMPLE
        ON UPDATE CASCADE
        ON DELETE CASCADE
)

CREATE TABLE public."tasks"
(
    id bigint NOT NULL,
    title character varying(50) NOT NULL,
    description character varying NOT NULL,
    due_date timestamp without time zone NOT NULL,
    "check" boolean NOT NULL,
    progress integer NOT NULL,
    user_id bigint NOT NULL,
    category_id bigint NOT NULL,
    CONSTRAINT tasks_pkey PRIMARY KEY (id),
    CONSTRAINT category_id_fk FOREIGN KEY (category_id)
        REFERENCES public.labels (id) MATCH SIMPLE
        ON UPDATE NO ACTION
        ON DELETE NO ACTION
        NOT VALID,
    CONSTRAINT user_id_fk FOREIGN KEY (user_id)
        REFERENCES public.users (id) MATCH SIMPLE
        ON UPDATE NO ACTION
        ON DELETE NO ACTION
)
~~~

## 3.Заполнить базу данных тестовыми значениями.

~~~
-- Заполнение таблицы "users"
INSERT INTO public."users" (id, username, email, password) VALUES
(1, 'user1', 'user1@example.com', 'password1'),
(2, 'user2', 'user2@example.com', 'password2'),
(3, 'user3', 'user3@example.com', 'password3');

-- Заполнение таблицы "posts"
INSERT INTO public."posts" (id, title, description, user_id) VALUES
(1, 'Post 1', 'Description for Post 1', 1),
(2, 'Post 2', 'Description for Post 2', 2),
(3, 'Post 3', 'Description for Post 3', 1);

-- Заполнение таблицы "comments"
INSERT INTO public."comments" (id, content, user_id, post_id) VALUES
(1, 'Comment 1 for Post 1', 2, 1),
(2, 'Comment 2 for Post 1', 3, 1),
(3, 'Comment 1 for Post 2', 1, 2);

-- Заполнение таблицы "achievments"
INSERT INTO public."achievments" (id, achievment_name, description, styles) VALUES
(1, 'Achievement 1', 'Description for Achievement 1', 'Style 1'),
(2, 'Achievement 2', 'Description for Achievement 2', 'Style 2');

-- Заполнение таблицы "reviews"
INSERT INTO public."reviews" (id, description, create_time, user_id) VALUES
(1, 'Review 1', '2023-10-17', 1),
(2, 'Review 2', '2023-10-18', 2);

-- Заполнение таблицы "l_achievments_users"
INSERT INTO public."l_achievments_users" (id, achievment_id, users_id) VALUES
(1, 1, 1),
(2, 2, 2);

-- Заполнение таблицы "products"
INSERT INTO public."products" (id, title, description, price) VALUES
(1, 'Product 1', 'Description for Product 1', 100),
(2, 'Product 2', 'Description for Product 2', 200);

-- Заполнение таблицы "l_products_users"
INSERT INTO public."l_products_users" (id, products_id, users_id) VALUES
(1, 1, 1),
(2, 2, 2);

-- Заполнение таблицы "labels"
INSERT INTO public."labels" (id, category, priority, style) VALUES
(1, 'Category 1', 'High', 'Style 1'),
(2, 'Category 2', 'Medium', 'Style 2');

-- Заполнение таблицы "purchase_history"
INSERT INTO public."purchase_history" (id, product_list, total_amount, user_id) VALUES
(1, ARRAY[1, 2], 300, 1),
(2, ARRAY[1], 100, 2);

-- Заполнение таблицы "roles"
INSERT INTO public."roles" (id, role_name, user_id) VALUES
(1, 'Role 1', 1),
(2, 'Role 2', 2);

-- Заполнение таблицы "tasks"
INSERT INTO public."tasks" (id, title, description, due_date, "check", progress, user_id, category_id) VALUES
(1, 'Task 1', 'Description for Task 1', '2023-10-20 00:00:00', true, 50, 1, 1),
(2, 'Task 2', 'Description for Task 2', '2023-10-21 00:00:00', false, 0, 2, 2);
~~~

## 4.Создать пул запросов, необходимых для простых операций над данными в БД.

~~~
INSERT (добавление данных):

-- Пример INSERT запроса для добавления нового пользователя
INSERT INTO public."users" (id, username, email, password)
VALUES (4, 'user4', 'user4@example.com', 'password4');

SELECT (извлечение данных):

-- Пример SELECT запроса для получения списка всех пользователей
SELECT * FROM public."users";

UPDATE (обновление данных):

-- Пример UPDATE запроса для изменения имени пользователя с id=1
UPDATE public."users" SET username = 'updated_user' WHERE id = 1;

DELETE (удаление данных):

-- Пример DELETE запроса для удаления пользователя с id=2
DELETE FROM public."users" WHERE id = 2;

CREATE INDEX:

-- Создание индекса на столбце "username" в таблице "users"
CREATE INDEX idx_username ON public."users" (username);
~~~

## 5.УСЛОЖНЕННЫЕ ЗАПРОСЫ:

~~~
Использование EXPLAIN для анализа выполнения запроса:
EXPLAIN SELECT * FROM public."users" WHERE username = 'user1';

Сортировка данных с помощью ORDER BY:
SELECT * FROM public."posts" ORDER BY title ASC;

Использование DISTINCT для выбора уникальных значений:
SELECT DISTINCT username FROM public."users";

Ограничение числа результатов с помощью LIMIT и OFFSET:
SELECT * FROM public."comments" LIMIT 10 OFFSET 20;

Использование WHERE и оператора BETWEEN:
SELECT * FROM public."products" WHERE price BETWEEN 1000 AND 2000;

Использование IN для выбора данных, соответствующих списку значений:
SELECT * FROM public."users" WHERE id IN (1, 2, 3);

Использование LIKE для выполнения поиска по шаблону:
SELECT * FROM public."labels" WHERE category LIKE 'Category%';

Сортировка данных в убывающем порядке с помощью DESC:
SELECT * FROM public."achievments" ORDER BY id DESC;

Использование FETCH FIRST для выбора первых N строк:
SELECT * FROM public."reviews" FETCH FIRST 5 ROWS ONLY;

Использование JOIN для объединения таблиц и выбора данных:
SELECT users.username, products.title
FROM public."l_products_users"
INNER JOIN public."users" ON l_products_users.users_id = users.id
INNER JOIN public."products" ON l_products_users.products_id = products.id;
~~~
