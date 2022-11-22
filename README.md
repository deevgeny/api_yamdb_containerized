# API Yamdb containerized web application

In this project we'll use [Docker](https://www.docker.com/) to deploy 
[api_yamdb](https://github.com/evgeny81d/api_yamdb) web application on local 
host. 


## Technology stack
- Python 3.7
- Django 2.2.16
- Django Filter 21.1
- Django REST Framework 3.12.4
- Simple JWT 5.2.0
- Postgresql 13.0
- Nginx 1.21.3
- Gunicorn 20.0.4


## Web application infrastructure
The web application will be deployed in 3 Docker containers on local host.

### Web application container
Django web application. Will run in docker container on WSGI 
[Gunicorn](https://gunicorn.org/) server.

More detailed information about the application is 
[here](https://github.com/evgeny81d/api_yamdb#readme).

Docker image is already build and pushed to 
[DockerHub](https://hub.docker.com/repository/docker/evgeny81d/api_yamdb).

### Database container
The web application is supposed to work with [Postgresql](https://www.postgresql.org/)
 database.

### Nginx container
Nginx sever will be started in a separate container. It will serve http 
requests and static files. 


## How to install and deploy

Docker should be already installed on local host before deployment.

### Prepare project infrastructure

1. Clone the repository.
```sh
# Run git clone command
git clone https://github.com/evgeny81d/api_yamdb_containerized.git
```

2. Create `.env` file with secrets in `api_yamdb_containerized/infra/` directory.
```python
# Filepath: api_yamdb_containerized/infra/.env
DB_ENGINE=django.db.backends.postgresql
DB_NAME=# your database name
POSTGRES_USER=# your database user name
POSTGRES_PASSWORD=# your database password
DB_HOST=db
DB_PORT=5432
```

### Deploy
```sh
# Go to the projects infra directory
cd api_yamdb_containerized/infra

# Start docker containers
sudo docker-compose up -d

# Run migrations in the web application container
sudo docker-compose exec web python3 manage.py migrate

# Collect static files in the web application container
sudo docker-compose exec web python3 manage.py collectstatic --no-input

# Create superuser in the web application container
sudo docker-compose exec web python3 manage.py createsuperuser
```


### Finally the web application is ready for use

[http://localhost/redoc/](http://localhost/redoc/) - API documentation

[http://localhost/admin/](http://localhost/admin) - admin site



## How to stop containers
```sh
# Stop and persist the data
sudo docker-compose stop

# Start containers again
sudo docker-compose start

# Stop with deleting all data
sudo docker-compose down -v
```


## Security notice
The above instructions how to install and deploy the project have only
demonstration purpose and can be used on local host. 

If you would like to deploy the project on a web server, please read the 
[documentation](https://docs.djangoproject.com/en/2.2/howto/deployment/) and
build your own docker image.
