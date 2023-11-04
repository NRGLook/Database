# 1. Триггеры:

### 1. **Триггер для регистрации действий пользователей:**

   Этот триггер будет записывать в журнал действий пользователя (user_actions_log) информацию о вставке, обновлении и удалении записей в таблицах "posts", "comments", "reviews" и других.

   ```sql
   CREATE OR REPLACE FUNCTION log_user_action() RETURNS TRIGGER AS $$
   BEGIN
       INSERT INTO user_actions_log (action_type, user_id, action_time)
       VALUES (TG_OP, NEW.user_id, NOW());
       RETURN NEW;
   END;
   $$ LANGUAGE plpgsql;

   CREATE TRIGGER log_user_actions
   AFTER INSERT OR UPDATE OR DELETE ON posts
   FOR EACH ROW
   EXECUTE FUNCTION log_user_action();
   ```

### 2. **Триггер для автоматического обновления суммы покупок:**

   Этот триггер будет автоматически обновлять поле "total_amount" в таблице "purchase_history" при вставке новых записей.

   ```sql
   CREATE OR REPLACE FUNCTION update_purchase_total() RETURNS TRIGGER AS $$
   BEGIN
       UPDATE purchase_history
       SET total_amount = total_amount + NEW.price
       WHERE id = NEW.purchase_id;
       RETURN NEW;
   END;
   $$ LANGUAGE plpgsql;

   CREATE TRIGGER update_purchase_total
   AFTER INSERT ON l_products_users
   FOR EACH ROW
   EXECUTE FUNCTION update_purchase_total();
   ```

### 3. **Триггер для автоматического обновления среднего прогресса пользователя:**

   Этот триггер будет автоматически пересчитывать средний прогресс пользователя при добавлении, изменении или удалении задачи.

   ```sql
   CREATE OR REPLACE FUNCTION update_user_average_progress() RETURNS TRIGGER AS $$
   BEGIN
       UPDATE users
       SET average_progress = (SELECT AVG(progress) FROM tasks WHERE user_id = NEW.user_id)
       WHERE id = NEW.user_id;
       RETURN NEW;
   END;
   $$ LANGUAGE plpgsql;

   CREATE TRIGGER update_average_progress
   AFTER INSERT OR UPDATE OR DELETE ON tasks
   FOR EACH ROW
   EXECUTE FUNCTION update_user_average_progress();
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
