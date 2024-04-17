# Tinder

---

## Содержание

* ### [1. Тема, целевая аудитория](#1)
* ### [2. Расчет нагрузки](#2)
* ### [3. Глобальная балансировка](#3)
* ### [4. Локальная балансировка](#4)
* ### [5. Логическая схема БД](#5)
* ### [6. Физическая схема БД](#6)
* ### [7. Алгоритмы](#7)
* ### [8. Технологии](#8)
* ### [9. Обеспечение надёжности](#9)

## 1. Тема и целевая аудитория<a name="1"></a>

**Tinder** - популярное приложение для мобильных платформ Android и iOS, предназначенное для романтических знакомств в соответствии с заданными параметрами и с учётом геолокации. Основными действиями в приложении являются пролистывания — «свайпы» (проведение пальцем в горизонтальном направлении): пользователю показывают фотографии и краткие биографии кандидатов (имя и возраст), и пользователь может провести вправо, если считает совпадение удачным. Если два пользователя отметили друг друга как удачные совпадения, они могут начать общение и договориться о встрече.

## MVP

- Регистрация/Создание профиля
- Лента профилей
- Возможность ставить лайки и пропуски (свайпы)
- Загрузка фотографий
- Уведомления о совпадениях
- Чат

## Целевая аудитория

Согласно информации с сайта (https://www.businessofapps.com/data/tinder-statistics/)

- 75 миллионов активных пользователей в месяц
- 400 миллионов скачиваний

### Распределение по странам

| Страна            | Процент пользователей                 |
|-------------------|:-------------------------------------:|
| Соединенные Штаты |                 10.13                 |
| Бразилия          |                 9.7                   |
| Польша            |                 5.77                  |
| Испания           |                 5.28                  |
| Аргентина         |                 4.25                  |

## 2. Расчет нагрузки<a name="2"></a>

### Продуктовые метрики

| Метрика                                      | Значение |
| -------------------------------------------- | -------- |
| Зарегистрированных пользователей             | 400 млн  |
| Месячная аудитория                           |  75 млн  |
| Суточная аудитория                           |  42 млн  |
| Среднее количество свайпов в день            | 1.6 млрд |
| Максимальное количество свайпов в день       | 3 млрд   |
| Совпадений в день                            | 50 млн   |

### Количество операций по типам

Количество свайпов на пользователя в день - `1.6 млрд / 42 млн = 38`   
Исходя из данных научной статьи (https://arxiv.org/pdf/1607.03320.pdf) можно вычислить, что в среднем пользователь отправляет около 9 сообщений в чат.  
Будем считать, что количество чатов в день примерно равно количеству совпадений.  
Количество сообщений в день - `9 * 2.75 = 24.75`  

Считаем, что пользователь меняет свою аватарку или одну из своих фото раз в месяц.

| Операция                         | Среднее кол-во в день на пользователя |
| -------------------------------- | :-----------------------------------: |
| Регистрация                      |                 0.0007                |
| Смена аватара                    |                 0.03                  |
| Свайпы(просмотр профилей)        |                  38                   |
| Совпадения                       |                  2.75                 |
| Отправка сообщения               |                  24.75                |
| Отправка вложения(гифки)         |                  0.28                 |
| Получение списка чатов           |                  4                    |
| Получение списка сообщений чата  |                  4                    |


### Технические метрики

#### Объем хранилища

Хранилище требуется для пользовательских данных (профилей) и хранения сообщений (также ещё нужно хранить информацию о свайпах, но её размером можно пренебречь).

Средний размер профиля пользователя включающим в себя персональные данные пользователя, никнейм и контактные данные равным 2 КБ. Размер фото в среднем равен 410 КБ (отображаются в клиенте размером 640*640 пикселей). Рекомендуется загружать примерно 5 фото.  

Средний размер хранилища на профиль - 2 КБ + 410 КБ * 5 = 2052 КБ , количество пользователей 400 млн => Размер хранилища на пользователей

`400 млн * 2052 КБ = 820 ТБ`

Среднюю длину сообщения, отправленного пользователем, будем считать равным 60 символов,
а размер символа - 2 байтам. Т.е. средний размер сообщения - 120 байт. За один день пользователь отправляет 25 сообщений. Переписки хранят полгода. => Размер хранилища сообщений

`75 млн * 120 байт * 25 * 180 = 40.5 ТБ`

Средний размер вложения(гифки) будем считать 10 МБ. В приложении отправляют 4.2 млн Gif в неделю. Приложение хранит переписки в течении полугода(примерно 25 недель) => Размер хранилища вложений

`75 млн * 10 МБ = 750 ТБ`

| Тип данных | Размер  |
| ---------- | ------- |
| Профили    | 820 Тб  |
| Сообщения  | 40.5 Тб |
| Вложения   | 750  Тб |

#### RPS

Учитывая 42 млн DAU: 
Считаем средний RPS по формуле: `42 млн * N / 24 / 3600`, где N - число операций определённого типа в день  
Считаем пиковый RPS, как 2 * средний RPS

| Операция                         | Средний RPS | Пиковый RPS |
| -------------------------------- | :---------: | :---------: |
| Регистрация                      |    0.33     |     0.66    |
| Смена аватара                    |     15      |      30     |
| Свайпы(просмотр профилей)        |    18 472   |    36 944   |
| Совпадения                       |     1337    |     2674    |
| Отправка сообщения               |    12 031   |    24 062   |
| Отправка вложения                |    137      |      274    |
| Получение списка чатов           |    1944     |     3888    |
| Получение списка сообщений чата  |    1944     |     3888    |
| **Сумма**                        | **41 227**  |  **82 454** |

#### Сетевой трафик

**Смена аватара(фото)**

Средний размер фото при отправке считаем равным 2 Mб.

**Регистрация**

Средний размер фото при отправке считаем равным 2 Mб. При регистрации отправляют примерно 5 фото и данные о себе примерно на 2 КБ.  
Общий объем данные при регистрации:  

`2 КБ + 2 Мб * 5 ~= 10 Мб`  

**Свайпы(просмотр профилей)**

Средний размер профиля пользователя считаем равным `2 КБ + 410 КБ * 5 = 2052 КБ` . Размером информации о свайпе можно пренебречь.

**Совпадения(уведомления)**

Будем считать, что после совпадения пользователь ещё раз просматривает профиль.

**Отправка сообщения**

Средний размер сообщения будем считать равным 120 байт.

**Отправка вложения**

Средний размер вложения будем считать равным 1 МБ.

**Получение списка чатов**

Считаем, что на каждый запрос пользователь получает информацию о 10 чатах,
при среднем размере аватара в 410 Кб имеем:
`10 * 410 = 4.1 МБ`

**Получение списка сообщений чата**

Считаем, что на каждый запрос пользователь получает последние 10 сообщений,
при среднем размере в 120 байт и среднем размере вложения в 1 Мб на один запрос имеем:  
`10 * (120 + 0.01 * 1e6) = 10 Кб`

**Трафик**

| Операция                    | Средний трафик, Гбит/с | Пиковый трафик, Гбит/с | Суточный трафик, Гбайт |
| ----------------------------| :--------------------: | :--------------------: | :--------------------: |
| Смена аватара(фото)         |          0.24          |          0.48          |          2 592         |
| Регистрация                 |          0.027         |          0.054         |          292           |
| Свайпы                      |          303           |          606           |       3 272 400        |
| Совпадения                  |          22            |          44            |         237 600        |
| Отправка сообщения          |          0.012         |          0.024         |          130           |
| Отправка вложения           |           1.1          |           2.2          |         95 040         |
| Получение списка чатов      |           64           |          128           |        168 480         |
| Получение списка сообщений  |           0.16         |          0.32          |         1728           |
| **Сумма**                   |       **372.94**       |       **745.88**       |     **3 588 182**      |

## 3. Глобальная балансировка нагрузки<a name="3"></a>

### 3.1 Физическое расположение датацентров
+ Северная Америка - 4 ДЦ 
    - США (Нью-Йорк, Чикаго, Сан-Франциско)
    - Мексика (Мехико)
+ Южная Америка - 3 ДЦ 
    - Бразилия (Сан-Паулу)
    - Аргентина (Буэнос-Айрес)
    - Чили (Сантьяго)
+ Европа - 4 ДЦ 
    - Франция (Париж)
    - Великобритания (Лондон)
    - Германия (Франкфурт)
    - Испания (Мадрид)
+ Азия - 3 ДЦ 
    - Индия (Мумбаи, Дели)
    - Япония (Токио)
+ Австралия - 1 ДЦ  (Сидней)

![Глобальная балансировка](globalbalancing.png)

Интерактивная карта: https://www.google.com/maps/d/edit?mid=1qLQRtAtQ3YvXZSvJlapSoeAva7ZfXsU&usp=sharing

Данные города были выбраны, исходя из списка стран, где чаще всего пользуются Tinder (https://worldpopulationreview.com/country-rankings/tinder-users-by-country)

### 3.2 Схема балансировки
- Определяем регион через Geo-based DNS, поскольку данный метод гарантирует подключение пользователей к ближайшему серверу, а значит минимальную задержку и минимальную перегрузку сети. Он подходит для таких компаний, как Tinder, которые имеют аудиторию по всему миру.
- В рамках региона выбираем датацентр через BGP Anycast
- Также существует схема перераспределения трафика в случае отказа датацентра. Например, если откажет датацентр в Сиднее, то трафик из Австралии будет направлен в датацентры в Токио и Мумбаи с учетом их текущей загрузки

## 4. Локальная балансировка нагрузки<a name="4"></a>

### Схема балансировки

1. На входе в ДЦ ставим несколько роутеров. Роутеры могут использовать протокол HSRP.
2. Далее трафик проксируется на nginx, при этом маршруты к nginx анонсируются через BGP
3. nginx балансирует трафик на application-сервера по round-robin стратегии

HSRP был разработан компанией Cisco и обеспечивает автоматическое переключение на резервный роутер, если основной роутер выходит из строя. Это достигается путем создания виртуального роутера, который имеет свой собственный IP-адрес и MAC-адрес. Все узлы в сети настроены на использование этого виртуального роутера как своего шлюза по умолчанию.

### Kubernetes для оркестрации

Kubernetes будет использоваться для управления развертыванием и масштабированием приложений. Трафик будет направляться на поды следующим образом:

1. Создается Kubernetes Service, который будет действовать как балансировщик нагрузки для приложений. Service определяет маршруты к подам с использованием механизма Service Discovery.

2. Внутри кластера Kubernetes, kube-proxy будет перенаправлять трафик с Service на поды, используя стратегию балансировки sticky sessions.

### Нагрузка по терминации SSL

При такой схеме балансировки можно терминировать SSL на уровне nginx (с кешированием SSL-тикетов для улучшения производительности)

Пиковый RPS равен 82 454, суммарный пиковый трафик - 312.14 Гбит/с  
Пусть на датацентр приходится в среднем 20% запросов/трафика, тогда на один ДЦ приходится:

82 454 * 0.2 = 16 491 RPS    
312.14 * 0.2 = 62.428 Гбит/с

### Обеспечение отказоустойчивости

1. Несколько роутеров на входе в ДЦ
2. Несколько инстансов nginx
3. Следует выделить железо, способное выдержать 80% пикового трафика
4. Проводить e2e health checkи каждые 30 минут + Availability checks каждую минуту + Functional checks каждые 5 минут + проверки бд и кеша каждые 5 минут

## 5. Логическая схема БД<a name="5"></a>

![Логическая схема БД](logicsheme.png)

### Размер таблиц

**User**

| Поле              | Тип              | Размер, байт | Средний размер, байт |
| ----------------- | -----------------| :----------: | :------------------: |
| Id                | int64            |      8       |          8           |
| Name              | varchar(40)      |      40      |          10          |
| Surname           | varchar(40)      |      40      |          10          |
| Password          | varchar(30)      |      30      |          15          |
| Birthday          | date             |      3       |          3           |
| Gender            | char             |      1       |          1           |
| Email             | varchar(254)     |      254     |          254         |
| Phone_number      | varchar(15)      |      15      |          15          | 
| About             | varchar(1000)    |      1000    |          100         |
| Icon_id           | int64            |      8       |          8           |
| LastGeoPosition   | numeric(8,5) * 2 |      44      |          44          |
| **Сумма**         | -                |   **1443**   |         **468**      |

Размер одной записи - около 468 байт.

При размере аудитории в 75 млн пользователей получим общий объём таблицы: 32.6 Гб

**Session**

| Поле              | Тип          | Размер, байт |
| ----------------- | ------------ | :----------: |
| Cookie            | varchar(50)  |      50      |
| User_id           | int64        |      8       |
| **Сумма**         | -            |   **58**     |

Размер одной записи - около 58 байт.

Допустим, что кука выдается на сутки. При размере суточной аудитории в 42 млн пользователей получим общий объём таблицы: 2.3 Гб

**Interest**

| Поле              | Тип          | Размер, байт |
| ----------------- | ------------ | :----------: |
| Id                | int64        |      8       |
| Name              | varchar(40)  |      40      |
| **Сумма**         | -            |   **48**     |

Размер одной записи - около 48 байт.

В Tinder ecть 11 видов интересов. Размер таблицы - 528 байт.

**User_interest**

| Поле              | Тип          | Размер, байт |
| ----------------- | ------------ | :----------: |
| Id                | int64        |      8       |
| User_id           | int64        |      8       |
| Interest_id       | int64        |      8       |
| **Сумма**         | -            |   **24**     |

Размер одной записи - около 24 байт.

В среднем у одного пользователя 4 интереса. При размере аудитории в 75 млн пользователей получим общий объём таблицы: 286 Мб

**User_image**

| Поле              | Тип          | Размер, байт |
| ----------------- | ------------ | :----------: |
| Id                | int64        |      8       |
| User_id           | int64        |      8       |
| Image_path        | varchar(2048)|      2048    |
| **Сумма**         | -            |   **2064**   |

Размер одной записи - около 2064 байт.

В среднем у одного пользователя 5 фото. При размере аудитории в 75 млн пользователей получим общий объём таблицы: 720 Гб

**Swipe**

| Поле              | Тип          | Размер, байт |
| ----------------- | ------------ | :----------: |
| Id                | int64        |      8       |
| User_id           | int64        |      8       |
| Swiped_user_id    | int64        |      8       |
| Is_right          | boolean      |      1       |
| **Сумма**         | -            |   **25**     |

Размер одной записи - около 25 байт.

Предположим, что информация о свайпах хранится 3 месяца.

При среднем количестве свайпов в день 1.6 млрд получим общий объём таблицы: 3.3 Тб

**Match**

| Поле              | Тип          | Размер, байт |
| ----------------- | ------------ | :----------: |
| Id                | int64        |      8       |
| First_user_id     | int64        |      8       |
| Second_user_id    | int64        |      8       |
| **Сумма**         | -            |   **24**     |

Размер одной записи - около 24 байт.

Предположим, что информация о совпадениях хранится 3 месяца.

При среднем количестве совпадений в день 50 млн получим общий объём таблицы: 100 Гб


**Chat**

| Поле              | Тип          | Размер, байт |
| ----------------- | ------------ | :----------: |
| Id                | int64        |      8       |
| First_user_id     | int64        |      8       |
| Second_user_id    | int64        |      8       |
| **Сумма**         | -            |   **24**     |

Размер одной записи - около 24 байт.

Информация о чатах хранится полгода.

Будем считать что каждый день появляется 50 млн чатов. Тогда общий объем таблицы: 402 Гб

**Message**

| Поле              | Тип          | Размер, байт |
| ----------------- | ------------ | :----------: |
| Id                | int64        |      8       |
| Text              | varchar(1000)|      1000    |
| Chat_id           | int64        |      8       |
| User_id           | int64        |      8       |
| Time              | datetime     |      8       |
| Is_read           | boolean      |      1       |
| **Сумма**         | -            |   **1032**   |

Следует учитывать что средний размер сообщения около 50 символов.

Размер одной записи - около 83 байт.

В среднем в одном чате хранится около 20 сообщений. Тогда общий объем таблицы: 13 Тб

**Attachment**

| Поле              | Тип          | Размер, байт |
| ----------------- | ------------ | :----------: |
| Id                | int64        |      8       |
| Type              | varchar(20)  |      20      |
| Path              | varchar(50)  |      2048    |
| Message_id        | int64        |      8       |
| **Сумма**         | -            |   **2084**     |

Размер одной записи - около 2084 байт.

В приложении отправляют 4.2 млн вложений в неделю. Приложение хранит переписки в течении полугода.

Тогда общий объем таблицы: 200 Гб

### Сводка:

| Таблица             | Размер одной строки | Кол-во строк | Общий размер | Нагрузка на чтение(RPS) | Нагрузка на запись(RPS) |
| ------------------- | ------------------- | ------------ | ------------ | ------------------ | ------------------ |
| User                | 468 Байт            | 75 млн       | 32.6 Гб      | 39618              | 0.66               |
| Session             | 58 Байт             | 42 млн       | 2.3 Гб       | 82180              | 486.11             |
| Interest            | 48 Байт             | 11           | 528 байт     | 39618              | 0                  |
| User_interest       | 24 Байта            | 300 млн      | 286 Мб       | 39618              | 0.66               |
| User_image          | 2064 Байт           | 375 млн      | 720 Гб       | 47394              | 30.66              | 
| Swipe               | 25 Байт             | 288 млрд     | 3.3 Тб       | 2674               | 36944              |
| Match               | 24 Байта            | 9 млрд       | 100 Гб       | 2674               | 2674               |
| Chat                | 24 Байта            | 9 млрд       | 201 Гб       | 3888               | 2674               |
| Message             | 83 Байта            | 180 млрд     | 13 Тб        | 3888               | 24062              |
| Attachment          | 2084 Байт           | 105 млн      | 200 Гб       | 3888               | 274                |

## 6. Физическая схема БД<a name="6"></a>

### Денормализация схемы БД

Для избежания большого количества join'ов можно:

1. Добавить список интересов сразу в User
2. Добавить информацию о текущих сессиях сразу в User
3. Добавить список фото сразу в User
4. Добавить информацию о вложениях сразу в Message

### Партиционирование ClickHouse

- Таблица Swipe: партиционирование по User_id
Сохранять сообщения батчами по 10 млн строк. 
- Таблица Message: партиционирование по Chat_id и Time
Сохранять сообщения батчами по 1 млн строк с учетом того, чтобы они хранились вместе своим чатом.
- В качестве буфера для хранения данных можно использовать Kafka. Будут два воркера: один будет сохранять сообщения в Redis и моментально отображать, а второй будет объединять их в пакеты и класть в ClickHouse.

![Физическая схема БД](physicalsheme.png)

### Хранилище файлов 

Фото пользователей и прикрепленные файлы к сообщениям можно хранить в собственном S3 хранилище.
Размер файлов: 1514 Тб

### Схема резервного копирования

Для всех таблиц делать резервные копии раз в месяц + делать инкрементные копии каждый день.

## 7. Алгоритмы<a name="7"></a>

- Глобальная балансировка нагрузки осуществляется следующим образом: определение региона пользователя происходит через Geo-based DNS, а внутри региона выбор оптимального дата-центра происходит с помощью BGP Anycast. Это позволяет маршрутизировать трафик пользователей к ближайшим и наименее загруженным серверам, обеспечивая высокую доступность и производительность платформы.  
- На уровне локальной балансировки нагрузки, nginx выступает в качестве распределителя трафика, перенаправляя запросы пользователей на группу application-серверов по алгоритму round-robin.  
- При регистрации новых пользователей вся информация о них записывается в базу данных MongoDB, обеспечивая надежное хранение и быстрый доступ к профильным данным.  
- Для эффективной загрузки ленты для свайпов, алгоритм подбирает пользователей батчами по 50 человек, учитывая их приближенную геопозицию и пол. Геопозиция каждого пользователя хранится только в обобщенном виде и удаляется при длительном отсутствии активности, поддерживая актуальность данных. Таким образом, свайпы будут происходить среди пользователей с похожим местоположением. Кроме того, в ленту добавляются люди, которым текущий пользователь уже понравился (свайпнул вправо).  
- В случае взаимной симпатии (совпадения свайпов) оба пользователя получают уведомление и они могут начать чат. Сообщения чатов сохраняются с использованием партиционирования по Chat_id и Time, что позволяет эффективно хранить данные в батчах, группируя сообщения по соответствующим чатам.  
- Для буферизации данных сообщений чатов перед их сохранением в базу данных используется Kafka, обеспечивающая надежную очередь сообщений и высокую пропускную способность.

## 8. Технологии<a name="8"></a>

| Технология    | Область применения                              | Мотивационная часть                              |
| --------------| ------------------------------------------------| ------------------------------------------------ |
| Go            | ЯП Backend'а                                    | Надёжность, скорость, простая асинхронная модель |
| React, Vue.js | Frontend                                        | Гибкость, высокая скорость разработки            |
| nginx         | Reverse proxy, балансировщик                    | Скорость, гибкость, конфигурируемость            |
| MongoDB       | БД для данных пользователей                     | Масштабируемость, высокая скорость, гибкость     | 
| ClickHouse    | БД для хранения свайпов, совпадений и чатов     | Масштабируемость, высокая скорость               |
| Prometheus    | Сбор метрик, их хранение                        | Стандарт индустрии                               |
| Grafana       | Визуализация собранных метрик                   | Лидер индустрии                                  |
| S3            | Хранилище пользовательских файлов               | Уменьшение задержки и исходящего трафика         |
| Kafka         | Брокер сообщений                                | Скорость, масштабируемость                       |

## 9. Обеспечение надёжности<a name="9"></a>

| Раздел                 | Обеспечение надёжности                                                                                      | 
| -----------------------| ------------------------------------------------------------------------------------------------------------| 
| ДЦ                     | Дублирование всех критических компонентов, разнесение их по разным помещениям                               |
| Локальная балансировка | Несколько роутеров на входе в ДЦ, несколько инстансов nginx                                                 |
| MongoDB                | Репликация данных                                                                                           |
| ClickHouse             | Репликация данных + горячее резервирование                                                                  |
| Kafka                  | Репликация данных                                                                                           |
| S3 хранилище           | Репликация данных + расположение серверов в двух различных помещениях и соединение их "растянутым" кластером|

Репликация MongoDB : репликационный набор, состоящий из трех узлов: один первичный узел, который обрабатывает все записи и чтения, и два вторичных узла, которые зеркалируют данные первичного узла + автоматическое переключение на вторичный узел в случае сбоя первичного узла 

# Источники
- https://roast.dating/blog/tinder-statistics 
- https://www.enterpriseappstoday.com/stats/tinder-statistics.html 
- https://datingzest.com/tinder-statistics/
- https://marketsplash.com/ru/statistika-tinder/ 
