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

   Этот триггер будет проверять сроки задач и обновлять статус задачи на "просрочено" при достижении срока выполнения.

   ```sql
   CREATE OR REPLACE FUNCTION check_task_due_date() RETURNS TRIGGER AS $$
   BEGIN
       IF NEW.due_date < NOW() THEN
           UPDATE public."tasks" SET "check" = false WHERE id = NEW.id;
       END IF;
       RETURN NEW;
   END;
   $$ LANGUAGE plpgsql;

   CREATE TRIGGER check_task_due_date
   BEFORE INSERT OR UPDATE ON public."tasks"
   FOR EACH ROW
   EXECUTE FUNCTION check_task_due_date();
   ```

# 2.Процедуры:

### 1. **Процедура для создания нового пользователя:**

   Создайте процедуру, которая позволяет администратору или модератору создать нового пользователя, указав имя пользователя, адрес электронной почты и пароль.

   ```sql
   CREATE OR REPLACE PROCEDURE create_new_user(
       username character varying,
       email character varying,
       password character varying
   ) AS $$
   BEGIN
       INSERT INTO public."users" (username, email, password)
       VALUES (username, email, password);
   END;
   $$ LANGUAGE plpgsql;
   ```

### 2. **Процедура для добавления нового поста:**

   Создайте процедуру, которая позволяет пользователям добавить новый пост с указанием заголовка и описания.

   ```sql
   CREATE OR REPLACE PROCEDURE create_new_post(
       user_id bigint,
       title character varying,
       description character varying
   ) AS $$
   BEGIN
       INSERT INTO public."posts" (title, description, user_id)
       VALUES (title, description, user_id);
   END;
   $$ LANGUAGE plpgsql;
   ```

### 3. **Процедура для получения списка задач для конкретного пользователя:**

   Создайте процедуру, которая принимает идентификатор пользователя и возвращает список его задач.

   ```sql
   CREATE OR REPLACE PROCEDURE get_user_tasks(user_id bigint) AS $$
   BEGIN
       SELECT * FROM public."tasks" WHERE user_id = user_id;
   END;
   $$ LANGUAGE plpgsql;
   ```

### 4. **Процедура для добавления нового отзыва:**

   Создайте процедуру, которая позволяет пользователям добавлять новые отзывы с указанием текста и даты.

   ```sql
   CREATE OR REPLACE PROCEDURE create_new_review(
       user_id bigint,
       description character varying,
       create_time date
   ) AS $$
   BEGIN
       INSERT INTO public."reviews" (description, create_time, user_id)
       VALUES (description, create_time, user_id);
   END;
   $$ LANGUAGE plpgsql;
   ```

### 5. **Процедура для обновления статуса задачи:**

   Создайте процедуру, которая позволяет пользователю обновить статус задачи по ее идентификатору.

   ```sql
   CREATE OR REPLACE PROCEDURE update_task_status(
       task_id bigint,
       new_status integer
   ) AS $$
   BEGIN
       UPDATE public."tasks"
       SET "check" = (new_status = 1)
       WHERE id = task_id;
   END;
   $$ LANGUAGE plpgsql;
   ```
