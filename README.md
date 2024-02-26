# Tinder

---

## 1. Тема и целевая аудитория

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
| Соединенные Штаты |                 12.0                  |
| Бразилия          |                 10.18                 |
| Польша            |                 6.08                  |
| Испания           |                 5.22                  |
| Аргентина         |                 4.79                  |

## 2. Расчет нагрузки

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

Считаем, что пользователь меняет свою аватарку раз в месяц.

| Операция                         | Среднее кол-во в день на пользователя |
| -------------------------------- | :-----------------------------------: |
| Аутентификация                   |                  11                   |
| Смена аватара                    |                 0.03                  |
| Свайпы(просмотр профилей)        |                  38                   |
| Совпадения                       |                  2.75                 |
| Отправка сообщения               |                  24.75                |
| Отправка вложения(гифки)         |                  0.28                 |
| Получение списка чатов           |                  4                    |
| Получение списка сообщений чата  |                  4                    |


### Технические метрики

#### Объем хранилища

Хранилище требуется для пользовательских данных (профилей) и хранения сообщений (также ещё нужно хранить инфоомацию о свайпах, но её размером можно пренебречь).

Средний размер профиля пользователя включающим в себя персональные данные пользователя, никнейм и контактные данные равным 2 КБ. Размер аватара в среднем равен 410 КБ (отображаются в клиенте размером 640*640 пикселей)  

Средний размер хранилища на профиль - 2 КБ + 410 КБ = 412 КБ , количество пользователей 400 млн => Размер хранилища на пользователей

`400 млн * 412 КБ = 165 ТБ`

Среднюю длину сообщения, отправленного пользователем, будем считать равным 60 символов,
а размер символа - 2 байтам. Т.е. средний размер сообщения - 120 байт. За один день пользователь отправляет 25 сообщений. Переписки хранят полгода. => Размер хранилища сообщений

`75 млн * 120 байт * 25 * 180 = 40.5 ТБ`

Средний размер вложения(гифки) будем считать 10 МБ. В приложении отправляют 4.2 млн Gif в неделю. Приложение хранит переписки в течении полугода(примерно 25 недель) => Размер хранилища вложений

`75 млн * 10 МБ = 750 ТБ`

| Тип данных | Размер  |
| ---------- | ------- |
| Профили    | 165 Тб  |
| Сообщения  | 40.5 Тб |
| Вложения   | 750  Тб |

#### RPS

Учитывая 42 млн DAU: 
Считаем средний RPS по формуле: `42 млн * N / 24 / 3600`, где N - число операций определённого типа в день  
Считаем пиковый RPS, как 2 * средний RPS

| Операция                         | Средний RPS | Пиковый RPS |
| -------------------------------- | :---------: | :---------: |
| Аутентификация                   |     5347    |    10 649   |
| Смена аватара                    |     15      |      30     |
| Свайпы(просмотр профилей)        |    18 472   |    36 944   |
| Совпадения                       |     1337    |     2674    |
| Отправка сообщения               |    12 031   |    24 062   |
| Отправка вложения                |    137      |      274    |
| Получение списка чатов           |    1944     |     3888    |
| Получение списка сообщений чата  |    1944     |     3888    |
| **Сумма**                        | **41 227**  |  **82 454** |

#### Сетевой трафик

**Смена аватара**

Средний размер аватара при отправке считаем равным 500 КБ.

**Свайпы(просмотр профилей)**

Средний размер профиля пользователя считаем равным 412 КБ. Размером информации о свайпе можно пренебречь.

**Совпадения(уведомления)**

Будем считать, что после совпдения пользователь ещё раз просматривает профиль.

**Отправка сообщения**

Средний размер сообщения будем считать равным 120 байт.

**Отправка вложения**

Средний размер вложения будем считать равным 10 МБ.

**Получение списка чатов**

Считаем, что на каждый запрос пользователь получает информацию о 10 чатах,
при среднем размере аватара в 410 Кб имеем:
`10 * 410 = 4.1 МБ`

**Получение списка сообщений чата**

Считаем, что на каждый запрос пользователь получает последние 10 сообщений,
при среднем размере в 120 байт и среднем размере вложения в 10 Мб на один запрос имеем:  
`10 * (120 + 0.01 * 10e6) = 1.001 Мб`

**Трафик**

| Операция                    | Средний трафик, Гбит/с | Пиковый трафик, Гбит/с | Суточный трафик, Гбайт |
| ----------------------------| :--------------------: | :--------------------: | :--------------------: |
| Смена аватара               |          0.06          |          0.12          |          648           |
| Свайпы                      |          61            |          122           |        658 800         |
| Совпадения                  |          4.4           |          8.8           |         47 520         |
| Отправка сообщения          |          0.012         |          0.024         |          130           |
| Отправка вложения           |           11           |           22           |        950 400         |
| Получение списка чатов      |           64           |          128           |        168 480         |
| Получение списка сообщений  |           15.6         |          31.2          |        691 200         |
| **Сумма**                   |       **156.07**       |       **312.14**       |     **2 517 178**      |



# Источники
- https://roast.dating/blog/tinder-statistics 
- https://www.enterpriseappstoday.com/stats/tinder-statistics.html 
- https://datingzest.com/tinder-statistics/
- https://marketsplash.com/ru/statistika-tinder/ 
