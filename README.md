# Учебный проект «Infra_sp2»

Цель работы над проектом - получить опыт автоматизации развертывания 
веб-приложений.

## Стек технологий
- Python 3.7.14 
- Django 2.2.16
- Django Filter 21.1
- Django REST Framework 3.12.4
- Simple JWT 5.2.0
- Postgresql 13.0
- Nginx 1.21.3
- Gunicorn 20.0.4

## Описание учебного проекта
В данном учебном проекте используется технология контейнеризации и инструмент 
[Docker](https://www.docker.com/) для развертывания веб-приложения
[api_yamdb](https://github.com/evgeny81d/api_yamdb) на локальном компьютере. 

Веб-приложение будет упаковано в три отдельных контейнера:
- web - контейнер с веб-приложением на фреймворке Django
- db - контейнер с базой данных postgresql
- nginx - контейнер с nginx сервером

### Веб-приложение
Создано на фрэймворке Django. Работает в отдельном контейнере на WSGI сервере 
[Gunicorn](https://gunicorn.org/)

Веб-приложение **YaMDb** собирает **отзывы (Review)** пользователей на 
**произведения (Titles)**. 

Произведения делятся на категории: «Книги», «Фильмы», «Музыка». 
**Список категорий (Category)** может быть расширен администратором.

Произведению может быть присвоен **жанр (Genre)** из списка предустановленных.
Новые жанры может создавать только администратор.

Благодарные или возмущённые пользователи оставляют к произведениям текстовые
**отзывы (Review)** и ставят произведению оценку (от одного до десяти).
Из пользовательских оценок формируется усреднённая оценка произведения — 
**рейтинг**.
На одно произведение пользователь может оставить только один отзыв.

Сами произведения в **YaMDb** не хранятся, здесь нельзя посмотреть фильм или 
послушать музыку.

Более детальную информацию по функционалу приложения можно посмотреть [здесь](https://github.com/evgeny81d/api_yamdb#readme)

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
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Проверяем, что все установилось корректно (запускаем образ hello-world)
 sudo service docker start
 sudo docker run hello-world
```

### Клонируем репозиторий и создаем файл с данными доступа к базе

Клонируйте репозиторий
```sh
# Клонируем репозиторий через https
git clone https://github.com/evgeny81d/infra_sp2.git

# Или

# Клонируем репозиторий через ssh
git clone git@github.com:evgeny81d/infra_sp2.git
```

Создайте файл `.env` в каталоге `infra_sp2/infra/`
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
Если не создать `.env` файл, то проект запустится с данными по умолчанию,
указанными в файле настроек `infra_sp2/api_yamdb/api_yamdb/settings.py`:
```python
DATABASES = {
    'default': {
        'ENGINE': os.getenv('DB_ENGINE', 'django.db.backends.postgresql'),
        'NAME': os.getenv('DB_NAME', 'postgres'),
        'USER': os.getenv('POSTGRES_USER', 'postgres'),
        'PASSWORD': os.getenv('POSTGRES_PASSWORD', 'postgres'),
        'HOST': os.getenv('DB_HOST', 'db'),
        'PORT': os.getenv('DB_PORT', '5432')
    }
}
```

### Запускаем проект
```sh
# Переходим в каталог с инфраструктурой запуска проекта
cd infra_sp2/infra

# Запускаем контейнеры
sudo docker-compose up -d --build

# Выполняем миграции в контейнере с веб-приложением
docker-compose exec web python3 manage.py migrate

# Выполняем сбор статических файлов в контейнере веб-приложения
sudo docker-compose exec web python3 manage.py collectstatic --no-input

# Выполняем команду для создания суперпользователя в контейнере веб-приложения
# и указываем все необходимые данные для нового пользователя
sudo docker-compose exec web python3 manage.py createsuperuser
```

### Проверяем

- Переходим в админ панель http://localhost/admin
- Выполняем вход с данными суперпользователя, который был создан на предыдущем
шаге.

### Останавливаем проект
```sh
# Остановка с сохранением базы данных и контейнеров
sudo docker-compose down

# Остановка с удалением всех контейнеров и базы данных
sudo docker-compose down -v
```