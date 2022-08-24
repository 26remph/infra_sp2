![](api_yamdb/static/header.png)  

# Docker build for YaMDb project
git: https://github.com/26remph/api_yamdb
(проект YaMDb собирает отзывы пользователей на различные произведения)

Сборка представляет из собой три образа: web, db, nginx. 
При создании контейнеров из образов, будет создано два тома для работы 
со статикой и медиа файлами. Название томов: `static_value`, `media_value`

- `web` образ содержит сам проект, все зависимости в том числе gunicorn server для
корректной работы, документация к API проекта YAMDB (v1.0)
- `nginx` образ содержит веб сервер,
- `db` образ содержит PostgresSQL базу данных проекта

Оркестрация контейнерами происходит по средствам `docker-compose` утилиты.

___
![DjangoREST](https://img.shields.io/badge/DJANGO-REST-ff1709?style=for-the-badge&logo=django&logoColor=white&color=ff1709&labelColor=gray)
![Django](https://img.shields.io/badge/django-%23092E20.svg?style=for-the-badge&logo=django&logoColor=white)
![Postgres](https://img.shields.io/badge/postgres-%23316192.svg?style=for-the-badge&logo=postgresql&logoColor=white)
![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)

![GitHub code size in bytes](https://img.shields.io/github/languages/code-size/26remph/api_final_yatube)
![GitHub repo size](https://img.shields.io/github/repo-size/26remph/api_yamdb)
![GitHub](https://img.shields.io/github/license/26remph/api_yamdb)
![pythonversion](https://img.shields.io/badge/python-%3E%3D3.7-blue)


## Оглавление
0. [Описание сборки](#описание-сборки)
1. [Описание проекта](#описание-проекта)
2. [Стек технологий](#стек-технологий)
3. [Как запустить проект](#как-запустить-проект)
4. [Как заполнить базу](#как-заполнить-базу)
5. [Документация по api](#документация-по-api)
6. [Примеры запросов к api](#примеры-запросов-к-api)
7. [Авторы проекта](#авторы-проекта)

## Описание сборки
  
Сборка представляет из собой три образа: `web`, `db`, `nginx`.   

- Конфигурация `web` образа описана в файле  `./api_yamdb/Dockerfile`, там же лежит файл с зависимостями `requirements.txt`
- `db` образ включает образ с **docker-hub** -- `postgres:13.0-alpine`, 
- `nginx` включает образ с **docker-hub** `nginx:1.21.3-alpine`

Инфраструктура проекта лежит в папке `./infra`
- `docket-compose.yaml` # файл конфигурации `docker-compose.`  
- `./nginx/default.conf` # файл настройки `nginx` 
- `fixture.json` # дамп базы данных с тестовыми публикациями.

При запуске контейнеров создаются два тома для хранения данных. 
- `static_value:`
- `media_value:`


## Описание проекта
Проект YaMDb собирает отзывы пользователей на произведения. Произведения делятся на категории: «Книги», «Фильмы», «Музыка». Произведению может быть присвоен жанр. Новые жанры может создавать только администратор. Читатели оставляют к произведениям текстовые отзывы и выставляют произведению рейтинг (оценку в диапазоне от одного до десяти). Из множества оценок автоматически высчитывается средняя оценка произведения.


## Стек технологий
- Docker
- Docker-compose
- проект написан на Python с использованием Django REST Framework
- библиотека Simple JWT - работа с JWT-токеном
- библиотека django-filter - фильтрация запросов
- база данных - PostgreSQL
- система управления версиями - git


## Как запустить проект

- Создать и заполнить .env файл:  
  (example for ubuntu os)
```
touch infra_sp2-1/infra/.env
nano infra_sp2-1/infra/.env
``` 

- Шаблон заполнения .env файла переменными окружения:
```
DB_ENGINE=django.db.backends.postgresql # указываем, что работаем с postgresql
DB_NAME=postgres # имя базы данных
POSTGRES_USER=postgres # логин для подключения к базе данных
POSTGRES_PASSWORD=postgres # пароль для подключения к БД (установите свой)
DB_HOST=db # название сервиса (контейнера)
DB_PORT=5432 # порт для подключения к БД
```

- проверить установлен ли docker на вашем компьютере:
```
$ docker -v
Docker version 20.10.17, build 100c701

$ docker-compose -v
docker-compose version 1.29.2, build 5becea4c
```

- Собрать образы и запустить проект на локальной машине:
```
- $ docker-compose up -d --build 
```


- Выполнить миграции:
```
$ docker-compose exec web python manage.py migrate
```
- Cоздать суперпользователя:
```
$ docker-compose exec web python manage.py createsuperuser
```
- Cобрать статику:
```
$ docker-compose exec web python manage.py createsuperuser
```

Теперь проект доступен по адресу [http://localhost/](http://localhost/).


## Как заполнить базу

1. Заполнение базы данных из ранее сделанного дампа
```
# Положить dump.json в папку с проектом

python3 manage.py shell  
# выполнить в открывшемся терминале:
>>> from django.contrib.contenttypes.models import ContentType
>>> ContentType.objects.all().delete()
>>> quit()

python manage.py loaddata dump.json
```
2. Заполнить тестовыми данными
```
python manage.py initdata
```

## Документация по api
Ознакомиться с полной документацией можно по адресу.  
[http://localhost/redoc/](http://localhost/redoc/)

### Алгоритм получения токена:
#### 1. Получить код подтвержения.
в теле передать JSON
{
  "username": "имя_пользователя",
  "email": "адрес эл. почты"
}
POST-запрос на эндпоинт:
```
http://localhost/api/v1/auth/signup/
```


в папке sent_emails найти письмо и скопировать полученный код подтверждения.

#### 2. Получить токен
в теле передать JSON
{
  "username": "имя_пользователя",
  "confirmation_code": "код_подтвержения"
}
POST-запрос на эндпоинт:
```
http://localhost/api/v1/auth/token/
```
Токен использовать для авторизации


## Примеры запросов к api

### Некоторые примеры запросов к API
Доступные энд-поинты:
GET-запросы
```
/api/v1/users/
```
```
/api/v1/titles/
```
```
/api/v1/categories/
```
```
/api/v1/genres/
```
```
/api/v1/titles/{title_id}/reviews/
```
```
/api/v1/titles/{title_id}/reviews/{review_id}/comments/
```


## Авторы проекта
_[Вадим Барсуков](https://github.com/26remph), python-developer
___
<p>
    <span>api_yamdb © 2022, developer git: 26Remph ツ </span>
</p>