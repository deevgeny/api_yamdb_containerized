# Развертывание многоконтейнерного веб-приложения

Цель проекта - практическое применение автоматизации развертывания 
многоконтейнерного веб-приложения на локальном компьютере.

## Стек технологий
- Python 3.7
- Django 2.2.16
- Django Filter 21.1
- Django REST Framework 3.12.4
- Simple JWT 5.2.0
- Postgresql 13.0
- Nginx 1.21.3
- Gunicorn 20.0.4

## Описание проекта
В данном проекте используется технология контейнеризации и инструмент 
[Docker](https://www.docker.com/) для развертывания веб-приложения
[api_yamdb](https://github.com/evgeny81d/api_yamdb) на локальном компьютере. 

Веб-приложение будет упаковано в три отдельных контейнера:
- web - контейнер с веб-приложением на фреймворке Django
- db - контейнер с базой данных postgresql
- nginx - контейнер с nginx сервером

### Веб-приложение
Создано на фрэймворке Django. Работает в отдельном контейнере на WSGI сервере 
[Gunicorn](https://gunicorn.org/)

Более детальную информацию по функционалу приложения можно посмотреть [здесь](https://github.com/evgeny81d/api_yamdb#readme)

Образ заранее подготовлен и загружен на [dockerhub](https://hub.docker.com/repository/docker/evgeny81d/api_yamdb).

### База данных
В проекте используется база данных [Postgresql](https://www.postgresql.org/).

### Nginx сервер
Nginx сервер работает в отдельном контейнере, обеспечивая доступ к приложению,
а так же раздает статические файлы. В рамках данного проекта используется
незащищенный протокол [http](https://ru.wikipedia.org/wiki/HTTP).


## Запуск учебного проекта

Перед запуском проекта необходимо установить Docker на ваш локальный компьютер
и создать файл с данными доступа к базе.

### Устанавливаем Docker
```sh
# Удаляем старые версии Docker
sudo apt-get remove docker docker-engine docker.io containerd runc

# Обновляем пакеты
sudo apt-get update

# Устанавливаем Docker
sudo apt-get install docker docker-compose

# Проверяем, что все установилось корректно (запускаем образ hello-world)
 sudo service docker start
 sudo docker run hello-world
```

### Подготавливаем репозиторий и файл с данными доступа к базе

1. Клонируeм репозиторий
```sh
# Клонируем репозиторий
git clone git@github.com:evgeny81d/infra_sp2.git
```

2. Создаем файл `.env` в каталоге `infra_sp2/infra/`
с содержимым указанным ниже:
```python
# infra_sp2/infra/.env
# ... - замените своими данными
DB_ENGINE=django.db.backends.postgresql # указываем, что работаем с postgresql
DB_NAME=... # имя базы данных (выберите свое)
POSTGRES_USER=... # логин для подключения к базе данных (установите свой)
POSTGRES_PASSWORD=... # пароль для подключения к БД (установите свой)
DB_HOST=db # название сервиса (контейнера)
DB_PORT=5432 # порт для подключения к БД 
```

### Запускаем проект
```sh
# Переходим в каталог с инфраструктурой запуска проекта
cd infra_sp2/infra

# Запускаем контейнеры
sudo docker-compose up -d --build

# Выполняем миграции в контейнере с веб-приложением
sudo docker-compose exec web python3 manage.py migrate

# Выполняем сбор статических файлов в контейнере веб-приложения
sudo docker-compose exec web python3 manage.py collectstatic --no-input

# Выполняем команду для создания суперпользователя в контейнере веб-приложения
# и указываем все необходимые данные для нового пользователя
sudo docker-compose exec web python3 manage.py createsuperuser
```

[http://localhost/redoc/](http://localhost/redoc/) - полная документация API

[http://localhost/admin/](http://localhost/admin) - админ панель веб-приложения



### Способы остановки
```sh
# Остановка с сохранением базы данных и контейнеров
sudo docker-compose stop

# Повторный запуск контейнеров
sudo docker-compose start

# Остановка с удалением всех контейнеров и сохраненных даннных
sudo docker-compose down -v
```
