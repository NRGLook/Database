# Database

## 1. Описание проекта:
* **Название проекта**: Приложение по саморазвитию для достижения целей.
* **Автор**: Тиханёнок Илья Александрович
* **Номер группы**: 153504

## 2. Минимальные функциональные требования:
* Идентификация / Аутентификация / Авторизация/(регистрация) пользователя
* Возможность у авторизированного/(зарегистрированного) пользователя:
   * возможность создавать / удалять / редактировать задачи:
      * добавление описания к задачи
      * добавления приоритетности задачи (добавление категории)
      * добавление времени выполнения на данную задачу
      * дата добавления будет задаваться автоматически / дата выполнения будет создаваться автоматически, после выполнения задачи пользователем
   * отслеживания прогресса по конкретной задаче (будет происходить при помощи поля времени, которое будет у каждой задачи создано пользователем, либо будет создано по умолчанию)
   * отслеживания прогресса по выполненным задачам (после выполнения задачи будет увеличен счетчик выполненных задач, который конвертируются в валюту(число выполненных задач) и можно будет использовать в магазине для покупки различных товаров(стилей(цветов для задачи, добавления картинок и тд)))
   * добавления обзора (это страница, которая будет предлагаться пользователю, по окончании дня для отслеживания его впечатлений / настроения - будет представлять обычное текстовое поле, чтобы пользователя ни в чём не ограничивать, где пользователь сможет делиться впечатлениями за день(добавить картинку по настроению и описание к ней и потом спустя некоторое время - будет возможность вернуться просмотреть каждый день))(это как личная страница, которая видна только автору данного "обзора")
   * добавление постов - это страница наподобие поста в соцсети, которую уже смогут видеть другие пользователи, а также смогут её комментировать 
   * возможная покупка неких вещей, за собственную валюту(счетчик выполненных задач) - пример (стили для задачи, задний фон приложения, кастомный таймер)
* Система ролей (авторизованный / неавторизованный пользователь; администратор(тот, кто будет отслеживать прогресс пользователя(будет выступать в качестве наблюдателя, он не сможет создавать пользователя)))
* Отслеживания прогресса пользователей (возможность администратора)

## 3. Описание сущностей:
1. **Пользователь (Users)**
   - Пользователь представляет собой основную сущность, хранящую информацию о зарегистрированных пользователях.
2. **Роль (Roles)**
   - Роль определяет уровень доступа и права пользователя в системе:
      - авторизованный / зарегестрированный 
      - администратор - пользователь, который является наблюдателем, может отслеживать прогресс зарегестрированных пользователей (выполненные задачи)
3. **Обзор (Review)**
   - Это это страница, которая будет предлагаться пользователю, по окончании дня для отслеживания его впечатлений / настроения - будет представлять обычное текстовое поле, чтобы пользователя ни в чём не ограничивать, где пользователь сможет делиться впечатлениями за день(добавить картинку по настроению и описание к ней и потом спустя некоторое время - будет возможность вернуться просмотреть каждый день))(это как личная страница, которая видна только автору данного "обзора"
4. **Задача (Tasks)**
   - Эта сущность представляет задачи, которые пользователи могут создавать и управлять(создавать описание задачи, давать приорететность(добавлять категорию)).
5. **Категория задачи (Label)**
   - Эта сущность позволяет группировать задачи по категориям (по приоритетам), добавлять стили(цвета и тп) для задачи, чтобы их различать и было удобно сортировать самому пользователю.
6. **Прогресс задачи (прогресс задачи) (UserProgress)**
   - Эта сущность отслеживает прогресс выполнения конкретной задачи пользователя.
7. **Пост (Posts)**
    - Эта сущность представляет собой страницу, которую сможет заполнять зарегестрированный пользователь, как пост в соцсети, который смогут видеть другие пользователи, а также его комментировать.
8. **Комментарии (Comments)**
    - Эта сущность позволит авторизованным пользователям оставлять комментарии к постам других пользователей. 
9. **Достижения (Achievements)**
    - Эта сущность позволяет пользователям зарабатывать награды и достижения за выполнение определенных задач, достижение определенных целей или активное использование приложения. Это может быть стимулом для участия и мотивации пользователей.
10. **Магазин (Store)**
    - Эта сущность представляет собой раздел в вашем приложении, где пользователи могут приобретать различные предметы, стили, улучшения и другие товары, используя собственную валюту, которая эквивалентна счетчику выполненных задач.
11. **Товар (Products)**
    - Эта сущность представляет собой конкретный товар, который доступен для покупки в магазине(стиль для задачи, кастомная картинка и тп).
      
## 4. Описание таблицы:
1. **Пользователь (Users)**
   - Поля:
     - Идентификатор (ID): integer, primary key, auto-increment.
     - Имя (Username): varchar(50), unique,  NOT NULL.
     - Электронная почта (Email): varchar(255), unique.
     - Пароль (Password): varchar(255), хешированный пароль, NOT NULL.
   - Связь:
     - У пользователя может быть одна роль (One-to-One).
     - У пользователя может быть много задач (One-to-Many).
     - У пользователя может быть один или несколько отзывов (One-to-Many).
     - У пользователя может быть одно или несколько достижений (One-to-Many).
     - У всех пользователя может быть только один магазин.(Many-to-One)
2. **Роль (Roles)**
   - Поля:
     - Идентификатор (ID): integer, primary key, auto-increment.
     - Название (Role_Name): varchar(50), unique, NOT NULL.
   - Связь:
     - У пользователя может быть одна роль (One-to-One).
3. **Обзор (Review)**
   - Поля:
     - Идентификатор (ID): integer, primary key, auto-increment.
     - Описание (description): text, NOT NULL.
     - Дата создания(create_time): datetime - timestamp NOT NULL.
   - Связь:
     - У одного пользователя может быть много постов (One-to-Many).
4. **Задача (Tasks)**
   - Поля:
     - Идентификатор (ID): integer, primary key, auto-increment.
     - Заголовок - название задачи (Title): varchar(255), NOT NULL.
     - Описание (Description): text.
     - Время выполнения (Due Date): дата и время - будет создаваться сразу после добавления задачи(datetime - timestamp), NOT NULL.
     - Выполнение (Check):boolean, NOT NULL.
   - Связь:
     - У одной задачи может быть несколько категорий и у одной категории может быть неск задач(label) (Many-to-Many).
     - Каждая задача имеет связь с прогрессом выполнения (One-to-One).
     - Каждая задача принадлежит определенному пользователю (One-to-One).
5. **Категория задачи (Label)**
   - Поля:
     - Идентификатор (ID): integer, primary key, auto-increment.
     - Категория (Category): varchar(255), NOT NULL.
     - Приоритет (Priority): varchar(255), NOT NULL.
     - Стиль (Style): varchar(100).
   - Связь:
     - Категории задач связаны с задачами - одна задача может иметь неск категорий и одна категория может иметь неск задач (Many-to-Many).
6. **Прогресс задачи (UserProgress)**/..
   - Поля:
     - Идентификатор (ID): integer, primary key, auto-increment.
     - Прогресс задачи (Progress): integer, NOT NULL.
     - Время начала (Start Time): дата и время - тоже будет добавляться автоматически (datetime - timestamp), NOT NULL.
     - Время завершения (End Time): дата и время - тоже будет добавляться автоматически (datetime - timestamp), NOT NULL.
   - Связь:
     - Каждая запись о прогрессе выполнения связана с определенной задачей (One-to-One).
7. **Пост (Posts)**
   - Поля:
     - Идентификатор (ID): integer, primary key, auto-increment.
     - Автор поста (Authot): varchar(255), NOT NULL.
     - Название поста (Title): varchar(255), NOT NULL.
     - Текст статьи (Content): text.
     - Дата создания (Creation Date): дата и время (datetime - timestamp), NOT NULL.
   - Связь:
     - У пользователя может быть несколько постов (Many-to-One).
8. **Комментарии (Comments)**
   - Поля:
     - Идентификатор (ID): integer, primary key, auto-increment.
     - Автор поста (Authot): varchar(255), NOT NULL.
     - Название поста (Title): varchar(255), NOT NULL.
     - Текст статьи (Content): text.
     - Дата создания (Creation Date): дата и время (datetime - timestamp), NOT NULL.
   - Связь:
     - К одному посту может быть много комментариев (Many-to-One).
9. **Достижения (Achievements)**
   - Поля:
     - Идентификатор (ID): integer, primary key, auto-increment.
     - Название (Name): varchar(255), NOT NULL.
     - Описание (Description): text.
   - Связь:
     - Достижения могут быть заработаны пользователями - у одного пользователя может быть неск достижений (например, выполнено 10 задач за день, выполнено за такое-то время и т.д.) (Many-to-One).
10. **Магазин (Store)**
     - Поля:
       - Идентификатор (ID): integer, primary key, auto-increment.
       - Название (Name): varchar(255), NOT NULL.
       - Описание (Description): text.
     - Связь:
       - У неск пользователей - 1 магазин (One-to-Many).
       - Товары в магазине доступны для покупки - 1 магазин может иметь неск товаров (One-to-Many).
11. **Товар (Products)**
    - Поля:
        - Идентификатор (ID): integer, primary key, auto-increment.
        - Название (Name): varchar(255), NOT NULL.
        - Описание (Description): text.
        - Цена (Price): float - decimal, NOT NULL.
        - Изображение (Image): string - путь к изображению товара.
        - Доступность (Availability): boolean, NOT NULL.
    - Связь:
        - Товары связаны с магазином - в 1 магазине может быть неск товаров (Many-to-One).
        
## 5. Схема:
![image](https://github.com/NRGLook/Database/assets/91383841/22012df7-e4b3-48c3-be6f-bbb36a289f06)
