# Database
## 1. Описание проекта:
* **Название проекта**: Приложение по саморазвитию для достижения целей.
* **Автор**: Тиханёнок Илья Александрович
* **Номер группы**: 153504

## 2. Минимальные функциональные требования:
* Авторизация пользователя
* Возможность у авторизированного пользователя отслеживания прогресса по выполненным целям
* Возможность у авторизированного пользователя отслеживания прогресса по конкретной цели
* Возможность у авторизированного пользователя создавать / удалять / редактировать цели
* Возможность у авторизированного пользователя редактирования цели по категории / добавления описания к цели
* Возможность у авторизированного пользователя добавления статьи (статья позволяет пользователю отследить свои впечатления за день)
* Управление пользователями (CRUD)
* Система ролей (авторизованный/неавторизованный пользователь)
* Журналирование действий пользователя

## 3. Описание сущностей:
1. **Пользователь (Users)**
   - Пользователь представляет собой основную сущность, хранящую информацию о зарегистрированных пользователях.
2. **Роль (Roles)**
   - Роль определяет уровень доступа и права пользователя в системе.
3. **Действие пользователя (UserActions)**
   - Эта сущность записывает действия, совершаемые пользователями в системе.
4. **Журнал действий (AuditLog)**
   - Эта сущность используется для журналирования важных событий и изменений в системе.
5. **Задача (Tasks)**
   - Эта сущность представляет задачи, которые пользователи могут создавать и управлять.
6. **Прогресс пользователя (UserProgress)**
   - Эта сущность отслеживает прогресс выполнения задачи пользователя.
7. **Категория задачи (TaskCategory)**
   - Эта сущность позволяет группировать задачи по категориям.
8. **Цель (Goals)**
   - Эта сущность представляет собой цели, которые пользователи могут устанавливать и отслеживать.
9. **Прогресс цели (GoalProgress)**
   - Эта сущность отслеживает прогресс в достижении цели пользователя.
10. **Статья (Articles)**
    - Эта сущность представляет собой статьи, которые пользователи могут публиковать.

## 4. Описание таблицы:
1. **Пользователь (Users)**
   - Пользователь представляет собой основную сущность, хранящую информацию о зарегистрированных пользователях.
   - Поля:
     - **ID**: Уникальный идентификатор пользователя (integer, primary key, auto-increment).
     - **Имя**: Имя пользователя (varchar(255), NOT NULL).
     - **Фамилия**: Фамилия пользователя (varchar(255), NOT NULL).
     - **Логин**: Уникальный логин пользователя, используется для аутентификации (varchar(50), unique, NOT NULL).
     - **Пароль**: Хешированный пароль пользователя для безопасной аутентификации (varchar(255), NOT NULL).
     - **Электронная почта**: Уникальный адрес электронной почты пользователя (varchar(255), unique, NOT NULL).
   - Связи:
     - **Множество Ролей**: Многие пользователи могут иметь множество ролей через сущность "Роль пользователя" Many-to-Many (foreign key).
     - **Множество Действий пользователя**: Пользователь может выполнять множество действий, которые записываются в "Действие пользователя" Many-to-Many (foreign key).
2. **Роль (Roles)**
   - Роль определяет уровень доступа и права пользователя в системе.
   - Поля:
     - **ID**: Уникальный идентификатор роли (integer, primary key, auto-increment).
     - **Название роли**: Название роли, например, "администратор" или "пользователь" (varchar(50), unique).
   - Связи:
     - **Множество Пользователей**: Множество пользователей может иметь одну или несколько ролей через сущность "Роль пользователя"; связь с пользователем через Many-to-Many (foreign key).
3. **Действие пользователя (UserActions)**
   - Эта сущность записывает действия, совершаемые пользователями в системе.
   - Поля:
     - **ID**: Уникальный идентификатор действия (integer, primary key, auto-increment).
     - **Действие**: Описание действия, например, "Вход в систему" или "Редактирование профиля" (varchar(255), NOT NULL).
     - **Время выполнения действия**: Фиксирует момент времени, когда действие было совершено (timestamp, NOT NULL).
     - **IP-адрес пользователя**: Записывает IP-адрес пользователя при совершении действия (varchar(50)).
   - Связи:
     - **Пользователь**: Связь с пользователем, который выполнил действие, через Many-to-One связь (foreign key).
4. **Журнал действий (AuditLog)**
   - Эта сущность используется для журналирования важных событий и изменений в системе.
   - Поля:
     - **ID**: Уникальный идентификатор записи в журнале (integer, primary key, auto-increment).
     - **Текстовое описание действия**: Описывает событие или действие, которое произошло (text).
     - **Время журналирования**: Фиксирует момент времени, когда событие было записано (timestamp, NOT NULL).
   - Связи:
     - **Пользователь**: Связь с пользователем, который вызвал событие, через Many-to-One связь (foreign key).
5. **Задача (Tasks)**
   - Эта сущность представляет задачи, которые пользователи могут создавать и управлять.
   - Поля:
     - **ID**: Уникальный идентификатор задачи (integer, primary key, auto-increment).
     - **Название задачи**: Описание задачи (varchar(255), NOT NULL).
     - **Описание задачи**: Подробное описание задачи (text).
     - **Дата создания**: Дата и время создания задачи (timestamp, NOT NULL).
     - **Дедлайн**: Дата и время, когда задачу необходимо завершить (timestamp).
     - **Статус задачи**: Отображает текущий статус задачи, например, "выполнено", "в процессе", "не выполнено" (varchar(50), NOT NULL).
   - Связи:
     - **Пользователь**: Связь с пользователем, который ответственен за задачу, через Many-to-One связь (foreign key).
6. **Прогресс пользователя (UserProgress)**
   - Эта сущность отслеживает прогресс выполнения задачи пользователя.
   - Поля:
     - **ID**: Уникальный идентификатор записи о прогрессе задачи пользователя (integer, primary key, auto-increment).
     - **Дата**: Дата и время, когда был зарегистрирован прогресс (timestamp, NOT NULL).
     - **Прогресс**: Значение, отражающее прогресс выполнения задачи, например, в процентах (float).
   - Связи:
     - **Пользователь**: Связь с пользователем, для которого записан прогресс, через One-to-One связь (foreign key).
     - **Задача**: Связь с задачей, к которой относится прогресс, через Many-to-One связь (foreign key).
7. **Категория задачи (TaskCategory)**
   - Эта сущность позволяет группировать задачи по категориям.
   - Поля:
     - **ID**: Уникальный идентификатор категории задачи (integer, primary key, auto-increment).
     - **Название категории**: Название категории, например, "работа", "личное", "проект A" (varchar(50)).
   - Связи:
     - **Множество Задач**: Множество задач может быть связано с одной категорией через One-to-Many связь (foreign key).
8. **Цель (Goals)**
   - Эта сущность представляет собой цели, которые пользователи могут устанавливать и отслеживать.
   - Поля:
     - **ID**: Уникальный идентификатор цели (integer, primary key, auto-increment).
     - **Название цели**: Описание цели (varchar(255), NOT NULL).
     - **Описание цели**: Подробное описание цели (text).
     - **Дата начала**: Дата и время начала работы над целью (timestamp, NOT NULL).
     - **Дата завершения**: Дата и время, когда цель должна быть достигнута (timestamp).
     - **Статус цели**: Отражает текущий статус цели, например, "достигнута", "в процессе", "не достигнута" (varchar(50), NOT NULL).
   - Связи:
     - **Пользователь**: Связь с пользователем, который является владельцем цели, через Many-to-One связь (foreign key).
9. **Прогресс цели (GoalProgress)**
   - Эта сущность отслеживает прогресс в достижении цели пользователя.
   - Поля:
     - **ID**: Уникальный идентификатор записи о прогрессе цели пользователя (integer, primary key, auto-increment).
     - **Дата**: Дата и время, когда был зарегистрирован прогресс (timestamp, NOT NULL).
     - **Прогресс**: Значение, отражающее прогресс в достижении цели, например, в процентах (float).
   - Связи:
     - **Цель**: Связь с целью, к которой относится прогресс, через One-to-One связь (foreign key).
     - **Пользователь**: Связь с пользователем, для которого записывается прогресс, через Many-to-One связь (foreign key).
10. **Статья (Articles)**
    - Эта сущность представляет собой статьи, которые пользователи могут публиковать.
    - Поля:
      - **ID**: Уникальный идентификатор статьи (integer, primary key, auto-increment).
      - **Заголовок статьи**: Заголовок статьи (varchar(255), NOT NULL).
      - **Текст статьи**: Содержание статьи (text).
      - **Дата создания**: Дата и время создания статьи (timestamp, NOT NULL).
    - Связи:
      - **Автор**: Связь с пользователем, который написал статью, через Many-to-One связь (foreign key).
        
## 5. Схема:
![DB1](https://github.com/NRGLook/Database/blob/main/DB1.png)

