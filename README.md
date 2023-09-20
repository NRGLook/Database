# Database

## 1. Описание проекта:
* **Название проекта**: Приложение по саморазвитию для достижения целей.
* **Автор**: Тиханёнок Илья Александрович
* **Номер группы**: 153504

## 2. Минимальные функциональные требования:
* Авторизация / аутентификация / регистрация  пользователя
* Возможность у авторизированного / зарегистрированного пользователя:
   * возможность создавать / удалять / редактировать задачи:
      * добавление описания к задачи
      * добавления приоритетности задачи (добавление категории)
      * добавление времени выполнения к каждой задаче
      * дата добавления будет задаваться автоматически / дата выполнения будет создаваться автоматически, после выполнения задачи пользователем
   * отслеживания прогресса по конкретной задаче (будет происходить при помощи поля времени, которое будет у каждой задачи создано пользователем, либо будет создано по умолчанию)
   * отслеживания прогресса по выполненным задачам (это будет как банк (банк - это хранилище, которое будет создаваться при регистрации нового пользователя), который будет пополняться после выполнения каждой задачи(1 выполненная задача - 1 единица некой собственной валюты в банке))
   * добавления статьи (статья - это страница, которая будет предлагаться пользователю, по окончании дня для отслеживания его впечатлений / настроения - будет представлять обычное текстовое поле, чтобы пользователя ни в чём не ограничивать)
   * возможная покупка неких вещей, за собственную валюту банку - пример (стили для задачи, задний фон приложения, кастомный таймер)
* Система ролей (авторизованный / неавторизованный пользователь; администратор(тот, кто будет отслеживать прогресс пользователя(будет выступать в качестве наблюдателя, он не сможет создавать пользователя)))
* Отслеживания прогресса пользователей (возможность администратора)

## 3. Описание сущностей:
1. **Пользователь (Users)**
   - Пользователь представляет собой основную сущность, хранящую информацию о зарегистрированных пользователях.
2. **Роль (Roles)**
   - Роль определяет уровень доступа и права пользователя в системе:
      - авторизованный / зарегестрированный 
      - администратор - пользователь, который является наблюдателем, может отслеживать прогресс зарегестрированных пользователей (выполненные задачи)
3. **Банк пользователя - прогресс (UserBank)**
   - Это хранилище, которое создаётся при регистрации пользователя. Хранилище, поможет отслеживать число выполненных задач - прогресс пользователя.
4. **Задача (Tasks)**
   - Эта сущность представляет задачи, которые пользователи могут создавать и управлять(создавать описание задачи, давать приорететность(добавлять категорию)).
5. **Категория задачи (TaskCategory)**
   - Эта сущность позволяет группировать задачи по категориям (по приоритетам).
6. **Прогресс задачи (прогресс задачи) (UserProgress)**
   - Эта сущность отслеживает прогресс выполнения конкретной задачи пользователя.
7. **Статья - Впечатление за день (Articles)**
    - Эта сущность представляет собой страницу, которую сможет заполнять зарегестрированный пользователь каждый день для описания впечатлений за день. 
8. **Достижения (Achievements)**
    - Эта сущность позволяет пользователям зарабатывать награды и достижения за выполнение определенных задач, достижение определенных целей или активное использование приложения. Это может быть стимулом для участия и мотивации пользователей.
9. **Магазин (Store)**
    - Эта сущность представляет собой раздел в вашем приложении, где пользователи могут приобретать различные предметы, стили, улучшения и другие товары, используя собственную валюту из "Банка пользователя - прогресс".
10. **Товар (Products)**
    - Эта сущность представляет собой конкретный товар, который доступен для покупки в магазине.
      
## 4. Описание таблицы:
1. **Пользователь (Users)**
   - Поля:
     - Идентификатор (ID): integer, primary key, auto-increment.
     - Имя (Username): varchar(50), unique,  NOT NULL.
     - Электронная почта (Email): varchar(255), unique.
     - Пароль (Password): varchar(255), хешированный пароль, NOT NULL.
   - Связь:
     - У пользователя может быть одна роль (One-to-One).
     - У пользователя может быть один банк (One-to-One).
     - У пользователя может быть одна или несколько статей (One-to-Many).
     - У пользователя может быть одно или несколько достижений (One-to-Many).
2. **Роль (Roles)**
   - Поля:
     - Идентификатор (ID): integer, primary key, auto-increment.
     - Название (Role_Name): varchar(50), unique, NOT NULL.
   - Связь:
     - У пользователя может быть одна роль (One-to-One).
3. **Банк пользователя - прогресс (UserBank)**
   - Поля:
     - Идентификатор (ID): integer, primary key, auto-increment.
     - Идентификатор пользователя (User ID): integer, foreign key (связь с таблицей Users).
     - Сумма (Amount): decimal, NOT NULL.
   - Связь:
     - Каждый банк прогресса принадлежит определенному пользователю (One-to-One).
4. **Задача (Tasks)**
   - Поля:
     - Идентификатор (ID): integer, primary key, auto-increment.
     - Заголовок - название задачи (Title): varchar(255), NOT NULL.
     - Описание (Description): text.
     - Приоритет (Priority): tinyint.
     - Категория (Category): varchar(255), foreign key (связь с таблицей TaskCategory).
     - Время выполнения (Due Date): дата и время - будет создаваться сразу после добавления задачи(datetime - timestamp), NOT NULL.
   - Связь:
     - Каждая задача связана с определенной категорией задачи (One-to-One).
     - Каждая задача имеет связь с прогрессом выполнения (One-to-One).
     - Каждая задача принадлежит определенному пользователю (One-to-One).
5. **Категория задачи (TaskCategory)**
   - Поля:
     - Идентификатор (ID): integer, primary key, auto-increment.
     - Название (Name): varchar(255), NOT NULL.
   - Связь:
     - Категории задач связаны с задачами - одна задача может иметь неск категорий (Many-to-One)
6. **Прогресс задачи (UserProgress)**
   - Поля:
     - Идентификатор (ID): integer, primary key, auto-increment.
     - Идентификатор задачи (Task ID): integer, foreign key (связь с таблицей Tasks).
     - Прогресс задачи (Progress): integer, NOT NULL.
     - Время начала (Start Time): дата и время - тоже будет добавляться автоматически (datetime - timestamp), NOT NULL.
     - Время завершения (End Time): дата и время - тоже будет добавляться автоматически (datetime - timestamp), NOT NULL.
   - Связь:
     - Каждая запись о прогрессе выполнения связана с определенной задачей (One-to-One).
7. **Статья - Впечатление за день (Articles)**
   - Поля:
     - Идентификатор (ID): integer, primary key, auto-increment.
     - Идентификатор пользователя (User ID): integer, foreign key (связь с таблицей Users).
     - Текст статьи (Content): text.
     - Дата создания (Creation Date): дата и время (datetime - timestamp), NOT NULL.
   - Связь:
     - Каждая статья связана с определенным пользователем (One-to-One).
8. **Достижения (Achievements)**
   - Поля:
     - Идентификатор (ID): integer, primary key, auto-increment.
     - Название (Name): varchar(255), NOT NULL.
     - Описание (Description): text.
   - Связь:
     - Достижения могут быть заработаны пользователями - у одного пользователя может быть неск достижений (например, выполнено 10 задач за день, выполнено за такое-то время и т.д.) (Many-to-One).
9. **Магазин (Store)**
     - Поля:
       - Идентификатор (ID): integer, primary key, auto-increment.
       - Название (Name): varchar(255), NOT NULL.
       - Описание (Description): text.
       - Цена (Price): float-decimal.
     - Связь:
       - Товары в магазине доступны для покупки - 1 магазин может иметь неск товаров (One-to-Many).
10. **Товар (Products)**
    - Поля:
        - Идентификатор (ID): integer, primary key, auto-increment.
        - Название (Name): varchar(255), NOT NULL.
        - Описание (Description): text.
        - Цена (Price): float - decimal, NOT NULL.
        - Изображение (Image): string - путь к изображению товара.
        - Доступность (Availability): boolean, NOT NULL.
    - Связь:
        - Товары связаны с магазином (One-to-One).
        
## 5. Схема:
![DB1](https://github.com/NRGLook/Database/blob/main/DB1.png)

