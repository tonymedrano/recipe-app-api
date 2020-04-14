
# Recipe App rest api
using Django rest framework, pyCharm, docker, flake, travis ci

#### Install
```shell script
- docker
```

#### Create and add to Dockerfile script
```shell script
FROM python:3.7-alpine
MAINTAINER Tony Medrano Developer

ENV PYTHONUNBUFFERED 1

COPY ./requirements.txt /requirements.txt
RUN pip install -r /requirements.txt

RUN mkdir /app
WORKDIR /app
COPY ./app /app

RUN adduser -D user
USER user
```
#### Run Dockerfile for creation
```shell script
sudo chmod 666 /var/run/docker.sock
docker build .
```

#### Create docker-compose.yml script
```shell script
version: "3.7"
services:
  app:
    build:
      context: .
    ports:
      - "8000:8000"
    volumes:
      - ./app:/app
    command: >
      sh -c "python manage.py runserver 0.0.0.0:8000
```
#### Run docker-compose.yml
```shell script
docker-compose build
```

#### Start new project
```shell script
docker-compose run app sh -c "django-admin.py startproject app ."
```

#### Create and add .travis.yml script
```shell script
language: python
python:
  - "3.6"
services:
  - docker
before_script: pip install docker-compose

script:
  - docker-compose run app sh -c "python manage.py test && flake8"
```

#### Create .flake8 script
```shell script
[flake8]
exclude =
  migrations
  __pycache__,
  manage.py,
  settings.py
```

#### Create new app named core
```shell script
docker-compose run app sh -c "python manage.py startapp core"
```

#### Make migrations
```shell script
docker-compose run app sh -c "python manage.py makemigrations"
```

#### Migrate
```shell script
docker-compose run app sh -c "python manage.py migrate"
```

#### Adding Database to docker-compose.yml script
```shell script
version: "3.7"
services:
  app:
    build:
      context: .
    ports:
      - "8000:8000"
    volumes:
      - ./app:/app
    command: >
      sh -c "python manage.py runserver 0.0.0.0:8000"
    ------- ADDING DB SETUP --------------
    environment:
      - DB_HOST=db
      - DB_NAME=app
      - DB_USER=postgres
      - DB_PASS=supersecretpassword
    depends_on:
      - db

  db:
    image: postgres:10-alpine
    environment:
      - POSTGRES_DB=app
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=supersecretpassword
```

#### Add to Dockerfile script
```shell script
FROM python:3.7-alpine
MAINTAINER Tony Medrano Developer

ENV PYTHONUNBUFFERED 1

COPY ./requirements.txt /requirements.txt

---------------- POSTGRESQL ---------------
RUN apk add --update --no-cache postgresql-client
RUN apk add --update --no-cache --virtual .tmp-build-deps gcc libc-dev linux-headers postgresql-dev
---------------- END POSTGRESQL ---------------

RUN pip install -r /requirements.txt

---------------- TMP-BUILD-DEPS ---------------
RUN apk del .tmp-build-deps
---------------- END TMP-BUILD-DEPS ---------------

RUN mkdir /app
WORKDIR /app
COPY ./app /app

RUN adduser -D user
USER user
```

#### Test dependencies and settings  y building the project
```shell script
docker-compose build
```

#### Adding PostgresQL to application in settings.py defined in docker-compose.yml environment variables
```shell script
----------------- REPLACE THIS --------------------------
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
------------- WITH THIS ---------------------------------
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'HOST': os.environ.get('DB_HOST'),
        'NAME': os.environ.get('DB_NAME'),
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASS'),
    }
}

```

#### Adding Database to docker-compose.yml script
```shell script
version: "3.7"
services:
  app:
    build:
      context: .
    ports:
      - "8000:8000"
    volumes:
      - ./app:/app
    command: >
--------------- REPLACE THIS --------------
sh -c "python manage.py runserver 0.0.0.0:8000"    
--------------- WITH THIS ------------------
sh -c "python manage.py wait_for_db &&
      python manage.py migrate &&
      python manage.py runserver 0.0.0.0:8000"
-----------------------------------------------
    environment:
      - DB_HOST=db
      - DB_NAME=app
      - DB_USER=postgres
      - DB_PASS=supersecretpassword
    depends_on:
      - db

  db:
    image: postgres:10-alpine
    environment:
      - POSTGRES_DB=app
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=supersecretpassword
```

### Run application
```shell script
docker-compose up
```

### Create admin/superuser (email-password)
```shell script
docker-compose run app sh -c "python manage.py createsuperuser"
```

### Start new app named user
```shell script
docker-compose run app sh -c "python manage.py startapp user"
```

### Register django rest_frameworks and apps
 (app/app/settings.py)
```shell script
INSTALLED_APPS = [
    ...
    'rest_framework',
    'rest_framework.authtoken',
    'core',
    'user',
]
```