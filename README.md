# social_network_system_design

# Социальна сеть для путешественников

### Functional requirements:

- публикация постов из путешествий с фотографиями, небольшим описанием и привязкой к конкретному месту путешествия;
- оценка и комментарии постов других путешественников;
- подписка на других путешественников, чтобы следить за их активностью;
- поиск популярных мест для путешествий и просмотр постов с этих мест;
- просмотр ленты других путешественников и ленты пользователя, основанной на подписках в обратном хронологическом порядке;
- мобильные устройства и браузер;


### Non-functional requirements:

- 10 000 000 DAU
- availability 99,95%
- посты - в среднем 1 пост в месяц от активного пользователя
  - от 1 до 5 фотографий, макс размер фотографии - 10 Мб
  - до 1000 символов описания
  - геолокация
  - максимум 100 постов в день
- оценки и комментарии - в среднем 10 комментариев в день от активного пользователя
  - до 100 символов
  - без картинок
  - максимум 1000 комментариев в день
- подписка на других путешественников - в среднем 1 подписка в день от активного пользователя
  - максимум 1000 подписок в день
  - максимум 1000 подписок всего у одного пользователя
  - не ограничений в подписках на 1 одного пользователя
- поиск популярных мест для путешествий и просмотр постов с этих мест - в среднем 10 поисков (просмотр по 100 постов) в день от активного пользователя
  - популярность считаем по кол-ву постов и комментариев
- просмотр ленты - в среднем 10 просмотров (просмотр по 10 постов) в день от активного пользователя
- посты и комментарии храним вечно
- максимальная длительность ленты - 1 год или 1000 событий
- гео-распределенность не требуется
- аудитория стран СНГ
- есть сезонность - считаем как нагрузку как х3 от обычной
  - лето
  - каникулы
  - новогодние праздники
  - длинные выходные

## Design overview


### Level 1. System context diagram

![](images/diagrams/Context.png "System context diagram")

![](images/diagrams/Container_Article.png "Article container diagram")

![](images/diagrams/Container_SearchArticle.png "SearchArticle container diagram")

![](images/diagrams/Container_Users.png "Users container diagram")

![](images/diagrams/Container_UserFeeds.png "UserFeeds container diagram")

## Basic calculations

RPS (создание постов):

    DAU = 10 000 000
    1 пост в месяц от активного пользователя
    RPS = 10 000 000 / 86 400 / 30 ~= 4

    от 1 до 5 фотографий, макс размер фотографии - 10 Мб
    до 1000 символов описания (~2kB) - по сравнению с размером фото можно принебречь
    Traffic = 5 * 10Мб * 4RPS = 200 Мб/с

RPS (оценки и комментарии):

    DAU = 10 000 000
    10 комментариев в день
    RPS = 10 000 000 * 10 / 86 400 ~= 1 200

    до 100 символов ~200B, без картинок
    Traffic = 200B * 1 200RPS = 240 000 B/c = 0.24 Мб/с

RPS (подписки):

    DAU = 10 000 000
    1 подписка в день
    RPS = 10 000 000 * 1 / 86 400 ~= 120

    Traffic = 8B * 120RPS = 960 B/c

RPS (поиск):

    DAU = 10 000 000
    10 поисков в день
    RPS = 10 000 000 * 10 / 86 400 ~= 1 200

    по 100 постов, не учитываем картинки - считаем, что отдаем ссылки на фотографии
    Traffic = 100 * 2kB * 1 200RPS = 240 000 kB/c = 240 Мб/с

    картинки без сжатия
    Traffic = 100 * 5 * 10MB * 1 200RPS = 6 000 000 Мб/c = 6000 Гб/с

RPS (просмотр ленты):

    DAU = 10 000 000
    10 просмотров ленты в день
    RPS = 10 000 000 * 10 / 86 400 ~= 1 200

    по 10 постов, не учитываем картинки - считаем, что отдаем ссылки на фотографии
    Traffic = 10 * 2kB * 1 200RPS = 24 000 kB/c = 24 Мб/с

    картинки без сжатия
    Traffic = 10 * 5 * 10MB * 1 200RPS = 600 000 Мб/c = 600 Гб/с

Requirements:

    RPS ~= 4 000
    Traffic ~= 7 000 Гб/с

    Хранилище картинок на 5 лет = 200 Мб/с * 86 400 * 365 * 5 = 31 536 000 000 Мб = 31 536 Тб
