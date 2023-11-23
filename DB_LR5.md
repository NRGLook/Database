# 1. Триггеры:

### 1. **Триггер для регистрации действий пользователей:**

   Выполняет логирование изменений в таблицах базы данных, выводя сообщение в консоль PostgreSQL и добавляя запись в таблицу log_table. 

   ```sql
CREATE OR REPLACE FUNCTION public.log_changes()
    RETURNS trigger
    LANGUAGE 'plpgsql'
    COST 100
    VOLATILE NOT LEAKPROOF
AS $BODY$
BEGIN
    -- Вывод сообщения в консоль о событии
    RAISE NOTICE 'Table %, ID %, Change field % to %', 
                 TG_TABLE_NAME, 
                 NEW.id, 
                 TG_OP, 
                 ROW(NEW.*)::text;
    -- Изменено: проверка наличия NEW.id перед вставкой
    INSERT INTO public.log_table (table_name, action_type, record_id, change_time)
    VALUES (
        TG_TABLE_NAME,
        TG_OP,
        CASE 
            WHEN TG_OP = 'DELETE' THEN OLD.id
            ELSE NEW.id
        END,
        CURRENT_TIMESTAMP
    );
    RETURN NEW;	
END;
$BODY$;


CREATE TRIGGER log_changes_trigger
AFTER INSERT OR UPDATE OR DELETE
ON public.comments
FOR EACH ROW
EXECUTE FUNCTION public.log_changes();
   ```

Пример использования:

```sql
-- Изменение столбца user_id в таблице "purchase_history" разрешать значение NULL
ALTER TABLE public."purchase_history" ALTER COLUMN user_id DROP NOT NULL;

-- Обновление записей в "purchase_history" перед удалением пользователя
UPDATE public."purchase_history" SET user_id = NULL WHERE user_id = 1;

-- Удаление пользователя
DELETE FROM public."users" WHERE id = 1;
```

### 2. **Триггер для автоматического обновления суммы покупок:**

   Этот триггер update_purchase_total предназначен для автоматического обновления данных в таблице purchase_history при вставке новых записей в таблицу l_products_users.  Триггер update_purchase_total автоматически поддерживает актуальные данные о покупках в таблице purchase_history для каждого пользователя в зависимости от вставленных данных в таблицу l_products_users:

   ```sql
   CREATE OR REPLACE FUNCTION public.update_purchase_total()
       RETURNS trigger
       LANGUAGE 'plpgsql'
       COST 100
       VOLATILE NOT LEAKPROOF
   AS $BODY$
   BEGIN
       -- Проверяем, существует ли запись с user_id в purchase_history
       IF EXISTS (SELECT 1 FROM purchase_history WHERE user_id = NEW.users_id) THEN
           -- Если запись существует, обновляем данные
           UPDATE purchase_history
           SET 
               product_list = array_append(purchase_history.product_list, NEW.products_id::text),
               total_amount = purchase_history.total_amount + (SELECT price FROM products WHERE id = NEW.products_id)
           WHERE user_id = NEW.users_id;
       ELSE
           -- Если запись не существует, вставляем новую запись
	INSERT INTO purchase_history (product_list, total_amount, user_id)
	VALUES (
	    ARRAY[NEW.products_id::text],
	    (SELECT price FROM products WHERE id = NEW.products_id),
	    NEW.users_id
	);
       END IF;
   
       RAISE NOTICE 'Purchase amount updated for user ID %, added amount: %', NEW.users_id, (SELECT price FROM products WHERE id = NEW.products_id);
   
       RETURN NEW;
   END;
   $BODY$;
   
   ALTER FUNCTION public.update_purchase_total()
       OWNER TO postgres;



CREATE OR REPLACE TRIGGER update_purchase_total_trigger
AFTER INSERT OR UPDATE OR DELETE
ON public.l_products_users
FOR EACH ROW
EXECUTE FUNCTION public.update_purchase_total();
   ```

   Пример использования:
   ```sql
   INSERT INTO l_products_users VALUES (1, 1, 2);
   ```

### 3. **Триггер для автоматического обновления среднего прогресса пользователя:**

   Этот триггер будет автоматически пересчитывать средний прогресс пользователя при добавлении, изменении или удалении задачи.

   ```sql
	-- Создание триггера
	CREATE OR REPLACE FUNCTION public.update_average_progress()
	    RETURNS trigger
	    LANGUAGE 'plpgsql'
	    COST 100
	    VOLATILE NOT LEAKPROOF
	AS $BODY$
	BEGIN
	    -- Обновление среднего прогресса пользователя
	    UPDATE public."users"
	    SET average_progress = (
	        SELECT AVG(progress) FROM public."tasks" WHERE user_id = NEW.user_id
	    )
	    WHERE id = NEW.user_id;
	
	    -- Вывод сообщения в консоль о событии
	    RAISE NOTICE 'Updated average progress for user ID %', NEW.user_id;
	
	    RETURN NULL;
	END;
	$BODY$;
	
	-- Привязка триггера к событию AFTER INSERT на таблице "tasks"
	CREATE TRIGGER update_average_progress
	AFTER INSERT ON public."tasks"
	FOR EACH ROW
	EXECUTE FUNCTION public.update_average_progress();
   ```

Пример использования:

```sql
-- Вставка новой задачи для пользователя с ID = 1
INSERT INTO public."tasks" (id, title, description, due_date, "check", progress, user_id, category_id)
VALUES (1, 'New Task', 'Description', '2023-12-01', false, 25, 1, 1);
```

### 4. **Триггер для автоматической проверки сроков задач:**

Этот триггер check_task_deadline предназначен для проверки просроченных задач. Вот его действия: 
* При срабатывании триггера перед вставкой (BEFORE INSERT) в таблицу "tasks" он проверяет, просрочена ли задача.
* Если дата завершения задачи (due_date) меньше текущей даты и времени (CURRENT_TIMESTAMP), то считается, что задача просрочена.
* Для отслеживания просроченных задач используется поле is_overdue. Если задача просрочена, устанавливается флаг is_overdue в true.
* В консоль выводится уведомление о просроченной задаче с указанием её ID.
  
   ```sql
	CREATE OR REPLACE FUNCTION public.check_task_deadline()
	    RETURNS trigger
	    LANGUAGE 'plpgsql'
	    COST 100
	    VOLATILE NOT LEAKPROOF
	AS $BODY$
	    BEGIN
	        -- Проверка, если дата завершения задачи прошла
	        IF NEW.due_date < CURRENT_TIMESTAMP THEN
	            -- Добавление сообщения в консоль о просроченной задаче
	            RAISE NOTICE 'Task with ID % has missed the deadline!', NEW.id;
	
	            -- Пример: Установка флага просроченной задачи
	            NEW.is_overdue := true;
	
	            -- Пример: Отправка уведомления пользователю (замените этот код на свой код уведомления)
	            INSERT INTO public.notifications (user_id, message)
	            VALUES (NEW.user_id, 'Your task is overdue!');
	
	        END IF;
	
	        RETURN NEW;
	    END;
	$BODY$;
	
	ALTER FUNCTION public.check_task_deadline()
	    OWNER TO postgres;
   ```

Пример использования:

```sql
-- Вставляем задачу с просроченным сроком
INSERT INTO public."tasks" (id, title, description, due_date, "check", progress, user_id, category_id)
VALUES (27, 'Просроченная задача', 'Описание просроченной задачи', '2023-11-18 12:00:00', false, 0, 2, 1);

-- Обновляем задачу с просроченным сроком
UPDATE public."tasks"
SET due_date = '2023-11-17 12:00:00'
WHERE id = 27;
```

# 2.Процедуры:

### 1. **Процедура для создания нового пользователя:**

   Создайте процедуру, которая позволяет администратору или модератору создать нового пользователя, указав имя пользователя, адрес электронной почты и пароль.

   ```sql
	CREATE OR REPLACE PROCEDURE public.create_new_user(
	    IN p_id INT,
	    IN p_username CHARACTER VARYING,
	    IN p_password CHARACTER VARYING,
	    IN p_email CHARACTER VARYING,
	    IN p_average_progress INT
	)
	LANGUAGE 'plpgsql'
	AS $BODY$
	DECLARE
	    new_user_id INT;
	BEGIN
	    -- Вставка нового пользователя в таблицу "users"
	    INSERT INTO public."users" (id, username, password, email, average_progress)
	    VALUES (p_id, p_username, p_password, p_email, p_average_progress) RETURNING id INTO new_user_id;
	
	    -- Вывод сообщения в консоль о создании нового пользователя
	    RAISE NOTICE 'New user created with ID %', new_user_id;
	END;
	$BODY$;
	ALTER PROCEDURE public.create_new_user(
	    INT, CHARACTER VARYING, CHARACTER VARYING, CHARACTER VARYING, INT
	)
	OWNER TO postgres;
   ```

Пример использования:
```sql
CALL public.create_new_user(
    17,
    'DmitriyTihon',
    'expensive123ABCD',
    'dima@yandex.by',
    100
);
```

### 2. **Процедура для добавления нового поста:**

   Создайте процедуру, которая позволяет пользователям добавить новый пост с указанием заголовка и описания.

   ```sql
	CREATE OR REPLACE PROCEDURE public.create_new_post(
	    IN p_id INT,
	    IN p_user_id INT,
	    IN p_title CHARACTER VARYING(255),
	    IN p_description TEXT
	)
	LANGUAGE 'plpgsql'
	AS $BODY$
	BEGIN
	    -- Вставка нового поста в таблицу "posts" с указанным id
	    INSERT INTO public."posts" (id, user_id, title, description)
	    VALUES (p_id, p_user_id, p_title, p_description);
	
	    -- Вывод сообщения в консоль о создании нового поста
	    RAISE NOTICE 'New post created with ID %', p_id;
	END;
	$BODY$;
	ALTER PROCEDURE public.create_new_post(INT, INT, CHARACTER VARYING(255), TEXT)
	OWNER TO postgres;
   ```

Пример использования:
```sql
CALL public.create_new_post(
    1,
    2,
    'Заголовок нового поста',
    'Это описание нового поста. Привет, мир!'
);
```

### 3. **Процедура для получения списка задач для конкретного пользователя:**

   Создайте процедуру, которая принимает имя пользователя и возвращает список его задач.

   ```sql
	CREATE OR REPLACE PROCEDURE get_user_tasks_procedure(p_username VARCHAR)
	AS $$
	DECLARE
	    user_name VARCHAR;
	    task_titles TEXT[];
	BEGIN
	    SELECT
	        users.username,
	        ARRAY_AGG(tasks.title)
	    INTO
	        user_name,
	        task_titles
	    FROM
	        tasks
	    JOIN users ON tasks.user_id = users.id
	    WHERE users.username = p_username
	    GROUP BY users.username;
	
	    -- Проверка наличия результатов
	    IF FOUND THEN
	        -- Ваш код для использования переменных user_name и task_titles
	
	        -- Например, вы можете вывести результаты
	        RAISE NOTICE 'User Name: %, Task Titles: %', user_name, task_titles;
	    ELSE
	        RAISE NOTICE 'User with username % not found', p_username;
	    END IF;
	END;
	$$ LANGUAGE plpgsql;
   ```

Пример использования:

```sql
CALL get_user_tasks_procedure('example_user');
```

### 4. **Процедура для добавления нового отзыва:**

   Создайте процедуру, которая позволяет пользователям добавлять новые отзывы с указанием текста и даты.

   ```sql
	CREATE OR REPLACE PROCEDURE add_review_procedure(
	    p_id INT,
	    p_user_id INT,
	    p_description TEXT,
	    p_create_time TIMESTAMP DEFAULT NULL
	)
	AS $$
	BEGIN
	    INSERT INTO reviews (id, user_id, description, create_time)
	    VALUES (p_id, p_user_id, p_description, COALESCE(p_create_time, NOW()));
	    RAISE NOTICE 'Review added for User ID: %', p_user_id;
	END;
	$$ LANGUAGE plpgsql;
   ```

Пример использования:

```sql
CALL add_review_procedure(1, 123, 'Отличная работа!');
```

### 5. **Процедура для обновления статуса задачи:**

   Создайте процедуру, которая позволяет пользователю обновить статус задачи по ее идентификатору и соответственно изменить прогресс задачи до 100, когда задача выполнена.

   ```sql
	   CREATE OR REPLACE PROCEDURE update_task_status_procedure(
	    p_task_id INT,
	    p_check BOOLEAN
	)
	AS $$
	BEGIN
	    UPDATE tasks
	    SET 
	        check = p_check,
	        progress = CASE WHEN p_check THEN 100 ELSE progress END
	    WHERE id = p_task_id;
	    RAISE NOTICE 'Status updated for Task ID: %', p_task_id;
	END;
	$$ LANGUAGE plpgsql;
   ```

Пример использования:

```sql
CALL update_task_status_procedure(123, TRUE);
```
